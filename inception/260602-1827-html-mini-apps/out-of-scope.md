---
type: out-of-scope
feature: html-mini-apps
created: 2026-06-02
tags:
  - inception/out-of-scope
  - feature/html-mini-apps
---

# Out of scope

> [!warning]
> Explicitly **not** in this feature.

- **Replacing the native component palette.** The 147 native components stay the default for Tier-A (structured, standard, themed) UI. This feature is the Tier-B escape hatch — additive, not a rewrite ([[decisions]] D1).
- **Tier-C native-only mini-apps** — live in-view camera viewfinder, a pannable native map, AR, 60fps native games, live native sensor-stream viz. These need a native component/screen, not HTML; they're a separate effort if/when wanted.
- **Backend / sync / cloud mini-apps.** No server. A mini-app's network access goes through the existing on-device allowlist policy, not a new BE.
- **Sharing / publishing mini-apps between users** (an app store, import/export). Mini-apps are agent-authored and locally saved in v1.
- **A general-purpose plugin/extension SDK** for third-party (non-agent) HTML. The bridge is scoped to agent-authored mini-apps inside this app.
- **Offline package management / bundled assets** (multi-file HTML apps, npm-style deps). v1 mini-apps are self-contained HTML docs.
- **Cross-mini-app communication** (one mini-app talking to another). Each mini-app is isolated in v1.
