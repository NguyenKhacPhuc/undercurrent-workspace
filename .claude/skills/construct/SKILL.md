---
name: construct
description: Run the AI-SDLC Construction phase. Pick up an Inception story from the inception MCP, decompose it into tech tasks for the lane's actual stack, and implement it via TDD (red → green → refactor) with one PR per story. Triggers on /construct, "implement the next issue", "claim and build #N", "let's start coding the OCR story".
---

# Construction phase

You are a dev's AI partner during the **Construction phase** of AI-SDLC. Your job is to take **one Inception story** and turn it into a merged PR via TDD, while honoring the lane's existing stack and conventions.

## Operating model

> Inception said WHAT. Construction says HOW. The driver supervises but you own the keyboard.

You are stack-aware: you read the lane's `CLAUDE.md` (or scout the actual code) to learn the real conventions before writing a line. You don't invent libraries — if the story seems to need one, you stop and ask the driver, just like Inception would have.

You also default to **TDD**: write the failing test first whenever the slice lends itself to it; for slices that genuinely don't (e.g. UI scaffolding with no behavior yet), document why and add tests *before* opening the PR.

## Project context (Undercurrent — Android + iOS KMP)

This is a forked copy of the upstream Construct skill, adapted for the
Undercurrent workspace. Key adaptations:

- **No orchestration MCP** — issue state lives in markdown frontmatter
  (`status: ready → in-progress → done`). All MCP calls in this doc
  (`claim_issue`, `next_unblocked_for`, `complete_issue`) are replaced
  with frontmatter edits.
- **Two submodules** — `weft/` (substrate) + `undercurrent/` (host app).
  Always `cd <subrepo>` before any `git`, `gh`, or `./gradlew` command.
  PRs live in the subrepo, not the workspace.
- **Lanes:** `kmp-common`, `android`, `ios`, `substrate`, `backend`
  (backend dormant). Lane → CLAUDE.md mapping is in
  `undercurrent-workspace/CLAUDE.md`.
- **Testing convention is BDD with kotest** — see Step 4 for the
  per-lane test commands and test-shape rules.

## Inputs

Before starting, resolve:

1. **The story.** The driver names a story (`/construct <feature-slug>/<issue-slug>`)
   or you read the feature's `_index.md` and pick the top wave-0 issue
   in the driver's lane whose `blocked-by` is empty. If lane isn't
   specified, ask.
2. **The lane's `CLAUDE.md`.** Per
   `undercurrent-workspace/CLAUDE.md`'s lane table:
   - `android` / `ios` / `kmp-common` → `undercurrent/CLAUDE.md`
   - `substrate` → `weft/CLAUDE.md`

   If absent, scout the actual code and propose creating one as a
   sub-task of this Construction.
3. **The story body + the api-contract section it references (if any)
   + the PRD section.** Read them from the markdown files at
   `undercurrent-workspace/inception/<feature>/`. The acceptance
   criteria are the contract — any "Implementation steps" or
   "Files to touch" inside the story are non-binding hints.

## Process

### Step 1 — Claim the issue (markdown frontmatter)

Open the issue file at
`undercurrent-workspace/inception/<feature>/issues/<lane>/<NN>-<slug>.md`.
Check its frontmatter:

- `status: ready` and `blocked-by: []` (or all blockers have
  `status: done` in their respective files) → continue.
- `status: in-progress` → another dev has it. Stop, tell the driver,
  ask whether to pick a different one.
- `status: done` → already finished. Stop and surface.
- `blocked-by` has open entries → wave model drifted. Stop and surface.

Edit the frontmatter: `status: ready → status: in-progress`. Add a
`claimed-by: <your GH login>` field. Commit alone:

```bash
cd undercurrent-workspace
git add inception/<feature>/issues/<lane>/<NN>-<slug>.md
git commit -m "Claim <feature>/<issue>"
git push
```

This is **not** atomic — coordinate with the team verbally if more
than one person is in the same lane. When we deploy
`orchestration-mcp` later, this step gets a real atomic-claim call.

### Step 2 — Branch (inside the relevant subrepo)

The branch lives in the subrepo whose code you're touching, not in
the workspace.

**Pick the subrepo by lane:**

| Lane | Subrepo |
|---|---|
| `kmp-common`, `android`, `ios` | `undercurrent/` |
| `substrate` | `weft/` |
| `backend` | (dormant — TBD) |

```bash
cd <subrepo>          # undercurrent/ or weft/
git fetch origin
git checkout -b feat/<feature-slug>/<NN-issue-slug> origin/main
```

**Branch naming convention** (override of upstream `feat/<N>`):
`feat/<feature-slug>/<NN-issue-slug>` — e.g.
`feat/pin-conversation/01-domain-field`. The `NN` prefix and slug
match the issue's filename so branch ↔ issue link is obvious.

If a story genuinely touches both subrepos (rare — usually means a new
substrate capability), open **two branches**, one per repo, and two
draft PRs. Coordinate the order so the dependent PR isn't merged
first.

Never branch from another feature branch — always from `origin/main`.

### Step 3 — Decompose into tech tasks (in TodoWrite, not files)

Read the story acceptance criteria, the api-contract section, and the lane's CLAUDE.md. Break the story into **tech tasks** sized for one TDD loop each (~5–20 min of focused work). Examples:

- "Add `OcrResponseDto` with kotlinx.serialization annotations + JSON round-trip test"
- "Add `OcrApi` class with single multipart `ocr()` method — test with MockEngine"
- "Register `OcrApi` in Koin AppModule + integration test that `get<OcrApi>()` resolves"

Use `TodoWrite` to track these — they are scratch state, not artifacts. Do not commit a "tasks.md" or "plan.md" to the repo.

**Show the breakdown to the driver** before starting Task 1. The driver may merge, split, or reorder. Don't proceed to code until they ack.

### Step 4 — TDD loop, one task at a time (draft PR open from task 1)

For each task in order:

1. **Red.** Write the smallest test that asserts the new behavior. Run it. **Confirm it fails for the *right* reason** (not a typo, not a missing import). If the test passes immediately, the test is wrong — fix it before continuing.
2. **Green.** Write the minimum code to pass. No bonus features. No extracted abstractions yet.
3. **Refactor.** Tighten naming, remove duplication, extract helpers — keep tests green throughout. Run tests after each refactor.
4. **Commit.** One commit per task is the default. Conventional message:
   ```
   feat(<lane>): <one-line task description>
   ```
   Example: `feat(kmp-common): ConversationSummary.pinnedAtMs sort order`.
   Drop the `(#N)` suffix — the branch name (`feat/<feature>/<issue>`)
   already carries the reference; we don't have GH issue numbers
   without orchestration-mcp.
5. **Push + sync the PR.**
   - **After task #1 only:** `git push -u origin feat/<feature>/<issue>` → `gh pr create --draft --title "<story title>" --body "<rendered PR body>"`. Body uses the Step 6 template, full task checklist populated, task #1 ticked, status `🚧 Draft — task 2 of <N> next.`
   - **After tasks #2..N:** `git push` → `gh pr edit <number-or-branch> --body "<re-rendered PR body>"` with the next checkbox ticked.
   - Idempotency: if `gh pr create` fails because a PR already exists, fall back to `gh pr edit`.
6. Update the matching TodoWrite item to `completed` and move on.

> [!info] **TDD strictness rule** (this project's policy):
> Test-first is preferred. **Tests must exist before the PR is marked ready (`gh pr ready`).** A task that genuinely cannot be tested in isolation (UI scaffolding with no behavior, type-only refactors, file moves) skips the red step but documents *why* in the commit message and adds an integration test in a later task that exercises it.

> [!tip] **Why draft-from-task-1**
> Two payoffs: (a) the driver and any reviewer can watch progress live in GitHub, with CI running on each commit; (b) if your session crashes mid-story, another agent can `gh pr view <branch>` and resume from the unticked task — no re-decomposition needed.

#### Test conventions (BDD with kotest)

The Red step uses **kotest's `BehaviorSpec`** with `Given` / `When` /
`Then` blocks (uppercase — avoids the `when` keyword backtick
collision). The convention plugin
(`KmpLibraryConventionPlugin`) auto-wires kotest engine + matchers +
Turbine + `kotlinx-coroutines-test` into every KMP library's
`commonTest`; MockK + kotest-runner-junit5 in `androidUnitTest`. No
per-module test deps to add.

**Where the test lives:**

| Lane | Test source set | Stack |
|---|---|---|
| `kmp-common` | `<module>/src/commonTest/kotlin/…` | kotest + hand-rolled fakes (KMP-portable, no MockK) |
| `android` (with collaborator-interaction needs) | `<module>/src/androidUnitTest/kotlin/…` | kotest + MockK (`coVerify(exactly = N)`) |
| `ios` (pure logic) | `<module>/src/commonTest/kotlin/…` if shared, else `iosTest/` | kotest + hand-rolled fakes |
| `substrate` | matches the substrate module's convention; check `weft/CLAUDE.md` | kotest + hand-rolled fakes preferred |

**The Red step's template:**

```kotlin
@OptIn(kotlinx.coroutines.ExperimentalCoroutinesApi::class)
class FooViewModelTest : BehaviorSpec({
    val mainDispatcher = StandardTestDispatcher()
    beforeTest { Dispatchers.setMain(mainDispatcher) }
    afterTest { Dispatchers.resetMain() }

    Given("a fresh VM wired to a fake repo") {
        When("Intent.X is dispatched") {
            Then("state.foo becomes 'bar'") {
                runTest {
                    val vm = FooViewModel(FakeFooRepository())
                    vm.dispatch(FooIntent.X)
                    advanceUntilIdle()                      // ← async dispatch
                    vm.state.value.foo shouldBe "bar"
                }
            }
        }
    }
})
```

**Always `runTest { advanceUntilIdle() }` before asserting state.**
`MviViewModel.dispatch` returns `Job` and is implemented as
`= launch { when { … } }` — every state mutation is async on the
test dispatcher. Asserting without `advanceUntilIdle()` is the most
common false-negative in this codebase.

**Hand-rolled fakes over MockK in commonTest.** MockK is JVM-only —
breaks iOS compilation. In `commonTest`, build small `FakeXxx`
classes that implement the repository interface and expose
`MutableStateFlow` slots + call-counters as needed. Reference
templates in this codebase:

- `feature/traces/src/commonTest/.../TracesViewModelStateTest.kt`
- `core/domain/src/commonTest/.../usecase/chat/FakeChatRepository.kt`

**Pure state-projection vs. collaborator-interaction tests** —
prefer to split into two specs when both styles apply:

- `<Name>ViewModelStateTest.kt` (commonTest) — state-projection
  assertions only (`vm.state.value shouldBe …`, Turbine on flows).
  Hand-rolled fakes.
- `<Name>ViewModelTest.kt` (androidUnitTest) — `coVerify` that the
  right collaborator method was called with the right args. MockK.

The split is by *what an assertion checks*, not by *what the test
exercises*. Don't duplicate state assertions in androidUnitTest if
the commonTest sibling already covers them.

**For Compose UI tasks** (`@Composable` scaffolding, layout changes):
no behavioral test stack is wired today (no Compose UI Test,
Paparazzi, or screenshot tests yet). Skip the Red step, document
in the commit message: `feat(kmp-common): UI-only — Preview added,
no behavior test (snapshot stack not wired)`. The story's
`@Preview` Composable is the visible proof of work. When a UI
snapshot stack ships, retrofit tests for prior UI work as a
separate Inception story.

**For trivial follow-ups** (rename, move file, extract helper for
clarity) fold into the parent task's commit rather than spawning
a one-line task. The TDD overhead has to earn its keep.

### Step 5 — Verify the lane

Before opening the PR, run the **lane's standard checks**. Per
`undercurrent/CLAUDE.md` and `weft/CLAUDE.md`:

| Lane | Commands |
|---|---|
| `kmp-common` | `./gradlew :<module>:test :<module>:compileDebugKotlinAndroid :<module>:compileKotlinIosSimulatorArm64` (verifies both targets compile + tests pass) |
| `android` | `./gradlew :<module>:testDebugUnitTest :androidApp:compileDebugKotlin` |
| `ios` | `./gradlew :<module>:test :composeApp:compileKotlinIosSimulatorArm64` |
| `substrate` | per `weft/CLAUDE.md` — usually `./gradlew :<module>:build :<module>:detekt` |
| `backend` | (dormant) |

If a feature has changed both the host app and the substrate, run
the verify commands in **both** repos.

If a pre-existing failure unrelated to this story shows up
(e.g. the known iOS-test-runner discovery issue), surface it to the
driver — don't paper over it, don't expand the story to fix it. Park
it for a separate Inception story.

### Step 6 — Mark the PR ready

The PR has been open as a draft since task #1. Now flip it to reviewable.

1. Re-render the PR body one final time — task checklist all ticked, verification block filled with the actual commands you ran, status → `✅ Ready for review`.
2. `gh pr edit <N> --body "<final body>"`
3. `gh pr ready` (flips `--draft` → ready).

**PR body template** (used from task #1 onward — re-render and `gh pr edit` after each commit):

```markdown
## Inception story
[`<NN>-<issue-slug>.md`](https://github.com/NguyenKhacPhuc/undercurrent-workspace/blob/main/inception/<feature>/issues/<lane>/<NN>-<issue-slug>.md)

## Tech tasks (TDD)
- [x] <task 1 — done>
- [x] <task 2 — done>
- [ ] <task 3 — in progress>
- [ ] <task 4 — pending>

## Summary
<2–4 bullet points of what this PR does at the behavioral level — not "added class X". Talk in user / API terms. Empty bullet point ok during draft; fill before marking ready.>

## TDD trail
<n> commits, each: failing test → impl → green. See commits for the loop.

## Verification
- [ ] <lane> tests pass locally: `<exact command>`
- [ ] <lane> build green: `<exact command>`
- [ ] (Reviewer) Manual smoke per story acceptance criteria

## Status
🚧 Draft — task 3 of 4 next.
<!-- Flip to "✅ Ready for review" before `gh pr ready` -->

## Notes
<Anything the reviewer should know — gotchas, deferred work surfaced, deps proposed.>
```

No `Closes #<N>` — we don't have GH issue numbers without
orchestration-mcp. The "Inception story" link points at the markdown
issue file in the workspace repo. After merge, you'll flip the
issue's frontmatter to `status: done` in Step 7.

### Step 7 — Hand off

After the PR is open, print a hand-off line for the driver:

```
PR opened: <url>
Story:     <feature-slug>/<NN-issue-slug> "<title>"
Lane:      <lane>
Repo:      undercurrent | weft
Status:    status/in-progress (frontmatter)

Once merged, run /construct complete <feature>/<NN-issue> <pr-url>
to flip the issue file's frontmatter to status: done and bump the
workspace's submodule pointer.
```

**Do NOT mark the issue done before the PR is actually merged.**
Downstream issues whose `blocked-by` references this one would
incorrectly unblock.

**On merge**, the driver invokes `/construct complete <feature>/<NN-issue> <pr-url>`. You then:

1. Edit the issue frontmatter:
   `status: in-progress → status: done`
   Add `merged-pr: <pr-url>` and `merged-at: <ISO timestamp>`.
2. Commit alone in the workspace repo:
   ```bash
   cd undercurrent-workspace
   git add inception/<feature>/issues/<lane>/<NN>-<slug>.md
   git commit -m "Done: <feature>/<NN-issue> — <pr-url>"
   ```
3. **Bump the submodule pointer** so future clones see the merged
   commit. From the workspace root:
   ```bash
   git add <subrepo>                 # weft/ or undercurrent/
   git commit -m "Bump <subrepo> to <sha> — <feature>/<NN-issue>"
   git push
   ```
4. Stop.

## Cross-cutting

### When the story seems to need a new dependency

You hit a moment where the cleanest implementation requires a library not already in the dep manifest. **Stop the TDD loop.** Ask the driver:

> "This task wants `<library>` for `<purpose>`. Alternatives: (a) use existing `<X>`, trade-off `<…>`; (b) write it ourselves, trade-off `<…>`. Which?"

If the driver picks the new dep, log it in `decisions.md` for the feature (open the file from the inception folder, append a `D<n>` entry), add the dependency in the manifest as **its own commit** with message `chore(<lane>): add <library> for <feature>/<issue>`, then resume TDD. If the driver is unsure, append to the feature's `open-questions.md` and pause this story.

### When the story body is wrong

Inception isn't infallible. You may discover during TDD that:

- The acceptance criterion is unachievable as written
- The api-contract diverges from what the BE actually does
- The story is twice the size Inception thought

Surface to the driver immediately. Two options:

1. **Soft fix:** edit the story body in place in the workspace repo, commit + push to `inception/<feature>/issues/<lane>/...md`, mention it in the PR.
2. **Hard fix:** flip the issue's frontmatter back to `status: ready` (releasing your claim) and recommend re-running `/inception` for this slice.

Don't silently re-interpret the story to make it fit what you built.

### When two devs (sessions) are in the same lane

There's no atomic claim today (no orchestration-mcp). Coordinate
verbally / on Slack: agree on who takes which wave-0 issue, then
edit each issue's frontmatter `status: in-progress` + `claimed-by`
before opening the branch. `git pull` in the workspace before
claiming so you see the latest `status` values. When the team grows
past 2 in the same lane, deploy `orchestration-mcp` for atomic
claims.

## Definition of done (for one story)

- [ ] Branch `feat/<feature>/<NN-issue>` exists in the relevant
      subrepo, pushed to origin
- [ ] Tests for every behavior the story asserts exist and pass locally
      (or commit message documents the no-test exception per the soft-TDD rule)
- [ ] Lane's standard build + test commands (Step 5 table) pass
- [ ] PR (was draft, now ready) with all task checkboxes ticked,
      no merge conflicts
- [ ] Driver notified with PR url
- [ ] **After merge:** issue frontmatter flipped to `status: done`
      in the workspace repo; submodule pointer bumped

## What you must NOT do

- Don't write code without a test (or a documented reason in the commit) — soft TDD applies.
- Don't introduce a library outside the dep manifest without asking.
- Don't expand the story to fix unrelated regressions — park them as new Inception stories.
- Don't skip the draft PR — task #1's commit is the trigger; without the draft PR, no one outside your session can see progress or resume on a crash.
- Don't mark the PR ready before all tech tasks are checked AND lane verify passes.
- Don't `gh pr ready` before the workspace repo's issue frontmatter
  + the PR body are consistent (story link present, all tasks ticked).
- Don't flip `status: done` before the PR is merged. Downstream
  `blocked-by` references would incorrectly unblock.
- Don't branch from another feature branch — always from `origin/main` of the relevant subrepo.
- Don't run `gh` from the workspace root — there's no PR there.
  Always `cd <subrepo>` first.

## Templates

- Branch: `feat/<feature-slug>/<NN-issue-slug>` in the relevant subrepo (mandatory)
- Commit: `feat(<lane>): <one-line>` (default; `fix`, `chore`, `test`, `refactor`, `docs` for other intents). No `(#N)` suffix.
- PR title: the story's one-line title verbatim
- PR body: see Step 6
