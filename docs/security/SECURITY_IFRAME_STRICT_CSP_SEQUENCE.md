# SECURITY: Iframe + Strict CSP Sequence (One-Page)

Date: 2026-02-27  
Status: Quick reference for debugging and onboarding  
Related: `docs/security/SECURITY_IFRAME_STRICT_CSP_DESIGN.md`

## Scope

This page shows the runtime sequence for hardened email-body rendering:
1. normal renderer-origin flow
2. fallback flow when renderer registration/view fails
3. CSP violation reporting path

## Normal Flow

```mermaid
sequenceDiagram
    participant U as User (ZWC UI)
    participant B as Bridge (/service/soap)
    participant J as Stalwart JMAP
    participant R as Bridge Renderer API
    participant O as Renderer Origin

    U->>B: GetMsgRequest
    B->>J: Email/get + body
    J-->>B: message payload
    B-->>U: sanitized HTML payload

    U->>R: POST /service/email-renderer/register (auth cookie)
    R-->>U: { ok, token, viewUrl }
    U->>O: iframe src = /service/email-renderer/view/:token
    O-->>U: strict-CSP renderer HTML
    Note over U: runtime height guards apply (msg + capsule views)
```

## Fallback Flow

```mermaid
sequenceDiagram
    participant U as User (ZWC UI)
    participant R as Bridge Renderer API
    participant T as Telemetry API

    U->>R: POST /service/email-renderer/register
    alt register/view fails
        R-->>U: non-ok / timeout / invalid payload
        U->>U: fallback local safe render path
        U->>T: POST /service/client-telemetry/email-iframe
    else success
        R-->>U: viewUrl
    end
```

## CSP Reporting Flow

```mermaid
sequenceDiagram
    participant C as Browser
    participant A as App Origin (/zimbra)
    participant L as CSP Report Endpoint

    A-->>C: CSP-Report-Only (report-uri /zimbra/csp-report)
    C->>L: POST /zimbra/csp-report
    L-->>C: 204 No Content
    Note over L: bridge logs document, directive, blocked URI
```

## Runtime Decision Model

```mermaid
flowchart TD
    A[Message open] --> B{Renderer enabled for user?}
    B -- No --> C[Legacy iframe render path]
    B -- Yes --> D[Register sanitized HTML]
    D --> E{Register/view success?}
    E -- Yes --> F[Load renderer-origin iframe]
    E -- No --> G[Fallback safe local render]
    F --> H[Height guards + telemetry]
    G --> H
```

## High-Value Debug Signals

1. Renderer success:
   - `iframe_renderer_loaded`
2. Renderer failures:
   - `iframe_renderer_failed`
3. Height containment:
   - `renderer_iframe_height_guard_applied`
   - `capsule_resize_*`
4. CSP pipeline:
   - `csp report ... violated_directive=... blocked_uri=...`

## First Commands to Run During Incident

```bash
./manage.sh logs | rg "iframe_renderer|height_guard|capsule_resize|csp report"
./manage.sh run env | rg "^BRIDGE_CSP|^BRIDGE_EMAIL_IFRAME"
curl -k -I -s https://<host>/zimbra | rg -i "content-security-policy|report-uri"
```
