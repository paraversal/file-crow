# M9 — Plugins, views, and operations (index)

M9 is done. Status and PRs also live in the [roadmap](../../../ROADMAP.md#m9--plugins-views-and-operations); the through-M9 [code audit](../../../m9-code-audit.md) records the performance baseline and findings. Plans and progress logs stay local under `docs/plans/done/m9/`.

## Shipped

| Piece | Delivered | PR |
| --- | --- | --- |
| P1A | Versioned Rhai packages, permissions, registration, and budgeted workers | [#61](https://github.com/paraversal/file-crow/pull/61) |
| P9A | Test and fuzz fixture ownership and cleanup | [#62](https://github.com/paraversal/file-crow/pull/62) |
| P9B | Cross-run test and fuzz reporting with a baseline | [#63](https://github.com/paraversal/file-crow/pull/63) |
| P9C | Palette and window module extraction | [#64](https://github.com/paraversal/file-crow/pull/64) |
| P8 | Read-only iCloud download status and metadata-only browsing for placeholders | [#65](https://github.com/paraversal/file-crow/pull/65) |
| P1B | App extension surfaces and [plugin author documentation](../../../plugins.md) | [#66](https://github.com/paraversal/file-crow/pull/66) |
| P2 | Read-only Git status, lines changed, and Projects with user-selected roots | [#67](https://github.com/paraversal/file-crow/pull/67) |
| P2 follow-up | Git review and diff, pinned repository status, and resizable side panes | [#68](https://github.com/paraversal/file-crow/pull/68) |
| Navigation follow-up | List and Columns transitions, reveal feedback, persistent sidebar width, and bulk selection scaling tests | [#69](https://github.com/paraversal/file-crow/pull/69) |
| Settings overhaul | Category sidebar, settings search, shared configuration errors, view and preview defaults | [#73](https://github.com/paraversal/file-crow/pull/73) |
| List fit | List fits beside Preview | [#72](https://github.com/paraversal/file-crow/pull/72) |
| P3A | Data-only icon packs | [#71](https://github.com/paraversal/file-crow/pull/71) |
| P3B | Sorted icon grid, marquee and Cmd-drag selection | [#74](https://github.com/paraversal/file-crow/pull/74) |
| P3C | Gallery view | [#75](https://github.com/paraversal/file-crow/pull/75) |
| P4A | Saved queries as live destinations | [#76](https://github.com/paraversal/file-crow/pull/76) |
| P4B | Finder tags in the sidebar, filter and saved queries | [#77](https://github.com/paraversal/file-crow/pull/77) |
| P5A | Resumable copy jobs and a conflict protocol | [#78](https://github.com/paraversal/file-crow/pull/78) |
| P5B | App-wide copy queue in Activity | [#79](https://github.com/paraversal/file-crow/pull/79) |
| P6 | Batch rename with preview and one undo | [#80](https://github.com/paraversal/file-crow/pull/80) |
| P9D | Explorer-style path bar setting | [#81](https://github.com/paraversal/file-crow/pull/81) |
| P7 | Native drop routing and spring-loaded folders | [#82](https://github.com/paraversal/file-crow/pull/82) |
| Audit | Through-M9 code audit report, size index and backlog | [#83](https://github.com/paraversal/file-crow/pull/83) |
| Audit fixes 1–7 | Drag routing, Git gating and Empty Trash, config safety, selection parity, live freezes, polish, fuzz coverage | [#84](https://github.com/paraversal/file-crow/pull/84)–[#87](https://github.com/paraversal/file-crow/pull/87), [#89](https://github.com/paraversal/file-crow/pull/89)–[#91](https://github.com/paraversal/file-crow/pull/91) |
| Finish fix | Saved query and tag results survive a batch rename and its Undo | [#92](https://github.com/paraversal/file-crow/pull/92) |

Not planned: P9E, Solarized Light contrast in the shared theme index (decided 2026-11-21).

## Finish verification (2026-11-21)

- **Checks at `5a626d0`:** workspace tests 1325 passed, 0 failed, 2 ignored (`bench_tag_filter_100k`, `fuzz_long`); fmt, clippy `-D warnings` and `check-test-temp.sh` clean.
- **Bench** (release, Apple M5 Pro, 120 Hz, FireBox, 1280×800, median of 3): warm first paint 6–13 ms (budget 50 ms); scroll p99 8.4–8.7 ms (budget 1.5 × 8.33 ms), for List, Filter, Icons and Gallery at 100k, Icons and Gallery with 300 images, and List with Git on. Search B4 on 200k: `needle` ≈ 1.6 s total, first hit ≈ 1 ms. A fresh build of the audit base gives the same warm first-paint range, so the difference from the audit's 3 ms is machine state.
- **Chains in the app:** plugin, Git and icon pack; grid selection, Gallery and file operations (cross-volume copy with conflicts and Undo); native drag and spring-loading; themes (FireBox, Solarized Light) at 1280 and ~640 pt; Vim layer; rapid tab and window changes (CPU 0.1–0.2 %, flat memory). The saved query, tag and batch rename chain found the regression fixed in #92.
- **Not run:** the batch-5 live fuzz replay (`crow --fuzz --seed 20261120 --tree small --until 6913`, minutes long, needs an OK).

## Hand checks left for the user

- Copy pause, resume and Cancel with a ≥ 10 GB file to a slow or external volume.
- Cmd-drag marquee in Icons with a real mouse, then Gallery and List keep the selection.
- Plain click inside a multi-selection (List and search results): Finder collapses to the clicked item on mouse-up.
- Spring cancel with a real mouse: leave the window and release over Finder; Crow should restore the original folder at once.
- iCloud: Remove Download on a file, then Icons, Gallery and Info show name, size and badge without downloading it.
- VoiceOver (Cmd-F5): Settings ▸ Plugins, conflict sheet, batch rename fields, Gallery rail, Activity controls.
- The "Git decorations flash during refresh" check in the [inbox](../../inbox.md).

The next milestone has not been chosen.
