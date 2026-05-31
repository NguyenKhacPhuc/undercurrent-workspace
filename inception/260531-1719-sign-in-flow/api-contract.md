---
type: api-contract
feature: sign-in-flow
created: 2026-05-31
backend-work: false
tags:
  - inception/api-contract
  - feature/sign-in-flow
---

# API contract

> [!success] **No backend changes** — feature is purely client-side.

The captured profile is persisted locally on the device only. There is no server, no network call, and no auth handshake in this feature.

When the backend lane wakes up (likely first BE feature: cross-device sync), the locally-captured profile will need a migration path. That work belongs to a future Inception — see [[open-questions#Q5]] and [[out-of-scope]].
