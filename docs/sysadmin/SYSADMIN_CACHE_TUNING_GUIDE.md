# Cache and Performance Tuning Guide

Status: current operator runbook. Re-audit after `.env.example` cache/default
changes, push/NoOp behavior changes, or perf-log wording changes.

This guide is for operators tuning bridge responsiveness vs upstream load.

Use this with:
- [`SYSADMIN_PERF_STATS_REFERENCE.md`](SYSADMIN_PERF_STATS_REFERENCE.md)
- [`SYSADMIN_LOG_READING_GUIDE.md`](SYSADMIN_LOG_READING_GUIDE.md)
- [`../developer/PERFORMANCE_CACHE_AUDIT.md`](../developer/PERFORMANCE_CACHE_AUDIT.md)

## Goals and Tradeoffs

The bridge can prioritize two things:
- lower user-visible latency ("feels instant")
- lower upstream JMAP/CPU load

Most knobs are a tradeoff between those two.

## Defaults and Why They Exist

These are the main defaults currently used by the shim:

- `BRIDGE_BUSY_OVERLAY_DELAY_MS=700`
  - Hides short busy-cursor flashes so normal clicks feel smooth.
- `BRIDGE_NOOP_WAIT_SECS=30`
  - Long-poll wait; reduces request churn while preserving near-live updates.
- `BRIDGE_CONVERSATION_CACHE_SECS=300`
  - Keeps folder search results warm for common mailbox clicks.
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE=1`
  - Serves a stale folder cache entry immediately, then refreshes in background.
- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE_MIN_INTERVAL_MS=2000`
  - Prevents revalidate storms for repeated clicks on same folder.
- `BRIDGE_EMAIL_QUERY_CHANGES_ENABLED=1`
  - Lets simple-folder stale revalidation use guarded `Email/queryChanges`
    before falling back to a full rebuild.
- `BRIDGE_EMAIL_QUERY_CHANGES_DELTA_ENABLED=1`
  - Allows small safe changed deltas to patch the offset-zero cached page.
- `BRIDGE_NOOP_MAILBOX_CHANGES_ENABLED=1`
  - Lets quiet NoOps use `Mailbox/changes` to avoid full mailbox/head refreshes.
- `BRIDGE_PUSH_WARM_ENABLED=1`
  - Uses upstream push events when available to warm hot queries.
- `BRIDGE_PUSH_HOT_MAX_QUERIES=4`
  - Chosen to keep overhead bounded: 3 default folders (`in:inbox`, `in:junk`, `in:sent`) plus ~1 recent folder.
- `BRIDGE_PUSH_PREWARM_LIMIT=100`
  - Matches common conversation list page size and avoids underfetch.
- `BRIDGE_PUSH_PREWARM_MIN_INTERVAL_MS=5000`
  - Prevents push bursts from causing repeated warm passes. NoOp fallback warming has its own interval and does not consume this Push throttle.
- `BRIDGE_PUSH_NOOP_PREWARM_ENABLED=1`
  - Keeps hot queries warm even when push is quiet or unavailable.
- `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS=300000`
  - Conservative default (5 minutes) to limit background load.
- `BRIDGE_CONTACTS_CF_CACHE_SECS=3600`
  - Contacts CF startup payload is heavy; caching avoids repeated rebuilds.
- `BRIDGE_MAILBOX_CACHE_SECS=300`
  - Mailbox tree/count fetches are reused, while still refreshed often.
- `BRIDGE_GETINFO_CACHE_SECS=86400`
  - `GetInfoRequest` is expensive and mostly stable for a user.
- `BRIDGE_APPOINTMENT_SEARCH_CACHE_SECS=3600`
  - Persists calendar appointment `SearchRequest` responses so background calendar bootstrap is fast after relogin/restart.
- `BRIDGE_GETMSG_CACHE_SECS=45`
  - Short-lived message payload cache for rapid open/re-open.
- `BRIDGE_GETMSG_CACHE_MAX_ENTRIES=512`
  - Caps per-session in-memory message payload cache size.
- `BRIDGE_GETMSG_PREFETCH_COUNT=24`
  - Prefetches likely next reads without overwhelming upstream.
- `BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES=5242880`
  - Caps body bytes requested by prefetch `Email/get` calls.
- `BRIDGE_THREAD_SUMMARY_CACHE_SECS=30`
  - Reuses recent conversation counts, aggregate flags, and sender participant
    summaries during list/read-ahead flows.
- `BRIDGE_THREAD_SUMMARY_CACHE_MAX_ENTRIES=4096`
  - Caps short-lived thread summary entries.
- `BRIDGE_FILTER_ADDRESS_BOOKS_CACHE_SECS=120`
  - Avoids repeated full contact expansion during filter operations.
- `BRIDGE_FILTER_MODIFY_SYNC_DEBOUNCE_MS=1500`
  - Coalesces rapid filter UI edits before upstream sync.

## Knob Reference

### Folder/search responsiveness

- `BRIDGE_CONVERSATION_CACHE_SECS` (default `300`)
  - `0` disables conversation result cache.
  - Lower = fresher but more upstream calls.
  - Higher = faster repeat folder clicks, more chance of stale brief view.

- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE` (default `1`)
  - `1` serves eligible simple-folder cache immediately and refreshes in background.
  - This includes stale cache and small folder-count mismatches such as new mail arriving since the last login.
  - `0` disables the background revalidate trigger; explicit internal prefetches still request fresh data.

- `BRIDGE_SEARCH_CACHE_STALE_REVALIDATE_MIN_INTERVAL_MS` (default `2000`, clamp `200..120000`)
  - Lower = more frequent background refreshes.
  - Higher = less refresh churn.

Bridge-owned mail actions on collapsed conversations:
- `Spam` / `Not Spam`, `Trash`, `Move`, and read/unread can return from a local projection when folder/search caches are warm and the selected rows are provable.
- The bridge patches source/destination folder rows, folder totals, unread counts, and related conversation seeds, then finishes the authoritative JMAP mutation in the background.
- Read/unread folder-count projection prefers current JMAP `mailboxIds` and `$seen` state so a post-move read in Junk decrements Junk immediately instead of waiting for the next NoOp/background reconciliation.
- Opening a cached unread message can return the cached body before the upstream mark-read `Email/set` finishes. The bridge projects the read flag and folder unread count locally, then reconciles live counts and conversation flags in the background.
- The fast path is guarded. If the bridge cannot prove the selected thread shape, source rows, destination folder, or mailbox count projection, it falls back to synchronous behavior or removes only known stale rows.
- Post-move cache revalidation is delayed until after the upstream move finishes so the bridge does not refresh folder caches from old server state.
- During an in-flight bridge-owned move/trash projection, larger first-window folder reads can be answered from a compatible warm projection so the UI does not briefly re-import pre-move upstream state.
- If a source first page becomes short and the next page is already cached, the bridge backfills from that continuation cache. If no continuation page is available, users may briefly see a correct but shorter list rather than waiting on upstream.
- Verification logs include `ConvActionRequest optimistic delete/trash: patched folder move cache`, `ConvActionRequest optimistic read: applied folder unread projection`, `ConvActionRequest optimistic background action scheduled`, `patched_folder_cache=true`, `ConvActionRequest background action completed`, `GetMsgRequest returning cached body before background mark-read reconciliation`, and `GetMsgRequest background mark-read completed`.

### Push and hot-folder prewarm

- `BRIDGE_PUSH_WARM_ENABLED` (default `1`)
- `BRIDGE_PUSH_HOT_MAX_QUERIES` (default `4`, clamp `3..16`)
- `BRIDGE_PUSH_PREWARM_LIMIT` (default `100`, clamp `20..200`)
- `BRIDGE_PUSH_PREWARM_MIN_INTERVAL_MS` (default `5000`, clamp `500..120000`)
- `BRIDGE_PUSH_NOOP_PREWARM_ENABLED` (default `1`)
- `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS` (default `300000`, clamp `30000..3600000`)

Verification logs:
- JMAP Push is connected when logs show `JMAP Push event source connected` with `source="jmap_push"`.
- JMAP Push actually received a mail/count event when logs show `JMAP Push event received` with `source="jmap_push"`, `mail_change=true`, and `reason="mail_change"`.
- JMAP Push caused cache warming when logs show `hot search prewarm started` with `source="jmap_push"` and `trigger="mail_change"`.
- `mail_change=false` is not automatically a problem. Check `reason`: `no_changed_section` is usually heartbeat/ping-style traffic, `account_not_changed` means the event was not for this account, and `non_mail_change` means this account changed but not Email/Mailbox.
- NoOp/poll discovered new mail when logs show `NoOpRequest: inbox changed` with `source="noop"`.
- NoOp fallback caused cache warming when logs show `NoOp hot search prewarm scheduled`, followed by `hot search prewarm started` with `source="noop"`.
- NoOp fallback and JMAP Push use separate warm gates; a periodic NoOp warm should not make a later real `mail_change=true` Push event skip its warm pass.

Operator note:
- Lower `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS` if first click after idle is slow.
- In production testing, `60000` (60s) often gives a noticeably "always warm" feel.

Why default hot queries is `4`:
- It intentionally caps speculative work per active session.
- It guarantees the three core folders stay hot and leaves one slot for a recent folder.
- Raising it improves "cold folder" first-click behavior, but increases continuous background load.

Prewarm cost model (rule of thumb):
- Background load scales with:
  - `hot_queries * (60 / prewarm_interval_seconds) * cost_per_query`
- Typical observed cost per query:
  - cache-hot query: usually a few milliseconds and often near-zero upstream work
  - cold refresh query: roughly ~1s class work and multiple upstream JMAP calls

Admin recommendation:
- Keep counts as the primary user signal (folder unread/total changes), then tune hot prewarm for click feel.
- Start with `BRIDGE_PUSH_HOT_MAX_QUERIES=4` and `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS=60000`.
- Increase hot queries only if users routinely jump across many non-default folders and you have upstream capacity.

NoOp foreground budget:
- NoOp still performs notification-critical checks after the long-poll wait, but feed polling and retention sweeps run as background maintenance. In logs, `noop_work_ms` should mostly reflect mailbox/Inbox/calendar notification checks rather than feed import or retention purge cost.
- When `Mailbox/changes` reports only updated existing folders, NoOp can fetch those folder records and patch cached mailbox metadata instead of fetching the full mailbox tree. Unsafe cases still fall back to the full path.
- Background maintenance logs as `NoOp background maintenance complete`. If it imports or destroys messages, it refreshes mailbox/GetInfo count caches for the next UI-visible update.

### Message-open acceleration

- `BRIDGE_GETMSG_CACHE_SECS` (default `45`)
- `BRIDGE_GETMSG_CACHE_MAX_ENTRIES` (default `512`, max `50000`)
- `BRIDGE_GETMSG_PREFETCH_COUNT` (default `24`, max `200`)
- `BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES` (default `5242880`, clamp `65536..26214400`)
- `BRIDGE_SEARCH_CONV_SEED_CACHE_SECS` (default `30`)
- `BRIDGE_SEARCH_CONV_SEED_CACHE_MAX_ENTRIES` (default `1024`, max `50000`)
- `BRIDGE_THREAD_SUMMARY_CACHE_SECS` (default `30`)
- `BRIDGE_THREAD_SUMMARY_CACHE_MAX_ENTRIES` (default `4096`, max `50000`)

### Above/Below-Fold Prefetch Controls

- `BRIDGE_CONV_ABOVE_FOLD_BODY_PREFETCH_ENABLED` (default `1`)
  - `1` warms message bodies for currently visible conversation rows.
  - Set `0` to reduce background load (at cost of colder first-click opens).

- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_ENABLED` (default `1`)
  - `1` prefetches next conversation pages' metadata (beneath current page).
  - First-page folder opens (`offset=0`) use delayed idle prefetch instead of immediate fan-out.
  - Set `0` to disable scroll-ahead prewarm.

- `BRIDGE_CONV_INITIAL_METADATA_PREFETCH_IDLE_DELAY_MS` (default `1500`, clamp `250..30000`)
  - Delay before first-page (`offset=0`) next-page conversation metadata prefetch runs.
  - The delayed prefetch only fires if the same folder/query is still the active hot query.
  - Lower values warm the next page sooner but can steal time from the first visible page.

- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_PAGES` (default `3`, clamp `1..3`)
  - Number of pages ahead to schedule for metadata prefetch.
  - Higher values can improve scroll smoothness but increase background work.
  - First-page conversation opens warm continuation pages using the continuation page size (`limit=50`) that ZWC requests while scrolling.

- `BRIDGE_CONV_SCROLL_METADATA_PREFETCH_MIN_INTERVAL_MS` (default `5000`, clamp `500..120000`)
  - Per session/query/page cooldown for scroll metadata prefetch scheduling.

- `BRIDGE_SEARCH_PREFETCH_NEXT_WINDOW_ENABLED` (default `0`)
  - Optional next-window body warmup for message-mode list paging.

- `BRIDGE_SEARCH_PREFETCH_NEXT_WINDOW_LIMIT` (default `24`, clamp `1..200`)
  - Max IDs warmed for next-window prefetch.
  - Ignored when `BRIDGE_SEARCH_PREFETCH_NEXT_WINDOW_ENABLED=0`.

### Login/startup and folder tree

- `BRIDGE_CONTACTS_CF_CACHE_SECS` (default `3600`)
- `BRIDGE_MAILBOX_CACHE_SECS` (default `300`)
- `BRIDGE_GETINFO_CACHE_SECS` (default `86400`)
- `BRIDGE_APPOINTMENT_SEARCH_CACHE_SECS` (default `3600`)

Operational note:
- Bridge-owned mail actions such as Spam/Not Spam, move, trash/delete, and read-state updates should not normally force a cold `GetInfoRequest` on the next login. The bridge patches cached folder counts when it already fetched live mailbox counts for the action notification. In logs, look for `GetInfo cache folder counts patched`; a follow-up relogin should show `GetInfoRequest ... cache_hit=true` with `upstream_jmap_calls=0`.
- Preference saves should also keep the startup cache hot. `ModifyPrefsRequest` overlays lightweight prefs at response-build time, Bridge Safety/Rspamd mail prefs patch the cached mail-pref/tag snapshot instead of invalidating the whole `GetInfoRequest` payload, and identical mail-pref saves skip the upstream Sieve rewrite.
- Collapsed-conversation Spam/Not Spam, Trash, Move, and read/unread can also patch active conversation lists and folder counts optimistically, then complete the upstream mutation in the background. For read/unread, the preferred count delta comes from current JMAP message state rather than a possibly stale cached conversation row. If the action touches a thread shape the bridge cannot verify exactly, synchronous behavior is expected and safer.
- New mail is batch-tolerant for simple folder views. A warm `in:inbox`/`in:junk` cache can be shown immediately even when counts changed, with `count_mismatch_revalidate=true` logging the background refresh path.
- Background calendar appointment searches can otherwise be multi-second on large calendars after restart. The persisted appointment search cache serves the prior response immediately and logs `SearchRequest appointment served from cache`; stale entries refresh asynchronously.

### UI feel and long-poll behavior

- `BRIDGE_BUSY_OVERLAY_DELAY_MS` (default `700`, max `60000`)
  - Increase if users notice frequent brief busy flashes.
  - Decrease if users need immediate "working" feedback.

- `BRIDGE_NOOP_WAIT_SECS` (default `30`, effective max `300`)
  - Longer reduces poll churn.
  - Shorter can feel more "chatty" but costs more request overhead.

- `BRIDGE_NOOP_MAILBOX_CHANGES_ENABLED` (default `1`)
  - Leave enabled unless debugging NoOp behavior.
  - When upstream supports it, quiet NoOps use JMAP `Mailbox/changes` to avoid full mailbox/head refreshes.
  - Simple updated-folder deltas can log `NoOpRequest: Mailbox/changes patched cached mailbox metadata`; created/destroyed mailbox changes, missing caches, and patch mismatches fall back to a full mailbox refresh.

- `BRIDGE_EMAIL_QUERY_CHANGES_ENABLED` (default `1`)
  - Leave enabled unless debugging conversation-cache revalidation.
  - Background simple-folder revalidation can use JMAP `Email/queryChanges` to refresh cache metadata when a cached `in:inbox` / `in:junk` style query is unchanged, avoiding the heavier full `Email/query` + `Email/get` + `Thread/get` rebuild.
  - Small safe changed deltas can patch the offset-zero cached page when `BRIDGE_EMAIL_QUERY_CHANGES_DELTA_ENABLED` is enabled.
  - If state is missing, upstream cannot calculate changes, or the changed delta is unsafe, the bridge falls back to the existing full revalidate path.

- `BRIDGE_EMAIL_QUERY_CHANGES_DELTA_ENABLED` (default `1`)
  - Leave enabled for normal operation.
  - Disable only when isolating simple-folder cache update behavior; disabling preserves quiet `Email/queryChanges` checks but returns changed deltas to the full rebuild path.

- Thread summary cache:
  - Short-lived per-session memory cache for conversation count, aggregate flags, and sender participant summaries.
  - Useful for repeated folder clicks and read-ahead scrolling over the same threads.
  - Bridge-owned mail mutations invalidate the cache for the affected account; external changes age out by TTL.

- `BRIDGE_MAIL_POLL_INTERVAL` (default `500`)
  - `500` keeps ZWC instant-notify behavior.

## Tuning Playbooks

### Playbook A: Make folder clicks feel instant

Use when users report slight lag after idle periods.

Recommended:
```env
BRIDGE_PUSH_WARM_ENABLED=1
BRIDGE_PUSH_NOOP_PREWARM_ENABLED=1
BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS=60000
BRIDGE_SEARCH_CACHE_STALE_REVALIDATE=1
BRIDGE_CONVERSATION_CACHE_SECS=300
BRIDGE_BUSY_OVERLAY_DELAY_MS=700
```

Expected effect:
- First click after idle is usually cache-served.
- Busy cursor appears less often for folder navigation.

### Playbook B: Reduce upstream/background load

Use when server CPU/network usage is high.

Recommended:
```env
BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS=300000
BRIDGE_PUSH_PREWARM_LIMIT=50
BRIDGE_GETMSG_PREFETCH_COUNT=4
BRIDGE_GETMSG_PREFETCH_MAX_BODY_BYTES=1048576
BRIDGE_CONVERSATION_CACHE_SECS=180
```

Expected effect:
- Less speculative work.
- Slightly higher chance of visible delay on cold clicks.

### Playbook C: Consistency-first (less stale view risk)

Use when operators prefer stricter freshness over raw speed.

Recommended:
```env
BRIDGE_SEARCH_CACHE_STALE_REVALIDATE=0
BRIDGE_CONVERSATION_CACHE_SECS=120
BRIDGE_MAILBOX_CACHE_SECS=120
```

Expected effect:
- More synchronous upstream fetches.
- Lower chance of briefly showing stale folder list content.

## How to Tune Safely

1. Reset baseline:
```bash
./manage.sh perf-stats-reset
```

2. Run a realistic workflow for 15-30 minutes.

3. Check:
```bash
./manage.sh perf-stats-show --foreground
```

4. If folder clicks feel slow after idle:
- lower `BRIDGE_PUSH_NOOP_PREWARM_INTERVAL_MS` first.

5. If upstream load is too high:
- lower prewarm aggressiveness (`BRIDGE_PUSH_PREWARM_LIMIT`, prewarm interval).

6. Re-test and compare snapshots.

## What to Watch in Logs

- Fast path hit:
  - `SearchRequest conversation served from cache ... stale_revalidate=true`
- Background refresh:
  - `SearchRequest stale cache revalidate started`
  - `mailboxes cache refreshed from upstream` for stale folder-list revalidation that needs trusted folder counts.
  - `SearchRequest conversation Email/queryChanges quiet; refreshed cache metadata without full revalidate` when a stale simple-folder cache was confirmed unchanged without rebuilding the list.
  - `SearchRequest conversation Email/queryChanges delta; patched cache without full revalidate` when a small safe changed query delta updated the cached first page.
  - `SearchRequest conversation Email/queryChanges delta unsafe; falling back to full revalidate` when the query changed but the bridge correctly chose the authoritative path.
  - `NoOpRequest: Mailbox/changes patched cached mailbox metadata` when a simple mailbox-count delta avoided a full mailbox-tree refresh.
- Periodic keep-warm:
  - `NoOp prewarm: warming hot search queries`

If you only see full `SearchRequest conversation perf` lines on frequent clicks, hot folders are not staying warm and intervals are likely too conservative.
