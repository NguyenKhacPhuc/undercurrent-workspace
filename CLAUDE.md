# Undercurrent workspace — Claude working notes

Auto-loaded when working at the workspace root. Carries
**project-shape facts**. AI-SDLC behavior is baked into the forked
skills at `.claude/skills/{inception,construct}/` — don't read those
to learn the conventions; they read this file to learn the project.

## Project shape

- **Two submodules:** `weft/` (substrate SDK) + `undercurrent/` (host
  app). Each is an independent git repo with its own GitHub remote,
  default branch, build, and `gh` issues.
- **Most work lands in `undercurrent/`.** Substrate changes (`weft/`)
  are pulled in reactively when the host app needs new capability.
- **No backend today.** LLM providers (Anthropic / OpenAI / OpenRouter
  / DeepSeek) are external APIs whose contracts we consume but don't
  author. The `backend` lane reactivates when auth / sync /
  multi-device ships.

## Lane → location → CLAUDE.md mapping

| Lane | Code lives under | CLAUDE.md to consult |
|---|---|---|
| `kmp-common` | `undercurrent/<module>/src/commonMain/` | `undercurrent/CLAUDE.md` |
| `android` | `undercurrent/<module>/src/androidMain/` or `androidApp/` | `undercurrent/CLAUDE.md` |
| `ios` | `undercurrent/<module>/src/iosMain/` or `composeApp/src/iosMain/` | `undercurrent/CLAUDE.md` |
| `substrate` | `weft/<module>/` (any source set) | `weft/CLAUDE.md` |
| `backend` | (dormant — TBD when first BE feature ships) | TBD |

**`kmp-common` is the default lane for new feature work.** Most slices
are 80% shared `commonMain` (`ViewModel`, `Screen`, `Repository`
interface) + small per-platform impls. Split into per-platform stories
only when there's genuinely platform-specific work (Android Weft
wrapper, iOS Keychain access, etc.).

## Workspace-level commands

```bash
# Pull latest of everything
git pull
git submodule update --recursive --remote

# Bump skills from upstream (forked — merge conflicts likely)
SRC=/path/to/simple-ai-sdlc/ai-sdlc/skills
diff -ru "$SRC/inception/SKILL.md" .claude/skills/inception/SKILL.md
diff -ru "$SRC/construct/SKILL.md" .claude/skills/construct/SKILL.md
# … manually merge wanted upstream changes

# Start a feature
/inception
```

## When BE arrives

When the first backend-touching feature ships (likely auth — Sign in
with Apple / Google for cross-device persona + memory sync), do:

1. Decide where BE code lives: new third submodule, inside `weft/`, or
   inside `undercurrent/`. Update the lane table above.
2. Add a `backend/CLAUDE.md` to that repo with build / test commands.
3. The forked Inception skill's BE default flips: re-engage the full
   api-contract flow (it's already there, just suppressed by
   `Project context` in `.claude/skills/inception/SKILL.md`).

## What NOT to do

- Don't run `/inception` or `/construct` from inside a submodule. Both
  are workspace-level commands — artifacts live in
  `undercurrent-workspace/inception/<feature>/`.
- Don't `gh pr create` from the workspace root — there's no upstream
  for the workspace's submodule pointers as a PR. PRs go to the
  subrepo's GitHub remote (`cd undercurrent/` or `cd weft/` first).
- Don't create per-platform stories for work that's actually
  commonMain. Use the `kmp-common` lane.
- Don't add a `backend/` issues dir for features that have no BE.
  Empty lane dirs confuse the `_index.md` wave layout.
- Don't edit upstream-style overrides into the vendored skills *and*
  this file. Behavior overrides belong in the skills (which read this
  file for project shape). Project-shape facts belong here.
