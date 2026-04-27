# Sysadmin Docs

This directory is for **operators** deploying Project Z-Bridge.

## Naming Convention

Use explicit `SYSADMIN_*` filenames for operator-focused docs.

Start here:

- [`SYSADMIN_DEPLOYMENT_TOPOLOGY.md`](SYSADMIN_DEPLOYMENT_TOPOLOGY.md) — preferred production topology (edge -> bridge -> Stalwart, plus MX/network separation).
- [`SYSADMIN_ZWC_STALWART_ENHANCED_FEATURES.md`](SYSADMIN_ZWC_STALWART_ENHANCED_FEATURES.md) — admin-facing inventory of bridge enhancements compared with stock ZWC + Zimbra.
- [`SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md`](SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md) — operator training guide for Rspamd score hover, detailed analysis, and raw-message export workflows.
- [`SYSADMIN_DEPLOY.md`](SYSADMIN_DEPLOY.md) — docker-compose dev + runtime deployment notes.
- [`SYSADMIN_ENV_REFERENCE.md`](SYSADMIN_ENV_REFERENCE.md) — operator-facing guide to the main `.env` settings and where to tune them.
- [`SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md`](SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md) — grouped command map for `./manage.sh` (what to run, when, prerequisites).
- [`SYSADMIN_LOG_READING_GUIDE.md`](SYSADMIN_LOG_READING_GUIDE.md) — how to read bridge logs (`soap method complete`, NoOp, cache and filter lines) and triage incidents.
- [`SYSADMIN_ATTACHMENT_SORT_TRIAGE.md`](SYSADMIN_ATTACHMENT_SORT_TRIAGE.md) — quick runbook for paperclip/attachment-sort regressions (log signatures + tuning).
- [`SYSADMIN_PERF_STATS_REFERENCE.md`](SYSADMIN_PERF_STATS_REFERENCE.md) — field-by-field reference for `./manage.sh perf-stats-*` outputs and interpretation.
- [`SYSADMIN_CACHE_TUNING_GUIDE.md`](SYSADMIN_CACHE_TUNING_GUIDE.md) — cache/push/prewarm tuning defaults, tradeoffs, and scenario playbooks.
- [`../developer/PERFORMANCE_CACHE_AUDIT.md`](../developer/PERFORMANCE_CACHE_AUDIT.md) — developer-facing audit of what is cached today, what is regenerated today, and which decisions should be settled before deeper cache changes.
- [`SYSADMIN_BUNDLE_EXPORT_IMPORT.md`](SYSADMIN_BUNDLE_EXPORT_IMPORT.md) — export/import “environment bundles” for moving a working setup between machines/users.
- [`SYSADMIN_PRODUCTION_HARDENING.md`](SYSADMIN_PRODUCTION_HARDENING.md) — recommended reverse proxy headers, TLS, and other hardening.
- [`SYSADMIN_EXTERNAL_HARDENING.md`](SYSADMIN_EXTERNAL_HARDENING.md) — rate limiting, abuse resistance, fail2ban/WAF integration.
- [`SYSADMIN_WEBCLIENT_EXTRACT.md`](SYSADMIN_WEBCLIENT_EXTRACT.md) — extracting `zimbra.war` into `static/zimbra/` and handling version bumps.
- [`SYSADMIN_ACCOUNT_SECURITY.md`](SYSADMIN_ACCOUNT_SECURITY.md) — password changes, 2FA, app passwords (Stalwart semantics and impact).

Legal / licensing:

- [`SYSADMIN_ZIMBRA_TRADEMARK_AND_LICENSING.md`](SYSADMIN_ZIMBRA_TRADEMARK_AND_LICENSING.md)

Operational notes / TODOs:

- [`SYSADMIN_RETENTION_POLICY.md`](SYSADMIN_RETENTION_POLICY.md)
- [`SYSADMIN_EMAILED_CONTACTS_CLEANUP.md`](SYSADMIN_EMAILED_CONTACTS_CLEANUP.md) — bulk-delete/reset Stalwart “Emailed Contacts” address book when it has unexpected volume.
- [`SYSADMIN_EXECUTIVE_SUMMARY.md`](SYSADMIN_EXECUTIVE_SUMMARY.md) (overview targeted at leadership; keep current and factual)
- [`SYSADMIN_OBSERVABILITY_PLAN.md`](SYSADMIN_OBSERVABILITY_PLAN.md) — phased plan for growing log/perf operator docs and runbooks.
