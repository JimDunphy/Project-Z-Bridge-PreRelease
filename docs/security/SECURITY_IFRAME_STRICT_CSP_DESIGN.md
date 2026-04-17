# SECURITY: Iframe + Strict CSP Design

Date: 2026-02-27  
Status: Active design and debugging reference  
Owner: Bridge/Webclient

Quick sequence view:
1. `docs/security/SECURITY_IFRAME_STRICT_CSP_SEQUENCE.md`
2. `docs/security/SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md`

## Purpose

Define the architecture and operations model for hardened mail-body rendering using:
1. iframe containment
2. optional separate renderer origin
3. CSP in staged rollout (`log` -> `report-only` -> `enforce`)

This doc is intended to be the primary reference for future debugging and rollout decisions.

## Problem Statement

Classic ZWC is a legacy web client with high script complexity and many dynamic rendering paths.  
Incoming HTML email is untrusted content and is the primary attack surface.

We need a model that:
1. preserves ZWC usability (reading pane, conversation view, links, images)
2. materially reduces XSS breakout impact
3. provides clear operator telemetry and rollback controls

## Security Goals

1. Prevent untrusted mail HTML from executing in app-shell origin.
2. Limit blast radius if sanitizer misses a payload.
3. Keep external resource fetches constrained and auditable.
4. Provide canary-by-user rollout controls.
5. Make failures debuggable with deterministic logs and toggles.

## Non-Goals

1. We do not attempt strict CSP for all classic ZWC scripts today (legacy compatibility constraints).
2. We do not claim this solves social engineering/phishing.
3. We do not remove the need for perimeter controls (TLS, firewall, reverse proxy hardening).

## Architecture

### Components

1. App shell origin:
   - Example: `https://stalwart.example.com`
   - Serves `/zimbra`, SOAP bridge, and telemetry endpoints.
2. Renderer origin:
   - Example: `https://renderer.example.com`
   - Hosts short-lived renderer views (`/service/email-renderer/view/:token`).
3. Telemetry and CSP report collectors:
   - `POST /service/client-telemetry/email-iframe`
   - `POST /csp-report`
   - `POST /zimbra/csp-report` (proxy-friendly alias)

### Mail Body Render Flow (Hardened Path)

1. Bridge sanitizes message HTML server-side.
2. ZWC message view creates iframe.
3. If renderer mode is enabled for user:
   - Browser posts sanitized HTML to `/service/email-renderer/register`.
   - Bridge returns signed/short-lived `viewUrl`.
   - iframe `src` is set to renderer view URL.
4. iframe is sandboxed under renderer policy.
5. Runtime height guards keep reading pane stable in both:
   - `ZmMailMsgView` (message mode)
   - `ZmMailMsgCapsuleView` (conversation/capsule mode)

### End-to-End Interaction Sequence (How It Works)

```text
User opens message
  -> ZWC requests message payload via SOAP (GetMsgRequest)
  -> Bridge fetches from JMAP and sanitizes HTML
  -> Client runtime patch decides render mode
      -> (A) Legacy iframe mode: write sanitized HTML to iframe directly
      -> (B) Renderer mode:
           POST /service/email-renderer/register (auth required)
           <- { ok, token, viewUrl }
           iframe.src = viewUrl (renderer origin)
  -> Renderer endpoint serves strict-CSP document
  -> Runtime height guards correct iframe sizing
  -> Optional telemetry emitted for errors/guards
```

### Renderer Token and Document Lifecycle

1. Client sends sanitized HTML to `POST /service/email-renderer/register`.
2. Bridge validates auth (`ZM_AUTH_TOKEN`) and renderer feature gates.
3. Bridge enforces size/capacity/TTL controls:
   - max bytes
   - max in-memory entries
   - expiration window
4. Bridge returns short-lived token and `viewUrl`.
5. Browser loads `GET /service/email-renderer/view/:token`.
6. Bridge checks token validity + expiration before serving HTML.
7. Expired/invalid tokens return non-success responses and are not rendered.

### Runtime Render State Machine

1. Bootstrap:
   - iframe is created with temporary compatible sandbox to avoid early ZWC breakage.
2. Register:
   - async call registers sanitized HTML with bridge renderer endpoint.
3. Swap:
   - iframe switches to renderer `viewUrl` and renderer sandbox policy.
4. Guard:
   - on load/resize, runtime enforces minimum height and re-checks delayed timers.
5. Fallback:
   - if register/view fails, runtime falls back to local safe text rendering and emits telemetry.

### Trust Boundaries

1. Untrusted: email HTML and metadata from remote senders.
2. Trusted: bridge sanitization pipeline and renderer token service.
3. High-value boundary: origin separation (`stalwart` vs `renderer`).

### Authentication and Authorization Model

1. Renderer register endpoint requires authenticated session cookie.
2. Renderer enablement is also gated by per-user allowlist.
3. Renderer documents are scoped by token lifecycle and are not long-term storage.
4. CSP report endpoint accepts report payloads and logs violations; no privileged mutation path.

## CSP Design

### Main App CSP (ZWC shell)

Classic ZWC compatibility usually requires:
1. `script-src 'unsafe-inline' 'unsafe-eval'`
2. gradual rollout with report collection

Recommended staged modes:
1. `BRIDGE_CSP_MODE=log` (adds report URI automatically)
2. `BRIDGE_CSP_MODE=report-only`
3. `BRIDGE_CSP_MODE=enforce` after sustained clean telemetry

### Renderer CSP (mail body view)

Renderer pages should remain strict:
1. `default-src 'none'`
2. `script-src 'none'`
3. `connect-src 'none'`
4. no top-level execution privileges

This is the core containment layer for untrusted mail HTML.

### Header Construction Model

1. Bridge builds main app CSP from mode + policy env:
   - `off`: no CSP header
   - `report-only`/`log`: `Content-Security-Policy-Report-Only`
   - `enforce`: `Content-Security-Policy`
2. In `log` mode, report URI is auto-added if missing.
3. Renderer origin is appended to app `frame-src` when configured.
4. Renderer view responses use a dedicated strict CSP independent of app-shell CSP.

## Runtime Controls

### Iframe Sandbox and Renderer

1. `BRIDGE_EMAIL_IFRAME_SANDBOX_ENABLED`
2. `BRIDGE_EMAIL_IFRAME_SANDBOX_POLICY`
3. `BRIDGE_EMAIL_IFRAME_SANDBOX_USER_ALLOWLIST`
4. `BRIDGE_EMAIL_IFRAME_RENDERER_ENABLED`
5. `BRIDGE_EMAIL_IFRAME_RENDERER_USER_ALLOWLIST`
6. `BRIDGE_EMAIL_IFRAME_RENDERER_ORIGIN`
7. `BRIDGE_EMAIL_IFRAME_RENDERER_SANDBOX_POLICY`
8. `BRIDGE_EMAIL_IFRAME_RENDERER_TTL_SECS`
9. `BRIDGE_EMAIL_IFRAME_RENDERER_MAX_BYTES`
10. `BRIDGE_EMAIL_IFRAME_RENDERER_MAX_ENTRIES`

### CSP

1. `BRIDGE_CSP_MODE`
2. `BRIDGE_CSP`
3. `BRIDGE_CSP_REPORT_URI`

Recommended reverse-proxy-safe value:
1. `BRIDGE_CSP_REPORT_URI=/zimbra/csp-report`

## Debugging Playbook

### Symptom: message body is clipped/truncated on first click

Likely causes:
1. iframe height collapsed by legacy ZWC resize path
2. cross-origin iframe resize assumptions in message/capsule code

Checks:
1. confirm renderer and sandbox flags for affected user
2. inspect telemetry events:
   - `renderer_iframe_height_guard_applied`
   - `capsule_resize_*`
   - `iframe_renderer_loaded`

Commands:

```bash
./manage.sh logs | rg "email iframe telemetry|renderer_iframe_height_guard|capsule_resize|iframe_renderer"
```

### Symptom: message body blank, then appears after switching messages

Likely causes:
1. async renderer registration race
2. iframe source not swapped after bootstrap

Checks:
1. look for `iframe_renderer_register_start` without `iframe_renderer_loaded`
2. look for `iframe_renderer_failed`

### Symptom: “Display Images” / external-image infobar disappears for some HTML mail

Likely causes:
1. renderer mode applied to deferred-media messages (`dfsrc` / `dfbackground`)
2. native ZWC deferred-image consent path bypassed by cross-origin renderer swap

Checks:
1. inspect message HTML summary logs for `dfsrc` / `dfbackground` counts
2. confirm telemetry includes `iframe_renderer_mode_skipped` with `reason=deferred_external_media`
3. verify those messages use native iframe flow and infobar click behavior is restored

Commands:

```bash
./manage.sh logs | rg "dfsrc=|dfbackground=|iframe_renderer_mode_skipped|deferred_external_media"
```

### Symptom: CSP violations not visible in logs

Likely causes:
1. `BRIDGE_CSP_MODE` not set to `log` or `report-only`
2. `BRIDGE_CSP_REPORT_URI` not reachable through proxy path
3. compose env did not pass CSP vars into container

Checks:

```bash
./manage.sh run env | rg "^BRIDGE_CSP"
curl -k -I -s https://<host>/zimbra | rg -i "content-security-policy|report-uri"
```

If needed, validate report ingestion:

```bash
curl -k -X POST https://<host>/zimbra/csp-report \
  -H "content-type: application/csp-report" \
  --data '{"csp-report":{"document-uri":"https://<host>/zimbra","violated-directive":"script-src","blocked-uri":"https://evil.example/x.js"}}'
```

Expected:
1. HTTP `204`
2. bridge log line containing `csp report`

### Symptom: renderer mode works for one user but not another

Likely causes:
1. user not matched by renderer allowlist
2. renderer origin mismatch with `frame-src` CSP

Checks:
1. verify `BRIDGE_EMAIL_IFRAME_RENDERER_USER_ALLOWLIST`
2. verify main CSP `frame-src` includes renderer origin

## Rollout Strategy

1. Canary users only (`exact user` allowlist entries).
2. Observe:
   - iframe telemetry
   - CSP report trends
   - user-visible rendering regressions
3. Expand by cohort (`@domain`, then `*`).
4. Move CSP to enforce only after stability window.

## Rollback Strategy

Fast rollback knobs:
1. disable renderer mode:
   - `BRIDGE_EMAIL_IFRAME_RENDERER_ENABLED=false`
2. disable sandbox mode:
   - `BRIDGE_EMAIL_IFRAME_SANDBOX_ENABLED=false`
3. reduce CSP strictness:
   - `BRIDGE_CSP_MODE=log` or `report-only`

All rollback steps are runtime config + bridge restart.

## Why This Is Stronger Than Stock ZWC + Zimbra (Security Lens)

1. Bridge adds server-side sanitization controls we can tune and test continuously.
2. Bridge adds image proxy enforcement with explicit SSRF/content-type guardrails.
3. Bridge can run renderer on a dedicated origin with short-lived tokenized views.
4. Bridge provides user-scoped canary rollout and telemetry-driven hardening.
5. Bridge provides explicit CSP collection pipeline integrated with operational logs.

In short: same familiar UI, stronger containment and observability controls.
