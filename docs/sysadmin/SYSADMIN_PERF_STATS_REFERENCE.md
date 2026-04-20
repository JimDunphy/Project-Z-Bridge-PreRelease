# Perf Stats Reference (Sysadmin)

This guide explains the output of:
- `./manage.sh perf-stats-show --foreground`
- `./manage.sh perf-stats-show --all`
- `./manage.sh perf-stats-snapshot`
- `./manage.sh perf-stats-reset`

Primary data file:
- `.dev/data/perf-stats.json` (dev default)
- `${BRIDGE_DATA_DIR}/perf-stats.json` (runtime path)

## 1) Modes

`--foreground`:
- Excludes `NoOpRequest` from totals and top-method ranking.
- Best for user-facing latency analysis.

`--all`:
- Includes all methods, including `NoOpRequest`.
- Best for total bridge workload analysis.

## 2) Header Fields

- `Source`: where stats were read from (dev host file/dev container/runtime container).
- `Updated`: last write timestamp for stats snapshot.
- `Started`: counter epoch start time (since last reset/init).

## 3) Totals Section

Timing and faults:
- `calls`: total SOAP method completions counted.
- `faults`: count of methods with SOAP faults.
- `fault_rate`: `faults / calls`.
- `avg_ms`: `total_ms / calls`.
- `max_ms`: slowest single method duration.

Upstream JMAP:
- `jmap_calls`: total upstream JMAP calls.
- `jmap_ms`: cumulative upstream JMAP time.
- `jmap_bytes`: cumulative upstream JMAP response bytes.
- `jmap_errors`: upstream JMAP call errors.

Slow-call counters:
- `slow_over_250ms`
- `slow_over_500ms`
- `slow_over_1000ms`
- `slow_over_3000ms`

NoOp breakdown helpers:
- `noop_wait_ms`: total long-poll wait time.
- `noop_work_ms`: total background work time performed in NoOp handlers.

## 4) NoOp Breakdown Line

When present:
- `calls`: number of `NoOpRequest` methods.
- `avg_wait_ms`: average long-poll wait portion.
- `avg_work_ms`: average in-handler work portion.
- `total_ms`: cumulative NoOp runtime.

Interpretation:
- High wait is expected.
- Rising work indicates heavier background maintenance activity.

## 5) Top Methods by `total_ms`

Per method fields:
- `calls`
- `avg_ms`
- `total_ms`
- `max_ms`
- `faults`

Use this to identify dominant contributors to perceived slowness.

## 6) Top Batch Submethods by Attributed `batch_ms`

These are inferred from `BatchRequest` contents and attributed batch duration.

Fields:
- `batches`: number of batches containing that submethod.
- `slow_batches`: count where parent batch exceeded slow threshold.
- `slow_rate`: `slow_batches / batches`.
- `avg_batch_ms`: average parent batch duration when this submethod appears.
- `total_batch_ms`: sum of parent batch durations.
- `max_batch_ms`: max parent batch duration seen.
- `faults`: parent batch faults.

Interpretation:
- This is attribution, not isolated per-submethod wall time.
- High `total_batch_ms` + high `slow_rate` is a strong optimization target signal.

## 7) Fault Reason Buckets

Tracked for:
- `AuthRequest`
- `ConvActionRequest`

Each bucket:
- fault reason string -> count

Use this to detect recurring failure classes without scanning raw logs.

## 8) Recommended Sampling Workflow

1. Reset counters before a controlled run:
- `./manage.sh perf-stats-reset`

2. Exercise scenario/workload:
- login, folder switches, message reads, filter edits, etc.

3. Capture summary:
- `./manage.sh perf-stats-show --foreground`

4. Save snapshot for history:
- `./manage.sh perf-stats-snapshot .dev/perf-stats/<label>.json`

5. Compare snapshots over time:
- focus on `avg_ms`, `max_ms`, slow-call counters, and top-method ordering.

## 9) Quick Interpretation Rules

- If foreground `avg_ms` looks good but users report slowness, inspect method-specific `max_ms`.
- If `SearchRequest` and `GetMsgRequest` dominate, investigate list/read-path JMAP calls first.
- If `ModifyFilterRulesRequest` is high, verify debounce/local-model behavior and upstream sync frequency.
- If `fault_rate` rises, read raw logs immediately for exact fault payloads and context.
