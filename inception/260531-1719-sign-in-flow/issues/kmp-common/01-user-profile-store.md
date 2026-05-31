---
type: issue
feature: sign-in-flow
lane: kmp-common
status: ready
wave: 0
estimate: 60m
blocked-by: []
tags:
  - inception/issue
  - lane/kmp-common
  - feature/sign-in-flow
  - status/ready
  - wave/0
---

# [kmp-common] A user profile can be saved and loaded locally

**Lane:** kmp-common
**PRD section:** [[PRD#Goals]] (the "survive an app kill + relaunch" goal)
**API contract section:** n/a — no backend

## Why

Sign-in and the Settings editor both need a single, shared place that owns *the* user's profile on this device. This story stands the storage up as a foundation, with its observable contract framed in domain terms, so the UI slices that follow ([[02-first-launch-sign-in-screen]] and [[03-edit-profile-from-settings]]) can be built and tested without each one re-inventing persistence.

## Acceptance criteria

This is a foundation slice — no user-visible UI yet. The contract is observable through the public domain surface (calls + persistence behavior across process restarts).

- [ ] The app can record a user profile consisting of a display name and an email.
- [ ] Once recorded, the same profile is returned on subsequent reads — both within the same process and after the app is force-killed and reopened.
- [ ] Before any profile has ever been recorded on this install, the "current profile" is observably absent (distinguishable from a profile whose fields happen to be empty strings).
- [ ] Recording a second profile overwrites the first (single-slot, no history).
- [ ] Reads are observable from both Android and iOS targets of the shared module — i.e. neither target falls back to a platform-only API and the shared module's verify commands stay green.

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape).
- The lane's standard build/test commands pass with no regressions. From `undercurrent/CLAUDE.md`: `./gradlew :androidApp:compileDebugKotlin`, `./gradlew :composeApp:compileKotlinIosSimulatorArm64`, and the affected module's `test` task.

## Blocked by

- nothing — independently grabbable.

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Construction reads
> the lane's CLAUDE.md + opens existing similar files and may diverge from
> any hint here without re-opening Inception.

- **Likely files affected:** somewhere under `core/domain/src/commonMain/.../` for the repository interface; persistence via the existing `data/datastore` KMP DataStore-Preferences plumbing. The Koin module wire-up follows the pattern other repositories already use.
- **Existing pattern to mirror:** look at how `ThemePrefs` is persisted (single global slot in DataStore-Preferences) — same shape applies here. See [[../../../../CONTEXT#ThemePrefs]].
- **Watch out for:** the "absent vs empty-string" distinction. `Flow<UserProfile?>` (null = absent) is one obvious shape; a sealed `LoadResult` is another. Construction picks.
- **Watch out for:** keep this 100% commonMain — no Keychain on iOS, no Credential Manager on Android. See [[../../decisions#D4]].

## Out of scope for this story

- The sign-in screen UI — that's [[02-first-launch-sign-in-screen]].
- The Settings editor UI — that's [[03-edit-profile-from-settings]].
- Any validation rules on the fields (the store accepts what it's given; UI layers enforce rules). See [[../../open-questions#Q1]] and [[../../open-questions#Q2]].
- A delete / wipe operation. Not needed in v1; see [[../../open-questions#Q7]].
