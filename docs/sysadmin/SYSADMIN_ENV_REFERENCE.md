# Environment Variable Reference

This document is the operator-facing guide to the main `.env` settings used by Project Z-Bridge.

Use it as the readable reference. Treat `.env.example` as the complete source of truth for the full variable list and current defaults.

## Required for basic local startup

At minimum, set:

- `STALWART_BASE_URL`
  - Base URL for the upstream Stalwart server.
  - Example: `https://mail.example.com`

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

## UI / user-experience defaults

- `BRIDGE_DEFAULT_SKIN`
  - Default ZWC theme/skin.

- `BRIDGE_TIMEZONE`
  - Default timezone presented to the ZWC UI.

- `BRIDGE_MINICAL_DELAY_MS`
  - Delay before showing the mini calendar.

- `BRIDGE_BUSY_OVERLAY_DELAY_MS`
  - Delay before showing the global busy overlay/cursor.

## Mail performance / cache tuning

Common hot-path tuning variables:

- `BRIDGE_GETMSG_CACHE_SECS`
- `BRIDGE_GETMSG_CACHE_MAX_ENTRIES`
- `BRIDGE_GETMSG_PREFETCH_COUNT`
- `BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES`
- `BRIDGE_CONVERSATION_CACHE_SECS`
- `BRIDGE_MAILBOX_CACHE_SECS`
- `BRIDGE_GETINFO_CACHE_SECS`
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE`
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE_MIN_INTERVAL_MS`

Use these only when tuning responsiveness or debugging cache behavior. For scenario guidance, see:

- `docs/sysadmin/SYSADMIN_CACHE_TUNING_GUIDE.md`

## Push / polling behavior

- `BRIDGE_MAIL_POLL_INTERVAL`
  - ZWC polling interval preference.

- `BRIDGE_NOOP_WAIT_SECS`
  - Server-side hold time for instant-notify `NoOpRequest`.

- `BRIDGE_PUSH_WARM_ENABLED`
- `BRIDGE_PUSH_HOT_MAX_QUERIES`
- `BRIDGE_PUSH_PREWARM_LIMIT`
- `BRIDGE_PUSH_PREWARM_MIN_INTERVAL_MS`
- `BRIDGE_PUSH_NOOP_PREWARM_ENABLED`
- `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS`

These control push-assisted hot-folder prewarming and the polling fallback path.

## Mail rendering / security

- `BRIDGE_CSP_MODE`
- `BRIDGE_CSP_REPORT_URI`
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

- `docs/security/SECURITY_EMAIL_IFRAME_SANDBOX_ROLLOUT_PLAN.md`
- `docs/security/SECURITY_IFRAME_CSP_HEADER_VERIFICATION.md`

## Filters / Sieve

- `BRIDGE_SIEVE_ACTIVATE`
  - Enables delivery-time filter activation against upstream Sieve.

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
- `BRIDGE_FILTER_ADDRESS_SET_MAX`
- `BRIDGE_FILTER_SIEVE_SCRIPT_MAX_BYTES`

For filter behavior and tradeoffs, see:

- `docs/developer/FILTERS_PLAN.md`

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
- `BRIDGE_AI_RUNNER_URL`
- `BRIDGE_AI_RUNNER_TOKEN`
- `BRIDGE_AI_RUNNER_BIND`
- `BRIDGE_AI_RUNNER_PORT`

These are used for the local AI automation extension and host-side runner integration.

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
