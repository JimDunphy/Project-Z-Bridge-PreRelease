# Developer Docs

This directory is for **project maintainers** working on the Rust shim and its ZWC compatibility layer.

## Naming Convention

Use area-first filenames so docs are easy to scan:

- `AI_*`
- `API_*`
- `ARCHITECTURE_*`
- `CALENDAR_*`
- `CONTACTS_*`
- `DEV_*`
- `FILTERS_*`
- `IMPORT_EXPORT_*`
- `JMAP_*`
- `MAIL_*`
- `PERFORMANCE_*`
- `PREFERENCES_*`
- `PRODUCT_*`
- `RSS_*`
- `SHARING_*`

## Documentation Rule

If a document is listed here or referenced for further reading, it should be a clickable Markdown link using a path relative to this file.

Start here:

- [`../PROJECT_Z_BRIDGE_ONBOARDING.md`](../PROJECT_Z_BRIDGE_ONBOARDING.md) — high-level bridge explanation for management, product, GUI, and new technical stakeholders.
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — how the shim is put together (request flows, state, trust boundaries).
- [`API_REFERENCE.md`](API_REFERENCE.md) — HTTP routes + SOAP methods implemented/stubbed.
- [`SOAP_COMPATIBILITY_MATRIX.md`](SOAP_COMPATIBILITY_MATRIX.md) — source-audited SOAP method matrix, SOAP XML caveats, batch coverage, and third-party middleware/MUA evaluation notes.
- [`ARCHITECTURE_MODULE_MAP.md`](ARCHITECTURE_MODULE_MAP.md) — where key behaviors live in the codebase.
- [`DEV_DEBUGGING.md`](DEV_DEBUGGING.md) — practical debugging techniques (DevTools, logs, cold/warm cache).
- [`DEV_SCREENSHOT_CAPTURE_GUIDE.md`](DEV_SCREENSHOT_CAPTURE_GUIDE.md) — workflow and expectations for public/admin UI screenshots (full-context captures, hover states, archive-to-repo copy flow, overwrite rules).
- [`DEV_WORKDIR.md`](DEV_WORKDIR.md) — what `.dev/` contains and how to clear caches safely.
- [`DEV_TEST_STRATEGY.md`](DEV_TEST_STRATEGY.md) — unit tests + smoke binaries + recommended E2E checks.
- [`TESTING_AND_MIDDLEWARE_INTEROP.md`](TESTING_AND_MIDDLEWARE_INTEROP.md) — how Rust tests, bridge/API tests, and Playwright tests differ, plus guidance for Zimbra middleware compatibility evaluation.
- [`DEV_CODE_AUDIT.md`](DEV_CODE_AUDIT.md) — known rough edges, duplication hotspots, and refactor backlog.
- [`DEV_PUBLISHING.md`](DEV_PUBLISHING.md) — sanitized exports to public repos (publish.json workflow).
- [`PERFORMANCE_LOGIN.md`](PERFORMANCE_LOGIN.md) — login-time stalls and cache strategy (contacts, mailboxes, GetInfo).
- [`LOGIN_STARTUP_LATENCY.md`](LOGIN_STARTUP_LATENCY.md) — visual diagram of relogin readiness, GetInfo cache serve, and deferred mini-calendar bold-day caching.
- [`PERFORMANCE_CONVERSATIONS.md`](PERFORMANCE_CONVERSATIONS.md) — conversation view slowness (large mailboxes) and `collapseThreads` fast path.
- [`PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md`](PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md) — attachment-column sort incident timeline, root causes, and tuning/debug playbook.
- [`PERFORMANCE_REVIEW.md`](PERFORMANCE_REVIEW.md) — phased performance sprint plan (budgets, P0/P1/P2 execution, measurement).
- [`PERFORMANCE_CACHE_AUDIT.md`](PERFORMANCE_CACHE_AUDIT.md) — implementation audit of current cache behavior, JS/CSS/HTML injection, perceived-speed paths, and unresolved design questions.
- [`PERFORMANCE_CACHE_ALGORITHM_DESKCHECK.md`](PERFORMANCE_CACHE_ALGORITHM_DESKCHECK.md) — current cache algorithm (login/bootstrap, above/below-fold prefetch, push vs NoOp) plus desk-check verification flow.
- [`PERFORMANCE_CACHE_TRACE_MATRIX.md`](PERFORMANCE_CACHE_TRACE_MATRIX.md) — scenario matrix (expected cache source + required logs + fail signatures) for repeatable perf triage.
- [`JMAP_USAGE_AUDIT_AND_PLAN.md`](JMAP_USAGE_AUDIT_AND_PLAN.md) — JMAP feature-usage audit (`changes`/`queryChanges`/state tokens) and phased optimization plan.
- [`ARCHITECTURE_REFACTOR_PLAN.md`](ARCHITECTURE_REFACTOR_PLAN.md) — phased bridge->view-API->modern-UI migration plan with compatibility gates.
- [`ARCHITECTURE_GOLDEN_BEHAVIOR_MATRIX.md`](ARCHITECTURE_GOLDEN_BEHAVIOR_MATRIX.md) — compatibility checklist used as rollout gate for migration phases.
- [`ARCHITECTURE_PHASE0_EXECUTION_CHECKLIST.md`](ARCHITECTURE_PHASE0_EXECUTION_CHECKLIST.md) — step-by-step Phase 0 no-regression execution and evidence capture.
- [`MAIL_RENDERER_CONVERSATION_DEBUGGING.md`](MAIL_RENDERER_CONVERSATION_DEBUGGING.md) — renderer-origin inline image, conversation-row sender, and fallback-iframe nested-scroll regression playbook (`Nick,Steve`, quoted inline images, deferred-media read-pane traps, nginx renderer resource path).
- [`MAIL_THREADING_SEMANTICS.md`](MAIL_THREADING_SEMANTICS.md) — conversation grouping rules when backed by JMAP `threadId`, including the Cloudflare same-subject alert example and why subject-only grouping is intentionally not emulated.
- [`CALENDAR_IMPORT_VIEW_DEBUGGING.md`](CALENDAR_IMPORT_VIEW_DEBUGGING.md) — classic ZWC REST `.ics` import postmortem: folder tree reload, checked-calendar state, Month-view blank-then-repaint behavior, and duplicate-overlay debugging/fix path (`Dunphy Family Events`).

Feature areas / deep dives:

- Extensions
  - [`EXTENSIONS_PLAN.md`](EXTENSIONS_PLAN.md)
  - [`AI_AUTOMATION_EXTENSION_PLAN.md`](AI_AUTOMATION_EXTENSION_PLAN.md)
  - [`AI_ORGANIZER_ALGORITHM.md`](AI_ORGANIZER_ALGORITHM.md)
- Calendar
  - [`CALENDAR_INVITES.md`](CALENDAR_INVITES.md)
  - [`CALENDAR_O365_INVITES_RSVP_DEBUG.md`](CALENDAR_O365_INVITES_RSVP_DEBUG.md)
  - [`CALENDAR_ALARMS.md`](CALENDAR_ALARMS.md)
  - [`CALENDAR_ALARMS_POSTMORTEM.md`](CALENDAR_ALARMS_POSTMORTEM.md)
  - [`CALENDAR_SCHEDULER.md`](CALENDAR_SCHEDULER.md)
  - [`CALENDAR_JMAP_FIX.md`](CALENDAR_JMAP_FIX.md)
- Contacts
  - [`CONTACTS_FIELD_MAPPING.md`](CONTACTS_FIELD_MAPPING.md)
  - [`CONTACTS_SEARCH.md`](CONTACTS_SEARCH.md)
  - [`CONTACTS_SHARING.md`](CONTACTS_SHARING.md)
  - [`CONTACTS_ZWC_ID_COLLISION.md`](CONTACTS_ZWC_ID_COLLISION.md)
  - [`CONTACTS_GROUPS.md`](CONTACTS_GROUPS.md)
- Filters (Sieve)
  - [`FILTERS_PLAN.md`](FILTERS_PLAN.md)
  - [`FILTERS_SIEVE_BACKUPS.md`](FILTERS_SIEVE_BACKUPS.md)
  - [`FILTERS_SOCIAL_PLAN.md`](FILTERS_SOCIAL_PLAN.md)
- Sharing
  - [`SHARING_SHARED_FOLDERS.md`](SHARING_SHARED_FOLDERS.md)
- Preferences
  - [`PREFERENCES_PLAN.md`](PREFERENCES_PLAN.md)
  - [`PREFERENCES_SPAM_EXTLISTS_PLAN.md`](PREFERENCES_SPAM_EXTLISTS_PLAN.md) (future migration of Spam Mail Options from inline Sieve rules to external lists / `extlists`)
  - [`PREFERENCES_OUT_OF_OFFICE_PLAN.md`](PREFERENCES_OUT_OF_OFFICE_PLAN.md) (vacation/OOO, including bridge per-recipient alias reply extension)
  - [`PREFERENCES_ACCOUNT_AUTH_PLAN.md`](PREFERENCES_ACCOUNT_AUTH_PLAN.md)
- Security
  - [`SECURITY_IMPLEMENTATION_PHASES.md`](../security/SECURITY_IMPLEMENTATION_PHASES.md) — phased plan (stability -> edge hardening -> shim backstops).
  - [`SECURITY_REVIEW.md`](../security/SECURITY_REVIEW.md)
  - [`SECURITY_AUDIT_PLAN.md`](../security/SECURITY_AUDIT_PLAN.md)
  - [`SECURITY_RED_TEAM_PLAN.md`](../security/SECURITY_RED_TEAM_PLAN.md)
  - [`SECURITY_SHIM_AUDIT.md`](../security/SECURITY_SHIM_AUDIT.md)
  - [`SECURITY_STATIC_ZWC_AUDIT.md`](../security/SECURITY_STATIC_ZWC_AUDIT.md)
  - [`SECURITY_STATIC_ASSET_AUDIT_PLAN.md`](../security/SECURITY_STATIC_ASSET_AUDIT_PLAN.md)
  - [`SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md`](../security/SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md)
  - [`SECURITY_IMAGE_PROXY_TESTING.md`](../security/SECURITY_IMAGE_PROXY_TESTING.md)
- Architecture / state
  - [`ARCHITECTURE_STATELESSNESS_PLAN.md`](ARCHITECTURE_STATELESSNESS_PLAN.md)
  - [`ARCHITECTURE_CONTEXT_PATH_PLAN.md`](ARCHITECTURE_CONTEXT_PATH_PLAN.md) (future: configurable `/zimbra/` mount point)
- Misc
  - [`PRODUCT_ENHANCEMENTS.md`](PRODUCT_ENHANCEMENTS.md)
  - [`PRODUCT_FUTURE_COMPLETION_PLAN.md`](PRODUCT_FUTURE_COMPLETION_PLAN.md)
  - [`API_STUBS.md`](API_STUBS.md)
  - [`ARCHITECTURE_DESIGN_HISTORICAL.md`](ARCHITECTURE_DESIGN_HISTORICAL.md) (early technical design; historical)
  - [`DEV_CODEX_CONTEXT.md`](DEV_CODEX_CONTEXT.md) (agent/context snapshot; historical)
