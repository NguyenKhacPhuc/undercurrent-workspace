# Undercurrent workspace — Claude working notes

Auto-loaded when working at the workspace root. Carries
**project-specific overrides** for the vendored ai-sdlc skills
(`/inception`, `/construct`). The implementation conventions live
inside each subrepo:

- `undercurrent/CLAUDE.md` — host app (KMP Android + iOS) architecture, MVI, UI rules
- `weft/CLAUDE.md` — substrate SDK module layout, tool-authoring rules

## Project shape

- **Two submodules:** `weft/` (substrate SDK) + `undercurrent/` (host
  app). Each is an independent git repo with its own GitHub remote,
  default branch, build, and `gh` issues.
- **Most work lands in `undercurrent/`.** Substrate changes (`weft/`)
  are pulled in reactively when the host app needs new capability.
- **No backend today.** LLM providers (Anthropic / OpenAI / OpenRouter
  / DeepSeek) are external APIs whose contracts we consume but don't
  author. BE lane will activate when auth / sync / multi-device ships —
  see [§ When BE arrives](#when-be-arrives).

## AI-SDLC overrides

The vendored skills at `.claude/skills/{inception,construct}/` assume a
single-repo monorepo with a backend. Our setup differs. Apply these
overrides whenever the skills run.

### Lanes

Use these lane labels (extends the skill's default `android | ios | backend`):

| Lane | Code lives under | CLAUDE.md to consult |
|---|---|---|
| `kmp-common` | `undercurrent/<module>/src/commonMain/` | `undercurrent/CLAUDE.md` |
| `android` | `undercurrent/<module>/src/androidMain/` or `androidApp/` | `undercurrent/CLAUDE.md` |
| `ios` | `undercurrent/<module>/src/iosMain/` or `composeApp/src/iosMain/` | `undercurrent/CLAUDE.md` |
| `substrate` | `weft/<module>/` (any source set) | `weft/CLAUDE.md` |
| `backend` | (not active today — see below) | TBD |

**`kmp-common` is the default lane for new feature work.** Most slices
in this codebase are 80% commonMain (shared `ViewModel`, `Screen`,
`Repository` interface) + small per-platform impls. Don't artificially
split the commonMain slice into "android" + "ios" stories that both
have to write the same code — make one `kmp-common` story for the
shared work, then per-platform stories only when there's genuinely
platform-specific work (Android `data:weft` impl, iOS Keychain impl,
SpeechRecognizer wiring, etc.).

### Backend lane

For features without BE work (~all current features):

- **Step 1.3 "Does this touch the backend?" → answer "no" by default.**
  Only flip to yes if the driver explicitly names a BE need.
- **Skip `api-contract.md`.** Write the one-line callout the template
  describes: `> [!success] **No backend changes** — feature is purely client-side.`
- **No `issues/backend/` directory.**

When BE *does* show up (planned: auth, sync, multi-device — see below),
re-engage the full api-contract flow as the skill describes.

### Issue artifacts — markdown only, no orchestration-mcp

We have not deployed `orchestration-mcp`. The Construct skill's MCP
calls (`claim_issue`, `next_unblocked_for`, `complete_issue`) won't
work. Replacements:

| Skill assumes | Our reality |
|---|---|
| `claim_issue(N)` returns `{ ok: true }` | Edit the issue's frontmatter: `status: ready → status: in-progress`. Commit alone. |
| `next_unblocked_for(lane)` returns top result | Read `_index.md` for the feature. Pick a wave-0 issue in your lane whose `blocked-by` is empty. Coordinate with the team verbally / on Slack. |
| `complete_issue(N, pr_url)` after merge | Edit the issue's frontmatter: `status: in-progress → status: done`. Commit. |
| GitHub issue number `N` | We don't have one. See branch naming below. |

If we decide to deploy `orchestration-mcp` later, these manual steps
go away and the skill's MCP flow takes over unchanged.

### Branch + PR discipline (two-repo dance)

The skill assumes one repo. We have two. Before any `git` / `gh`
command, **`cd` into the correct subrepo**:

```bash
# substrate change
cd weft/
git checkout -b feat/<feature-slug>/<issue-slug> origin/main
# … work, commit, push, gh pr create
cd ..

# host-app change
cd undercurrent/
git checkout -b feat/<feature-slug>/<issue-slug> origin/main
# … work, commit, push, gh pr create
cd ..

# bump the workspace's submodule pointer once the PR merges
git add weft/        # or undercurrent/
git commit -m "Bump <weft|undercurrent> to <sha> — <feature-slug>"
git push
```

**Branch naming (override of skill's `feat/<N>`):** since we have no GH
issue number from the MCP, use
`feat/<feature-slug>/<issue-slug>` — e.g.
`feat/image-attachments/01-pick-from-gallery`. Slug must match the
issue's filename so the issue and branch are obviously linked.

**PR title** still the issue's one-line title verbatim.

**PR body:** drop `Closes #<N>` (no GH issue exists). Replace with:

```markdown
## Inception story
[issue file](../../undercurrent-workspace/inception/<feature>/issues/<lane>/<N>-<slug>.md)
```

If we deploy orchestration-mcp later, real GH issues get created and
the standard `Closes #<N>` flow comes back.

### Construct skill — what to ignore

- The Step 1.2 MCP-required check ("if `/mcp` shows no `inception` server,
  stop") — we know there isn't one. Skip it, proceed in markdown mode.
- All `claim_issue` / `next_unblocked_for` / `complete_issue` invocations
  — replace with markdown frontmatter edits as in the table above.

Everything else in the Construct skill — TDD loop, branch naming
shape, draft PR from task #1, per-task commits, lane verify before
ready, PR body template — applies as-is.

## When BE arrives

When the first backend-touching feature ships (likely **auth** — Sign
in with Apple / Google for cross-device persona + memory sync), do:

1. Decide whether BE lives in `weft/` (substrate-level, every host gets
   it), in `undercurrent/` (host-specific), or in a **new third
   submodule** added to this workspace.
2. Add that location to the lane table above (point `backend` lane at it).
3. Add a `backend/CLAUDE.md` to that repo describing build / test
   commands the Construct skill's lane-verify step needs.
4. Re-engage the skill's full api-contract flow — it's already there,
   just dormant.

Don't pre-build BE scaffolding before there's a feature that needs it.

## Workspace-level commands

```bash
# Pull latest of everything
git pull
git submodule update --recursive --remote

# Bump skills from upstream when ai-sdlc releases something useful
SRC=/path/to/simple-ai-sdlc/ai-sdlc/skills
rsync -av --delete "$SRC/inception/" .claude/skills/inception/
rsync -av --delete "$SRC/construct/"  .claude/skills/construct/
git add .claude/skills && git commit -m "Bump ai-sdlc skills"

# Start a feature
/inception
```

## What NOT to do

- Don't run `/inception` from inside a submodule. It's a
  workspace-level command — artifacts land in
  `undercurrent-workspace/inception/<feature>/`, not inside
  `weft/inception/` or `undercurrent/inception/`.
- Don't `gh pr create` from the workspace root — there's no upstream
  for the workspace's submodule pointers as a PR. PRs go to the
  subrepo's GitHub remote.
- Don't create per-platform stories for work that's actually
  commonMain. Use the `kmp-common` lane.
- Don't add a `backend/` issues dir for features that have no BE.
  Empty lane dirs confuse the `_index.md` wave layout.
- Don't edit `.claude/skills/` to fix project-specific friction —
  edit *this* file instead. Vendored skills stay pristine so `rsync`
  updates from upstream don't merge-conflict.
