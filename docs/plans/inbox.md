# Crow inbox

Ideas and unscheduled work go here. Move accepted milestone work to the [roadmap](../ROADMAP.md).

## Untriaged

- **Custom keybindings.** Let users set, see, and persist a key for a command, including Open Terminal Here. Decide on the config format and how to report conflicts across keymap layers.
- **Sidebar text filter.** Filter native and contributed sidebar sections. Plan matching, focus, cursor fallback, and Escape together. The existing `/` binding filters the file pane.
- **Design-board follow-ups.** Line icons per favorite, clock icons in Recent, syntax highlighting and line numbers in previews, 196 px columns, a stronger focused-column tint, and a scope token in the search field. See the [UI design overview](ui-overhaul/README.md).

## Triaged backlog

### Git decorations flash during refresh

**Medium priority · Fixed in [P5B PR #79](https://github.com/paraversal/file-crow/pull/79) · Pending bundled hand check.** Already decorated rows intermittently reverted to their default appearance for about 100 ms. For example, an ignored file flashed white before returning to blue.

- A repository rescan used to clear every cached Git decoration, even when the status matched. Folder watcher batches also evicted results for untouched siblings. Crow now retains ready decorations for unchanged entries, refreshes the affected worktree, and replaces old styling when the current worker reply arrives.
- In the bundle, open a subfolder with a visible ignored file and wait for its Git styling. Touch an already ignored file in a sibling folder in the same worktree, then touch a different file in the open folder. The first ignored row and Git status bar should stay steady. Remove its ignore rule and check that it loses the ignored styling. Disable Git and check that its remaining decorations clear immediately.

### App-wide UI scaling

**Medium priority · Unscheduled · 3–5 engineering days.** UI zoom would improve legibility and let users fit more on screen when they need it. The List view width fix addresses the immediate Preview layout issue, so this can wait behind scheduled M9 work.

- Scale text, icons, controls, and spacing across Crow windows, including Settings and sheets. Add ⌘+, ⌘−, and ⌘0 with matching View menu and command palette actions.
- Start with a one-day GPUI prototype covering a list with Preview open. Check row and pane layout, scroll position, and pointer hit targets before extending scaling to other views.
- Decide the zoom range and steps, whether zoom persists across launches, and how zoom combines with the existing font-size setting. Verify the shortcuts in both keymap layers, the layout at minimum window width, and performance on the 100k-folder benchmark.

### Deferred from the M9 audit (`fb3c58e`, triaged 2026-11-20)

- **Git cache lifetime, watchers, and Projects repo reuse (M9 audit #35, carried P1).** `crates/crow-app/src/git.rs` retains `folders`, `repos`, and `lines` maps for the entire enabled session, keeps up to 16 recursive watchers alive after all windows close, and re-opens repositories independently during Projects scans instead of reusing cached snapshots. Bound retention by active scope/LRU, stop watchers when no window uses a worktree, and share snapshots with Projects.
- **Search channel backpressure and off-main display names (M9 audit #36, carried P7).** `crow-search` sends individual name hits over an unbounded channel (`engine.rs:405, 517`) and `crow-app` calls `crow_macos::display_components` (`NSFileManager::displayNameAtPath`) synchronously on cache misses during render (`style.rs:245-261`, duplicated in `sidebar.rs:839-850`). Batch/bound search hit transport and move display-name resolution off the render thread with one shared cache.
- **Hidden-tab watchers, unfocused caret blink loops, and thumbnail queue scoping (M9 audit #37).** Hidden tabs keep `FolderWatcher` + 100 ms poll loops active (`folder.rs:351-361`), every live `TextInput` keeps a 500 ms blink loop alive while unfocused (`text_input.rs:646-689`), retained `SearchView` instances in history keep a 60 s timer (`search.rs:687-710`), and `thumbnails.rs` uses a single app-wide queue where one window's request can evict another window's queued jobs. Pause idle timers/watchers when hidden/blurred and scope thumbnail queues per window.
- **Structural splits and UI-free crate placement (M9 audit #38, carried P3–P6).** Split growing app files (`pane.rs` 6.4k lines, `settings_window.rs` 5.3k, `search.rs` 4.4k, `window.rs` 3.6k, `folder.rs` 3.1k), split `plugins/coordinator.rs` (`finish`, O(N) `RowCache::evict`, 7 `too_many_arguments` allows), split `crow-plugin/src/registry.rs::validate_registration`, unify `palette.rs::CommandId::key` (72 arms) with command metadata, and move UI-free logic (`coordinator.rs`, `icon_packs.rs` loader, `operations.rs` drop routing, `saved_query_counts.rs`, `tags.rs` index) into `crow-plugin` / `crow-core` / `crow-fs` / `crow-search`.
- **Remaining live-app freezes (M9 audit #47, partial in batch 5).** Batch 5 cut freezes over 250 ms in the live-fuzz replay (`crow --fuzz --seed 20261120 --tree small --until 6913`) from about 25 to 6. Leaf `displayNameAtPath` calls (#36 above) and creating the Settings window still block the main thread.
- **Remaining Settings UI for TOML-only keys (M9 audit #39).** Add Settings or contextual UI coverage for `views.toml` (`icon_source`, default `icon_size`/`icon_caption`/`columns`), `sidebar.toml` (`pinned_tags`, `[sections].order`, `[section].show`, `[projects].roots`), and `settings.toml` (`[window].sidebar_width`), plus switch first-launch `settings.toml` creation (`settings.rs:465`) to `write_atomic`.
