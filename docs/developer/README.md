# Developer Docs

Last reviewed: 2026-06-06.

This directory is for **project maintainers** working on the Rust shim and its ZWC compatibility layer.

## Naming Convention

Use area-first filenames so docs are easy to scan:

- `AI_*`
- `API_*`
- `ARCHITECTURE_*`
- `BRIEFCASE_*`
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
- `STALWART_*`
- `TASKS_*`

## Documentation Rule

If a document is listed here or referenced for further reading, it should be a clickable Markdown link using a path relative to this file.

Start here:

- [`../PROJECT_Z_BRIDGE_ONBOARDING.md`](../PROJECT_Z_BRIDGE_ONBOARDING.md) — high-level bridge explanation for management, product, GUI, and new technical stakeholders.
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — how the shim is put together (request flows, state, trust boundaries).
- [`ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md`](ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md) — how classic ZWC uses Zimbra SOAP, how the bridge intercepts those calls, and what happens from a mail-row click to `GetMsgResponse`.
- [`API_REFERENCE.md`](API_REFERENCE.md) — HTTP routes + SOAP methods implemented/stubbed.
- [`SOAP_COMPATIBILITY_MATRIX.md`](SOAP_COMPATIBILITY_MATRIX.md) — source-audited SOAP method matrix, SOAP XML caveats, batch coverage, and third-party middleware/MUA evaluation notes.
- [`PRODUCT_ZIMBRA_COMPATIBLE_APPLICATION_LAYER.md`](PRODUCT_ZIMBRA_COMPATIBLE_APPLICATION_LAYER.md) — roadmap for evolving from ZWC bridge to a tested Zimbra-shaped SOAP/REST application layer for Stalwart.
- [`ARCHITECTURE_MODULE_MAP.md`](ARCHITECTURE_MODULE_MAP.md) — where key behaviors live in the codebase.
- [`DEV_DEBUGGING.md`](DEV_DEBUGGING.md) — practical debugging techniques (DevTools, logs, cold/warm cache).
- [`DEV_RUNTIME_EVALUATION_PACKAGING.md`](DEV_RUNTIME_EVALUATION_PACKAGING.md) — maintainer workflow for the runtime-only partner evaluation package, including current manual steps and future automation boundary.
- [`DEV_SCREENSHOT_CAPTURE_GUIDE.md`](DEV_SCREENSHOT_CAPTURE_GUIDE.md) — workflow and expectations for public/admin UI screenshots (full-context captures, hover states, archive-to-repo copy flow, overwrite rules).
- [`DEV_WORKDIR.md`](DEV_WORKDIR.md) — what `.dev/` contains and how to clear caches safely.
- [`DEV_TEST_STRATEGY.md`](DEV_TEST_STRATEGY.md) — unit tests + smoke binaries + recommended E2E checks.
- [`DEV_DEPENDENCY_MAINTENANCE.md`](DEV_DEPENDENCY_MAINTENANCE.md) — pinned Rust/toolchain policy, crate refresh procedure, `cargo audit`, and verification gate.
- [`TESTING_AND_MIDDLEWARE_INTEROP.md`](TESTING_AND_MIDDLEWARE_INTEROP.md) — how Rust tests, bridge/API tests, and Playwright tests differ, plus guidance for Zimbra middleware compatibility evaluation.
- [`COMPAT_TRACE.md`](COMPAT_TRACE.md) — optional sanitized SOAP/REST compatibility trace, human report, privacy boundary, and partner-evaluation workflow.
- [`DEV_CODE_AUDIT.md`](DEV_CODE_AUDIT.md) — known rough edges, duplication hotspots, and refactor backlog.
- [`DEV_PUBLISHING.md`](DEV_PUBLISHING.md) — sanitized exports to public repos (publish.json workflow).
- [`PERFORMANCE_LOGIN.md`](PERFORMANCE_LOGIN.md) — current login-time cache strategy (contacts, mailboxes, GetInfo, MiniCal, appointment search) plus regression memory.
- [`LOGIN_STARTUP_LATENCY.md`](LOGIN_STARTUP_LATENCY.md) — measured debugging snapshot of relogin readiness, cached GetInfo serve, and deferred mini-calendar bold-day caching.
- [`PERFORMANCE_CONVERSATIONS.md`](PERFORMANCE_CONVERSATIONS.md) — current conversation-view performance behavior for large mailboxes and the guarded `collapseThreads` fast path.
- [`PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md`](PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md) — attachment-column sort postmortem/debugging memory.
- [`PERFORMANCE_REVIEW.md`](PERFORMANCE_REVIEW.md) — Feb 2026 performance sprint plan and historical execution record; use the cache audit/desk-check/trace docs for current behavior.
- [`PERFORMANCE_CACHE_AUDIT.md`](PERFORMANCE_CACHE_AUDIT.md) — current implementation audit of cache behavior, JS/CSS/HTML injection, perceived-speed paths, and unresolved design questions.
- [`PERFORMANCE_CACHE_ALGORITHM_DESKCHECK.md`](PERFORMANCE_CACHE_ALGORITHM_DESKCHECK.md) — current cache algorithm (login/bootstrap, above/below-fold prefetch, push vs NoOp) plus desk-check verification flow.
- [`PERFORMANCE_CACHE_TRACE_MATRIX.md`](PERFORMANCE_CACHE_TRACE_MATRIX.md) — current scenario matrix (expected cache source + required logs + fail signatures) for repeatable perf triage.
- [`JMAP_USAGE_AUDIT_AND_PLAN.md`](JMAP_USAGE_AUDIT_AND_PLAN.md) — JMAP feature-usage audit (`changes`/`queryChanges`/state tokens) and phased optimization plan.
- [`JMAP_PUSH_AND_NOOP.md`](JMAP_PUSH_AND_NOOP.md) — how classic ZWC `NoOpRequest` browser notifications differ from bridge-side JMAP Push cache warming, including log signatures and latency testing.
- [`STALWART_0_14_TO_0_16_UPGRADE_READINESS.md`](STALWART_0_14_TO_0_16_UPGRADE_READINESS.md) — bridge-facing Stalwart 0.14.1 to 0.16.4 differences, compatibility rules, smoke matrix, and cutover gates.
- [`ARCHITECTURE_BRIDGE_USER_METADATA_STORE.md`](ARCHITECTURE_BRIDGE_USER_METADATA_STORE.md) — design note for durable per-user bridge metadata under runtime `./data`, shared-filesystem deployments, and future transparent Stalwart-backed metadata.
- [`ARCHITECTURE_REFACTOR_PLAN.md`](ARCHITECTURE_REFACTOR_PLAN.md) — future bridge->view-API->modern-UI migration plan with compatibility gates.
- [`ARCHITECTURE_GOLDEN_BEHAVIOR_MATRIX.md`](ARCHITECTURE_GOLDEN_BEHAVIOR_MATRIX.md) — future migration guardrail; capture fresh dated evidence before using it as a rollout gate.
- [`ARCHITECTURE_PHASE0_EXECUTION_CHECKLIST.md`](ARCHITECTURE_PHASE0_EXECUTION_CHECKLIST.md) — future Phase 0 no-regression execution and evidence-capture checklist.
- [`MAIL_RENDERER_CONVERSATION_DEBUGGING.md`](MAIL_RENDERER_CONVERSATION_DEBUGGING.md) — renderer-origin inline image, conversation-row sender, and fallback-iframe nested-scroll regression playbook (`Nick,Steve`, quoted inline images, deferred-media read-pane traps, nginx renderer resource path).
- [`MAIL_THREADING_SEMANTICS.md`](MAIL_THREADING_SEMANTICS.md) — conversation grouping rules when backed by JMAP `threadId`, including the Cloudflare same-subject alert example and why subject-only grouping is intentionally not emulated.
- [`CALENDAR_IMPORT_VIEW_DEBUGGING.md`](CALENDAR_IMPORT_VIEW_DEBUGGING.md) — classic ZWC REST `.ics` import postmortem: folder tree reload, checked-calendar state, Month-view blank-then-repaint behavior, and duplicate-overlay debugging/fix path (`Example Family Events`).

Debugging memory / postmortems:

These docs are intentionally kept to prevent repeated debugging mistakes. Treat
them as incident memory unless the document says it is a current reference.

- [`CALENDAR_IMPORT_VIEW_DEBUGGING.md`](CALENDAR_IMPORT_VIEW_DEBUGGING.md)
- [`CALENDAR_O365_INVITES_RSVP_DEBUG.md`](CALENDAR_O365_INVITES_RSVP_DEBUG.md)
- [`CALENDAR_ALARMS_POSTMORTEM.md`](CALENDAR_ALARMS_POSTMORTEM.md)
- [`FLAG_ACTION_LATENCY.md`](FLAG_ACTION_LATENCY.md)
- [`API_STUBS.md`](API_STUBS.md)
- [`LOGIN_STARTUP_LATENCY.md`](LOGIN_STARTUP_LATENCY.md)
- [`MAIL_RENDERER_CONVERSATION_DEBUGGING.md`](MAIL_RENDERER_CONVERSATION_DEBUGGING.md)
- [`PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md`](PERFORMANCE_ATTACHMENT_SORT_DEBUGGING.md)
- [`RSS_FEEDS_POSTMORTEM.md`](RSS_FEEDS_POSTMORTEM.md)

Feature areas / deep dives:

- Extensions
  - [`EXTENSIONS_PLAN.md`](EXTENSIONS_PLAN.md)
  - [`AI_AUTOMATION_EXTENSION_PLAN.md`](AI_AUTOMATION_EXTENSION_PLAN.md)
  - [`AI_COMPOSE_WORKFLOW.md`](AI_COMPOSE_WORKFLOW.md)
  - [`AI_ORGANIZER_ALGORITHM.md`](AI_ORGANIZER_ALGORITHM.md)
  - [`AI_RUNNER_PERSISTENT_SESSIONS_DESIGN.md`](AI_RUNNER_PERSISTENT_SESSIONS_DESIGN.md)
- Calendar
  - [`CALENDAR_INVITES.md`](CALENDAR_INVITES.md) (current RSVP/invite reference)
  - [`CALENDAR_O365_INVITES_RSVP_DEBUG.md`](CALENDAR_O365_INVITES_RSVP_DEBUG.md) (postmortem/debugging memory)
  - [`CALENDAR_ALARMS.md`](CALENDAR_ALARMS.md) (current alarms reference)
  - [`CALENDAR_ALARMS_POSTMORTEM.md`](CALENDAR_ALARMS_POSTMORTEM.md) (postmortem/debugging memory)
  - [`CALENDAR_SCHEDULER.md`](CALENDAR_SCHEDULER.md) (current free/busy and working-hours reference)
  - [`../archive/CALENDAR_JMAP_FIX.md`](../archive/CALENDAR_JMAP_FIX.md) (generic LLM guardrail; archived)
- Contacts
  - [`CONTACTS_FIELD_MAPPING.md`](CONTACTS_FIELD_MAPPING.md) (current field-mapping reference)
  - [`CONTACTS_SEARCH.md`](CONTACTS_SEARCH.md) (current search parsing reference)
  - [`CONTACTS_SHARING.md`](CONTACTS_SHARING.md) (current shared address-book reference)
  - [`CONTACTS_ZWC_ID_COLLISION.md`](CONTACTS_ZWC_ID_COLLISION.md) (debugging memory/current id guardrail)
  - [`CONTACTS_GROUPS.md`](CONTACTS_GROUPS.md) (current group-contact reference)
- Filters (Sieve)
  - [`FILTERS_PLAN.md`](FILTERS_PLAN.md) (hybrid current/future filter and Sieve reference)
  - [`FILTERS_SIEVE_BACKUPS.md`](FILTERS_SIEVE_BACKUPS.md) (current Sieve backup/restore runbook)
  - [`FILTERS_SOCIAL_PLAN.md`](FILTERS_SOCIAL_PLAN.md) (future Social filter design; blocked on real Zimbra evidence)
- Sharing
  - [`SHARING_SHARED_FOLDERS.md`](SHARING_SHARED_FOLDERS.md) (current internal shared mail-folder reference)
- Import / Export
  - [`IMPORT_EXPORT_FEATURE_PLAN.md`](IMPORT_EXPORT_FEATURE_PLAN.md) (current feature reference with future gaps)
- RSS
  - [`RSS_FEEDS.md`](RSS_FEEDS.md) (current RSS/Atom feed-folder reference)
  - [`RSS_FEEDS_POSTMORTEM.md`](RSS_FEEDS_POSTMORTEM.md) (postmortem/debugging memory)
- Optional large surfaces
  - [`TASKS_DESIGN.md`](TASKS_DESIGN.md) (future ZWC Tasks design and Stalwart/JMAP foundation check)
  - [`BRIEFCASE_DESIGN.md`](BRIEFCASE_DESIGN.md) (future ZWC Briefcase design and Stalwart file-storage foundation check)
- Preferences
  - [`PREFERENCES_PLAN.md`](PREFERENCES_PLAN.md)
  - [`PREFERENCES_SPAM_EXTLISTS_PLAN.md`](PREFERENCES_SPAM_EXTLISTS_PLAN.md) (future migration of Spam Mail Options from inline Sieve rules to external lists / `extlists`)
  - [`PREFERENCES_OUT_OF_OFFICE_PLAN.md`](PREFERENCES_OUT_OF_OFFICE_PLAN.md) (vacation/OOO, including bridge per-recipient alias reply extension)
  - [`PREFERENCES_ACCOUNT_AUTH_PLAN.md`](PREFERENCES_ACCOUNT_AUTH_PLAN.md)
- Security
  - [`SECURITY_IMPLEMENTATION_PHASES.md`](../security/SECURITY_IMPLEMENTATION_PHASES.md) — developer/security implementation worklist for code, tests, and audit follow-through.
  - [`SECURITY_REVIEW.md`](../security/SECURITY_REVIEW.md)
  - [`SECURITY_AUDIT_PLAN.md`](../security/SECURITY_AUDIT_PLAN.md)
  - [`SECURITY_RED_TEAM_PLAN.md`](../security/SECURITY_RED_TEAM_PLAN.md)
  - [`SECURITY_SHIM_AUDIT.md`](../security/SECURITY_SHIM_AUDIT.md)
  - [`SECURITY_STATIC_ZWC_AUDIT.md`](../security/SECURITY_STATIC_ZWC_AUDIT.md)
  - [`SECURITY_STATIC_ASSET_AUDIT_PLAN.md`](../security/SECURITY_STATIC_ASSET_AUDIT_PLAN.md)
  - [`SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md`](../security/SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md)
  - [`SECURITY_IMAGE_PROXY_TESTING.md`](../security/SECURITY_IMAGE_PROXY_TESTING.md)
- Architecture / state
  - [`ARCHITECTURE_BRIDGE_USER_METADATA_STORE.md`](ARCHITECTURE_BRIDGE_USER_METADATA_STORE.md)
  - [`ARCHITECTURE_STATELESSNESS_PLAN.md`](ARCHITECTURE_STATELESSNESS_PLAN.md)
  - [`ARCHITECTURE_CONTEXT_PATH_PLAN.md`](ARCHITECTURE_CONTEXT_PATH_PLAN.md) (future: configurable `/zimbra/` mount point)
  - [`ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md`](ARCHITECTURE_ZWC_SOAP_LIFECYCLE.md)
- Misc
  - [`PRODUCT_ENHANCEMENTS.md`](PRODUCT_ENHANCEMENTS.md) (current maintainer inventory of bridge-only enhancements)
  - [`PRODUCT_FUTURE_COMPLETION_PLAN.md`](PRODUCT_FUTURE_COMPLETION_PLAN.md) (future roadmap / planning note)
  - [`PRODUCT_ZIMBRA_COMPATIBLE_APPLICATION_LAYER.md`](PRODUCT_ZIMBRA_COMPATIBLE_APPLICATION_LAYER.md) (roadmap/product positioning note, not a full-compatibility claim)
  - [`PRODUCT_VNCMAIL_COMPATIBILITY_OPTIONS.md`](PRODUCT_VNCMAIL_COMPATIBILITY_OPTIONS.md) (VNCmail/VNClagoon investigation and architecture options)
  - [`../archive/ARCHITECTURE_DESIGN_HISTORICAL.md`](../archive/ARCHITECTURE_DESIGN_HISTORICAL.md) (early technical design; historical)
  - [`../archive/DEV_CODEX_CONTEXT.md`](../archive/DEV_CODEX_CONTEXT.md) (agent/context snapshot; historical)
