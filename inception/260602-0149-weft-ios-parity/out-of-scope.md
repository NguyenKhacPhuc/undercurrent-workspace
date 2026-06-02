---
type: out-of-scope
feature: weft-ios-parity
created: 2026-06-02
tags:
  - inception/out-of-scope
  - feature/weft-ios-parity
---

# Out of scope

> [!warning]
> Explicitly **not** in this feature. The point of the feature is the *common mechanism*; once it exists, each item below becomes a single, cheap capability slice against it — they are deferred, not abandoned.

## Deferred device capabilities (the backlog)

Each is one future capability slice once the iOS mechanism lands. Grouped by rough priority for whoever picks up the next feature.

- **Notifications** — schedule / cancel / list local notifications on iOS. Persistence already shared; needs the iOS notification-center binding.
- **Location** — current location, geocode, reverse-geocode. Needs location permission + the iOS location manager.
- **Camera** — capture a photo on iOS.
- **Media picker** — pick photos/videos from the iOS library.
- **Vision** — OCR + barcode scanning on iOS.
- **PDF** — extract text / render pages / create PDFs on iOS.
- **Calendar** — read/create/update/delete events on iOS.
- **Contacts** — list/search/create contacts on iOS.
- **Audio** — record / play audio on iOS.
- **Media library** — query/load/delete library assets on iOS.
- **Translation + language ID** — on-device translate / detect language on iOS.
- **Volume** — read/set system volume on iOS (set is limited by the platform).
- **Wifi** — current network info on iOS (SSID needs extra entitlement).
- **Installed apps** — best-effort only on iOS (platform hides the installed-app list).
- **Telephony** — carrier/network info on iOS (deprecated surface).
- **App shortcuts** — home-screen / Spotlight shortcuts on iOS.
- **System settings deep links** — open specific iOS Settings panels.
- **Bluetooth** — paired-device listing / scan on iOS (platform-limited; larger effort).
- **Sensors** — pedometer / ambient light on iOS.

## Other exclusions

- **Backend / sync changes** — none; this feature is client/SDK-side. Provider sign-in consumes external OAuth we don't author.
- **New user-facing app features in undercurrent** — the host change is *adoption only* (switch to the common setup + delete duplicated impls). No new screens or behaviors.
- **Android behavior changes** — only incidental refactors required to share composition code; no intended change to how Android hosts behave.
- **Lifting the runtime composition root beyond what iOS parity needs** — the shared-composition refactor goes only as far as making iOS and Android share the mechanism; broader composition-root redesign is its own future effort.
- **iOS 16/17-only capabilities** — excluded by the iOS 15 floor (see [[decisions]] D2). Revisit per-capability if the floor rises.
