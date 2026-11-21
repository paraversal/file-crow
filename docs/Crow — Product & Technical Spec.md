# Crow — Product & Technical Spec

Nov 12, 2026 · @paraversal

Crow is a macOS file explorer that keeps Finder's directness and fixes where it fails: it should be fast and correct, behave predictably, make every action easy to read, put the keyboard first and be fully yours to customize. This spec turns the feasibility study into decisions. It replaces that study's stack choice with Rust.

## Principles

Ranked. When two principles conflict, the higher one wins.

| # | Principle | Rule | Consequence |
| --- | --- | --- | --- |
| 1 | Correct | No operation loses data; every change can be undone or is clearly marked as permanent | Journaled operation engine; trash by default; copy to a temp file, then rename |
| 2 | Fast | No frame ever waits on the disk | All I/O on background tasks; the UI renders a snapshot and applies diffs |
| 3 | Predictable | Same input, same layout | One view-settings model; no free icon placement; fixed window geometry rules |
| 4 | Legible | Every action shows its target and its effect before it runs | Focused pane is always marked; live queries shown as text; batch operations preview their result |
| 5 | Keyboard-first, mouse-complete | Finder shortcuts work unchanged; Vim-style motions sit on top | Every command reachable by key, by menu and by command palette |
| 6 | Reuse before build | Use proven crates first; replace one only for a measured reason | ripgrep's libraries for walking and search, a Helix-derived matcher for fuzzy search |
| 7 | Extensible | Anything outside the core ships as a plugin on the public API | Git integration is the first plugin, not core code |
| 8 | Yours | Theme, icons, keymap and settings are files you own | Reads `~/.config/themes`; settings and keymap in TOML |
| 9 | Fun | Polish and motion where they help understanding | No captions or explanatory text in the UI: show, don't tell |

## Stack

Rust throughout, with GPUI (the UI framework behind the Zed editor) for the interface, decided after a 2–3 week spike. A fully custom-drawn UI makes theming total and layouts deterministic. GPUI also brings a keymap system with contexts and a Metal renderer built for large lists.

| Option | Theming | macOS integration | Maturity | Verdict |
| --- | --- | --- | --- | --- |
| **GPUI** | Every pixel from theme roles | Through `objc2`: Quick Look, icons, drag and drop | Pre-1.0, changing API, thin docs; proven at scale by Zed | Recommended, pending spike |
| Swift/AppKit shell + Rust core (UniFFI) | Partial: menus and scrollers resist custom colors | Best | Stable | Fallback if the spike fails |
| Iced, Slint, egui | Full | Weak: drag out to other apps and Quick Look need custom work | Usable | Not chosen |
| Tauri | Full, through CSS | Needs native bridges | Stable | Rejected: a web view works against "Fast" |

**Spike exit criteria.** All must pass before Phase 1:

- [x] 100k-row folder scrolls at display refresh rate, with live updates. 120 fps on a 120 Hz display (median frame 8.3 ms, 99th percentile 9.0 ms, worst 17 ms over 600 frames). A new file shows up about 0.5 s after it is created.
- [ ] Drag a file from Crow into Mail and into Finder. Built with a native drag session; needs a hand test.
- [ ] Open the native Quick Look panel from a GPUI window. Built (Space); needs a hand test.
- [x] Theme switch through the theme index, live, without restart. Gruvbox → Catppuccin Mocha → Solarized Dark while running; follows macOS light/dark.
- [ ] Keymap contexts: the same key does different things in the sidebar and in a pane. Built (Return opens in the sidebar, renames in the list); needs a hand test.
- [x] VoiceOver reads the file list. Passes on Zed's current GPUI: VoiceOver received each row as "crates, Folder, modified Today at 14:34", 1 of 7, with the selected row as the focused item. Fails on crates.io GPUI 0.2.2, which has no accessibility layer.

**Spike findings** (code in `spike/`, theme reader in `crates/crow-theme`):

- Rendering is not a concern: GPUI holds the display's refresh rate on 100,000 rows.
- Reading a 100k folder with metadata takes about 450 ms, and every change triggers a full rescan. Phase 1 needs streamed loading and diffs.
- Native icons work: Finder's icon for each file type, converted once and cached.
- Accessibility exists upstream: Zed merged AccessKit support into GPUI in May 2026 (Zed PR #56065), with macOS support and the row and set attributes a scrolling file list needs. It is not on crates.io yet, so Crow depends on a pinned Zed commit (fetched by scripts/fetch-gpui.sh). The spike now runs on it with unchanged results: 120 fps on 100k rows, live folder updates, live themes. VoiceOver passes.
- The crates.io release of GPUI (0.2.2) lags far behind Zed's own repository. Building it normally needs Apple's Metal Toolchain download; the `runtime_shaders` feature avoids that.

&#91;embedded content: proposed Cargo workspace · 7 crates + first plugin\]

Everything except `crow-macos` is portable Rust and testable without a window. `crow-app` also calls `crow-macos` directly for Quick Look and drag and drop. Compared with the feasibility study's AppKit plan, theming and keymaps get cheaper, while drag and drop and accessibility get more expensive.

## Reuse before build

Use ripgrep's and Helix's libraries in-process, not the `rg` and `fzf` binaries. They are the same engines, but results come back as Rust structs: no output parsing, no install dependency, and cancellation is just dropping a task. `fzf` cannot be embedded anyway; it is an interactive terminal program.

| Capability | Use | Why | Revisit when |
| --- | --- | --- | --- |
| Directory walking | `ignore` (ripgrep's walker) | Parallel; understands `.gitignore`, `.ignore` and override globs | Not expected |
| Content search | `grep-searcher` + `grep-regex` (ripgrep's libraries) | ripgrep speed, streamed matches with line numbers | Not expected; `rg --json` is a fallback if the API proves awkward |
| Fuzzy filtering | `nucleo` (from the Helix editor) | fzf-style scoring, incremental, multithreaded, handles millions of items | Not expected |
| Glob and pattern matching | `globset` (ripgrep) | Compiles hundreds of exclusion globs into one fast matcher | Not expected |
| Whole-disk filename search | Spotlight (`NSMetadataQuery`) | Already indexed, zero cost | Spotlight misses excluded or system folders often enough to need a Crow index |
| Watching folders | `notify` (FSEvents backend) | Cross-checked, debounced events | Event latency or coalescing is not good enough; then raw FSEvents |
| Copy | `copyfile(3)` with `COPYFILE_CLONE` | APFS clones, extended attributes, ACLs, Finder metadata | Not expected |
| Trash | `trash` crate, Finder method | Keeps Finder's "Put Back" working | Not expected |
| Git status (plugin) | `gix` (gitoxide) | Pure Rust, fast status | A feature `gix` lacks; then `git2` or the `git` CLI |
| macOS APIs | `objc2` + `objc2-app-kit` / `objc2-quick-look-ui` | Generated, maintained bindings for Quick Look, icons, drag and drop, UTTypes | Not expected |
| Settings and keymap | `serde` + `toml` | Human-editable, diffable in dotfiles | Not expected |

The one thing Crow builds itself is the glue: a query model that combines walking, exclusion rules, fuzzy and content matching into one streamed result set.

## Interaction model

One pane has focus at a time, and every keystroke acts on that pane only. This fixes Finder's column-view ambiguity, where typing sometimes searches the last column and sometimes the one before it.

**Focus and scope rules**

1. Exactly one pane is focused: a column, a list or one side of a split. Its selection is drawn in the accent color; selections in unfocused panes are drawn muted.
2. In column view, the focused column is the one holding the keyboard cursor. A column that only shows the children of a selection is never focused until you move into it with → or `l`.
3. A query lives in the pane it filters. It is drawn in that pane as literal text (`/ conf`), with a match count (`4 / 212`). Esc clears it.
4. The status bar always shows: focused path, active query, active exclusion rules, selection count and size.
5. A key sequence in progress (`g…`, `m…`) is shown in the status bar. After 400 ms a small list of what can follow appears.
6. Batch and destructive operations preview their result before running: renames as old → new, moves with conflicts listed.
7. Every operation lands in an activity log, and each entry can be undone.

**Keymap.** The Finder layer is always on. The Vim layer is off by default; switched on in settings, it turns plain letters into commands; Finder's type-to-select then moves to `/`. Vim-layer letters follow Yazi's conventions where they exist, so terminal users feel at home.

| Action | Finder layer | Vim layer (opt-in) |
| --- | --- | --- |
| Move cursor | Arrow keys | `h` `j` `k` `l`, counts (`5j`) |
| Open / enter folder | Cmd-↓ | `l`, Enter |
| Parent folder | Cmd-↑ | `h`, `-` |
| First / last item | Option-↑ / Option-↓ | `gg` / `G` |
| Filter focused pane | `/` | `/` |
| Jump to name | Type letters | `/` then Enter |
| Go to path | Cmd-Shift-G | `:cd`, `gp` |
| Command palette | Cmd-K | `:` |
| Quick Look | Space | Space |
| Rename | Return | `r` |
| Copy / cut / paste | Cmd-C / Cmd-X / Cmd-V | `y` / `x` / `p` |
| Move to Trash | Cmd-Delete | `d` |
| Delete permanently | Cmd-Option-Delete | `D` |
| Range select | Shift-arrows | `v` |
| Undo | Cmd-Z | `u` |
| Split pane / switch pane | Cmd-\\ / Ctrl-Tab | `Ctrl-w v` / `Ctrl-w w` |
| New tab / next tab | Cmd-T / Cmd-Shift-\] | `gt` |
| Toggle hidden files | Cmd-Shift-. | `.` |
| Add favorite / jump to it | Cmd-Ctrl-T / sidebar | `m` + letter / `'` + letter |
| Switch view | Cmd-1 to Cmd-4 | Cmd-1 to Cmd-4 |

Cut (Cmd-X) works on files in both layers; Finder only offers it as Cmd-Option-V. Vim marks double as sidebar favorites, so `ma` pins a folder to the sidebar under the letter `a`.

## Predictable views

A folder's view comes from three layers, resolved the same way every time: global defaults, then path rules, then an explicit folder override. Nothing is stored in `.DS_Store`, and nothing changes without a visible action.

| Layer | Example | Stored in | Visible as |
| --- | --- | --- | --- |
| Global default | List view, sort by name, folders first | `~/.config/crow/views.toml` | Settings |
| Path rule | `~/Downloads`: sort by date added, newest first; `**/src`: list view | Same file, in order; the first match wins | Rule name in the view menu |
| Folder override | This one folder in icon view | Same file, keyed by path | A dot in the path bar; one click resets it |

The view menu has a "Why this view?" entry that names the layer behind each setting.

**What each setting can vary by**

| Setting | Global | Path rule | Folder |
| --- | --- | --- | --- |
| View mode | Yes | Yes | Yes |
| Sort key and direction | Yes | Yes | Yes |
| Grouping (kind, date) | Yes | Yes | Yes |
| List columns | Yes | Yes | No |
| Icon and text size | Yes | No | No |
| Hidden files | Yes (one toggle, in the status bar) | No | No |

**Layout rules**

- Icon view is always a sorted grid. There is no free placement and no "Clean Up", so toggling hidden files only reflows the grid; it never leaves gaps.
- Sorting is stable, with natural order as the tie-breaker (`file2` before `file10`), the same in every view.
- New windows open at one default size, offset by 24 pt from the last one. Restoring a session restores exact sizes.
- Column view uses one column width, set globally; a single column widens to fit a long name, up to a cap.
- Renaming or adding a file never moves the cursor to another item.

## Exclusions

Exclusions are named rule sets, and each one applies only to the scopes you pick. Whenever a rule hides something, Crow says how much and why, and one key runs the same query with every rule off.

**Rule sources**

| Source | Matches | Default scopes |
| --- | --- | --- |
| Dev sentinels (in the style of asimov / asimeow) | A folder is skipped only when a sibling file proves what it is: `node_modules` next to `package.json`, `target` next to `Cargo.toml`, `.venv` next to `pyproject.toml`, `Pods` next to `Podfile` | Search, folder sizes |
| Ignore files | `.gitignore` and `.ignore` inside repositories | Search |
| System | `~/Library`, caches, `.Trash`, `/System` | Search |
| Your rules | Paths, globs and regexes in `~/.config/crow/exclude.toml` | Your choice |

No rule applies to plain browsing by default: an excluded folder still shows up in its parent, dimmed, with the rule's name on hover.

```toml
[[ruleset]]
name = "dev-deps"
scopes = ["search", "sizes"]
sentinels = [
  { dir = "node_modules", sibling = "package.json" },
  { dir = "target", sibling = "Cargo.toml" },
]

[[ruleset]]
name = "library"
scopes = ["search"]
paths = ["~/Library"]
globs = ["**/*.photoslibrary"]
```

**Transparency**

- Results end with a count of what was skipped: `1,204 results · 38 folders skipped by dev-deps, library`. Clicking it lists every skipped folder and the rule behind it.
- Cmd-Shift-A (or `!` in the Vim layer) reruns the current query with all rules off.
- A rule set can be switched off for one query from the query line; the query text shows it (`conf -library`).
- Folder sizes say when they leave something out: `2.1 GB (+14 GB excluded)`.

## Sidebar

The sidebar is a full pane: it can take focus, be filtered with `/`, and hold live content, not just links. Favorites stay at the top, as in Finder.

| Section | Holds | Beyond Finder | MVP |
| --- | --- | --- | --- |
| Favorites | Folders and files, in nestable groups | Custom icon and color per item; a mark letter (`'a`) for instant jumps; drop onto an item to move, Option-drop to copy | Yes |
| Recent | Frequently and recently used folders | Ranked by frecency; uses zoxide's database when installed (`zoxide query --list --score`) | Yes |
| Saved queries | Live searches, e.g. "PDFs in Documents, last 30 days" | Live result count; open as a virtual folder | M9 |
| Projects | Git repositories under chosen roots | Provided by the git plugin; branch and changed-file count | M9 |
| Locations | Volumes, iCloud, network shares | Free space bar; eject on hover | Yes |
| Tags | Finder tags | Same tags Finder sees | M9 |
| Plugin sections | Anything a plugin provides | Same API as built-in sections | M9 |

Every section can be collapsed, reordered or hidden. The sidebar's contents live in `~/.config/crow/sidebar.toml`, so a setup can be synced through dotfiles.

## Preview pane

The preview pane is in the MVP and follows Finder's layout: a preview, then the file's name, kind and size, then Information, Tags and More. It appears as the last column in column view and as a toggleable right pane in the other views.

| Part | Shows | Beyond Finder |
| --- | --- | --- |
| Preview | Quick Look thumbnail; text and code as scrollable, highlighted text | Text previews are selectable and searchable with `/` |
| Name | File name, editable in place |  |
| Kind and size | `Markdown · 12 KB`; folders show item count and size | Folder size states what exclusion rules left out |
| Information | Created, Modified, Last opened; image dimensions, audio length, text line count | Plugins can add rows (the git plugin adds last commit and status) |
| Tags | Finder tags, editable |  |
| More… | Opens the full Get Info view |  |

## Icons and theming

**Decided for the MVP:** Crow uses native macOS file icons and Finder's path names (Macintosh HD › Users › …), so it is easy to pick up. The themed glyph icons and path styles below become settings after the MVP.

Icons are single-color vector glyphs tinted from the active theme, so every icon follows a theme switch. Native macOS icons and thumbnails stay available as a per-view option, and image files always show thumbnails in icon and gallery views.

**Icon sources**

| Need | Source | License | Notes |
| --- | --- | --- | --- |
| Interface glyphs, generic files and folders | Lucide | ISC | About 1,600 icons with one consistent stroke style |
| Programming languages and dev files | Devicon / Nerd Font glyph sets | MIT (check each set) | The same mapping Yazi and eza use |
| Brands and services | Simple Icons | CC0 | 3,000+ logos (GitHub, Figma, Docker…) |
| Apps | sketchybar-app-font | Check before bundling | Maps app names to glyphs; matches your SketchyBar setup |

These counts and licenses are from memory; confirm them before bundling.

**Icon resolution**, first match wins:

1. A custom icon set on this exact item (any file, folder or favorite)
2. Exact file name: `Cargo.toml`, `.gitignore`, `Dockerfile`
3. Folder name: `src`, `.github`, `node_modules`
4. Extension: `.rs`, `.ts`, `.md`
5. File type (UTType): image, audio, archive
6. Generic file or folder

Icon packs are TOML files mapping these keys to an icon id and a color. Colors name a theme role (`accent`, `fg-muted`) or one of the palette hues (`base08`–`base0F`, `base12`–`base17`), which the theme index allows for exactly this many-hue case. Plugins can ship icon packs.

**Theming.** Crow implements the theme index reader protocol from `~/.config/themes`:

- It styles only with the 16 roles and derives hover, pressed and disabled states in OKLCH.
- It follows "System" through `current.json`, then `defaults.json` (currently FireBox dark, Solarized Light).
- It live-reloads by watching the real (de-symlinked) directories, debounced by 100 ms, and keeps the last good theme on a parse error.
- It stores only the theme id in its own settings, embeds a fallback theme, and offers "Apply to all apps".
- The interface uses one font: the system font by default, configurable to any installed font.

## Plugins

Use Rhai for plugins, with heavy work done by native host modules that scripts call. Built-in features use the same extension points as third-party plugins, so the API is exercised from day one.

| Option | For | Against |
| --- | --- | --- |
| **Rhai** (chosen) | Pure Rust, Rust-like syntax, sandboxed by default (no I/O unless the host grants it), operation limits built in | Slower than Lua; small ecosystem; no async inside scripts |
| Lua (`mlua`) | Fast with LuaJIT; Yazi proves it for file managers | A C dependency; a second language style |
| WebAssembly (`wasmtime` / Extism) | Any language, strongest isolation | Heavy tooling for plugin authors; a lot of boilerplate for UI hooks |

Rhai's speed is a non-issue if scripts only describe and combine: anything slow (walking, diffing, git status) runs in Rust host modules, called from scripts. WebAssembly can be added later as a second plugin tier if a plugin genuinely needs its own heavy code.

**Extension points**

| Point | What a plugin provides | Example |
| --- | --- | --- |
| Command | A named action for the palette and keymap | "Copy relative path" |
| Column | A value per entry, computed in the background and cached | Lines changed |
| Decoration | Badge, name color or dimming per entry | Git status letter |
| Sidebar section | A live list of items | Projects |
| Previewer | A preview pane renderer for some file types | Markdown, CSV |
| Context menu item | An entry scoped by file type | "Open in Zed" |
| Hook | A reaction to navigate, select, file change or operation done | Auto-refresh a column |
| Icon pack | Data only, no code | Language icons |

**Permissions and isolation.** A `plugin.toml` manifest declares what the plugin needs, shown once on install:

```toml
name = "git"
version = "0.1.0"
permissions = { fs = "read", spawn = ["git"], net = false }
provides = ["column", "decoration", "sidebar", "command"]
```

Each plugin runs on its own worker with an operation budget per call. A plugin that exceeds it is paused and named in the status bar; the UI never waits on a plugin.

**Git, the first plugin (after the MVP).** Inside a repository it shows status letters (M, A, ?) and dims ignored files, adds a "lines changed" column (`+12 −3`), puts branch and ahead/behind counts in the status bar, and fills the Projects sidebar section. It does not stage, commit or push: that stays with the `git` CLI and tools like lazygit.

## MVP scope

The MVP is a keyboard-driven column and list browser with fast, visible search: the parts where Crow differs most from Finder. The plugin host is in the MVP as an internal API that built-in features already use; public Rhai scripting follows right after.

| Feature | Scope | Notes |
| --- | --- | --- |
| Column and list views | MVP |  |
| Tabs and split panes | MVP |  |
| Sidebar: favorites, recent, locations | MVP | Custom icons and mark letters included |
| Filter focused pane with visible query | MVP |  |
| Command palette and fuzzy go-to | MVP |  |
| Name and content search with exclusion rule sets | MVP | Including skip counts and the rerun-with-everything key |
| Finder and Vim keymap layers | MVP |  |
| Basic file operations, undo, activity log | MVP | New folder, rename, move, copy, trash |
| Finder-style preview pane, Quick Look, drag out to other apps | MVP |  |
| Theme index with live reload | MVP |  |
| Native macOS icons and Finder path names | MVP | Icon packs and the Explorer-style path bar followed as settings in M9 |
| View-settings model (global, path rules, folder) | MVP |  |
| Public Rhai plugin API | M9 | [Plugin author docs](plugins.md) |
| Git plugin | M9 | First public plugin |
| Icon and gallery views | M9 |  |
| Copy queue: pause, resume, conflict dialog | M9 |  |
| Drop in from other apps, spring-loaded folders | M9 |  |
| Saved queries, Projects and Tags in the sidebar | M9 |  |
| Folder sizes, batch rename with preview | M8, M9 | Folder sizes in M8, batch rename in M9 |
| iCloud download status | M9 |  |
| Staging, committing, pushing | Not planned | Stays with the `git` CLI |
| Desktop icons, Open/Save panels | Not planned | macOS keeps these for Finder |
| Free icon placement | Not planned | Breaks principle 3 |
| Remote file systems (SFTP, S3) | Not planned | Not before 1.0 |
| Mac App Store release | Not planned | The sandbox rules it out |

## Open questions

- [ ] GPUI or a Swift shell? Decided by the spike (see Stack).
- [ ] Personal tool, open source from day one, or a product later? This sets the license and how stable the plugin API must be.
- [x] Vim layer on or off by default? Off; opt-in in settings.
- [ ] Which rule sets are on by default: only dev sentinels, or also `~/Library`?
- [x] Preview pane in the MVP? Yes, Finder-style (see Preview pane).
- [x] Icons and paths in the MVP? Native macOS icons and Finder path names; configurable later.
- [ ] Is Spotlight good enough for whole-disk name search in the MVP?
- [ ] Embedded fallback theme: FireBox, or something neutral?
