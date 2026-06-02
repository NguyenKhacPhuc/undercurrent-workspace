---
type: api-contract
feature: html-mini-apps
created: 2026-06-02
backend-work: false
tags:
  - inception/api-contract
  - feature/html-mini-apps
---

# API contract

> [!success] **No backend changes.** This is a substrate + host feature. The bridge runs entirely on-device; a mini-app's network access goes through the existing host-allowlist `NetworkPolicy` (an *external*-provider request the agent already makes), not a new backend. There is, separately, a **client-side JS API contract** (`window.weft.*`) the substrate exposes to mini-app code — that lives in the substrate stories, not here.
