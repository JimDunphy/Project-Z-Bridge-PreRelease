# Project Z-Bridge: Executive Summary

**Strategic Infrastructure Modernization for the Zimbra Ecosystem**

Updated: 2026-04-07

Project Z-Bridge modernizes legacy email infrastructure **without disrupting end-user workflow**. It is a **Backend-for-Frontend (BFF)** compatibility layer that allows the classic **Zimbra Web Client (ZWC)** interface to run on top of a **Stalwart Mail Server** backend (JMAP/IMAP/SMTP), by translating ZWC’s `/service/soap` calls into standards-based operations.

This architecture decouples the user interface from the legacy backend, allowing organizations to replace resource-intensive Java/Jetty services with a secure, Rust-based JMAP translation engine.

## Operational Impact

*   **Familiar Experience:** Minimizes user retraining by preserving the familiar interface.
*   **Infrastructure Simplification:** Replaces the Zimbra server-side runtime with Stalwart while preserving the ZWC user experience.
*   **Protocol Modernization:** Translates legacy SOAP (JSON or XML) and ZWC REST flows to modern JMAP-backed operations, reducing round trips via batching and targeted bridge-side caches where possible.
*   **Extensible Platform:** The bridge is evolving into an extensible application surface, with bridge-managed extensions enabling new capabilities without forking the legacy client.
*   **Governed AI Integration:** The extension model already supports policy-controlled AI workflows, giving administrators and users fine-grained control over how AI-powered automation is introduced.
*   **Production-Oriented Compatibility:** The bridge now delivers a broad, credible feature set across mail, calendar, contacts, sharing, import/export, filtering, retention controls, and common user actions, with validation increasingly centered on live daily-use mailboxes rather than synthetic demos.

## Executive Highlights (Since 2026-03-27)

*   **Product Readiness:** The bridge continues to move from technical compatibility layer to deployable product, with more of the classic Zimbra experience now operating cohesively on a modern backend.
*   **Feature Breadth:** Mail, calendar, contacts, sharing, filtering, import/export, and retention controls now form a denser end-to-end workflow story for pilot, migration, and day-to-day use scenarios.
*   **Extensibility:** The product now has a credible extension story, allowing value-added capabilities to be layered onto the familiar client experience; the AI Organizer is the first concrete example of that model.
*   **Governed AI Workflows:** The AI Organizer is backed by a policy editor and provider controls, enabling fine-grained oversight of what AI tools are allowed to do while preserving a review-first rollout model.
*   **Quality and Rollout Confidence:** Expanded automated regression coverage and clearer operator controls reduce delivery risk, improve release confidence, and support more disciplined staging, pilot programs, and production-style evaluation.
*   **Scalability Trajectory:** Continued performance tuning is improving responsiveness for larger and more complex mailboxes while preserving the familiar end-user experience.

## Current Product Coverage

Project Z-Bridge is no longer just a login-and-read prototype. Current coverage is centered on the workflows that matter most in real deployments:

*   **Mail:** robust day-to-day workflow coverage across reading, search, conversation handling, attachments, image controls, document viewing, and common mailbox actions.
*   **Calendars:** month/day/list views, reminders, create/edit/delete flows, ICS import/export, calendar shares, and calendar-folder management.
*   **Contacts:** contact listing, import/export, create/edit/delete, and compose autocomplete.
*   **Extensions / AI:** a bridge-managed extension model is in place, with per-user enablement and the AI Organizer shipped as the first extension path for future premium and automation features. It includes a policy editor for fine-grained control over AI-assisted actions and supports multiple CLI-based providers, including Codex, Claude, and Gemini.
*   **Migration / Restore:** `smmailbox` account migration plus bridge-side contacts/calendar/mail import/export paths for staged cutovers and repair workflows.
*   **Operational Hardening:** broader browser regression coverage now strengthens release confidence as feature breadth increases, helping the product mature without sacrificing stability.

## Security Architecture

The platform is engineered with a "Secure by Design" philosophy, leveraging the memory safety of Rust and proactive content neutralization.

### 1. Active Threat Mitigation
*   **Content Sanitization:** All email content is processed server-side through a strict HTML sanitization engine (`ammonia`) to neutralize Cross-Site Scripting (XSS) vectors before they reach the client.
*   **Image Proxying:** External resources are routed through a secure, same-origin image proxy (`/service/image-proxy`). This prevents third-party tracking pixels from leaking client IP addresses and enforces strict content-type validation.
*   **Sandboxed Mail Rendering:** Mail body rendering now uses iframe-based containment as a first-class part of the design, not just a future option. The bridge supports a dedicated renderer origin with short-lived tokens and a strict enforced CSP for the rendering pane, while preserving a compatibility fallback for deferred external-media messages so ZWC’s native “Display Images” workflow still works.
*   **Layered CSP Strategy:** The classic ZWC app shell still requires a **best-effort** compatibility CSP because legacy assets depend on inline script and eval-era behavior. In practice, strict CSP is enforced on the isolated renderer view, while the main app shell is typically deployed with report-only/logging first and tightened carefully behind a reverse proxy.
*   **Regression Validation:** Mail-rendering security changes are now backed by committed browser regressions for deferred-media scrolling, quoted inline CID images, external-image opt-in, and compact conversation-sender rendering. This reduces the chance of silently reintroducing read-path regressions while hardening the client.

### 2. Hybrid State Model
The system is designed so primary user data lives upstream, with minimal bridge-local state:
*   **Core Data:** Email, calendars, contacts, and delivery-time filtering are stored and executed on Stalwart.
*   **UI Preferences / Metadata:** Some UI-only preferences and metadata are currently persisted via the shim’s storage abstraction (for example: theme/skin, some appearance defaults, and some UI metadata). For horizontal scaling, this should be backed by a shared store (Redis/SQL/shared volume) or moved upstream where feasible.

### 3. Secure Authentication
Integrates directly with Stalwart authentication (JMAP session establishment), and uses a session cookie (`ZM_AUTH_TOKEN`) with secure attributes (`HttpOnly`, `SameSite=Lax`). When deployed behind HTTPS, the cookie should be marked `Secure`.

## Migration Capability

The platform includes **`smmailbox`**, a specialized CLI utility designed for enterprise migration. This tool orchestrates the transition from legacy Zimbra environments to the new architecture by:
*   Synchronizing mail data via high-performance IMAP transfer.
*   Cloning key metadata including **Contacts, Calendars, and Filters**.
*   Preserving organizational taxonomy (**Folders, tags and tag colors**), including mapping Zimbra tags to IMAP keywords.
*   Supporting iterative migration and repair workflows, where bridge-side import/export can be used to validate or correct targeted contacts/calendar/mail datasets without re-running a full account migration.

## Technical Specifications

*   **Runtime:** Rust (Axum, Tokio)
*   **Protocol Standard:** JMAP (RFC 8620)
*   **Deployment:** Containerized (Docker/Kubernetes)
*   **Compatibility:** Verified against Zimbra 10.x classic web client assets. Other classic asset versions may work but should be validated in staging.

## Performance & Scalability

Project Z-Bridge is engineered for high-concurrency enterprise environments.

*   **JMAP Efficiency:** JMAP’s batching and structured data model can reduce round trips and bandwidth versus legacy SOAP patterns, especially on high-latency links.
*   **Rust Runtime:** Built on Tokio’s async runtime for efficient concurrency and predictable memory behavior versus GC-managed server runtimes.
*   **Large Mailbox Readiness:** Startup paths are explicitly optimized for “power user” accounts (e.g., **50K+ contacts** and **15K+ folders**) using best-effort shim caches for expensive startup data (contact exports and `GetInfoRequest` folder/tag/prefs discovery). If a cache is missing or stale, the bridge rebuilds from upstream and continues normally.
*   **Read-Path Caching:** Hot mail search, message-open, and calendar-navigation paths now rely on targeted read-side caches rather than repeated full upstream fetches. Recent tuning has further improved shared-folder startup and conversation browsing, reinforcing the product's production-readiness at larger mailbox scales.
*   **Horizontal Scaling:** The target deployment model is multiple bridge replicas behind standard load balancers (nginx/HAProxy/ALB), with no “special” knowledge required by end users.
    * The bridge treats any local state as a **cache** behind a storage abstraction (so it can be moved to a shared backend).
    * Session handling is intended to be portable across replicas (shared store or stateless token scheme). As an interim deployment option, sticky sessions can be used.

## Licensing & Lifecycle

**Open Source Foundation:**
Project Z-Bridge is released under the **MIT License**, ensuring broad compatibility with enterprise requirements and no vendor lock-in.

**Lifecycle Management:**
The platform adopts a "Bring Your Own Client" model to respect intellectual property.
*   **Installation:** Project Z-Bridge does not redistribute classic ZWC assets, preserving the legal and licensing boundary around the client. Operationally, however, the process is becoming much simpler: `manage.sh` can automate the upstream build-and-extract workflow so administrators do not need deep knowledge of the legacy Zimbra webclient build chain to provision the required static assets.
    * In practice, the bridge can drive Docker-based WAR generation and extraction workflows from upstream open-source build inputs, producing a local `zimbra.war` and unpacking the client assets into `static/zimbra/` for use by the shim. To further streamline deployment, those build outputs can also be stored in a separate build repository outside Project Z-Bridge itself, allowing faster provisioning without changing the project's redistribution posture.
*   **Upgrading:** Front-end lifecycle management is increasingly treated as a managed refresh of upstream client assets rather than a bespoke manual packaging exercise. This preserves the Bring Your Own Client model while materially reducing operator friction and making version refreshes easier to standardize across environments.
This decouples the frontend update cycle from the backend, allowing for faster client refreshes and easier provisioning while keeping redistribution boundaries for Project Z-Bridge intact.

## Strategic Value

For organizations committed to the Zimbra user experience but constrained by legacy backend limitations, Z-Bridge offers a practical modernization path. It delivers the security, performance, and maintainability of next-generation infrastructure while maintaining workflow continuity for the end user.

## Recommended Evaluation Approach (Executive-Friendly)

1. **Pilot with a small cohort:** Select a subset of accounts and run the bridge behind your existing TLS reverse proxy pattern (nginx) with observability enabled.
2. **Prove interoperability:** Validate that mail, calendars, contacts, shares, and filtering interoperate across the bridge and any secondary clients you expect to support (JMAP/CalDAV/CardDAV where applicable).
3. **Stage migration:** Use `smmailbox` to clone accounts ahead of cutover. Plan a short final sync window before switching delivery (MX/forwarding), and keep bridge import/export available for targeted repair or validation during the transition.
4. **Security review gate:** Treat classic ZWC as a high-risk webmail client and require: image proxy enabled, iframe/render isolation enabled for the intended cohort, CSP reporting enabled before enforcement changes, rate limiting/fail2ban, and HTTPS-only cookie hardening.

## Notes / Non‑Goals

- Project Z‑Bridge is **not** a Zimbra server and does not include Zimbra's server-side runtime, admin console, or zimlet infrastructure.
- Some ZWC features (Tasks, Briefcase, full classic zimlet parity) are not yet implemented; current focus is core mail, calendar, contacts, sharing, migration, operational hardening, and bridge-managed extensions.
- Zimbra trademarks and classic web client asset licensing should be reviewed by your legal/compliance team; this project intentionally avoids redistributing ZWC assets, which administrators can obtain separately from existing Zimbra builds or public open-source build workflows.
