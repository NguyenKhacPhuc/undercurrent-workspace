---
type: issue
feature: mini-app-exchange
lane: android
status: ready
wave: 2
estimate: 45m
blocked-by: ["[[03-install-preview]]"]
tags:
  - inception/issue
  - lane/android
  - feature/mini-app-exchange
  - status/ready
  - wave/2
---

# [Android] Opening a share link or scanning its QR opens the preview on Android

**Lane:** Android
**PRD section:** [[PRD#Story 2 — Preview a shared mini-app]]
**API contract section:** consumes the share identifier carried in the deep link → drives [[03-install-preview]]

## Why

A recipient on Android needs the share link / scanned QR to actually open the app on the install preview. This is the Android-specific glue: registering the install link so the OS hands it to the app, and routing it to the preview.

## Acceptance criteria

- [ ] Scanning the share QR with the Android camera offers to open Undercurrent and lands on the shared mini-app's preview.
- [ ] Opening the install deep link on a device with the app installed lands on the preview for that share.
- [ ] A malformed or empty install link does not crash the app — it lands somewhere sensible (e.g. the app's normal start) without attempting an install.
- [ ] Following an install link while the app is already open still reaches the preview.

## Blocked by

- [[03-install-preview]] — routes into the shared preview screen.

## Hints (non-binding)

> [!tip]
> Verify per `undercurrent/CLAUDE.md` (Android lane). Force-stop after manifest changes.

- **Likely work:** register the custom URL scheme (`undercurrent://install/<shareId>`) in the Android manifest and route the incoming identifier into the preview. No app-association files needed for the custom scheme (see [[decisions#D3]]).
- **Watch out for:** the app may already be running — handle both cold-start and warm-resume entry into the preview.

## Out of scope for this story

- The preview/install behavior itself (kmp-common).
- A web landing page when the app is not installed (out of scope for v1).
