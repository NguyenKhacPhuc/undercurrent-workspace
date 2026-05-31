---
type: issue
feature: <feature-slug>
lane: <android|ios|backend|web|...>
status: ready
wave: <0..N>
estimate: <Xm>     # rough order-of-magnitude only — Construction sizes the real work
blocked-by: []     # or list of "[[NN-other-story]]" wikilinks (same lane)
tags:
  - inception/issue
  - lane/<lane>
  - feature/<feature-slug>
  - status/ready
  - wave/<0..N>
---

# [<Lane>] <one-line story title — what user-observable outcome this delivers>

**Lane:** <Android | iOS | Backend | …>
**PRD section:** <link or heading>
**API contract section:** <endpoint(s) consumed, if any — or "n/a">

## Why

<1–3 sentences. What user-visible outcome this enables, or why it's a needed step toward one. Stay above implementation: name behavior, not classes.>

## Acceptance criteria

The user-visible contract Construction will satisfy. Write each
bullet as a **plain-language user outcome** — what someone using the
app would see or be able to do. Construction translates these into
test code (kotest Given/When/Then) in its red step.

**Rules for AC bullets:**

- Use user-facing nouns and verbs only ("pin a conversation",
  "the conversation list", "the pin icon"). No field names
  (`pinnedAtMs`), no class names (`ConversationsViewModel`), no test
  framing (`Given X When Y Then Z`).
- Each bullet stands on its own as a sentence a product person would
  read. If a non-engineer can't follow it, rewrite.
- Cover the happy path AND the obvious edge cases (empty, error,
  user cancels mid-flow, two-of-them, etc.) — Construction won't
  invent edge cases for you.

```markdown
- [ ] Pinned conversations appear at the top of the conversations list.
- [ ] Tapping the pin icon on an already-pinned conversation removes the pin.
- [ ] When multiple conversations are pinned, the most-recently-pinned one is first.
- [ ] Pin state survives app restart.
```

Two procedural bullets are added by Construction (not Inception):

- Test for this slice exists and passes (Construction chooses the test shape).
- The lane's standard build/test commands pass with no regressions. (See [CLAUDE.md](CLAUDE.md) or `<lane>/CLAUDE.md` for the exact commands.)

## Blocked by

- <wikilinks to other stories in the same lane, or "nothing — independently grabbable">

## Hints (non-binding)

> [!tip]
> Hints help Construction orient. They are NOT a contract. Construction reads
> the lane's CLAUDE.md + opens existing similar files and may diverge from
> any hint here without re-opening Inception.

- **Likely files affected:** `<rough paths>` — confirm against the actual code before editing.
- **Existing pattern to mirror:** `<file>` — Construction inspects this and matches its conventions.
- **Watch out for:** `<known constraint, regression risk, or coordination point>`

## Out of scope for this story

- <Things adjacent that look related but belong to a different story or feature.>
