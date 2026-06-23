---
type: open-questions
feature: mini-app-exchange
created: 2026-06-19
tags:
  - inception/open-questions
  - feature/mini-app-exchange
---

# Open questions — Mini-App Exchange

> [!question] For the mob to resolve. Driver guesses are marked so there's a starting point. Nothing here blocks Construction of Wave 0; these refine edges.

## Q1 — Exact HTML size cap

What's the right ceiling on a shared HTML document? **[DRIVER GUESS: 256 KB]** — generous for self-contained inline HTML/JS/CSS while bounding backend storage and QR/transfer cost. Mob: confirm or adjust. (Drives the `413 payload_too_large` boundary in [[api-contract]].)

## Q2 — Per-IP rate limit on public preview fetches

The preview/install fetch (`GET …/share/{id}`) is unauthenticated, so it needs a coarse per-IP limit to prevent enumeration/scraping. **[DRIVER GUESS: reuse the existing limiter at a generous per-IP rate]** — exact number is BE's call. Mob: any abuse concern that wants a tighter bound?

## Q3 — What does the raw https link do when the app isn't installed?

v1's QR path uses a custom scheme and works when the app is present. If someone taps the **https** link on a device without the app, v1 has no polished landing page (see [[out-of-scope]]). **[DRIVER GUESS: best-effort — backend returns a minimal "get the app" message; polished landing deferred.]** Mob: acceptable for launch, or is a basic landing page table-stakes?

## Q4 — Should the sharer be able to share the same mini-app again after revoking?

After "stop sharing", can the user re-share and mint a fresh link? **[DRIVER GUESS: yes — re-sharing is just a new POST, yielding a new link; the old one stays dead.]** Mob: confirm this is the expected mental model.
