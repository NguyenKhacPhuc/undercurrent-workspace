# Undercurrent Workspace

One-clone bootstrap for the Undercurrent app and its substrate. Two git
submodules:

| Path | Repo | What it is |
|---|---|---|
| [`weft/`](weft/) | [NguyenKhacPhuc/android-harness](https://github.com/NguyenKhacPhuc/android-harness) | KMP SDK for building LLM-orchestrated apps (agent runtime, tools, compose components, OAuth, MCP). |
| [`undercurrent/`](undercurrent/) | [NguyenKhacPhuc/undercurrent](https://github.com/NguyenKhacPhuc/undercurrent) | Reference host app — KMP Android + iOS. Personal assistant with streaming chat, custom personas, OAuth integrations. |

## Clone

```bash
git clone --recurse-submodules https://github.com/NguyenKhacPhuc/undercurrent-workspace.git
cd undercurrent-workspace
```

If you forgot `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Why two repos behind one workspace

`undercurrent` consumes `weft` via Gradle composite-build
(`includeBuild("../weft")`) — the on-disk layout must be `weft/` and
`undercurrent/` as siblings. The workspace just locks that layout in
and pins each subrepo to a known-good commit.

The two repos stay independent — each has its own history, issues, and
PRs. The workspace only records *which commits* belong together.

## Build

All Gradle work happens inside `undercurrent/`:

```bash
cd undercurrent

# Android debug APK
./gradlew :androidApp:assembleDebug

# Cross-platform compile check (catches Android-only APIs leaking
# into commonMain)
./gradlew :androidApp:compileDebugKotlin
./gradlew :composeApp:compileKotlinIosSimulatorArm64

# Tests
./gradlew test

# Coverage
./gradlew koverHtmlReport   # build/reports/kover/html/index.html
```

For iOS, open `undercurrent/iosApp/iosApp.xcodeproj` in Xcode.

## Pull latest

```bash
git pull
git submodule update --recursive --remote   # bump submodules to their latest main
```

To bump only one submodule:

```bash
cd weft && git pull origin main && cd ..
git add weft && git commit -m "Bump weft submodule"
```

## Working in the submodules

Each submodule is a normal git repo. You can `cd weft` or `cd
undercurrent` and commit / push as usual. The workspace's pointer
only moves when you run `git add <submodule-path>` in the workspace
root.

```bash
cd undercurrent
# make changes, commit, push to NguyenKhacPhuc/undercurrent
git push

cd ..
git add undercurrent             # update workspace's pinned commit
git commit -m "Bump undercurrent to <sha>"
git push
```

## Project docs

- `weft/docs/architecture-vision.md` — the SDK ↔ app split
- `undercurrent/CLAUDE.md` — host-app architecture, MVI conventions, UI rules
- `weft/CLAUDE.md` — substrate module layout, tool-authoring rules
