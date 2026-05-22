# Project Z-Bridge — Status / Handoff

Status: current handoff snapshot. Last condensed: 2026-05-16.

Project Z-Bridge runs the classic Zimbra Web Client (ZWC) against Stalwart by
serving ZWC assets and translating Zimbra-shaped `/service/soap` and REST calls
into Stalwart/JMAP-backed behavior.

For the prior dense handoff snapshot, use
[`STATUS_DETAILED_HANDOFF_ARCHIVE.md`](STATUS_DETAILED_HANDOFF_ARCHIVE.md). That
archive is debugging memory, not the current source of truth.

## Quick Run

Dev loop:

1. Put `zimbra.war` in the repo root, or run `./manage.sh webclient-build`.
2. Run `./manage.sh webclient-extract` to populate `static/zimbra/`.
3. Copy `.env.example` to `.env` and set at least `STALWART_BASE_URL`.
4. Run `./manage.sh up` or `./manage.sh restart`.
5. Open `http://127.0.0.1:${BRIDGE_PORT:-7777}/zimbra/`.

Runtime image:

1. Run `./manage.sh image-build`.
2. Ensure webclient assets are extracted.
3. Run `./manage.sh up-runtime`.
4. Use `./manage.sh logs-runtime` for logs.

## Current Shape

- Core daily-use surfaces are implemented broadly enough for active testing:
  mail, folders, tags, contacts, calendar, compose/send, preferences, filters,
  internal sharing, import/export, Rspamd analysis, retention, and selected
  bridge-only enhancements.
- The bridge is compatibility-first for classic ZWC. It is not a full Zimbra
  server and should not be described as a complete `mailboxd` replacement.
- The authoritative SOAP method list is source-audited by
  `./manage.sh soap-docs-check`.
- The operator and developer docs now distinguish current references, future
  plans, resolved debugging memory, and historical notes.

## Current Issue Sources

- Live bugs and pending verification: [`BUGS.md`](BUGS.md)
- Deferred gaps/future debt: [`FUTURE_GAPS.md`](FUTURE_GAPS.md)
- Resolved incident memory: [`BUGS_RESOLVED.md`](BUGS_RESOLVED.md)
- Product future roadmap: [`developer/PRODUCT_FUTURE_COMPLETION_PLAN.md`](developer/PRODUCT_FUTURE_COMPLETION_PLAN.md)

Current high-value checks from `BUGS.md`:

- hard-refresh nested login / transient server-unreachable UX
- persona account-name isolation
- pending browser verification for display-name preference and retention UI fixes
- user-facing error-message sanitization
- attachment `Remove` action behavior

## Authoritative References

- Project onboarding: [`PROJECT_Z_BRIDGE_ONBOARDING.md`](PROJECT_Z_BRIDGE_ONBOARDING.md)
- ZWC/SOAP lifecycle: [`developer/ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md`](developer/ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md)
- Architecture overview: [`developer/ARCHITECTURE.md`](developer/ARCHITECTURE.md)
- API routes and method groups: [`developer/API_REFERENCE.md`](developer/API_REFERENCE.md)
- SOAP method matrix: [`developer/SOAP_COMPATIBILITY_MATRIX.md`](developer/SOAP_COMPATIBILITY_MATRIX.md)
- Env reference: [`sysadmin/SYSADMIN_ENV_REFERENCE.md`](sysadmin/SYSADMIN_ENV_REFERENCE.md)
- Command reference: [`sysadmin/SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md`](sysadmin/SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md)
- Security posture: [`security/SECURITY_REVIEW.md`](security/SECURITY_REVIEW.md)
- Documentation audit: [`DOCUMENTATION_AUDIT.md`](DOCUMENTATION_AUDIT.md)

## Safety Notes

- Do not commit `.env`, `zimbra.war`, `static/zimbra/`, or `.dev/` artifacts.
- Keep deployment-specific hostnames, user data, screenshots, and transcripts out
  of public-bound docs unless they are sanitized examples.
- Before sharing docs, run `./manage.sh docs-check` and
  `./manage.sh docs-audit`, then review the diff manually.
