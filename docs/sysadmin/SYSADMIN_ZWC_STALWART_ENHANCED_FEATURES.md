# ZWC + Stalwart Enhanced Features

Project Z-Bridge keeps the classic Zimbra Web Client (ZWC) user experience while replacing the Zimbra server runtime with Stalwart plus a Rust compatibility bridge.

This document tracks admin-facing enhancements that are available in Project Z-Bridge and are not simply stock ZWC talking to stock Zimbra `mailboxd`.

For deeper maintainer notes, see [`../developer/PRODUCT_ENHANCEMENTS.md`](../developer/PRODUCT_ENHANCEMENTS.md).

## Summary Table

| Area | Enhancement | Admin Value |
| --- | --- | --- |
| Rspamd operations | Message-list score hover and detailed Rspamd header badge | Faster spam/ham triage without opening every message |
| Rspamd training | `Save Original` and `Save Originals as Tar` context-menu actions | Easy collection of raw `.eml` samples for rule writing |
| Mail security | Shim-side HTML sanitization, image defanging, image proxy, safer link handling | Reduces tracking and mail-rendering attack surface |
| Mail isolation | Optional sandboxed/renderer-origin mail body mode | Stronger containment for untrusted mail HTML |
| Tags | Sort mail list by Tags column | Better filter QA and tagged-message review |
| Retention | Per-folder retention persistence and optional purge enforcement | ZWC retention UI works against Stalwart-backed mail |
| Filters | Bridge-managed Sieve compilation and backups | ZWC filters become delivery-time Stalwart Sieve behavior |
| Preferences | Bridge-managed safety controls and Rspamd analysis toggle | Admin/security features exposed in familiar Preferences UI |
| Out-of-office | Alias/recipient-specific auto-reply extension | More precise vacation replies than stock global-only UI |
| RSS folders | Shim-managed RSS/Atom importer and polling | Feed folders behave as mail-like unread items |
| Branding | Stalwart/bridge branding overrides | Deployable UI without presenting as a full Zimbra server |
| Reliability | Soft network warning, hard auth expiry on restart | Clearer user behavior during outages and bridge restarts |
| Performance | Cache, prewarm, and startup optimizations | Better large-mailbox daily usability |

## Rspamd Analysis and Training

Project Z-Bridge adds Rspamd-specific operational tooling in the ZWC mail UI:

- Compact `X-Spam-Status` hover in the message list flag column.
- Detailed `X-Spamd-Result` badge and symbol popover in the reading pane.
- Single-message `.eml` source download.
- Multi-message plain `.tar` export containing `.eml` files.

Admin training guide:

- [`SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md`](SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md)

## Mail Rendering Security

Classic ZWC expects Zimbra server-side mail HTML processing. Project Z-Bridge replaces that with shim-side controls:

- Sanitizes message HTML.
- Defangs external images by default.
- Proxies remote images through authenticated same-origin URLs.
- Adds safer external-link behavior for browser compatibility.
- Normalizes IDN hostnames to punycode for better phishing visibility.
- Optionally isolates rendered mail bodies in a stricter renderer-origin mode.

Security references:

- [`../security/SECURITY_REVIEW.md`](../security/SECURITY_REVIEW.md)
- [`../security/SECURITY_IFRAME_STRICT_CSP_DESIGN.md`](../security/SECURITY_IFRAME_STRICT_CSP_DESIGN.md)

## Mail Operations

Project Z-Bridge adds or improves several operator-visible mail behaviors:

- Tags-column sorting for message and conversation review.
- Source export directly from the mail context menu.
- Archive action integrated into normal move behavior.
- Large-selection continuation fixes for common bulk actions.
- More predictable connection-lost and session-expired behavior.

These are intended to preserve classic ZWC workflows while making the Stalwart-backed deployment more usable for large real-world mailboxes.

## Preferences and Policy Controls

Bridge-managed preference extensions appear inside familiar ZWC Preferences screens:

- Rspamd Analysis toggle.
- Bridge Safety controls for phishing/foreign-country tagging.
- Stored appearance and signature preferences backed by the bridge store.
- Extended Out-of-Office behavior, including alias-targeted replies.

Some settings are user-facing controls; others materialize as Stalwart-backed Sieve behavior.

## Filters, Sieve, and Retention

Project Z-Bridge maps ZWC filter behavior into Stalwart-compatible delivery-time Sieve where supported. It also keeps backups of active scripts before activation.

Retention support bridges a ZWC UI feature that Stalwart does not expose in the same form:

- Folder retention policies can be saved through ZWC.
- Optional bridge sweeps can enforce purge rules.
- Admins must still align this with Stalwart global cleanup settings.

Retention reference:

- [`SYSADMIN_RETENTION_POLICY.md`](SYSADMIN_RETENTION_POLICY.md)

## Performance and Large Mailboxes

Project Z-Bridge includes bridge-side performance work that is not part of stock ZWC alone:

- Cached `GetInfoResponse` startup data.
- Search and message-open caches.
- Conversation seed caching.
- Optional push-driven hot-folder prewarm.
- Deferred mini-calendar startup behavior with cache reuse.

Operator tuning reference:

- [`SYSADMIN_CACHE_TUNING_GUIDE.md`](SYSADMIN_CACHE_TUNING_GUIDE.md)
- [`SYSADMIN_PERF_STATS_REFERENCE.md`](SYSADMIN_PERF_STATS_REFERENCE.md)

## How To Maintain This List

Use this document for admin-facing feature inventory.

Use [`../developer/PRODUCT_ENHANCEMENTS.md`](../developer/PRODUCT_ENHANCEMENTS.md) for implementation details, design notes, and maintainer caveats.

When adding a new enhancement:

1. Add a concise admin-facing entry here.
2. Add operational training or runbook documentation under `docs/sysadmin/` if admins need to use or support it.
3. Add deeper implementation notes to the developer enhancement tracker when the behavior changes ZWC internals or bridge architecture.

