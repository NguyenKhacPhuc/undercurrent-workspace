---
type: out-of-scope
feature: sign-in-flow
created: 2026-05-31
tags:
  - inception/out-of-scope
  - feature/sign-in-flow
---

# Out of scope

> [!warning]
> Things this feature is explicitly **not** doing. Be generous. The cheapest argument to prevent is one you wrote down.

- **Email verification.** No backend → nothing to send a verification email from. The captured email is trusted as typed.
- **Authentication.** No password, no Sign in with Apple, no Google sign-in, no biometric gate. The profile is *identity capture*, not *auth*.
- **Cross-device sync of profile, conversations, personas, mini-apps, or memory.** That entire space waits for the backend lane to wake up; see [[../../CONTEXT#Backend dormant]] and the workspace `CLAUDE.md`.
- **Stable local user id reserved for future BE.** Driver chose not to future-proof; see [[decisions#D2]].
- **Surfacing the captured display name in the agent's responses or greetings.** Separate Inception when product wants it. [[open-questions#Q3]] tracks the question.
- **Avatar / profile photo.** Not in v1. Storage, image-pickers, and per-platform permission flows are non-trivial; ship the text fields first.
- **Sign out / wipe profile.** Not in v1. Reinstall is the escape hatch. [[open-questions#Q7]] tracks if anyone needs it sooner.
- **Re-routing existing onboarding (provider picker / API key) into a single combined flow.** Sign-in is its own screen, ordered before the existing onboarding. [[open-questions#Q6]] confirms the ordering.
- **Internationalization of the sign-in screen copy beyond English.** Picked up by whatever the workspace's broader i18n initiative is — this feature doesn't move that needle.
- **Telemetry on email content.** Counting completions is fine; phoning the email home is explicitly not happening.
