---
type: decisions
feature: ios-mini-app-render
created: 2026-06-19
tags:
  - inception/decisions
  - feature/ios-mini-app-render
---

# Decisions

> [!info]
> ADR-lite log. Driver-made calls for the mob to ratify.

### D1 — Lift the mini-app orchestrator to shared code, don't re-implement on iOS — 2026-06-19

- **Context:** The Android mini-app orchestrator (the logic that turns a
  tap into a consent prompt, an HTML render, a state cache, and a
  save-as-mini-app) lives in an Android-only source set, but a scout
  confirmed it contains **no Android-specific logic** — every type it
  depends on, including the agent runtime and its UI bridge, is already
  defined in shared code with an existing iOS form.
- **Options considered:** (a) write a parallel iOS twin of the
  orchestrator; (b) lift the existing orchestrator into shared code so
  both platforms use one copy.
- **Decision:** Lift it into shared code; both platforms use the same
  orchestrator.
- **Why:** Single source of truth, zero behavior drift between
  platforms, and it's strictly less work than maintaining a twin. The
  earlier Android-only placement was historical, not a design boundary.
- **Consequences:** Both platforms are committed to identical mini-app
  orchestration. Any future genuinely-platform-specific behavior would
  need an explicit shared/per-platform seam introduced at that point.

### D2 — Ship full Android parity in one story, not a phased subset — 2026-06-19

- **Context:** Because the orchestrator lifts as one unit, carving out a
  subset (e.g. render-only, defer consent/save) is *more* work than
  shipping the whole thing.
- **Options considered:** Full parity in one slice; render+consent only;
  render-only.
- **Decision:** Full parity — consent, render, scope-gated actions,
  state, and save-as — in a single story.
- **Why:** Least effort, no security gap (consent ships with render),
  and no awkward intermediate state where iOS renders un-consented
  mini-apps.
- **Consequences:** One slightly larger story instead of several tiny
  ones; the feature is a single PR.

### D3 — On-device smoke deferred; verification is compile + simulator — 2026-06-19

- **Context:** No iOS hardware is available in the current dev loop,
  consistent with the deferred device-smoke posture of the prior mobile
  features in this workspace (weft-ios-parity, ios-agent-bringup,
  html-mini-apps).
- **Decision:** Definition of done is "both targets compile + the
  mini-app flow runs in the iOS simulator." A real-device pass is a
  tracked follow-up, not a blocker for this story.
- **Why:** Matches established workspace practice; the simulator
  exercises the WKWebView render + bridge path that's the actual risk.
- **Consequences:** A real-device regression (e.g. WKWebView quirk only
  on hardware) could slip past this story; accepted, same as prior
  features.
