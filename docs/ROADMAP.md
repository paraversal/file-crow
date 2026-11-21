# Crow roadmap

Last updated 2026-11-21. The daily-driver MVP, Finder parity, search reliability and M9 (plugins, views and operations) milestones are complete. The next milestone has not been chosen.

The [inbox](plans/inbox.md) holds work awaiting triage, including custom keybindings and the deferred M9 audit items. Search will continue to walk the live filesystem. The [product spec](Crow%20%E2%80%94%20Product%20%26%20Technical%20Spec.md#mvp-scope) records the broader scope boundaries.

## Completed milestones

| Milestone | What shipped | Where |
| --- | --- | --- |
| M1–M4 · Foundation | App shell, fast folder listing, List and Columns, preview, and file operations with undo | [#1](https://github.com/paraversal/file-crow/pull/1), [#2](https://github.com/paraversal/file-crow/pull/2), [#3](https://github.com/paraversal/file-crow/pull/3), [#4](https://github.com/paraversal/file-crow/pull/4) |
| M5 · Sidebar and views | Favorites, per-folder view settings, and themes | [#5](https://github.com/paraversal/file-crow/pull/5), [#6](https://github.com/paraversal/file-crow/pull/6) |
| M6 · Search and navigation | Inline filter, command palette, filename and content search, and Spotlight scope | [#7](https://github.com/paraversal/file-crow/pull/7) |
| UI overhaul | Toolbar, menus, search, palette, sidebar, tips, motion, and sound | [Design overview and PRs](plans/ui-overhaul/README.md) |
| M6.5 · Settings | Live configuration from the Settings window | [#28](https://github.com/paraversal/file-crow/pull/28) |
| M7 · Hardening | Seeded fuzzing, edge trees, frame budgets, main-thread I/O cleanup, path editing, and accessibility | [Findings and replays](plans/done/m7/findings.md) |
| Housekeeping and context menus | App lifecycle, command-line launcher, menus, text bindings, and drag and drop | [#24](https://github.com/paraversal/file-crow/pull/24), [#25](https://github.com/paraversal/file-crow/pull/25), [#27](https://github.com/paraversal/file-crow/pull/27), [#36](https://github.com/paraversal/file-crow/pull/36) |

### M8 — Finder parity

| Piece | What shipped | PR |
| --- | --- | --- |
| F1 | Selection islands, delete focus, removal motion, and copy/cut feedback | [#48](https://github.com/paraversal/file-crow/pull/48), [#49](https://github.com/paraversal/file-crow/pull/49) |
| F2 | Visible type-to-select, configurable Return, and context menus | [#50](https://github.com/paraversal/file-crow/pull/50) |
| F3 | Info pane | [#53](https://github.com/paraversal/file-crow/pull/53) |
| F4 | Draggable path segments and discoverable path editing | [#52](https://github.com/paraversal/file-crow/pull/52) |
| F5 and F8 | Tips, first launch, and one focus target in the filter | [#51](https://github.com/paraversal/file-crow/pull/51) |
| F6 | Folder sizes | [#54](https://github.com/paraversal/file-crow/pull/54) |
| F7 | Open Terminal Here, new-item placement, window drag, and Empty Trash | [#55](https://github.com/paraversal/file-crow/pull/55) |

### M8.5 — Search reliability

| Batch | What shipped | PR |
| --- | --- | --- |
| B1 | Batched streaming, name hits before content reads, typed skips, and search tracing | [#56](https://github.com/paraversal/file-crow/pull/56) |
| B2 | Hidden-name matching and current-folder files in Cmd-K | [#57](https://github.com/paraversal/file-crow/pull/57) |
| B3 | Search Home, explicit Folder scope, and ways to widen a search | [#58](https://github.com/paraversal/file-crow/pull/58) |
| B4 | Separate result budgets and stable selection while results arrive | [#59](https://github.com/paraversal/file-crow/pull/59) |
| B5 | Regression corpus, randomized search oracle, and fuzz coverage | [#60](https://github.com/paraversal/file-crow/pull/60) |

### M9 — Plugins, views and operations

[Index and verification](plans/done/m9/README.md) · [Code audit](m9-code-audit.md)

| Area | What shipped | PRs |
| --- | --- | --- |
| Plugins | Versioned Rhai packages with permissions and budgeted workers, app extension surfaces, and [plugin author docs](plugins.md) | [#61](https://github.com/paraversal/file-crow/pull/61), [#66](https://github.com/paraversal/file-crow/pull/66) |
| Git and Projects | Read-only Git status and lines changed, Projects under chosen roots, review and diff, pinned repository status | [#67](https://github.com/paraversal/file-crow/pull/67), [#68](https://github.com/paraversal/file-crow/pull/68) |
| iCloud | Download status and metadata-only browsing of placeholders | [#65](https://github.com/paraversal/file-crow/pull/65) |
| Settings | Category sidebar, settings search, shared configuration errors, view and preview defaults | [#73](https://github.com/paraversal/file-crow/pull/73) |
| Views and icons | Data-only icon packs, icon grid with rectangle selection, Gallery, List fit beside Preview, navigation motion | [#71](https://github.com/paraversal/file-crow/pull/71), [#74](https://github.com/paraversal/file-crow/pull/74), [#75](https://github.com/paraversal/file-crow/pull/75), [#72](https://github.com/paraversal/file-crow/pull/72), [#69](https://github.com/paraversal/file-crow/pull/69) |
| Search and organization | Saved queries as live destinations, Finder tags in the sidebar, filter and queries | [#76](https://github.com/paraversal/file-crow/pull/76), [#77](https://github.com/paraversal/file-crow/pull/77) |
| File operations | Resumable copy jobs, app-wide copy queue with pause, resume and conflicts, batch rename with preview and one undo | [#78](https://github.com/paraversal/file-crow/pull/78), [#79](https://github.com/paraversal/file-crow/pull/79), [#80](https://github.com/paraversal/file-crow/pull/80) |
| Paths and drops | Explorer-style path bar, drops from other apps and spring-loaded folders | [#81](https://github.com/paraversal/file-crow/pull/81), [#82](https://github.com/paraversal/file-crow/pull/82) |
| Test tooling | Owned temp fixtures, cross-run test and fuzz reports, module extraction | [#62](https://github.com/paraversal/file-crow/pull/62), [#63](https://github.com/paraversal/file-crow/pull/63), [#64](https://github.com/paraversal/file-crow/pull/64) |
| Audit and fixes | Through-M9 audit, seven fix batches and the saved-rename fix | [#83](https://github.com/paraversal/file-crow/pull/83)–[#87](https://github.com/paraversal/file-crow/pull/87), [#89](https://github.com/paraversal/file-crow/pull/89)–[#92](https://github.com/paraversal/file-crow/pull/92) |

Not planned: Solarized Light contrast in the shared theme index (P9E); Crow uses the theme index as it is.

Update the relevant roadmap and record the PR when a piece merges. Keep implementation steps and progress logs in the local planning files described in [AGENTS.md](../AGENTS.md).
