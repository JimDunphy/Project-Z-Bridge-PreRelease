# Recommended Deployment Topology

This document describes the **preferred production topology** for Project Z-Bridge.

It answers the operational question:

- "Where should the shim live relative to Stalwart, MX hosts, and Internet-facing edge controls?"

## Preferred Model

Project Z-Bridge should normally be deployed as a **separate webmail tier** in front of Stalwart, not as a browser-facing extension of the Stalwart host itself.

Preferred flow:

```text
Browser
  -> Cloudflare / WAF / reverse proxy
  -> Project Z-Bridge (webmail tier)
  -> Stalwart (mailbox tier, private/trusted IP only)

MX hosts
  -> Stalwart SMTP (port 25, private/trusted source list only)
```

Key rule:

- Browsers should talk to the **shim**
- the shim should talk to **Stalwart**
- Stalwart should not need to be exposed as the public webmail surface

## Why This Is The Preferred Topology

### 1) Stalwart stays on trusted/internal network space

Recommended:

- restrict Stalwart JMAP/management access to trusted IPs
- allow SMTP `25` only from the approved MX tier
- let Stalwart deliver outbound mail as needed

Why:

- reduces the exposed attack surface of the mailbox tier
- keeps the browser/web rendering boundary separate from the mail store/protocol server
- makes firewall policy simpler and more auditable

### 2) The shim becomes the hostile-content boundary

The shim is where hostile browser-facing content should be handled:

- mail HTML sanitization
- CSS hardening
- external-image defanging / proxying
- active-content attachment hardening
- renderer CSP / iframe containment

Why:

- classic ZWC is legacy client code and should not be treated as the primary trust boundary
- the browser should receive untrusted mail content only after the shim has transformed and constrained it

### 3) The edge handles Internet-facing abuse controls

Recommended edge controls:

- TLS termination
- HSTS
- rate limiting
- connection limits
- body-size limits
- bot/abuse detection
- WAF / captcha / challenge workflows
- auth-failure monitoring and blocking

Examples:

- Cloudflare in front of the reverse proxy
- reverse proxy only, with trusted-IP restriction instead of public exposure

Both are valid depending on the environment.

## Supported Exposure Models

### A) Public webmail behind Cloudflare / reverse proxy

Typical production model:

```text
Internet
  -> Cloudflare / WAF
  -> HTTPS reverse proxy
  -> Project Z-Bridge
  -> Stalwart (private)
```

Use this when:

- webmail is intended for public Internet access
- you want layered abuse controls before traffic reaches the shim

### B) Trusted-IP-only webmail

Also valid:

```text
Trusted network / VPN / private IP space
  -> reverse proxy or direct internal access
  -> Project Z-Bridge
  -> Stalwart (private)
```

Use this when:

- access is limited to internal networks, VPN users, or private address space
- Cloudflare/public exposure is not required

This is lower risk than public exposure and is a reasonable deployment model.

## What To Avoid

### 1) Do not expose Stalwart as the public webmail entrypoint

Avoid a topology where:

- browsers talk directly to Stalwart for webmail rendering paths
- the shim is bypassed for mail HTML / attachment rendering

Why:

- the shim is the application layer where Project Z-Bridge's content hardening lives

### 2) Do not expose the raw shim port casually

Avoid:

- direct Internet exposure of the bridge container port with minimal edge controls

Even though public webmail is normal, it should still be deployed like a serious web application:

- reverse proxy
- TLS
- logging
- rate limiting
- monitoring

### 3) Do not make multiple topologies equally "primary" too early

Possible but not recommended as the main support target:

- per-user local bridge instances
- mixed environments where some users hit the shim and others bypass it

Why:

- increases support complexity
- increases config drift
- makes the security boundary less clear

## Local Per-User Bridge

This is possible, but should be treated as a niche/internal model, not the default production story.

Pros:

- per-user isolation
- useful for special/internal workflows

Cons:

- harder to support consistently
- more endpoint drift
- weaker central observability
- less clear operational boundary

If used, the architectural rule is still the same:

- the shim must remain the browser-facing security boundary for hostile content

## Short Admin Answer

If an admin asks for the intended deployment model:

1. MX hosts deliver mail to Stalwart.
2. Stalwart lives on trusted/internal IP space.
3. Project Z-Bridge is the public or semi-public webmail tier.
4. Browsers talk to the shim, not directly to Stalwart for webmail.
5. Edge controls (Cloudflare, reverse proxy, firewall policy) sit in front of the shim.

## Related Documents

- [`SYSADMIN_DEPLOY.md`](SYSADMIN_DEPLOY.md)
- [`SYSADMIN_PRODUCTION_HARDENING.md`](SYSADMIN_PRODUCTION_HARDENING.md)
- [`SYSADMIN_EXTERNAL_HARDENING.md`](SYSADMIN_EXTERNAL_HARDENING.md)
- [`SYSADMIN_SECURITY_HARDENING_SUMMARY.md`](SYSADMIN_SECURITY_HARDENING_SUMMARY.md)
