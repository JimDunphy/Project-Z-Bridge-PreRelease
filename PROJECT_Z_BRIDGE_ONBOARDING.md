# Project Z-Bridge Onboarding Overview

This document is a high-level orientation for management, product, GUI, and new
technical stakeholders. It explains what Project Z-Bridge is, why the browser
still looks like classic Zimbra Web Client, how runtime injection works, and how
the bridge is designed to feel faster than ZWC backed by Zimbra mailboxd.

## One Sentence Version

Project Z-Bridge lets the classic Zimbra Web Client run against Stalwart by
translating Zimbra-style web requests into Stalwart/JMAP operations while
preserving the familiar ZWC user experience.

## Executive Summary

Project Z-Bridge is a compatibility bridge between the classic Zimbra Web Client
and Stalwart. The browser loads the familiar ZWC interface, but the backend is
not Zimbra mailboxd. Instead, the bridge accepts the Zimbra-style requests that
ZWC already knows how to send and maps them to Stalwart services, primarily
through JMAP.

The goal is not to fork ZWC, rewrite the UI, or clone every internal mailboxd
behavior. The goal is to preserve the working ZWC user experience while moving
mail, calendar, contacts, filtering, and related state onto a faster and simpler
Stalwart-backed architecture.

## Plain-English View

The simplest way to describe the system is:

```text
Browser running classic ZWC
        |
        v
Project Z-Bridge
        |
        +-- fast bridge-side cache
        |
        v
Stalwart through JMAP and related APIs
```

In plain terms, the user still opens the familiar Zimbra-style web client. The
bridge sits behind that web client and makes Stalwart look enough like Zimbra
mailboxd for the UI to keep working.

## Plain-English FAQ

### What changes for users?

Most users still see classic ZWC. The goal is that ordinary actions such as
reading mail, moving messages, marking spam, using calendar views, and changing
preferences feel familiar.

### What changes behind the scenes?

Zimbra mailboxd is no longer the backend serving the UI. Project Z-Bridge handles
the ZWC-compatible web requests and talks to Stalwart for the real mailbox data.

### Why should this be faster?

Stalwart is fast at the mail-server layer, and JMAP gives the bridge a cleaner
way to group related mailbox reads instead of repeatedly rebuilding the same
state. If the bridge already has a safe cached view of a folder, message,
preference, or calendar projection, it can answer the UI quickly and refresh in
the background.

The improvement is most visible in common UI paths such as folder switching,
reading messages, deleting or moving messages, and spam/ham actions. Once those
paths are decoupled from repeated mailboxd-style rebuilds and avoidable upstream
round trips, classic ZWC can feel substantially more responsive during normal
use.

### What does injection mean?

Injection means the bridge adds small runtime adjustments while serving the web
client. It does not permanently edit the stock ZWC files. The browser receives
the adjusted response, but the original extracted asset stays unchanged on disk.

### Why not modify ZWC directly?

Directly modifying ZWC would make upgrades, audits, and rollback harder. Keeping
the stock assets clean lets the project replace the ZWC artifact more easily and
keep Project Z-Bridge behavior in one clear compatibility layer.

### How real-time is it?

The goal is near real-time mail behavior, not a full upstream reload on every
click. JMAP Push can tell the bridge that mailbox state changed, and the bridge
can warm or refresh affected cached views. Polling and ZWC NoOp behavior remain
as fallback paths.

### What is the business value?

The project preserves the familiar ZWC interface while removing the heavy
mailboxd dependency from the active path. That creates a practical migration
route: keep users productive today, use Stalwart for the mail backend, and build
toward a cleaner bridge/API layer for future UI or middleware work.

### What are the main risks?

The main risks are compatibility gaps, cache correctness bugs, and unsupported
Zimbra SOAP/XML features. The project manages those risks by keeping coverage
docs current, adding regression tests for repaired behavior, and avoiding hidden
"success" responses for features that are not actually implemented.

## What The Bridge Is

Project Z-Bridge is a backend-for-frontend for classic ZWC.

It does these jobs:

- Serves the extracted ZWC static application files to the browser.
- Provides Zimbra-compatible SOAP and REST endpoints expected by ZWC.
- Translates implemented ZWC operations into Stalwart/JMAP operations.
- Shapes Stalwart data back into Zimbra-style responses.
- Adds small runtime compatibility and enhancement patches where stock ZWC has
  assumptions that do not match Stalwart.
- Maintains bridge-side caches so common read paths can return quickly without
  repeatedly waiting on upstream mailbox reads.

It is not these things:

- It is not a modified copy of ZWC committed into this project.
- It is not a replacement mail server.
- It is not a full mailboxd reimplementation.
- It is not intended to make every unsupported Zimbra SOAP feature appear
  complete before the feature is actually mapped and tested.

## Request Flow

At a high level, the flow is:

1. The browser requests the ZWC application under `/zimbra/`.
2. The bridge serves the extracted stock ZWC assets.
3. The bridge injects runtime configuration and narrow compatibility patches
   into selected generated or patched responses.
4. ZWC sends normal Zimbra-style SOAP and REST calls such as `/service/soap`.
5. The bridge handles those calls, uses cached projections when safe, and talks
   to Stalwart/JMAP when fresh upstream state is required.
6. The bridge returns Zimbra-shaped responses so the existing ZWC UI can keep
   operating.

From the user's point of view, the UI remains familiar. From the deployment
point of view, mailboxd is removed from the active request path.

## Why Static Assets Are Not Modified

The bridge intentionally does not edit the stock ZWC static files on disk.

This matters for several reasons:

- Upgrades are cleaner because the extracted ZWC artifact can be replaced
  without merging local edits into vendor files.
- The bridge's compatibility patches stay separate, auditable, and easier to
  review.
- Rollback is safer because a bad bridge patch can be changed or disabled
  without rebuilding a patched ZWC distribution.
- The legal and operational boundary is clearer because the project is not
  publishing a forked set of edited ZWC assets as its source of truth.
- Product and GUI teams can tell the difference between stock ZWC behavior and
  Project Z-Bridge enhancements.

In practical terms, `static/zimbra/` is treated as an extracted upstream
artifact. Project Z-Bridge owns the bridge code, the generated responses, and the
runtime patches layered on top of that artifact.

## How Runtime Injection Works

Runtime injection means the bridge changes selected responses while serving
them, rather than editing the source asset files permanently.

The bridge commonly uses injection for:

- Session-specific configuration needed by the web client.
- Compatibility fixes where ZWC expects mailboxd-specific behavior.
- Small UI enhancements such as bridge safety options, Rspamd score hover
  behavior, and save-original actions.
- Security and rendering plumbing that keeps untrusted message HTML isolated
  from the main application.
- Performance behavior such as avoiding unnecessary foreground reloads when a
  local projection can be updated safely.

The important distinction is that the original asset remains unchanged on disk.
The browser receives the bridge-adjusted response for that session and request.
That keeps the customization layer explicit and reduces the long-term cost of
upgrading or auditing the ZWC artifact.

## Performance Model

Project Z-Bridge is designed to make normal high-frequency UI paths feel more
responsive than classic ZWC backed by Zimbra mailboxd. Stalwart is fast at the
mail-server layer, JMAP gives the bridge a better model for grouping related
mailbox reads, and bridge-side caches avoid repeating expensive foreground work
when the bridge already has enough state to answer safely.

The bridge uses a cache-first model for common read paths:

- Startup and account context use persisted `GetInfo` projections for folders,
  preferences, tags, shares, aliases, and related metadata.
- Folder and conversation lists can be served from cached projections while a
  background refresh reconciles with Stalwart.
- Message detail reads use short-lived message caches so reopening or quickly
  navigating messages does not require a fresh upstream read every time.
- Visible-row prefetch warms likely next messages so the reading pane can open
  quickly.
- Calendar and mini-calendar views use cached appointment projections where safe.
- Preference saves and other small mutations avoid invalidating large caches
  when the changed data does not require it.

The bridge tries to keep the user's foreground path small. If a user marks a
message as spam, moves a message, deletes messages, or changes a preference, the
bridge should update the local UI projection immediately when it can do so
safely, then reconcile with Stalwart in the background.

That is different from a design where every click blocks on a full upstream
folder rebuild. The source of truth remains Stalwart, but the bridge should avoid
making the user wait for data it already has.

The model is cache-first, not cache-only. Cached projections make the UI fast,
while Stalwart remains authoritative and the bridge reconciles through normal
refreshes, ZWC NoOp fallback behavior, and JMAP Push when available.

## Near Real-Time Updates

Stalwart remains the authoritative data store. Project Z-Bridge uses JMAP to
read and mutate that state. For near real-time behavior, the bridge can also use
JMAP Push so Stalwart can notify the bridge that mailbox state changed.

The expected behavior is:

- JMAP Push tells the bridge that mail or mailbox state changed.
- The bridge warms affected hot-folder caches in the background.
- The UI can continue to receive fast cached responses while refresh work is
  happening.
- ZWC NoOp/long-poll behavior remains a compatibility path and fallback.
- Polling still exists as a safety net when push is unavailable, quiet, or
  disconnected.

The user does not need every mailbox change to block the visible UI in real time.
Email is naturally tolerant of a few seconds of background reconciliation. The
important part is that new state arrives reliably, visible folders stay fresh
enough, and common navigation does not feel like a full mailbox reload.

## GUI And Product Implications

Most of the visible interface is still classic ZWC. That is deliberate. It lets
users keep the UI they already understand while the backend changes underneath.

Project Z-Bridge can still add targeted GUI improvements without turning into a
large front-end fork. Examples include:

- Rspamd score hover behavior in the message list.
- Save-original actions for one or more selected messages.
- Bridge Safety preference controls.
- Safer message rendering and external-image handling.
- Reduced busy-overlay behavior when cached state can answer immediately.

GUI changes should stay narrow, follow existing ZWC interaction patterns, and be
covered by Playwright regression checks where practical. The project benefits
from preserving the familiar ZWC experience while improving the slow or missing
parts that matter for a Stalwart-backed deployment.

## Security Boundaries

The bridge treats email content as untrusted. Message HTML is sanitized and
rendered in an isolated frame with strict controls so hostile email content does
not become trusted application code.

Runtime-injected bridge code is different from message content. Bridge patches
are controlled application code served by the bridge. Message HTML is
attacker-controlled content and must stay isolated from the main ZWC application.

For more detail, see the security documentation under
[security/](security/), especially the iframe and external image documents.

## Operational Implications

For management and operations planning, the important implications are:

- Stalwart does the mail-server work it is designed to do.
- The bridge handles ZWC compatibility, request shaping, runtime patches, and
  cache projections.
- The bridge is expected to be much lighter than mailboxd, but cache design and
  clustering strategy still matter for large deployments.
- Multi-node deployments need a deliberate strategy for shared bridge state,
  sticky sessions, or shared cache/BridgeStore behavior.
- Unsupported SOAP operations should remain visible in the implementation matrix
  rather than being hidden behind generic success responses.

The architecture works best when the bridge stays focused: translate, cache,
patch narrowly, and keep Stalwart authoritative.

## Current Constraints

There are still constraints that stakeholders should understand:

- Some Zimbra SOAP/XML calls are implemented, some are partial, and some are not
  yet supported.
- A few generated or patched assets are intentionally conservative about browser
  caching because they include runtime/session-sensitive behavior.
- Cache correctness must be protected with regression tests because fast stale
  responses are only useful when background reconciliation is reliable.
- Compatibility with classic ZWC requires careful testing because small UI
  assumptions can depend on mailboxd behavior.

These are normal constraints for a compatibility bridge. They are also why the
project keeps separate documentation for architecture, cache behavior, SOAP
coverage, and regression testing.

## Deeper Docs In The Full Project

The full in-development tree contains deeper technical docs for architecture,
performance, security, SOAP/XML coverage, testing, and operations. Some of these
documents may not be included in the prerelease preview.

- [developer/ARCHITECTURE.md](developer/ARCHITECTURE.md) for the technical
  architecture map.
- [developer/PERFORMANCE_CACHE_AUDIT.md](developer/PERFORMANCE_CACHE_AUDIT.md)
  for cache behavior and known performance paths.
- [developer/JMAP_USAGE_AUDIT_AND_PLAN.md](developer/JMAP_USAGE_AUDIT_AND_PLAN.md)
  for current and planned JMAP usage.
- [developer/SOAP_COMPATIBILITY_MATRIX.md](developer/SOAP_COMPATIBILITY_MATRIX.md)
  for SOAP/XML implementation coverage.
- [developer/TESTING_AND_MIDDLEWARE_INTEROP.md](developer/TESTING_AND_MIDDLEWARE_INTEROP.md)
  for test strategy and middleware integration context.
- [sysadmin/SYSADMIN_CACHE_TUNING_GUIDE.md](sysadmin/SYSADMIN_CACHE_TUNING_GUIDE.md)
  for operational cache tuning.
- [developer/PRODUCT_ENHANCEMENTS.md](developer/PRODUCT_ENHANCEMENTS.md) for
  ZWC+Stalwart enhancements over stock ZWC+Zimbra behavior.
