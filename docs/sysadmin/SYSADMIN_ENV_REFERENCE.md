# Environment Variable Reference

Last reviewed: 2026-05-16.

This document is the operator-facing guide to the main `.env` settings used by Project Z-Bridge.

Use it as the readable reference. Treat `.env.example` as the complete source
of truth for bridge-process `.env` variables and current defaults. Host-side
helper variables used by `manage.sh` or the AI runner are documented separately
below because they may be exported in the shell instead of stored in `.env`.

## Required for basic local startup

At minimum, set:

- `STALWART_BASE_URL`
  - Base URL for the upstream Stalwart server.
  - Example: `https://mail.example.com`

- `STALWART_API_BASE_URL`
  - Optional base URL for Stalwart admin/account API calls such as account
    security and dumpster support.
  - Defaults to `STALWART_BASE_URL` when unset.
  - Use this when the browser-facing mail URL differs from the internal API URL.

Common local-development auth variables:

- `STALWART_USERNAME`
- `STALWART_PASSWORD`
  - Used by helper commands, probes, and browser regression checks.
  - Recommended: point these to a Stalwart account used for repeated verification and smoke testing.

## Common bridge/runtime settings

- `BRIDGE_PORT`
  - Host port for the bridge.
  - Default: `7777`

- `BRIDGE_BIND`
  - Optional bind address when running the shim directly.
  - In docker-compose dev, leave the default container binding alone unless you know why you are changing it.

- `BRIDGE_PUBLIC_BASE_URL`
  - Public URL used when the bridge is deployed behind a reverse proxy.
  - Important for absolute URLs and reverse-proxy correctness.
  - Also used by the bridge as an HTTPS signal for cookie behavior when the public URL is `https://...`.

- `BRIDGE_COOKIE_SECURE`
  - Set to `1` behind HTTPS so `ZM_AUTH_TOKEN` is marked `Secure`.
  - Prevents browsers from sending the auth cookie over plain HTTP.
  - Important: the bridge does not infer this from `X-Forwarded-Proto` alone; use this flag and/or `BRIDGE_PUBLIC_BASE_URL=https://...`.

- `BRIDGE_DATA_DIR`
  - Persistent local data/cache directory for the shim.
  - In dev compose this typically points into `.dev/data`.

- `BRIDGE_COMPAT_TRACE_ENABLED`
  - Optional SOAP/REST compatibility trace for middleware integration debugging.
  - Default: `0`.
  - When enabled, the bridge appends JSONL records for observed request shapes, response status, timing, and payload sizes.
  - Default `shape` detail omits auth tokens, cookies, raw message bodies, upload content, filenames, email addresses, and private search text.

- `BRIDGE_COMPAT_TRACE_DETAIL`
  - Compatibility trace detail level.
  - Default: `shape`.
  - `shape`: sanitized public request shape only.
  - `values`: adds `containsPrivate=true` and a `.private` object with exact developer-debug values such as ids, folder paths, queries, action targets, and filenames.
  - `full`: adds full parsed SOAP request/response JSON for local developer replay with `compat-trace-dump`.
  - `full` mode may include passwords, auth tokens, message bodies, contacts, calendar data, and other private mailbox content. Do not share full traces.

- `BRIDGE_COMPAT_TRACE_PATH`
  - Optional output path for the compatibility trace.
  - Default: `${BRIDGE_DATA_DIR}/compat-trace.jsonl`.
  - Render the trace for humans with `./manage.sh compat-trace-show --tail 25`.
  - Follow new calls live with `./manage.sh compat-trace-follow`.
  - Pretty-print full developer replay records with `./manage.sh compat-trace-dump` when `BRIDGE_COMPAT_TRACE_DETAIL=full`.
  - Redact private detail blocks with `./manage.sh compat-trace-redact`.

- `WEBCLIENT_ROOT`
  - Filesystem path to extracted ZWC assets.
  - Dev default: `../static/zimbra`.
  - Runtime image default: `/webclient`.

- `BRIDGE_ASSET_VERSION`
  - Optional cache-buster version string for shim-served web assets.
  - If unset, the shim uses a fresh value per start.

## `manage.sh` / build helper environment

These are host-side helper variables. They are read by `manage.sh` or Docker
Compose before the bridge process starts.

- `HOST_UID`
- `HOST_GID`
  - UID/GID used for the development container build/user mapping.
  - Defaults to the current host user's `id -u` / `id -g`.

- `BRIDGE_IMAGE`
  - Runtime image tag used by runtime-image commands.
  - Default: `project-z-bridge:runtime`.

- `BUILD_ZIMBRA_RELEASE_REPO`
- `BUILD_ZIMBRA_RELEASE_ASSET`
- `BUILD_ZIMBRA_WAR_VERSION`
  - Optional overrides for `./manage.sh webclient-build`.
  - Defaults to the configured supported `zimbra.war` release repository and
    latest release asset.

## UI / user-experience defaults

- `BRIDGE_DEFAULT_SKIN`
  - Default ZWC theme/skin.

- `BRIDGE_TIMEZONE`
  - Default timezone presented to the ZWC UI.

- `BRIDGE_MINICAL_DELAY_MS`
  - Delay before showing the mini calendar.
  - Default: `5000`.
  - Set `0` for immediate creation.

- `BRIDGE_BUSY_OVERLAY_DELAY_MS`
  - Delay before showing the global busy overlay/cursor.

Branding/about variables:

- `BRIDGE_BRAND_NAME`
- `BRIDGE_BRAND_LINK_URL`
- `BRIDGE_BRAND_LOGO_URL`
- `BRIDGE_BRAND_LOGO_PATH`
- `BRIDGE_BRAND_FAVICON_URL`
- `BRIDGE_ABOUT_PRODUCT_NAME`
  - Control the ZWC chrome branding and Help/About product text.
  - `BRIDGE_BRAND_LOGO_URL` is browser-visible; `BRIDGE_BRAND_LOGO_PATH` is the
    filesystem source for the shim-served default logo.

## Mail performance / cache tuning

Common hot-path tuning variables:

- `BRIDGE_GETMSG_CACHE_SECS`
- `BRIDGE_GETMSG_CACHE_MAX_ENTRIES`
- `BRIDGE_GETMSG_PREFETCH_COUNT`
- `BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES`
- `BRIDGE_CONVERSATION_CACHE_SECS`
- `BRIDGE_CONTACTS_CF_CACHE_SECS`
- `BRIDGE_MAILBOX_CACHE_SECS`
- `BRIDGE_GETINFO_CACHE_SECS`
- `BRIDGE_APPOINTMENT_SEARCH_CACHE_SECS`
- `BRIDGE_JMAP_INCREMENTAL_MODE`
- `BRIDGE_NOOP_MAILBOX_CHANGES_ENABLED`
- `BRIDGE_SHARED_BRIEFCASE_NOOP_REFRESH_MS`
- `BRIDGE_EMAIL_QUERY_CHANGES_ENABLED`
- `BRIDGE_EMAIL_QUERY_CHANGES_DELTA_ENABLED`
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE`
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE_MIN_INTERVAL_MS`
- `BRIDGE_CONV_ABOVE_FOLD_BODY_PREFETCH_ENABLED`
- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_ENABLED`
- `BRIDGE_CONV_INITIAL_METADATA_PREFETCH_IDLE_DELAY_MS`
- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_PAGES`
- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_MIN_INTERVAL_MS`
- `BRIDGE_SEARCH_PREFETCH_NEXT_WINDOW_ENABLED`
- `BRIDGE_SEARCH_PREFETCH_NEXT_WINDOW_LIMIT`
- `BRIDGE_SEARCH_CONV_SEED_CACHE_SECS`
- `BRIDGE_SEARCH_CONV_SEED_CACHE_MAX_ENTRIES`
- `BRIDGE_THREAD_SUMMARY_CACHE_SECS`
- `BRIDGE_THREAD_SUMMARY_CACHE_MAX_ENTRIES`
- `BRIDGE_EMAIL_GET_CHUNK_SIZE`
- `BRIDGE_ATTACHMENT_SORT_SCAN_THREADS`
- `BRIDGE_THREAD_GET_CHUNK_SIZE`

Use these only when tuning responsiveness or debugging cache behavior. For scenario guidance, see:

- [`SYSADMIN_CACHE_TUNING_GUIDE.md`](SYSADMIN_CACHE_TUNING_GUIDE.md)
- [`SYSADMIN_ATTACHMENT_SORT_TRIAGE.md`](SYSADMIN_ATTACHMENT_SORT_TRIAGE.md)

## Upload limits

- `BRIDGE_UPLOAD_MAX_BYTES`
  - Optional lower limit for uploads accepted by `/service/upload` and advertised to ZWC for attachments and Briefcase documents.
  - Default: Stalwart's JMAP core `maxSizeUpload`.
  - If the override is higher than Stalwart's advertised maximum, the bridge logs a warning and uses Stalwart's maximum.

## Push / polling behavior

- `BRIDGE_MAIL_POLL_INTERVAL`
  - ZWC polling interval preference.

- `BRIDGE_NOOP_WAIT_SECS`
  - Server-side hold time for instant-notify `NoOpRequest`.

- `BRIDGE_NOOP_MAILBOX_CHANGES_ENABLED`
  - Default `true`. Lets NoOp use JMAP `Mailbox/changes` to skip full mailbox/head refreshes when upstream reports no mailbox changes. Simple updated-folder deltas can patch cached mailbox metadata from targeted `Mailbox/get` results. Falls back to full refresh if state is missing, expired, unsupported, created/destroyed folders are present, or the patch is unsafe.

- `BRIDGE_SHARED_BRIEFCASE_NOOP_REFRESH_MS`
  - Default `5000`. Rate-limits active-session shared Briefcase mount refreshes during `NoOpRequest`. Lower values make background grant/revoke mount checks more eager for already-logged-in grantees; higher values reduce share-state checks. Owner-side grant/revoke actions also mark active grantee sessions as pending so their sleeping NoOp can wake early. The bridge emits ZWC `notify.created.link` / `notify.deleted.id` updates when the visible mount set changes.

- `BRIDGE_EMAIL_QUERY_CHANGES_ENABLED`
  - Default `true`. Lets background simple-folder conversation-cache revalidation use JMAP `Email/queryChanges` to skip a full list rebuild when the cached query state is still unchanged. It may also handle count-mismatch cases when the query delta is small and safe. Falls back to a full revalidate if query state is missing, upstream cannot calculate changes, or the query cannot be safely updated.

- `BRIDGE_EMAIL_QUERY_CHANGES_DELTA_ENABLED`
  - Default `true`. Allows small safe changed `Email/queryChanges` deltas to patch offset-zero simple-folder conversation caches. The bridge still falls back to full revalidate for `hasMoreChanges`, missing query state, shared-account caches, missing added-row details, or removals that would require unknown backfill.

- `BRIDGE_PUSH_WARM_ENABLED`
- `BRIDGE_PUSH_HOT_MAX_QUERIES`
- `BRIDGE_PUSH_PREWARM_LIMIT`
- `BRIDGE_PUSH_PREWARM_MIN_INTERVAL_MS`
- `BRIDGE_PUSH_NOOP_PREWARM_ENABLED`
- `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS`
- `BRIDGE_PUSH_CLOSEAFTER`
- `BRIDGE_PUSH_PING_SECS`
- `BRIDGE_PUSH_EVENT_TYPES`

These control push-assisted hot-folder prewarming and the polling fallback path.

## Mail rendering / security

- `BRIDGE_CSP_MODE`
- `BRIDGE_CSP_REPORT_URI`
- `BRIDGE_CSP`
  - Controls app-shell CSP mode and report ingestion.

- `BRIDGE_IMAGE_PROXY_SECRET`
  - Signing secret for the bridge image proxy.
  - Required for stable `/service/image-proxy/...` URLs across restart.
  - Also prevents callers from fabricating arbitrary proxy fetch URLs without a valid signature.

- `BRIDGE_SECURITY_IDN_PUNYCODE`
  - Best-effort punycode normalization for IDN links in HTML mail.

- `BRIDGE_EMAIL_IFRAME_SANDBOX_ENABLED`
- `BRIDGE_EMAIL_IFRAME_SANDBOX_POLICY`
- `BRIDGE_EMAIL_IFRAME_SANDBOX_USER_ALLOWLIST`
  - Controls the fallback sandboxed mail iframe path.

- `BRIDGE_EMAIL_IFRAME_RENDERER_ENABLED`
- `BRIDGE_EMAIL_IFRAME_RENDERER_USER_ALLOWLIST`
- `BRIDGE_EMAIL_IFRAME_RENDERER_ORIGIN`
- `BRIDGE_EMAIL_IFRAME_RENDERER_TTL_SECS`
- `BRIDGE_EMAIL_IFRAME_RENDERER_MAX_BYTES`
- `BRIDGE_EMAIL_IFRAME_RENDERER_MAX_ENTRIES`
- `BRIDGE_EMAIL_IFRAME_RENDERER_SANDBOX_POLICY`
  - Controls the dedicated renderer-origin mail view path.

For the security model behind these settings, see:

- [`../security/SECURITY_EMAIL_IFRAME_SANDBOX_ROLLOUT_PLAN.md`](../security/SECURITY_EMAIL_IFRAME_SANDBOX_ROLLOUT_PLAN.md)
- [`../security/SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md`](../security/SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md)

## Filters / Sieve

- `BRIDGE_SIEVE_ACTIVATE`
  - Enables delivery-time filter activation against upstream Sieve.
  - Compose files default this to `0` unless `.env` sets it. `.env.example`
    currently sets it to `1` as an explicit opt-in sample for active
    delivery-time filtering.

- `BRIDGE_SIEVE_INCOMING_SCRIPT_NAME`
- `BRIDGE_SIEVE_INCOMING_MODEL_SCRIPT_NAME`
- `BRIDGE_SIEVE_OUTGOING_SCRIPT_NAME`
- `BRIDGE_SIEVE_BASE_ORDER`
- `BRIDGE_MANAGED_FOREIGN_COUNTRY_SYMBOLS`
  - Comma-separated `X-Spamd-Result` tokens used by the hidden managed `Foreign Country` tag.
  - Default: `BLACKLIST_COUNTRY(`.
  - Recommendation: keep the opening `(` when matching normal Rspamd `SYMBOL(score)` output because the generated hidden Sieve rule uses raw `header :contains` matching, not structured symbol parsing. If your deployment emits a bare symbol with no score, override this with `BLACKLIST_COUNTRY`.
- `BRIDGE_SIEVE_BACKUP_PREFIX`
- `BRIDGE_SIEVE_BACKUP_KEEP`
- `BRIDGE_SIEVE_BACKUP_MIN_INTERVAL_SECS`
- `BRIDGE_FILTER_MODIFY_SYNC_DEBOUNCE_MS`
- `BRIDGE_FILTER_ADDRESS_BOOKS_CACHE_SECS`
- `BRIDGE_FILTER_ADDRESS_SET_MAX`
- `BRIDGE_FILTER_SIEVE_SCRIPT_MAX_BYTES`

For filter behavior and tradeoffs, see:

- [`../developer/FILTERS_PLAN.md`](../developer/FILTERS_PLAN.md)

## RSS / retention / dumpster

- `BRIDGE_FEED_POLL_SECS`
  - Background RSS/Atom polling interval while users are logged in.

- `BRIDGE_RETENTION_SWEEP_SECS`
  - Folder retention enforcement sweep interval.
  - Default: `86400` (`24h`).
  - Sweeps only run while a user session is active.

- `BRIDGE_RETENTION_MAX_DESTROY_PER_SWEEP`
  - Retention destroy cap per sweep across all folders.
  - Default: `2000`.
  - Applies to both save-triggered immediate sweeps and later automatic sweeps.
  - The sweep interval gate itself is in-memory; a bridge restart resets it.

- `BRIDGE_DUMPSTER_ENABLED`
- `BRIDGE_DUMPSTER_API_BASE_URL`
- `BRIDGE_DUMPSTER_ADMIN_BEARER_TOKEN`
- `BRIDGE_DUMPSTER_ADMIN_USERNAME`
- `BRIDGE_DUMPSTER_ADMIN_PASSWORD`
- `BRIDGE_DUMPSTER_PAGE_SIZE`

## AI / extension controls

- `BRIDGE_AI_ENABLED`
- `BRIDGE_AI_DEPLOYMENT_MODE`
- `BRIDGE_AI_PROVIDERS`
- `BRIDGE_AI_PROVIDER_PREFLIGHT_TIMEOUT_MS`
- `BRIDGE_AI_PROVIDER_PLAN_TIMEOUT_MS`
- `BRIDGE_AI_CODEX_MODEL`
- `BRIDGE_AI_PROVIDER_FAILOVER_ENABLED`
- `BRIDGE_AI_PLAN_ALLOW_HEURISTIC_FALLBACK`
- `BRIDGE_AI_RUNNER_URL`
- `BRIDGE_AI_RUNNER_TOKEN`
- `BRIDGE_AI_RUNNER_BIND`
- `BRIDGE_AI_RUNNER_PORT`
- `BRIDGE_AI_RUNNER_CONTEXT_DIR`
- `BRIDGE_AI_RUNNER_COMPOSE_SESSION_DIR`
- `BRIDGE_AI_RUNNER_COMPOSE_SESSION_TTL_HOURS`
- `BRIDGE_AI_RUNNER_COMPOSE_HISTORY_LIMIT`
- `BRIDGE_AI_RUNNER_DEFAULT_TIMEOUT_MS`
- `BRIDGE_AI_RUNNER_MAX_TIMEOUT_MS`
- `BRIDGE_AI_RUNNER_ALLOW_PROVIDERS`
- `BRIDGE_AI_RUNNER_STRICT_SCHEMA`
- `BRIDGE_AI_RUNNER_ALLOW_LOCAL_FALLBACK`

These are used for the local AI automation extension and host-side runner integration.
Compose session variables control the runner-side short-lived workspace used by
the AI Compose Assistant to let Codex/Claude/Gemini work on the same reply.
The default session TTL is 24 hours and the default prior-suggestion history
included in later prompts is 3 records.

## Developer/debug-only switches

- `BRIDGE_DEBUG_CONTACTS_CF`
  - Enables extra contact canonical-form debug logging.
  - Leave unset/disabled unless actively debugging contacts export or contact
    REST canonical-form behavior.

## Logging

- `RUST_LOG`
  - Standard Rust logging override.
  - Example: `RUST_LOG=debug`
  - Narrow bridge-focused example: `RUST_LOG=zbridge_shim=debug`

## Recommended operator workflow

1. Copy `.env.example` to `.env`.
2. Set the minimum required values first.
3. Start the bridge with defaults.
4. Change tuning/security flags only after you have a working baseline.
5. When changing behavior-sensitive settings, restart the bridge:

```bash
./manage.sh restart
```
