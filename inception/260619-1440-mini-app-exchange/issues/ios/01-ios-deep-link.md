---
type: issue
feature: mini-app-exchange
lane: ios
status: ready
wave: 2
estimate: 45m
blocked-by: ["[[03-install-preview]]"]
tags:
  - inception/issue
  - lane/ios
  - feature/mini-app-exchange
  - status/ready
  - wave/2
---

# [iOS] Opening a share link or scanning its QR opens the preview on iOS

**Lane:** iOS
**PRD section:** [[PRD#Story 2 — Preview a shared mini-app]]
**API contract section:** consumes the share identifier carried in the deep link → drives [[03-install-preview]]

## Why

The iOS-specific glue mirroring the Android deep-link story: register the install link so iOS hands it to the app, and route it to the shared mini-app preview.

## Acceptance criteria

- [ ] Scanning the share QR with the iOS camera offers to open Undercurrent and lands on the shared mini-app's preview.
- [ ] Opening the install deep link on a device with the app installed lands on the preview for that share.
- [ ] A malformed or empty install link does not crash the app — it lands somewhere sensible without attempting an install.
- [ ] Following an install link while the app is already open still reaches the preview.

## Blocked by

- [[03-install-preview]] — routes into the shared preview screen.

## Hints (non-binding)

> [!tip]
> Verify per `undercurrent/CLAUDE.md` (iOS lane).

- **Likely work:** register the custom URL scheme (`undercurrent://install/<shareId>`) in the iOS app config and route the incoming identifier into the preview. Custom scheme avoids associated-domains setup (see [[decisions#D3]]).
- **Watch out for:** cold-start vs. already-running entry into the preview; the Compose entry point on iOS receives the URL from the native app shell.

## Out of scope for this story

- The preview/install behavior itself (kmp-common).
- A web landing page when the app is not installed (out of scope for v1).
