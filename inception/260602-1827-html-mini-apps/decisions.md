---
type: decisions
feature: html-mini-apps
created: 2026-06-02
tags:
  - inception/decisions
  - feature/html-mini-apps
---

# Decisions

> [!info]
> ADR-lite log. Driver decisions for the mob to ratify.

---

### D1 — HTML+bridge is an escape hatch, not a replacement for the native palette — 2026-06-02

- **Context:** The agent renders mini-app UI from 147 native components — excellent for structured/standard UI, but it can't express novel shapes without an engineer shipping a new component. The substrate's `HtmlComponent` already renders arbitrary HTML/JS but is sandboxed (no bridge).
- **Options considered:** (a) keep extending the native palette; (b) replace it with HTML; (c) keep native as the default + add HTML+bridge for the flexible long tail.
- **Decision:** **(c).** Native palette stays the default for Tier-A (structured, standard, themed) UI; HTML+bridge becomes the Tier-B path for custom logic / novel interaction / bespoke viz. A thin Tier-C (live camera/map/AR/heavy real-time) stays native-only and is out of scope.
- **Why:** 147 hand-built components don't scale — every novel mini-app needs an engineer. HTML+bridge lets the agent author the long tail itself, while native stays better for the common, structured cases.
- **Consequences:** Two rendering paths to maintain. Guidance needed (preamble/docs) on when the agent should reach for HTML vs a native component.

### D2 — Security: per-mini-app declared scopes + user approval — 2026-06-02

- **Context:** Mini-app JS is agent-authored and semi-untrusted (prompt-injection risk). The bridge is the boundary between untrusted JS and real device/app actions.
- **Options considered:** conservative default allowlist + host extend; **per-mini-app declared scopes + user approval**; agent-mediated only (no direct tool calls).
- **Decision:** **Per-mini-app declared scopes + approval.** Each mini-app declares the actions it needs; the user approves/denies on first run (app-permission style); the substrate enforces the approved set at the bridge boundary; the host stores the grants and surfaces the consent UX. Network only via the existing allowlist; CSP + WebView sandbox enforced.
- **Why:** Strongest user control, and the consent UX makes the capability legible — the user always knows what a flexible mini-app can touch. Worth the extra consent-UX + grant-storage cost given untrusted JS.
- **Consequences:** Adds a consent screen + a per-mini-app grant store + a scope-enforcement gate. The host must define the **offerable** action menu (which actions are ever approvable). Destructive/sensitive actions may warrant extra friction (flagged in [[open-questions]]).

### D3 — Full bridge surface in v1 — 2026-06-02

- **Context:** The `window.weft` API could ship minimal (callTool only) or full.
- **Decision:** **Full:** `callTool` (invoke an approved action) + `getState`/`setState` (per-mini-app persistence) + theme tokens (CSS vars) + `sendMessage` (fire an agent turn) + `onData` (native→JS live push) + lifecycle (open/close) + multi-instance.
- **Why:** The driver wants a genuinely usable mini-app runtime, not a proof of concept.
- **Consequences:** Larger surface → more stories. `onData` / lifecycle / multi-instance are the most speculative before real usage; sliced so they can be deprioritized if needed (see [[_index]] waves).

### D4 — Scope: substrate bridge + host catalog (end-to-end) — 2026-06-02

- **Context:** The feature could stop at the substrate bridge mechanism, or include the host change that makes a mini-app *be* an HTML doc.
- **Decision:** **Both.** The substrate ships the bridge; the host (`undercurrent/`) makes a mini-app saveable as an HTML doc (+ declared scopes + state), cached for render-on-tap, with the approval UX and the agent-turn wiring.
- **Why:** Delivers an actually-usable HTML mini-app end-to-end, not just a mechanism with no consumer.
- **Consequences:** Cross-repo feature (weft + undercurrent). Host stories depend on substrate stories landing + being bumped into the workspace `weft` submodule.

### D5 — Mini-app HTTP stays unrestricted for now — 2026-06-05

- **Context:** [[05-tighten-miniapp-http-allowlist]] proposed replacing the
  mini-app `http_fetch` client's `NetworkPolicy.OPEN` with a real,
  user-curated allowlist.
- **Decision:** **No limit for now.** Mini-app `http_fetch` keeps
  `NetworkPolicy.OPEN` (a mini-app may fetch any host), matching the rest
  of the app's posture. k05 is deferred, not built.
- **Why:** The driver doesn't want the friction/complexity (an allowlist
  store + a settings UI + a default-host call) before there's a real need.
- **Consequences:** A script-enabled, approved mini-app can reach any
  host. The allowlist *seam* is already in place (the dedicated client
  installs weft's `NetworkAllowlistPlugin`), so tightening later is a
  config change — revisit if mini-app HTTP handles sensitive data.

### D6 — Mini-app CSP allows remote https images — 2026-06-05

- **Context:** [[08-sandbox-hardening]] set `img-src data:`, blocking remote
  images. Device testing hit it immediately: an image-gallery mini-app
  that takes image URLs couldn't display them.
- **Decision:** Loosen to `img-src https: data:` — remote https images
  load; `connect-src 'none'` is unchanged, so `fetch`/`XHR`/`WebSocket`
  and scripts/styles/frames/navigation stay blocked.
- **Why:** A gallery that can't show images is broken in practice; remote
  images are a normal mini-app need. The substrate's real network seal is
  `connect-src`, not `img-src`.
- **Consequences:** Residual exfiltration surface = a GET-only image-URL
  ping with no readable response (size-limited). Accepted for usability.
  Shipped as weft fix PR #27, bumped into the workspace `weft` submodule.

### D7 — Open the mini-app sandbox (network, remote resources, iframes) — 2026-06-05

- **Context:** Device testing showed the [[08-sandbox-hardening]] CSP was
  too strict for real widgets — no network, no remote CSS/fonts/media, no
  iframes.
- **Decision:** Loosen the CSP so a mini-app is a sandboxed web page that
  can use the network and remote resources over https:
  `connect-src https: wss:` (fetch/XHR/WebSocket), `frame-src https:`
  (iframes), `media-src https: data:`, `style-src 'unsafe-inline' https:`,
  `font-src https: data:`. Nav guards updated to allow sub-frame (iframe)
  loads while still blocking main-frame navigation away.
- **Kept blocked:** remote **top-frame scripts** (`script-src` stays
  inline-only — no `<script src=cdn>`), navigating away, base/form hijack.
- **Why:** Full-featured widgets need the network + embeds; a sealed page
  was breaking common cases (galleries, video, API calls).
- **Consequences:** **This supersedes the network-sealing intent of s08.**
  A mini-app's JS can now exfiltrate data directly, so the per-action
  consent gate (`http_fetch`) no longer fences network access — it's now
  about *device/app* actions (store, share, …), not network. `connect-src`
  / `frame-src` can be tightened back to an allowlist if wanted. Shipped
  as weft PR #28, bumped into the workspace `weft` submodule.
