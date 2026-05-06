# Testing and Middleware Interoperability

This document explains how Project Z-Bridge is tested and what those tests imply for teams that already have middleware built around Zimbra.

The short version:

- Project Z-Bridge exposes a **Zimbra-shaped SOAP/REST compatibility surface** for classic ZWC and related tooling.
- There is no private test backdoor into the running bridge.
- Browser and API tests call the same HTTP endpoints that ZWC calls.
- Rust unit tests are different: they compile test binaries and call internal functions directly.

## What Project Z-Bridge Replaces

Project Z-Bridge is not a full Zimbra server. It replaces the parts of `mailboxd` needed to run classic ZWC against Stalwart:

- Serves classic ZWC assets under `/zimbra/*`.
- Accepts ZWC SOAP traffic under `/service/soap` and `/service/soap/*`.
- Accepts selected ZWC REST-like flows under `/home/*`, `/service/home/*`, and `/service/upload`.
- Translates supported operations to Stalwart JMAP, Sieve, submission, and blob APIs.
- Stores some UI-only compatibility metadata in `BridgeStore`.

It does not provide Zimbra admin SOAP, mailboxd JSP behavior, Zimlet server infrastructure, ZimbraSync, or arbitrary private mailboxd internals.

## Test Paths

| Path | What It Runs | Needs Running Bridge? | Needs Rebuild/Restart After Rust Changes? | Best For |
| --- | --- | --- | --- | --- |
| Rust unit tests | `cargo test` compiled test binaries that call Rust functions directly | No | No running bridge involved; tests compile their own binary | Pure transforms, parsers, SOAP shape, prefs, cache algorithms |
| Smoke binaries | `shim/src/bin/*` tools run by `manage.sh` | Sometimes | Only if the smoke talks to the running bridge and server code changed | JMAP/upstream invariants and narrow bridge flows |
| Direct HTTP/SOAP tests | External scripts call `/service/soap`, `/home/*`, etc. | Yes | Yes, if server behavior changed | Middleware compatibility, endpoint-level regression checks |
| Playwright UI regression | Chrome logs into ZWC and drives the real UI | Yes | Yes, if server behavior changed | ZWC/browser contracts, context menus, downloads, rendering, performance evidence |
| Manual browser checks | A human uses ZWC | Yes | Yes, if server behavior changed | Exploratory testing and product feel |

## Rust Unit Tests

Rust tests are run with commands such as:

```bash
cargo test --manifest-path shim/Cargo.toml modify_prefs_updates -- --nocapture
```

These tests do not talk to the bridge process on port `7777`. Cargo compiles a separate test binary, links the Rust code under test, and runs functions directly.

Use this path for:

- Zimbra-to-JMAP shape transforms.
- Preference parsing and `GetInfoResponse` defaults.
- SOAP XML parsing.
- HTML sanitization/linkification logic.
- Header parsing.
- Cache-key and cache-update algorithms.

Do not use this path as proof that the running bridge has the new behavior. If the browser or an external client will call the bridge over HTTP, restart the bridge before validating that path.

## Smoke Binaries

Smoke tests live under `shim/src/bin/*` and are exposed through `manage.sh`.

Examples:

```bash
./manage.sh probe
./manage.sh share-probe
./manage.sh calendar-probe
./manage.sh filters-smoke
./manage.sh email-render-smoke
./manage.sh invite-xss-smoke
```

There are two patterns:

- **Upstream-only smoke tests** call Stalwart/JMAP directly using `.env` credentials.
- **Bridge smoke tests** call the running bridge and therefore require `./manage.sh up` or an equivalent running service.

If a smoke test calls the running bridge, it uses the same public HTTP surface that clients use. Rebuild/restart first when Rust server code changed.

## Direct HTTP/SOAP API Testing

External tests can call the bridge like any ZWC-compatible client.

Core endpoints:

- `POST /service/soap`
- `POST /service/soap/`
- `POST /service/soap/<RequestName>`
- `GET/POST /home/*`
- `GET/POST /service/home/*`
- `POST /service/upload`

Auth flow:

1. Send `AuthRequest` to `/service/soap`.
2. Store the returned `ZM_AUTH_TOKEN` cookie or `AuthResponse.authToken`.
3. Send later SOAP calls with the cookie and/or `Header.context.authToken`.

The bridge returns Zimbra-shaped JSON envelopes. SOAP faults are usually `200 OK` with `Body.Fault`, matching ZWC expectations.

Minimal direct SOAP example:

```bash
curl -sS http://127.0.0.1:7777/service/soap \
  -H 'content-type: application/json' \
  -d '{
    "Body": {
      "AuthRequest": {
        "_jsns": "urn:zimbraAccount",
        "account": { "by": "name", "_content": "user@example.com" },
        "password": "secret"
      }
    }
  }'
```

For the current implemented surface, use:

- [`API_REFERENCE.md`](API_REFERENCE.md)
- [`SOAP_COMPATIBILITY_MATRIX.md`](SOAP_COMPATIBILITY_MATRIX.md)
- [`API_STUBS.md`](API_STUBS.md)

## Playwright UI Regression

The browser harness uses `playwright-core` and a local Chrome/Chromium. It logs into ZWC and exercises the real browser UI.

Primary command:

```bash
./manage.sh email-ui-regression --case all
```

Focused example:

```bash
./manage.sh email-ui-regression \
  --case mail-list-fragments \
  --user user@example.com \
  --password 'CHANGE_ME'
```

What it validates:

- The ZWC JavaScript model sees the expected data.
- The DOM renders expected behavior.
- SOAP requests complete without faults.
- Per-case SOAP timing and bridge-log summaries are captured.

Artifacts are written under:

```text
.dev/email-ui-regressions/<case>-<timestamp>/
```

The harness has two important modes:

- **Non-destructive cases** can run against live mailboxes with read-only or reversible checks.
- **Destructive cases** require `--allow-destructive` and an allowlisted account. They use `ZZ-Regress...` folders/tags or restore original state where possible.

More detail:

- [`DEV_TEST_STRATEGY.md`](DEV_TEST_STRATEGY.md)
- [`REGRESSION_COVERAGE_MATRIX.md`](REGRESSION_COVERAGE_MATRIX.md)
- [`../security/SECURITY_EMAIL_UI_REGRESSION_AUTOMATION.md`](../security/SECURITY_EMAIL_UI_REGRESSION_AUTOMATION.md)

## When a Bridge Restart Is Required

Use this rule:

- If you changed Rust server code and then test via browser, Playwright, curl, or any external client, restart/rebuild the bridge first.
- If you run `cargo test`, Cargo compiles the code under test and does not need the running bridge.
- If you changed only docs or test scripts, no bridge restart is needed.
- If you changed served static assets or injected client JavaScript, a browser reload may be enough, but a bridge restart is still safest if the asset is generated by Rust.

Common restart paths:

```bash
./manage.sh restart
```

For detached dev operation:

```bash
docker compose -f docker-compose.yml up -d --build
```

Confirm the bridge is reachable:

```bash
curl -fsS http://127.0.0.1:7777/healthz
```

## Middleware Compatibility Model

External middleware can integrate with Project Z-Bridge in three practical ways.

### 1. Existing ZWC-style SOAP/REST middleware

If the middleware already calls the same user-facing Zimbra SOAP/REST endpoints that classic ZWC uses, it may work by pointing at Project Z-Bridge.

Expect to validate method by method:

- `AuthRequest`
- `GetInfoRequest`
- `SearchRequest`
- `GetMsgRequest`
- `MsgActionRequest`
- `ConvActionRequest`
- contacts/calendar/folder/preference methods as needed
- `/home/*` export/download flows
- `/service/upload`

Start with [`SOAP_COMPATIBILITY_MATRIX.md`](SOAP_COMPATIBILITY_MATRIX.md) because it calls out
top-level support, batch-dispatch gaps, SOAP XML caveats, and stubbed compatibility responses.

This is the best fit.

### 2. Middleware that uses Zimbra admin/private APIs

If the middleware depends on Zimbra admin SOAP, mailboxd internals, JSP endpoints, Zimlet server APIs, or ZCS-specific provisioning behavior, it should not be assumed compatible.

Options:

1. Repoint that part of the middleware to Stalwart APIs directly.
2. Add a new Project Z-Bridge endpoint if it is truly a user-facing compatibility need.
3. Keep a separate administrative integration layer outside the bridge.

Project Z-Bridge intentionally keeps the runtime smaller than mailboxd.

### 3. Middleware that wants JMAP-native integration

If the middleware is not tied to Zimbra SOAP, prefer Stalwart JMAP directly for new work.

Use the bridge when you need:

- Classic ZWC compatibility.
- Zimbra-shaped SOAP/REST behavior.
- Bridge-specific enhancements such as source export, Rspamd UI support, or ZWC preference compatibility.

Use Stalwart directly when you need:

- Backend automation independent of ZWC.
- Admin/provisioning behavior.
- Non-Zimbra clients.
- High-volume integration where Zimbra-shaped compatibility is unnecessary.

## Compatibility Validation Checklist for External Middleware

For each middleware product or integration, capture:

1. Which endpoints it calls.
2. Whether it uses SOAP JSON, SOAP XML, REST, upload, or browser-only flows.
3. Whether it expects Zimbra admin SOAP or user SOAP.
4. Whether it depends on Zimbra-specific IDs, folder ids, tag ids, or mountpoint semantics.
5. Whether it expects Zimbra mailboxd side effects such as server-side HTML defanging, retention, filters, or zimlet behavior.
6. Whether it treats SOAP faults as HTTP errors or in-body `Body.Fault`.
7. Whether it requires long-lived sessions across bridge restarts.
8. Whether it can authenticate using Stalwart credentials through `AuthRequest`.

Then build a focused test lane:

1. Start with direct SOAP/API calls for every endpoint it needs.
2. Add Playwright only when browser behavior matters.
3. Add Rust unit tests only for bridge-side transforms or compatibility logic added for that middleware.
4. Add a row to [`REGRESSION_COVERAGE_MATRIX.md`](REGRESSION_COVERAGE_MATRIX.md) for any real regression found during validation.

## Known Differences From Zimbra `mailboxd`

Important differences to communicate early:

- Session tokens are bridge session tokens, not Zimbra mailboxd tokens.
- Sessions are in memory today; bridge restart causes re-auth.
- Core user data lives in Stalwart.
- Some UI metadata lives in `BridgeStore`; horizontal deployments need shared storage or sticky sessions plus shared `BRIDGE_DATA_DIR`.
- Unknown SOAP methods currently receive permissive generic responses where possible, but that does not mean the underlying behavior exists.
- Unsupported server-side Zimbra products such as admin console, ZimbraSync, Briefcase, and full zimlet server parity are not provided.
- ZWC assets are bring-your-own and are not redistributed by this repo.

## Recommended Evidence Package for Middleware Vendors

For an external team evaluating compatibility, ask for:

- Their endpoint list and sample payloads.
- A login-to-action trace from their current Zimbra environment.
- Expected response snippets for each call.
- Whether their client is browser-driven, server-to-server, or both.
- Their session, retry, and fault-handling assumptions.
- Their required Zimbra version and classic ZWC asset version.

Then produce:

- A direct HTTP/SOAP smoke script for their required calls.
- A Playwright case only if their behavior depends on the ZWC browser runtime.
- A short gap list: supported, shim-compatible with small fix, Stalwart-direct replacement, or out of scope.
