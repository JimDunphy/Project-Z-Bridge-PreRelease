# Security Docs

Last reviewed: 2026-05-16.

This directory contains Project Z Bridge security architecture, audits,
hardening plans, and security-focused validation guides.

Use the section headings as part of the document status. "Current posture" docs
describe what the bridge relies on today. Plans, proposals, and guardrails can
be correct and useful, but should not be treated as implemented behavior without
checking the linked implementation docs and source.

## Naming Convention

Security docs use explicit `SECURITY_*` filenames.

## Current Posture

- [`SECURITY_REVIEW.md`](SECURITY_REVIEW.md) — shim-focused security posture and route inventory.
- [`SECURITY_SHIM_AUDIT.md`](SECURITY_SHIM_AUDIT.md) — current bridge controls for hostile mail content and web-origin threats.
- [`SECURITY_STATIC_ZWC_AUDIT.md`](SECURITY_STATIC_ZWC_AUDIT.md) — background scan of inherited classic ZWC client-side constraints.
- [`SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md`](SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md) — inbound HTML email attack-surface model.
- [`SECURITY_ZIMBRA_10_1_17_REVIEW.md`](SECURITY_ZIMBRA_10_1_17_REVIEW.md) — targeted applicability review for Zimbra 10.1.17 Classic UI security fixes.

## Designs, Rationale, And Guardrails

- [`SECURITY_IFRAME_STRICT_CSP_DESIGN.md`](SECURITY_IFRAME_STRICT_CSP_DESIGN.md) — active iframe renderer + strict CSP design and debugging reference.
- [`SECURITY_IFRAME_STRICT_CSP_SEQUENCE.md`](SECURITY_IFRAME_STRICT_CSP_SEQUENCE.md) — one-page request/response sequence reference.
- [`SECURITY_IFRAME_STRICT_CSP_RATIONALE.md`](SECURITY_IFRAME_STRICT_CSP_RATIONALE.md) — why iframe isolation was required, and which alternatives were rejected.
- [`SECURITY_EMAIL_IFRAME_SANDBOX_ROLLOUT_PLAN.md`](SECURITY_EMAIL_IFRAME_SANDBOX_ROLLOUT_PLAN.md) — rollout plan and phase tracking for iframe rendering.
- [`SECURITY_CALENDAR_COUNTER_INVITES.md`](SECURITY_CALENDAR_COUNTER_INVITES.md) — guardrail for future proposed-new-time (`METHOD:COUNTER`) support; organizer-side acceptance is not implemented today.

## Validation And Regression Guides

- [`SECURITY_AUDIT_PLAN.md`](SECURITY_AUDIT_PLAN.md)
- [`SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md`](SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md)
- [`SECURITY_EMAIL_RENDERING_E2E_VALIDATION.md`](SECURITY_EMAIL_RENDERING_E2E_VALIDATION.md)
- [`SECURITY_EMAIL_RENDER_SMOKE_AUTOMATION.md`](SECURITY_EMAIL_RENDER_SMOKE_AUTOMATION.md)
- [`SECURITY_IMAGE_PROXY_TESTING.md`](SECURITY_IMAGE_PROXY_TESTING.md)
- [`SECURITY_RED_TEAM_PLAN.md`](SECURITY_RED_TEAM_PLAN.md)

## Input Audits, Plans, And Proposals

- [`SECURITY_AUDIT.md`](SECURITY_AUDIT.md) — input/static-asset risk scan; not the current bridge security posture by itself.
- [`SECURITY_STATIC_ASSET_AUDIT_PLAN.md`](SECURITY_STATIC_ASSET_AUDIT_PLAN.md) — triage plan that maps static-asset findings to the bridge runtime.
- [`SECURITY_IMPLEMENTATION_PHASES.md`](SECURITY_IMPLEMENTATION_PHASES.md) — developer/security implementation worklist; operator-facing roadmap is [`../sysadmin/SYSADMIN_SECURITY_ROADMAP.md`](../sysadmin/SYSADMIN_SECURITY_ROADMAP.md).
- [`SECURITY_ZIMBRA_STATIC_FINDINGS_PROPOSAL.md`](SECURITY_ZIMBRA_STATIC_FINDINGS_PROPOSAL.md) — candidate findings against extracted Classic ZWC static assets; proposal only, no approved runtime changes.
