# AGENTS.md — Crow

Crow is a Finder-style file explorer for macOS in Rust on GPUI (vendored Zed, `vendor/`, fetched by `scripts/fetch-gpui.sh`). Read `README.md` (layout, config, keys) and `docs/testing.md` (tests, bench, hand checks).

## Working on a plan
Shared milestone status lives in [`docs/ROADMAP.md`](docs/ROADMAP.md) and the [M9 index](docs/plans/done/m9/README.md). Keep detailed implementation plans, fix lists, and progress logs in the ignored paths under `docs/plans/`. Implement them with the personal `execute-plan` skill. Keep the `<plan>.progress.md` beside its plan current after every step, and **ask the user about anything unclear or undecided instead of choosing yourself.** Give the critic the plan and progress paths at hand-off. From a separate worktree, use the paths in the checkout that holds the local files.
When the work is done, the executing agent commits locally and stops at "Ready for critique"; a second agent reviews the branch with the personal `critique` skill, fixes what is clearly wrong, and opens the PR.
After merge, record the outcome and PR in the shared roadmap and move the local plan and progress file to `docs/plans/done/`. Keep untriaged work in the [inbox](docs/plans/inbox.md) and milestone-level decisions in the current roadmap.
Both skills are generic. The Crow specifics are in the next section and win where the two differ.

## Plans and critique in Crow
**Hand-off.** Progress file states: in progress → **Ready for critique** → In critique → PR open. Even when a plan's Finish says "push", the executing agent commits locally and stops at Ready for critique without pushing; the critic opens the PR, so the PR the user hand-tests is the reviewed one. Hand checks are steps in the app built by `./scripts/bundle.sh`.

**Checks.** The Before every commit checks below, plus the bench when the change touches listing, scrolling, first paint or anything in a 100k folder (compare with `main`, not with the agent's numbers).

**Fuzzer budget.** Long fuzz runs make turns many times slower, and the user waits for them. `cargo test --workspace` already runs `fuzz_smoke`, which is the per-item fuzz check.
- Anything over ~30 s of fuzzing needs the user's OK first: `fuzz_long`, `crow --fuzz`, `CROW_FUZZ_MINUTES` > 0.5, big `CROW_FUZZ_STEPS`. Ask in one line with the command, the expected wall time and what it tells you that the smoke and a replay don't.
- Exception: a run the plan names (e.g. a "Finish (per batch)" step: "`fuzz_long` 10 minutes"). The executing agent does it once, at that point, as written, with `run_in_background`. The critic checks that the progress file records it (seeds, minutes, result) and asks before re-running it.
- A failure shrinks for up to `CROW_FUZZ_SHRINK_SECS` (default 60). Set it to 0 when replaying a known seed; raise it only with the user's OK.
- Checking a finding's fix means replaying its seed and steps (`findings.md` has the command) plus a regression test, not a fresh long run.

**Bugs.** When fixing a cause, look for the same pattern in the other views, rows and inputs built the same way (M5-fix fixed the `drag_over` flicker in the sidebar, but list and column rows needed a second plan). Spike platform, AppKit or GPUI behavior nobody has seen yet (menus, focus, drag, IME) before building on it. Workarounds that need a check-in include new `patches/gpui-*.patch` changes.

**Critique: product intent.** Judge "does it make sense" against the ranked **Principles** in `docs/Crow — Product & Technical Spec.md` (the higher one wins), the plan's Target images and, for UI plans, `docs/plans/ui-overhaul/README.md` and its `img/` boards. The principles critiques cite most: 3 Predictable (nothing jumps), 4 Legible, 5 Finder conventions first with Vim motions on top, and every feature reachable by key, menu and palette; 6 no dependency doing what an existing one does; 9 show, don't tell (no captions, helper labels or "Tip:" copy unless the plan asks).

**Critique: use the app.** Build the branch with `scripts/bundle.sh --open` and `main` the same way in a separate worktree, then compare them on the same folder. Check light and dark themes, a 640 px window and a wide one, an empty folder and `target/fixture-100k`, the Vim layer on and off, and VoiceOver (Cmd-F5) for new roles or labels. Walk the touched flows by mouse, then by keyboard only. Also walk the neighbors: the same key in other focus contexts, and the same row style in list, column and search results. When the app can't be driven, read rendered bounds in a throwaway GPUI test and list what you couldn't see as hand checks.

**Critique: Crow-specific code checks.** The Rules below, item by item. Edge cases: empty and unreadable folders, symlinks, very long names, non-UTF-8 and combining characters, a folder deleted while it's shown. Search `chrome`, `style`, `keys`, `icons`, `crow_core`, `crow_fs` and `crow_search` before accepting new helpers. UI-free logic belongs in `crow-core`/`crow-fs` with unit tests, and AppKit only in `crow-macos`. Consider whether a growing `window.rs` should split. Interaction states must not shift the layout (U1's `cursor_ring` is an inset shadow for this reason).

**Critique: finish.** Commit fixes as `<plan id> critique: <what>`. The PR body lists each item with its approach and how it was verified, plus a Critique section (fixed / user decided / noted, with literal and semantic findings listed separately). End the report with the next piece from the milestone roadmap and the exact request for the executing agent ("execute plan P3A"). If the plan feeds a fix list (a milestone's `findings.md`, e.g. `docs/plans/done/m7/findings.md`), count the open and "must fix" findings and say whether the fix batch comes before the next piece.

## Crates
`crow-app` (GPUI binary: windows, views, actions, keymaps) · `crow-core` (UI-free models: settings, views, sidebar, recent, tabs, selection; pure Rust, unit-tested) · `crow-fs` (listing, sort, watch, file-operation engine with undo) · `crow-macos` (the only AppKit/Foundation code) · `crow-search` (search engine) · `crow-plugin` (Rhai package runtime) · `crow-plugin-git` (read-only Git integration) · `crow-theme` (theme index reader).

## Rules
- No disk I/O on the main thread (use `background_spawn`, then update on the main thread). Startup loads are the exception.
- Every file operation goes through `crow_fs::ops`. Nothing is ever overwritten in place; trash by default.
- Every new keyboard behavior gets a GPUI keystroke test in `crates/crow-app/src/window.rs` (`simulate_keystrokes`; tests use `ui::init_for_tests`, which gives each test a temp config dir, never the real `~/.config/crow`).
- Every interactive element has a role and an aria label.
- Config files (`~/.config/crow/*.toml`): documented default written on first launch, live reload, errors in the status line, Crow's own edits through `crow_core::edit_toml` so user comments survive.
- Style with theme roles only (`theme.role("fg")` etc.), never raw palette slots except for named colors.
- Performance budgets (100k folder): warm first paint < 50 ms, scroll p99 < 1.5 × frame time. Bench: `cargo run --release -p crow-app -- --bench target/fixture-100k` (fixture: `--make-fixture target/fixture-100k 100000`). Bench numbers vary by a few ms run to run; compare with `main` before calling something a regression.

## Before every commit
`cargo fmt --check` · `cargo clippy --workspace --all-targets -- -D warnings` · `cargo test --workspace`. CI runs the same on macOS.

## Git
One branch and PR per milestone or fix list; squash-merged to `main` by the user after CI is green and they've hand-tested. Don't merge yourself. `./scripts/bundle.sh` builds `target/bundle/Crow.app` for hand tests.

## Ignore
`vendor/zed/CLAUDE.md` and other files under `vendor/` describe Zed's own repo. Their rules (e.g. editing README banners, PR title formats) don't apply to Crow.

<!-- OPENWIKI:START -->

## OpenWiki

This repository has a generated `openwiki/` evidence index. It is optional just-in-time context, not required startup reading.

- Do not enumerate, preload, or search wikis at task start. Use retrieval when the user asks for it, when unfamiliar architecture or dependency behavior materially affects the task, or when source inspection leaves an important uncertainty. Stop once the question is grounded.
- When those conditions apply and OpenWiki retrieval tools are available, use `openwiki_search` for just-in-time context and `openwiki_read` for the relevant complete sections. If search returns `workspace_required`, ask which listed workspace to use and retry with its ID.
- Use `openwiki_list_workspaces` or `openwiki_list_wikis` when workspace membership itself needs to be discovered.
- If the retrieval tools are unavailable, read `openwiki/quickstart.md` and follow its links to the relevant pages.
- Treat source code and tests as authoritative. A brief's unknowns and review items are verification gaps, not automatic requirements.
- Prefer the narrowest quiet validation that proves the changed behavior. Preserve complete failure output.

Use the repository's documented maintenance process to refresh generated pages after source changes. Check its configuration and completed runs before relying on automatic updates. Do not hand-edit generated OpenWiki pages unless explicitly asked; prefer updating source code/docs and letting OpenWiki regenerate.

<!-- OPENWIKI:END -->
