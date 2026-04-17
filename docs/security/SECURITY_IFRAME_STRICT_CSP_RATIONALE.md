# SECURITY: Why Iframe Isolation Was Required for Strict CSP

Date: 2026-04-17
Status: Rationale / design-decision reference
Owner: Bridge/Webclient

Related reading:
- [SECURITY_IFRAME_STRICT_CSP_DESIGN.md](SECURITY_IFRAME_STRICT_CSP_DESIGN.md)
- [SECURITY_IFRAME_STRICT_CSP_SEQUENCE.md](SECURITY_IFRAME_STRICT_CSP_SEQUENCE.md)
- [SECURITY_AUDIT.md](SECURITY_AUDIT.md)
- [SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md](SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md)

## Purpose

Explain **why** the bridge uses an iframe + optional separate renderer origin to achieve strict CSP for mail-body rendering, and document the alternatives that were considered and rejected. This is the "why not just…?" document for future maintainers who will inevitably ask the question again.

## TL;DR

The iframe wasn't the goal. **Independent CSP isolation was the goal.** The iframe is the only mechanism a modern browser offers to get a second, independently-governed document context inside an otherwise unmodified legacy SPA. Every cheaper alternative fails to shrink the blast radius when the sanitizer misses a payload; every more principled alternative requires the ~year-long ZWC rewrite that Project Z-Bridge explicitly chose to avoid.

## The Core Constraint: Strict CSP Is Architecturally Impossible for the App Shell

From `SECURITY_AUDIT.md` (Finding **C-1**) and `SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md` §"Residual Risks":

1. **Dynamic code evaluation is load-bearing.** `zimbra/js/ajax/boot/AjxPackage.js:274–293` is the Zimbra package loader. It fetches JS packages over XHR and executes them in the global scope via three browser-specific `eval`-equivalent fallbacks. The entire application boot process depends on this pattern. Any CSP under which ZWC boots **must** include `'unsafe-eval'`.

2. **UI templates are compiled at runtime.** Template compilation (`ZmSearch.js:1380` and many others) generates functions from strings at runtime. Also requires `'unsafe-eval'`.

3. **Inline script is everywhere.** 31 inline `<script>` blocks across 14 JSP/HTML files. `launchZCS.jsp` alone has 6 blocks with server-rendered values (`${csrfToken}`, auth expiry). `login.jsp` contains roughly 380 lines of inline JS. Every one of these requires `'unsafe-inline'` or a per-block nonce.

4. **`innerHTML` is everywhere.** `Dwt.setInnerHtml` is called from hundreds of sites; the audit counts 1,113 `innerHTML` assignments across 50 files. CSP cannot police `innerHTML`-sourced XSS at all.

The audit's own remediation estimate to make the app shell strict-CSP-compatible: **8–16 months of dedicated engineering**. Until that work ships, the minimum viable CSP for the shell requires both `'unsafe-eval'` and `'unsafe-inline'`, which — per the audit — "provides effectively zero script protection."

So we cannot get strict CSP on the app shell. Period. That is the hard ceiling.

## Why Iframe Containment Is the Right Workaround

The iframe approach flips the question: instead of trying to harden the shell (impossible), put the **untrusted content** (email HTML) into a document context where we **can** enforce strict CSP.

The browser-enforced iframe boundary gives us three properties we cannot get any other way in an unmodified SPA:

1. **Independent CSP context.** The inner document's CSP is enforced by the browser independently of the outer page. The renderer document ships with `default-src 'none'; script-src 'none'; connect-src 'none'; object-src 'none'; frame-src 'none'; font-src 'none'; media-src 'none';` — regardless of what the shell's CSP says.
2. **Origin isolation.** When the iframe is served from a dedicated renderer origin (e.g. `renderer.example.com` while the shell is at `stalwart.example.com`), a sanitizer bypass cannot read `ZM_AUTH_TOKEN`, touch ZWC globals, navigate the top frame, or exfiltrate via same-origin XHR.
3. **Compatibility preservation.** The classic "Display Images" deferred-media consent infobar keeps working because the design explicitly falls back to the legacy iframe flow for messages with `dfsrc` / `dfbackground` attributes. See `SECURITY_IFRAME_STRICT_CSP_DESIGN.md` §"External-image infobar".

These are properties of the browser's document/origin model, not of our code. That is why they are worth the complexity.

## Alternatives Considered and Rejected

| Alternative | Why it was rejected |
|---|---|
| **Strict CSP on the whole app shell** | Requires removing `eval`-based package loading, runtime template compilation, all inline scripts, and all `innerHTML` sinks. Audit estimate: 8–16 months. This is precisely the rewrite Project Z-Bridge was chartered to avoid. |
| **Nonces instead of `'unsafe-inline'`** | Nonces only solve inline-script CSP, not `eval`. And they would still require editing every JSP/JS template that emits an inline block — comparable cost to the rewrite above, without solving the `AjxPackage` loader. |
| **Shadow DOM + Trusted Types** | Shadow DOM provides style/DOM **encapsulation**, not a security boundary. A sanitizer miss still runs with full app-shell privileges — cookie access, ZWC internals, all of it. Trusted Types help inside modern code you control; they can't retrofit a legacy SPA that uses `innerHTML` in 1,113 places. |
| **Sanitization alone (no iframe)** | This is what stock ZWC had. The bridge's `ammonia`-based sanitizer (`shim/src/html.rs`) is significantly stronger than ZWC's native defense, but `SECURITY_MAIL_HTML_ATTACK_SURFACE_ASSESSMENT.md` §"Residual Risks" is explicit: sanitization alone leaves an unacceptable blast radius because CSP cannot be made strict. Belt **and** suspenders is the design. |
| **Service Worker CSP rewriting** | A Service Worker can intercept responses but cannot grant a document a stricter `script-src` than its own Response header declares, and the SW still runs in the shell's origin. No origin isolation either. |
| **Full server-side render to a separate page** | Breaks ZWC's SPA reading-pane UX. Also gives up progressive rendering, conversation view, and the inline compose/reply flows. |
| **`srcdoc` iframe (inline HTML, same origin)** | This is actually the legacy iframe fallback the design still supports. It buys browser-enforced CSP isolation but not origin isolation — `window.top`, same-origin XHR, and cookie reads remain reachable unless the iframe is `sandbox`ed without `allow-same-origin`, at which point links and image loading change behavior. The design keeps this mode as a compatibility path and treats the cross-origin renderer as the hardened path. |
| **Rewrite ZWC in a modern framework** | The 8–16 month option again. Out of scope by design. |

## Design Consequences Worth Remembering

The iframe + renderer-origin model has follow-on costs that show up in the surrounding documentation and code:

1. **Token lifecycle.** Because the inner document is a different origin, we can't just write HTML into it; we need a register/view handshake (`POST /service/email-renderer/register` → `GET /service/email-renderer/view/:token`) with short-lived, capacity-limited tokens. Full spec in `SECURITY_IFRAME_STRICT_CSP_DESIGN.md` §"Renderer Token and Document Lifecycle".
2. **Height guards.** Cross-origin iframes can't self-size. The runtime patches (`ZmMailMsgView`, `ZmMailMsgCapsuleView`) add explicit height guards to keep the reading pane stable. These are not cosmetic — they exist because of the origin boundary.
3. **Deferred-media path must be preserved.** The "Display Images" consent flow expects native ZWC iframe behavior. The renderer path explicitly skips messages with `dfsrc` / `dfbackground` and falls back to the legacy iframe flow so the infobar keeps working.
4. **Canary rollout is mandatory.** Per-user allowlists (`BRIDGE_EMAIL_IFRAME_RENDERER_USER_ALLOWLIST`) plus staged CSP modes (`log` → `report-only` → `enforce`) exist because the design acknowledges it cannot predict every legacy-client interaction. Telemetry-driven rollout is part of the safety argument, not an afterthought.

## Why This Is Still a Net Security Win

Even though the app shell remains on a permissive CSP, the mail-body rendering path — which is the **primary untrusted-content attack surface** in a webmail client — gains:

1. Server-side HTML sanitization with a modern allowlist sanitizer (`ammonia`) that is independently testable and continuously hardened.
2. Browser-enforced strict CSP on the rendering document (`default-src 'none'; script-src 'none'; connect-src 'none'`).
3. Origin isolation from auth cookies and ZWC globals when the renderer origin is configured.
4. Image proxying with content-type and SSRF guardrails.
5. Canary rollout with CSP violation telemetry.

The net effect is that the practical exploitability of a hostile HTML email is materially lower than stock ZWC, without paying the cost of rewriting the legacy client.

## One-Line Summary

We used an iframe because, in a browser in 2026, it is the only primitive that gives an untrusted HTML payload its own CSP and its own origin inside an unmodified legacy SPA — and every cheaper workaround leaves the sanitizer as the only line of defense, which the threat model explicitly refuses to accept.
