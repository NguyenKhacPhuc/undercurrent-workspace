---
type: out-of-scope
feature: ios-agent-bringup
created: 2026-06-02
tags:
  - inception/out-of-scope
  - feature/ios-agent-bringup
---

# Out of scope

> [!warning]
> Explicitly **not** in this feature. Each is externally blocked, not merely unfinished.

- **Live model catalog on iOS** — `ModelCatalogRepository` needs Koog's LLM clients (JVM/Android-only). iOS keeps the synthetic placeholder pool so screens render. Unblocks when Koog goes KMP.
- **Key validation ("Connect" ping) on iOS** — `KeyValidationRepository` pings the provider via Koog's clients (JVM-only). iOS keeps the no-op stub; a bad key surfaces as an agent error on first use instead. Unblocks with Koog-KMP.
- **Voice input on iOS** — `SpeechRepository` is blocked by the AVAudioSession Kotlin/Native binding gap (weft-ios-parity story 11, deferred; needs a custom cinterop `.def` or a K/N binding refresh). The mic CTA stays hidden on iOS.
- **Backend / sync changes** — none. Agent consumes the substrate + external LLM providers; OAuth consumes external provider endpoints.
- **New user-facing features** — this feature reaches Android parity for the agent experience on iOS; no net-new product surface.
- **Substrate changes** — the substrate (weft-ios-parity) is already merged; if a gap is found, it's a new `weft` story, not part of this host feature.
