# Project Z-Bridge

Rust “shim” that translates the classic Zimbra web client `/service/soap` API into JMAP calls for Stalwart Mail Server.

## Documentation index

Start with these “entry points” (by audience):

- Project onboarding: [`PROJECT_Z_BRIDGE_ONBOARDING.md`](PROJECT_Z_BRIDGE_ONBOARDING.md)
- Developer docs: [`developer/README.md`](developer/README.md)
- Sysadmin docs: [`sysadmin/README.md`](sysadmin/README.md)
- Public-facing notes: [`public/README.md`](public/README.md)

Security docs:

- Security/audit docs: [`security/README.md`](security/README.md)

Project trackers:

- Known bugs / gaps: [`BUGS.md`](BUGS.md)
- Status / handoff notes: [`STATUS.md`](STATUS.md)

## Cross-reference rule

If a document is listed in an index or referenced for follow-up reading, it should be a clickable Markdown link.

Use paths relative to the document doing the linking:

- good: `[developer/README.md](developer/README.md)` from `docs/README.md`
- good: [`ARCHITECTURE.md`](developer/ARCHITECTURE.md) from `docs/developer/README.md`
- avoid plain backtick path listings when the intent is navigation

## How to use the docs (types)

The docs in this repo fall into a few common types:

- **Reference**: stable “what exists” documentation (API routes/methods, module map).
- **Plan**: design notes for future work (kept as current as practical, but not always implemented yet).
- **Postmortem / fix notes**: what broke, why, and how it was fixed (helpful when regressions return).
- **Audit**: security and threat-model notes.
- **Historical**: early explorations or one-time agent snapshots (useful context, but not required day-to-day).

If you’re not sure where to start for a problem, use:

- [`developer/DEV_DEBUGGING.md`](developer/DEV_DEBUGGING.md)
- [`developer/DEV_WORKDIR.md`](developer/DEV_WORKDIR.md) (what caches exist and what to delete first)

## High-value docs (common tasks)

- **Project onboarding overview**: [`PROJECT_Z_BRIDGE_ONBOARDING.md`](PROJECT_Z_BRIDGE_ONBOARDING.md)
- **Deploy / operate**: [`sysadmin/SYSADMIN_DEPLOY.md`](sysadmin/SYSADMIN_DEPLOY.md)
- **ZWC + Stalwart enhanced features**: [`sysadmin/SYSADMIN_ZWC_STALWART_ENHANCED_FEATURES.md`](sysadmin/SYSADMIN_ZWC_STALWART_ENHANCED_FEATURES.md)
- **Rspamd admin training workflow**: [`sysadmin/SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md`](sysadmin/SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md)
- **Operational log triage**: [`sysadmin/SYSADMIN_LOG_READING_GUIDE.md`](sysadmin/SYSADMIN_LOG_READING_GUIDE.md)
- **Attachment-sort operator triage**: [`sysadmin/SYSADMIN_ATTACHMENT_SORT_TRIAGE.md`](sysadmin/SYSADMIN_ATTACHMENT_SORT_TRIAGE.md)
- **Perf counter reference**: [`sysadmin/SYSADMIN_PERF_STATS_REFERENCE.md`](sysadmin/SYSADMIN_PERF_STATS_REFERENCE.md)
- **Reverse proxy / TLS / hardening**: [`sysadmin/SYSADMIN_PRODUCTION_HARDENING.md`](sysadmin/SYSADMIN_PRODUCTION_HARDENING.md)
- **ZWC asset version bumps**: [`sysadmin/SYSADMIN_WEBCLIENT_EXTRACT.md`](sysadmin/SYSADMIN_WEBCLIENT_EXTRACT.md)
- **Publishing sanitized trees**: [`developer/DEV_PUBLISHING.md`](developer/DEV_PUBLISHING.md)
- **Login performance**: [`developer/PERFORMANCE_LOGIN.md`](developer/PERFORMANCE_LOGIN.md)
- **Attachment sort debugging/perf**: [`developer/PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md`](developer/PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md)
- **Implemented/stubbed API surface**: [`developer/API_REFERENCE.md`](developer/API_REFERENCE.md)
- **Testing paths and middleware interop**: [`developer/TESTING_AND_MIDDLEWARE_INTEROP.md`](developer/TESTING_AND_MIDDLEWARE_INTEROP.md)
- **Future completion roadmap**: [`developer/PRODUCT_FUTURE_COMPLETION_PLAN.md`](developer/PRODUCT_FUTURE_COMPLETION_PLAN.md)
- **Architecture / trust boundaries**: [`developer/ARCHITECTURE.md`](developer/ARCHITECTURE.md)
- **Mail rendering security design (iframe + CSP)**: [`security/SECURITY_IFRAME_STRICT_CSP_DESIGN.md`](security/SECURITY_IFRAME_STRICT_CSP_DESIGN.md)

Feature deep-dives:

- Mail sharing (internal): [`developer/SHARING_SHARED_FOLDERS.md`](developer/SHARING_SHARED_FOLDERS.md)
- Filters (Sieve): [`developer/FILTERS_PLAN.md`](developer/FILTERS_PLAN.md)
- Rspamd analysis/training: [`sysadmin/SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md`](sysadmin/SYSADMIN_RSPAMD_ANALYSIS_TRAINING.md)
- Out-of-Office / vacation (including per-recipient alias reply extension): [`developer/PREFERENCES_OUT_OF_OFFICE_PLAN.md`](developer/PREFERENCES_OUT_OF_OFFICE_PLAN.md)
- Contacts: [`developer/CONTACTS_FIELD_MAPPING.md`](developer/CONTACTS_FIELD_MAPPING.md), [`developer/CONTACTS_SEARCH.md`](developer/CONTACTS_SEARCH.md), [`developer/CONTACTS_SHARING.md`](developer/CONTACTS_SHARING.md)
- Calendar: [`developer/CALENDAR_INVITES.md`](developer/CALENDAR_INVITES.md), [`developer/CALENDAR_SCHEDULER.md`](developer/CALENDAR_SCHEDULER.md), [`developer/CALENDAR_ALARMS.md`](developer/CALENDAR_ALARMS.md)
- RSS feeds: [`developer/RSS_FEEDS.md`](developer/RSS_FEEDS.md)

## Suggested archive candidates

These are still useful references, but are not required for day-to-day maintenance and could be moved under a future `docs/archive/` without losing critical “how to run” guidance:

- [`developer/ARCHITECTURE_DESIGN_HISTORICAL.md`](developer/ARCHITECTURE_DESIGN_HISTORICAL.md) (early technical design; historical)
- [`developer/DEV_CODEX_CONTEXT.md`](developer/DEV_CODEX_CONTEXT.md) (agent snapshot; historical)
- [`DependencyAnalysis.md`](DependencyAnalysis.md) (one-off analysis; historical)
- [`WhyUseRust.md`](WhyUseRust.md) (background/positioning; optional)

## Dev workflow

- Configure port via `BRIDGE_PORT` (defaults to `7777`).
- Optional: configure default timezone via `BRIDGE_TIMEZONE` (e.g. `America/Los_Angeles`).
- Optional: configure instant-notify wait time via `BRIDGE_NOOP_WAIT_SECS` (defaults to `30`).
- Optional: set `BRIDGE_DATA_DIR` to persist shim-side UI metadata via the `BridgeStore` (default: a JSON `FileStore` under `../.dev/data` in docker-compose dev). This is **not stateless**: a horizontally scaled deployment must use a shared backend (shared volume/Redis/SQL) or move that metadata upstream where feasible. See [`developer/ARCHITECTURE_STATELESSNESS_PLAN.md`](developer/ARCHITECTURE_STATELESSNESS_PLAN.md).
- Filters (Sieve): in docker-compose dev, `BRIDGE_SIEVE_ACTIVATE` defaults to `0` (model-only; does not change Stalwart’s active delivery script). Set `BRIDGE_SIEVE_ACTIVATE=1` to compile+validate+activate delivery-time filters on save (see [`developer/FILTERS_PLAN.md`](developer/FILTERS_PLAN.md) and [`developer/FILTERS_SIEVE_BACKUPS.md`](developer/FILTERS_SIEVE_BACKUPS.md)).
  - For large rule sets and frequent reorder edits, tune:
    - `BRIDGE_FILTER_ADDRESS_BOOKS_CACHE_SECS` (cache Contacts/My Frequent Emails expansion).
    - `BRIDGE_SIEVE_BACKUP_MIN_INTERVAL_SECS` (throttle backup frequency during repeated saves).
    - `BRIDGE_FILTER_MODIFY_SYNC_DEBOUNCE_MS` (coalesce rapid filter edits before upstream sync).
- Optional: set `BRIDGE_BRAND_*` env vars to replace the top-left ZWC banner logo/link (defaults to Stalwart).
- Optional: override mini calendar delay via `BRIDGE_MINICAL_DELAY_MS` (defaults to `5000`; set `0` for immediate mini calendar creation).
- Optional: delay global busy cursor/overlay via `BRIDGE_BUSY_OVERLAY_DELAY_MS` (defaults to `700`; set `0` for stock behavior).
- Optional: enable push-driven hot-folder prewarm via `BRIDGE_PUSH_WARM_ENABLED` and tune with `BRIDGE_PUSH_HOT_MAX_QUERIES`, `BRIDGE_PUSH_PREWARM_LIMIT`, `BRIDGE_PUSH_PREWARM_MIN_INTERVAL_MS`, `BRIDGE_PUSH_NOOP_PREWARM_ENABLED`, and `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS`.
- Optional: enable dumpster listing/restore via `BRIDGE_DUMPSTER_ENABLED` (`inDumpster` search + `recover` action path). Tune with `BRIDGE_DUMPSTER_API_BASE_URL` and `BRIDGE_DUMPSTER_PAGE_SIZE`; optional admin override auth is available via `BRIDGE_DUMPSTER_ADMIN_BEARER_TOKEN` or `BRIDGE_DUMPSTER_ADMIN_USERNAME`/`BRIDGE_DUMPSTER_ADMIN_PASSWORD`. Current limitation: restore destination picker is not fully honored yet (Stalwart undelete restores to Inbox).
- Optional: tune folder-search cache lifetime via `BRIDGE_CONVERSATION_CACHE_SECS` (set `0` to disable).
- Optional: enable stale-while-revalidate for simple folder clicks via `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE` and `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE_MIN_INTERVAL_MS`.
- Optional: tune message-open acceleration via `BRIDGE_GETMSG_CACHE_SECS`, `BRIDGE_GETMSG_CACHE_MAX_ENTRIES`, `BRIDGE_GETMSG_PREFETCH_COUNT`, and `BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES`.
- Optional: tune single-message conversation expansion caching via `BRIDGE_SEARCH_CONV_SEED_CACHE_SECS` and `BRIDGE_SEARCH_CONV_SEED_CACHE_MAX_ENTRIES`.
- For operator-level tuning guidance and scenario playbooks, see [`sysadmin/SYSADMIN_CACHE_TUNING_GUIDE.md`](sysadmin/SYSADMIN_CACHE_TUNING_GUIDE.md).
- Optional: persist ZWC themes per user via `ModifyPrefsRequest` (`zimbraPrefSkin`); set `BRIDGE_DEFAULT_SKIN` to change the default.
- Optional: enable best-effort CSP via `BRIDGE_CSP_MODE`/`BRIDGE_CSP`; use `BRIDGE_CSP_MODE=log` + `BRIDGE_CSP_REPORT_URI=/zimbra/csp-report` (proxy-safe default) to collect violation reports in the shim logs.
- Use `./manage.sh` to build/run/test via Docker.

## Quick start

- Put `zimbra.war` (classic web client build) in the repo root.
  - If you don't already have it, you can build the classic webclient WAR from open-source sources.
    The simplest current documented path is `./manage.sh webclient-build`, which downloads the latest supported `zimbra.war` release artifact from `DockerZimbraRHEL8`.
    If you need a specific release, use `./manage.sh webclient-build --version 10.1.16`.
    If you want to build your own WAR manually, use `DockerZimbraRHEL8` (`https://github.com/JimDunphy/DockerZimbraRHEL8`) with `./docker.sh --build-war 10.1`.
- Extract it: `./manage.sh webclient-extract` (writes to `static/zimbra/`, gitignored)
  - If you are swapping to a newer ZWC version and something regresses, see [`sysadmin/SYSADMIN_WEBCLIENT_EXTRACT.md`](sysadmin/SYSADMIN_WEBCLIENT_EXTRACT.md) for the known failure modes and first checks.
- Copy `.env.example` → `.env` and set at least `STALWART_BASE_URL`.
- `./manage.sh build`
- `./manage.sh up` (publishes `127.0.0.1:${BRIDGE_PORT:-7777}` -> container `:7777`)
- `./manage.sh shell` (no published ports)

- Open: `http://127.0.0.1:${BRIDGE_PORT:-7777}/zimbra/`
- Login with the same credentials you use for Stalwart (see `.env`).

Note: `/zimbra/` is the current (compatibility-first) mount point for the ZWC UI. A future enhancement will allow configuring the mount point (example: `/mail/`) via env var; see [`developer/ARCHITECTURE_CONTEXT_PATH_PLAN.md`](developer/ARCHITECTURE_CONTEXT_PATH_PLAN.md).

Note: ZWC assets (`zimbra.war` / `static/zimbra/`) are intentionally not redistributed by this repo; see [`sysadmin/SYSADMIN_ZIMBRA_TRADEMARK_AND_LICENSING.md`](sysadmin/SYSADMIN_ZIMBRA_TRADEMARK_AND_LICENSING.md).

Endpoints: `GET /healthz`, `POST /service/soap`, `GET /zimbra/*`

## Upstream (Stalwart)

- For the UI, you only need `STALWART_BASE_URL` set; the login screen uses the username/password you type.
- `STALWART_USERNAME` / `STALWART_PASSWORD` are used by dev helpers (`./manage.sh probe`, `./manage.sh share-probe`, `./manage.sh filters-smoke`, `cargo run --bin soap_smoke`).

## `manage.sh` commands

Use `./manage.sh help` for the raw CLI usage text.

For the maintained, grouped command reference (including prerequisites, when-to-use guidance, and workflow recipes), use:

- [`sysadmin/SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md`](sysadmin/SYSADMIN_MANAGE_SH_COMMAND_REFERENCE.md)

## Notes

- The shim implements enough of classic ZWC to boot and show Mail:
  - SOAP: `AuthRequest`, `GetInfoRequest`, `BatchRequest` (JSON + SOAP XML), `GetFolderRequest`, `SearchRequest` (JMAP-backed), `GetMsgRequest` (JMAP-backed), plus a few startup stubs (`GetMailboxMetadataRequest`, `DiscoverRightsRequest`, `/home/*`).
- Shared mail folders (internal-only): shared mailboxes discovered via Stalwart mailbox ACLs (`Mailbox.shareWith`) are exposed as ZWC mountpoints; see [`developer/SHARING_SHARED_FOLDERS.md`](developer/SHARING_SHARED_FOLDERS.md).
- Local artifacts are intentionally not committed: `zimbra.war`, `static/zimbra/`, `.env`, and `.dev/` are gitignored.

Known issues are tracked in [`BUGS.md`](BUGS.md).

Bridge-side enhancements beyond stock ZWC are documented in [`developer/PRODUCT_ENHANCEMENTS.md`](developer/PRODUCT_ENHANCEMENTS.md).

## Migration tooling (early)

- `smmailbox/smmailbox` — local CLI wrapper around `imapsync` + helpers for exporting Zimbra tag metadata and mapping it to IMAP keywords. See [`../smmailbox/docs/SMMAILBOX_QUICKSTART.md`](../smmailbox/docs/SMMAILBOX_QUICKSTART.md).
