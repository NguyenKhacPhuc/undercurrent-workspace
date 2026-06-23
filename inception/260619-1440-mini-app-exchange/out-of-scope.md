---
type: out-of-scope
feature: mini-app-exchange
created: 2026-06-19
tags:
  - inception/out-of-scope
  - feature/mini-app-exchange
---

# Out of scope — Mini-App Exchange v1

> [!info] Explicitly excluded. The cheapest argument to prevent is one we wrote down. Several of these are the natural Phase 2+ of the ecosystem.

- **Sharing native (trigger-prompt) mini-apps.** v1 is HTML mini-apps only — they are self-contained and portable. Native mini-apps re-run the recipient's agent, so their result depends on the recipient's model/keys (unpredictable). Deferred.
- **Remix in chat.** Opening an installed shared mini-app into the creator context so the recipient's assistant can modify it. This is the marquee Phase 2 feature — deliberately held back to keep v1 a clean vertical slice.
- **Discovery catalog.** A browsable/trending feed of shared mini-apps. Needs the share store (which v1 builds) but also listing, ranking, and a lot of trust-and-safety. Phase 3.
- **Trust & safety:** reporting, flagging, moderation, takedowns, ratings, capability badges beyond the plain list. Phase 4. v1's only safety guarantees are the capability clamp + the "author not vetted" framing.
- **In-app QR scanner.** Recipients use their phone's normal camera (see [[decisions#D3]]).
- **Web landing page for "app not installed".** When a raw https share link is opened on a device without the app, v1 does not present a polished install-the-app web page. Best-effort only; a landing page is a later add.
- **Universal / App Links** (association-file-backed https deep linking). v1 uses a custom URL scheme for the QR/deep-link path (see [[decisions#D3]]). Upgrading the https link to open the app directly is deferred.
- **Sharing saved state.** A recipient always starts the mini-app fresh; the sharer's per-device state is never bundled (see [[decisions#D5]]).
- **Versioning / update push.** Once shared, a link is a frozen snapshot. Editing your mini-app later does not update anyone's installed copy or the shared bundle. No "new version available". Deferred.
- **Agent-initiated sharing.** A `share_mini_app` tool the assistant could call ("share my water tracker with a link") is a natural fast-follow but not in v1 — sharing is a user-driven UI action only.
- **A "my shares" management screen.** v1 lets you stop sharing from the mini-app itself (it knows it's shared). A dedicated list of everything you've ever shared is deferred.
