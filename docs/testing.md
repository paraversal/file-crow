# Testing Crow

## Automated

```bash
cargo test --workspace          # unit tests and GPUI keystroke tests
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --check
./scripts/check-test-temp.sh    # the same tests under a private TMPDIR; fails on leftover fixtures
```

Tests own their temporary folders and delete them when the test or app ends,
including on an unwind. Window and palette fixtures stay under `/tmp` on purpose
(palette scoring sees whole paths), and the check inventories new `/tmp` entries
with Crow's `crow-*` and `.trash-*` prefixes as well as the private `TMPDIR`
tree. It never sweeps or deletes pre-existing temp files. A forced `kill -9` of
a test binary bypasses destructors; remove only the exact paths it named, never
a shared prefix.

The keystroke tests (`crates/crow-app/src/window.rs`) drive a real window through
GPUI's test platform: key contexts, history, tabs, hidden files, multiple
selection, type-to-select (its bottom-bar prefix, timeout, Escape, and staying out of the filter and Vim), sorting, Go to Folder (Shift-Cmd-G into the palette), views kept per window, the column view's focus rules, file operations
(new folder and new file with inline rename, batch rename with a live preview, one Activity entry and grouped undo, trash and undo, duplicate, copy and paste, conflicts, permanent delete,
New File/Folder with a folder or file selected, Open Terminal Here and its failure, Empty Trash and its sheet,
cancel, empty, partial failure and text-field block), the sidebar's line icons, responsive Columns view widths, the syntax
preview's numbered lines,
drops, the sidebar (reordering, removing, Add to Sidebar, drops on favorites and group titles, Recent,
twin hints across Favorites and Recent, the current-folder fill and cursor ring, mark letters, the disk
row's free space and warning tone, arrows showing the folder in the center, Option-Up/Down group jumps),
live change batches (a late batch cannot bring back a deleted file, watcher paths through a
symlinked ancestor still match),
view settings (per-folder memory, live rules, Why This View?, broken files), the theme picker
(preview, revert, save), the `/` filter (its field in the focused column and list frame, match ranges and highlights, the ⌘F hand-off, the no-match state), the command palette (sections, two-line rows and match ranges, the home row matching its account name, the Commands token and chips, path mode and Tab completion, recently used commands), the search panel (scope bar, its folder glyph and spoken path, switch,
Search Home from the field and the results, footer counts and the grouped skipped list, the Full Disk Access action,
idle examples, empty-state widen buttons (everything, Home, names and contents), best-row selection while results stream and the user's
row after they move, contiguous name hits before content and weak fuzzy names after, one-line regex
errors, singular skipped wording, preview in list and column view at the hit line, match cycling), the top band
(search field click, Cmd-F refocus and select-all, Done/Esc, Go to, All/Names/Contents, switching
tabs closing search, the Tab cycle while searching, ↓ into the results and ↑ back, focus never left on an element that is gone), the right column
(⇧⌘P in list and column view, `show_preview` live reload, the palette chip, ⌘I opening Info and a second ⌘I closing it, the tab strip and its
click targets, the tab choice across folders, the Info fields for a file, folder, package and symlink, a multi-selection summary, the Where link,
search and sidebar targets, and a stale metadata load dropped when the selection changes), the blinking caret,
the bottom bar
(its type-to-select prefix, hints by focus, retiring idle hints, progress with Cancel, a config error beside progress, counts, narrow-window
path folding and hint dropping, messages that retire, the editable path bar in both `path_bar` modes
(⌘L, a single click on empty space, the always-visible pencil at the end of the path, the pill fading in
on hover and clearing when the pointer leaves the window, Explorer's resting field outline and Go
chevron, a mode reload while editing, the field expanding across the path and hints, the muted completion
suffix after the text (hidden mid-text), Tab completion, Return to a folder or a file, a missing path's
beep and bar error, Escape and a click-away, segment drags (a click still navigates, a drag does not; the
folded "…" drags its deepest folder; the path field has no drag), the path slot keeping its height, and a
virtual destination's title with no false editor), the contextual tips (a long-folder scroll, two
zero-match filters, a typed path for a recent folder, Got it, `hints = "off"`, one at a time, closing on a
folder change or an overlay, and the retired set surviving a relaunch), the Return setting (rename by default, open with `enter = "open"`, F2 always renames, the sidebar and search results still open), the search-result context menu, closing or quitting during an operation, ⌥ word moves in
text fields, the list's Name column following the window width, the menu bar
(every registry command listed, Finder keys, enabled by focus, dispatch matching the key) and the
Vim layer (motions, marks). The finish pass adds the whole-window checks: the assembled layout at
1280×800 (band, sidebar, row, filter field and scope bar heights), one key for each of ShowSearch,
ShowCommandPalette, StartFilter, GoToFolder and SearchReveal in every scope and both layers, exactly
one accent fill (none in the pane while the sidebar has the keys), the preview column taking the
strip's remaining width, the folder summary for a folder cursor and with no matches, and the filter
and search counts as announced. Shared chrome has its own tests: key text from the live keymap (including the
Vim layer and the palette's old strings), keycap weights, tooltip text and a hover, the sheet scrim
(closing outside, staying open inside), and the selection tint roles. Each test gets its own temp
config folder. The settings window has its own tests: ⌘, opens one window and a second ⌘, focuses
it, ⌘W/Esc close it, each control's keys and aria state, every setting writes only its line, the
the theme drop-down previews while open, keeps on Return and reverts on Esc, Tab or ⌘W; the font drop-down writes only when kept; a click opens, closes and picks; the open menu floats over the page; a broken file disables
the writing controls and fixing it clears them, external edits update the controls, the focus ring
never resizes a control, Tab scrolls the page, and every control has a role and label.

`crates/crow-core` tests the config models on their own: view layers and glob rules, `exclude.toml`
defaults and validation, editing `views.toml` and `sidebar.toml` without losing comments, and
frecency with zoxide's scores. It also tests the remembered palette commands: move to front, cap,
round trip, and a broken file starting empty; and the tip state: round trip with unknown ids kept,
a broken file starting empty, showing once, the key-use and `hints = "off"` refusals, the cooldown,
and the zero-match streak.

`crates/crow-search` tests the engine on temp trees: dev rules and sentinels skip and are counted
with typed reasons, Search everything walks them, dot names are visited and matched while Show Hidden
Files is off (dot folders are skipped, not descended), `search_includes_hidden = false` leaves them
out, packages are a name hit but not descended while Search everything enters them, fuzzy names score,
literal and `/regex/` contents report the first matching line, text and ranges with the rest counted,
batches stay bounded (one excerpt per file, 256 hits at most) and the first hit leaves at once, a name
hit arrives before its own content read, a FIFO does not stall the walk, an unreadable folder is
reported as a skip, binary files are not read, and cancelling stops the stream while stopping
contents keeps every name streaming. Both controls interrupt active content reads between buffers,
including nonmatching files; buffered matching lines stop too, and the searcher can be reused.
`corpus_cases.rs`
builds the deterministic corpus (`crow_search::corpus`) and runs its 23 named cases through the real
engine — the user's `.backu` case, hidden on and off, the exclusion sets, Search everything, the
symlink and its child, the package inside and entered, the unreadable folder and the FIFO — each under a 10 s deadline.
`oracle_tests.rs` adds the check that does not depend on someone thinking of the example: 25 seeded
trees of colliding names, exclusion sentinels, packages, symlinks and unreadable folders, walked by
an independent recursive `read_dir` oracle and compared as hit and skip sets with the engine across
four queries, hidden on/off, Search everything and all three match modes. (300 seeds took 57 s in
debug; 25 keep it near 5 s.) The search panel adds keystroke tests for names and contents, Search everything and its switch,
scope buttons and the spoken folder path, Search Home, Cmd-Shift-M cycling, idle examples, the empty
state's widen buttons, footer counts and errors, the grouped skipped reasons and the privacy action,
name-before-contents order, preview loading and the hit line, navigating and revealing results,
copy and trash, Vim j/k in the results, and the two result budgets (late strong names, dropped other
rows, the strong-name cancel, Contents-mode cancel, and a moved cursor that keeps its row).
`search_corpus_matrix` runs both corpus queries through All/Names/Contents, hidden on/off and Search
everything (24 runs), and compares the panel's result paths, footer and skipped count with
`crow_search::search` directly, so an app-side filter cannot silently hide engine results again.

The operation engine (`crates/crow-fs/src/ops_tests.rs`) is tested against real temp folders:
every operation and its undo, each conflict choice, cancelling midway, a read-only destination,
a source vanishing mid-operation, undo refusing to overwrite, a forced cross-volume move,
and 40 randomized rounds of "do, undo, compare the tree". Batch rename is checked against real folders too: multiple parents, folders/packages/symlinks, exact duplicates and occupied targets, an unselected hard link, ancestor/descendant refusal, non-UTF-8 round-trip refusal, case-only renames and canonically equivalent Unicode names on APFS, a swap and a longer cycle, injected stage/placement failures with byte-level restore assertions, a blocked restore that keeps and reports the exact staging paths, grouped undo/redo, and undo refusing an unrelated occupant. The job engine's deterministic
gates cover a pause acknowledged inside a byte callback, a clone paused after it completes,
cancel while paused and while a conflict waits, a late conflict keyed by attempt, a stale
answer, apply-to-all, an approved target replaced while paused, a source changed while
paused, Keep Both collision retry, a Replace whose placement fails and is restored (or whose
restore is blocked), a cancel between Replace's Trash and placement, a same-volume Move with
Replace and its undo, a symlinked parent counted as the same folder, a cross-volume move that
keeps its destination when the source cannot be retired, monotonic progress,
metadata/xattrs/ACL on clone and non-clone copies, case-folded targets, dangling symlinks,
special files refused in planning, and undo of a committed prefix.
`window/tests.rs` covers the app-wide coordinator: two windows share one ordered queue,
planning is held or released deterministically, a paused head keeps later work waiting,
Pause waits for the worker's acknowledgement and a late acknowledgement is ignored,
cancelling a planning head logs without copying, a failed plan or jail refusal advances
the next request, a conflict is re-hosted when its origin window closes and only the
current job/item/attempt is accepted, Activity lists active before queued jobs, the bar
shows the queue and the indeterminate planning state, and Activity and the conflict sheet
work by mouse and keyboard. Result highlighting stays with the original folder surface,
including after navigation away and back. A cancelled terminal keeps its committed prefix
undoable. A confirmed close cancels paused and conflict-waiting jobs, refuses late submissions,
and waits for cleanup. Empty Trash releases the queue even if its window closes meanwhile.
Pause/Resume through the bundled UI is P5B's hand check.

Batch staging is a private `0700` container named `.crow-batch-<pid>-<sequence>` beside each real parent, created exclusively and registered only while this process owns it. Reconciliation and listings ignore exactly the registered paths; a folder that merely looks similar is never hidden or removed. A successful batch removes its containers. A blocked restore keeps them and the error names each original, current and intended path, which is the session's recovery record. A crash leaves the containers on disk; they are not guessed at or cleaned by name on the next launch.

## Test and fuzz reports

`scripts/test-report.py` wraps test and fuzz commands into one report per run. It
runs the exact argv without a shell, streams merged stdout/stderr to the terminal
and `run.log`, and writes `run.json` atomically. The child's exit status is kept
even when parsing or writing the report fails. `run.json` records the schema,
actual revision (and whether the tree was dirty), UTC start, host/arch, mode,
command, wall time, result, per-binary cargo test counts, full failing test
output, fuzz seeds and optional live frame numbers.

```bash
python3 scripts/test-report.py run --mode workspace --out target/test-reports/workspace -- cargo test --workspace
python3 scripts/test-report.py run --mode smoke --out target/test-reports/smoke -- cargo test -p crow-app fuzz_smoke
python3 scripts/test-report.py summary target/test-reports
```

Without `--out`, a run goes to `target/test-reports/<mode>-<UTC>`. Results are
`passed`, `failed`, `interrupted`, `empty`, `incomplete`, `paused` or
`collector-error`; `collector_error` is separate from the child's `exit_code`.
A zero exit with no cargo `test result` and no fuzz seed evidence is `empty`,
not a pass. `summary` reads `run.json` recursively, validates the schema and
skips a malformed or unknown-version report with a warning; the valid reports
still render. It sorts by UTC, revision and mode and links each available
`run.log` and replay artifact.

The fuzz reporters emit machine events only when `CROW_TEST_EVENTS=1`; the
collector sets it for `--mode fuzz` and `--mode fuzz-live`, so direct runs and
their existing human output and replay flags are unchanged. `seed-start`,
`seed-end`, `seed-failure` and `frame-summary` use the same decoder for
`fuzz_long` and `crow --fuzz`. A live run's frame table is separate from the
headless rows because hardware and modes are not comparable; a missing
`frame-summary` renders as `—`, not zero.

### Replays

A headless failure carries the same options as its run. Copy the values from
`seeds[].replay` in `run.json`:

```bash
CROW_FUZZ_SEED=42 CROW_FUZZ_TREE=small CROW_FUZZ_CHURN=0 CROW_FUZZ_STEPS=3000 \
  cargo test -p crow-app fuzz_long -- --ignored --nocapture
```

The failure's `.rs` artifact is the shrinker's script for a regression test. A
live failure's `.txt` artifact is the saved step format; the collector rewrites
the command's `--script` path to that relative artifact, so run it from the
report directory:

```bash
cd target/test-reports/<fuzz-live-run>
cargo run --release -p crow-app --features fuzz -- --fuzz --seed 42 --tree small \
  --script replays/<run-id>-seed-42-step-123.txt --until 123
```

Add `--pause-ms 2000` to pause before that step for a profiler. The human
`replay:` line in `run.log` remains the same. A user-supplied `--script` is not
copied into the report; the replay command keeps its path, so run it where that
file lives.

The collector fixtures cover a pass, a failing test, a lost subprocess, a GPUI
failure after a seed start, a start without an end, a missing command,
zero-output evidence, malformed events and nested fields, an unknown schema, two
revisions in one table, environment non-copying, a user-supplied script and
absent live frame fields:

```bash
python3 scripts/test_report_test.py
```

CI wraps its Tests and Fuzz steps with the collector and uploads
`test-report-workspace` / `test-report-fuzz` on success and failure. To aggregate
downloaded CI artifacts with local reports, unzip or download them into one tree
and point `summary` at it:

```bash
gh run download <run-id> -D target/test-reports/ci-<run-id>
python3 scripts/test-report.py summary target/test-reports
```

### First baseline

Recorded from `1220af2a13ad` on 2026-11-18; the worktree also held this file's
uncommitted reporting section and the user's docs reorganization, so `run.json`
says `revision_dirty: true`. The machine was `Julians-MacBook-Pro.local` (arm64).

| Run | UTC | Revision | Mode | Result | Evidence | Report |
| --- | --- | --- | --- | --- | --- | --- |
| `workspace-20261118T170422Z` | 2026-11-18T17:04:22Z | `1220af2a13ad` | workspace | passed | 14 binaries · 905 passed · 0 failed · 1 ignored · 16.1 s | `target/test-reports/baseline-workspace-final/` |
| `smoke-20261118T170438Z` | 2026-11-18T17:04:38Z | `1220af2a13ad` | smoke | passed | 2 passed · 0 failed · 4.0 s | `target/test-reports/baseline-smoke-final/` |

Compare a later run with the same mode and workload. Keep live `fuzz-live` frame
numbers in their own table; do not compare them with the headless smoke.

## Fuzzing

`crates/crow-app/src/fuzz.rs` generates seeded steps from the live keymap and the
elements on screen, applies them to a real GPUI window, and checks invariants after
each one. Escaping effects go through `effects::RecordingEffects`, so a fuzz run
never opens an app, plays a sound or touches the pasteboard or Trash.

A run is confined to its temp tree: every file operation outside it is refused
(`Operations::jail`, reported as a `jail` violation), "home" is the tree (Go Home,
`~` and search's Home scope stay inside), Trash goes to a folder next to the tree,
config and state live in a temp folder, and the sidebar lists only the tree.

```bash
cargo test -p crow-app fuzz_smoke                 # 3 seeds x 30 steps on a 40-file tree, in the normal suite
cargo test -p crow-app fuzz_settings_smoke        # 2 seeds x 30 steps on the Settings window
cargo test -p crow-app fuzz_smoke_edge_trees -- --nocapture  # every small edge kind
CROW_FUZZ_MINUTES=2 cargo test -p crow-app fuzz_long -- --ignored --nocapture
CROW_FUZZ_SEED=42 CROW_FUZZ_STEPS=5000 cargo test -p crow-app fuzz_long -- --ignored --nocapture
```

`fuzz_long` prints `CROW_FUZZ_SEED=<n>` for each seed. With no seed it picks one
from the clock; `CROW_FUZZ_MINUTES` (default 1) stops after that many minutes,
`CROW_FUZZ_STEPS` (default 3000) caps each seed, `CROW_FUZZ_SHRINK_SECS` (default 60; the scheduled workflow sets 600) caps shrinking a failure (it prints the best script so far; the seed and step always reproduce it), and the long tree has 200 files
(the smoke uses 40 to stay near 10 s). The scheduled workflow
(`.github/workflows/fuzz.yml`) runs 20 minutes every three days.

### Edge trees and churn

`crates/crow-fs/src/testing.rs` (the `testing` feature) builds awkward trees and
changes them while the fuzzer acts. `--make-fixture DIR N` and
`EdgeTree::wide(N)` share one writer, so the bench fixture's names and contents
are unchanged.

```bash
CROW_FUZZ_TREE=small CROW_FUZZ_CHURN=1 cargo test -p crow-app fuzz_long -- --ignored --nocapture
CROW_FUZZ_TREE=long-name,deep-20 CROW_FUZZ_CHURN=0 cargo test -p crow-app fuzz_long -- --ignored --nocapture
CROW_FUZZ_TREE=wide-100000 CROW_FUZZ_MINUTES=2 cargo test -p crow-app fuzz_long -- --ignored --nocapture
```

`CROW_FUZZ_TREE` is `small` (every kind except `Wide` and `HugeFile`), `all`, or a
comma-separated list: `long-name`, `nfc-nfd-twins`, `emoji`, `rtl`,
`leading-dot`, `spaces`, `newline`, `deep-<n>`, `wide-<n>`, `symlink-loop`,
`dangling-symlink`, `unreadable`, `package`, `hardlinks`, `empty-folder`,
`huge-file` (sparse 10 GiB). `fuzz_smoke_edge_trees` covers every small kind.

`CROW_FUZZ_CHURN=1` starts a thread that creates, renames, deletes, rewrites,
chmods and moves entries in the shown folder and one level down every 1–50 ms.
With the variable unset, `fuzz_long` alternates churn off/on per seed. A `Settle`
step pauses churn before the deep listing check and resumes after it, so the
check compares a quiet disk. Churn makes replay nondeterministic: a failure
prints whether its shrunk script still fails with churn off.

`tree.remove()` / `crow_fs::testing::remove_tree` restore permissions before
deleting, so a mode-000 folder from an `Unreadable` tree cannot leak into the
next run.

A failure prints the seed, the failing step, the violation and a shrunk script:

```rust
// paste into crates/crow-app/src/window/tests.rs
use crate::fuzz::Step;
#[gpui::test]
fn regression(cx: &mut TestAppContext) {
    let tree = fuzz_tree("regression", 200);
    let found = replay(cx, &tree, 200, crate::script![Step::Keys("cmd-k".into()), Step::Settle]);
    assert!(found.is_empty(), "{found:?}");
}
```

The runner prints the row for `docs/plans/done/m7/findings.md`. Deduplicate by violation
and place, not by seed; H4 fixes rows and adds the named regression test. For an
edge-tree or churn finding, replay with the same tree and churn setting:
`replay_with(cx, &tree, 200, &EdgeTree::single(EdgeKind::LongName), seed, churn, script)`.

A right-click step carries the menu item it picks (`RightClick(selector, Some(slot))`;
`None` dismisses the menu): the native menu never opens, the recorder answers with
that slot and the item runs as if clicked.

What the fuzzer does not drive: the accessibility tree (GPUI's test platform disables
it; the invariants check every painted click/focus element's role and label through
the debug registry instead). `fuzz_settings_smoke` drives the Settings window as a
second surface with its own keymap and control selectors. The second browser window is opened
and checked by the invariants but only the first is driven. `cmd-m` is excluded
because the test platform has no minimize implementation.

### Real frames: `crow --fuzz`

The test platform has no real frames, so H1's invariant runs cannot see a freeze.
`crow --fuzz` drives the same generator in a real window and runs a 16 ms
main-thread heartbeat; its lateness is the frame budget:

```bash
cargo run --release -p crow-app --features fuzz -- --fuzz --seed 42 --minutes 10
cargo run --release -p crow-app --features fuzz -- --fuzz --seed 42 --minutes 10 --churn
cargo run --release -p crow-app --features fuzz -- --fuzz --tree long-name,deep-20 --minutes 10
cargo run --release -p crow-app --features fuzz -- --fuzz target/fixture-100k --dir-is-scratch --minutes 10
```

With no folder it builds a temp H1 tree (`--tree small` by default: every edge kind
except the 100k folder and the sparse 10 GiB file). Config, state, Trash, the log
(`logs/crow.log`) and the saved steps (`script.txt`) go to a support folder under the
system temp folder, never `~/.config/crow` or `~/Library/Logs/Crow`; the first line
printed names each. Copy Path and text-field copy, cut and paste use the recorded
effects' own clipboard, so the user's clipboard is left alone. A user folder is refused unless
`--dir-is-scratch` says it may be changed. `--tree none` is the ordinary 200-file
tree; `--tree all` adds `wide-100000` and `huge-file`. Churn is off unless `--churn`
is passed.

The runner dispatches keys and typing only (H3 Q3); clicks and the Settings window
come later. It drops `cmd-h`/`cmd-alt-h`/`cmd-m`/`cmd-q`: the test platform has no
hide/minimize implementation, and driving a hidden window is not a user path. A step
runs per frame at a 16 ms pace. It checks H1's invariants every 25 steps (the check
itself is main-thread work), and after a `Settle` (and at the end) it compares the
listing to disk on a background thread. The focus invariant requires focus only in the
active window: an inactive browser window may have no focused element (the Settings
window or a second window is key). `Step::AdvanceClock` is a test-clock step and
does nothing here; real timers run on their own.

A heartbeat over 50 ms is printed as a slow step with the seed, the step index, the
last five steps and the `tracing` spans that overlapped it (`listing apply`, `rebuild
rows`, `preview load`, `search batch`, `theme reload`, `icon fetch`; spans are at
`debug`, which this mode enables for the `crow` and `crow_macos` targets). GPUI's `on_next_frame` is
throttled for an inactive or occluded window, so the heartbeat, not a frame
callback, is the source. A step that acts on more than 10 000 paths (select-all,
then copy/cut/paste/trash/undo on the 100k fixture) is tagged `(bulk)`: its budget
is 100 ms, the others keep 50 ms. The report counts heartbeat intervals, groups the
slow ones by step kind and counts the ones over their own limit:

```
step                  over      worst   limit  replay
Keys (bulk)              2    312.4 ms    100 ms  --seed 42 --script …/script.txt --until 1234  Keys("cmd-k")
```

Exit code is 0 unless an invariant fails or a heartbeat is over 250 ms (a freeze).
A freeze prints `FREEZE` and the run keeps going; a stall over 30 s prints `HANG`
and exits 1, so a wedged step cannot swallow the rest of the run. A watchdog thread
watches the heartbeat, so this works even for a step that never returns; a 100k
resize can take 5–6 s and still finish.

Every run saves the steps it sends to `script.txt`, one per line, before sending
each, so a `HANG` or a killed run keeps the step that stalled. The seed alone does
not replay a run: the generator reads the live keymap and focus, which depend on
timing, and two runs of one seed can split after a hundred steps. To profile a slow
step, replay the saved script to just before it, with the same seed and tree
options (the seed builds the tree and drives churn), as the report prints it.
`--pause-ms` waits at the pause for a sampler, then keeps replaying:

```bash
cargo run --release -p crow-app --features fuzz -- --fuzz --seed 42 --script /path/to/script.txt --until 1234 --pause-ms 2000
sample <pid> 5 -file /tmp/crow.sample   # during the pause; `sample` prints a call tree
```

`sample` needs the process to be visible (a GUI app is) and `xcrun xctrace` needs a
full Xcode install, not just the Command Line Tools. Without `--pause-ms` the run
pauses before the step and the window stays open until Crow quits, which is enough
to reproduce the action by hand.

A replay of a `--dir-is-scratch` folder needs it in the same state as the original
run (a fresh `--make-fixture` for the 100k fixture). The window stays open until
Crow quits. A clean run removes its temp tree and support folder; a run with a slow
step, a violation or a freeze removes the tree and keeps the support folder (`kept
for replay:`). A paused run, a `HANG` and a killed run leave both (`crow-fuzz-*` in
the system temp folder); remove them by hand, with `chmod -R u+rwx` first because
the edge tree has an unreadable folder. Slow steps go into
`docs/plans/done/m7/findings.md` with their replay command; `findings.md` is the input
to the next H4 batch. This runner needs a real GUI session, so it is local only;
the scheduled workflow runs H1's `fuzz_long`.

Removable and cloud volumes are hand checks, not simulated. Create a disk image
(`hdiutil create -size 200m -fs APFS -volname CrowTest /tmp/crow.dmg`), attach it,
browse it in Crow, create and rename a folder on it (⌘⇧N, type `a.b`, Return), then
eject it while shown: the bottom bar reports "Can't open …: No such file or
directory", with no panic and no beachball (the rows stay until you navigate away).
A sleeping disk or a pulled SMB share is the slow case: the window keeps drawing
because the app's file-system calls run off the main thread (`crates/crow-app/src/io_guard.rs`
fails the tests on a new unmarked one). iCloud Drive placeholders (download on open,
eviction) are out of scope for the test platform; check a cloud-backed folder by hand.

## Folder sizes

Set `folder_sizes = "visible"` in `settings.toml` (or Settings ▸ Folder sizes) and open a folder with subfolders in list view: the Size column shows `--`, then sizes fill in top first and fade in; a stale value dims until it is re-measured. Compare a folder with Finder's Get Info (logical size). Edit a file deep in a measured folder: the folder and its parents update within a couple of seconds. Quit Crow, change something, relaunch: the change shows. With the setting off, View ▸ Calculate Folder Sizes measures the current folder once. Scroll a 100k folder of folders quickly: no stutter, and Activity Monitor shows the workers stop for rows that scrolled away. No permission prompt appears just from viewing `~`; an iCloud folder's not-downloaded files stay not downloaded.

## Performance

```bash
cargo run --release -p crow-app -- --make-fixture target/fixture-100k 100000
cargo run --release -p crow-app -- --bench target/fixture-100k
cargo run --release -p crow-app -- --bench-filter e target/fixture-100k
cargo run --release -p crow-fs --example list_bench -- target/fixture-100k
cargo run --release -p crow-app -- --make-search-fixture target/fixture-search-200k 200000
cargo run --release -p crow-app -- --make-search-corpus target/fixture-search-corpus
cargo run --release -p crow-app -- --bench-search target/fixture-search-200k
cargo run --release -p crow-app -- --bench-search target/fixture-search-200k backup
cargo run --release -p crow-app -- --bench-search target/fixture-search-200k file-01-02 --names
cargo test --release -p crow-app --bin crow palette::tests::path_completion_filters_many_files -- --nocapture
cargo test --release -p crow-app bench_tag_filter_100k -- --ignored --nocapture
```

The palette test reports matcher time for five partials over 78k names. Compare release times
with `main` on the same machine. The full palette refresh budget is in the manual checks below.

`--bench-search DIR [QUERY] [--names|--contents]` times the walk, fuzzy names and content reads;
`QUERY` defaults to `needle`, and the match flags restrict it as the panel's control does. It prints
`first_hit`, `first_name_hit`, `total`, `hits`, `scanned`, `skipped` and `contents_stopped`. It is an
engine bench and does not apply the panel's result budgets. `--make-search-corpus` writes the
deterministic tree the `crow-search` corpus cases use; the unreadable folder in it is mode 000, so
remove it with `chmod -R u+rwx` first.

`bench_tag_filter_100k` applies a synthetic tag filter to the 100k fixture and prints the first
background batch and the settled snapshot. It reads no real metadata, so it measures Crow's scan,
classification and row-build work; the tag predicate itself is read through the injected reader.
The idle cost is the ordinary `--bench` run, since the tag control only exists inside `/`.

`--bench` scrolls 600 frames and prints first paint, load time and frame times.
`CROW_BENCH_VIEW=icons` (or `list`/`columns`/`gallery`) forces the view for that run, so the
icon grid is measured on the same fixture and path as the list.
`--bench-filter <query>` applies the name filter first, so the scroll times are for
highlighted rows. It needs a GPU, so it runs locally, not in CI. `--make-fixture`
uses `crow_fs::testing::EdgeTree::wide`; the 100k fixture's manifest hash (names
and sizes; the files are empty) is `b659b9a8…`, unchanged since before H2.

| Measure | Budget |
| --- | --- |
| First rows on screen, 100k folder (warm app) | under 50 ms (M3: 9–10 ms; M5: 10–11 ms; M5-fix: 11 ms; M5-fix-2: 10 ms; M6: 10 ms; U1: 10 ms; U4: 10–11 ms, filtered 10–17 ms; U5: 10–11 ms; U11: 8 ms warm, 39–70 ms cold; H2: 7 ms warm, 132 ms cold) |
| Scroll, 99th percentile frame | under 1.5 × the display's frame time (M3: 8.6–8.8 ms at 120 Hz; M5: 9.7–13 ms, main on the same machine 8.8–14.6 ms; M5-fix: 8.9 ms; M5-fix-2: 9.0 ms; M6: 8.9–9.1 ms; U1: 8.79 ms, main on the same machine 8.93 ms; U4: 8.85–8.95 ms, main on the same machine 8.84–8.86 ms, filtered 8.82–9.03 ms; U5: 8.87–9.06 ms, main on the same machine 8.86–8.95 ms; U10: Full 9.73–9.83 ms, Off 9.83 ms, Low 9.97 ms; warm first paint 8–10 ms in all three; UI overhaul as a whole (U11): 8.66–9.86 ms, `main` on the same machine 9.64–9.77 ms, M6 8.66–8.69 ms; H2: 8.70 ms p99, avg 8.42, max 40.00) |
| Search, 200k nested files (`--bench-search`, median of 3) | first hit ≤ `main` + 5 ms; total ≤ `main` × 1.10. B4 critique medians, branch vs `main`: `needle` (all) 1.1 ms / 1518.8 ms vs 2 ms / 1576 ms; `back` (all) no hit / 1462.3 ms vs no hit / 1462 ms; `backup` (all) no hit / 1470.5 ms vs no hit / 1479 ms; `file-01-02 --names` 3.9 ms / 183.8 ms vs 4 ms / 185 ms. The fixture's files all contain `needle`, so `back` and `backup` measure a full walk with no hits. A regression past the budget is a blocker: profile with `samply` or Instruments before changing code. |

### Search diagnostics

`crow --search-trace DIR QUERY` runs the real engine — the same walk and matcher as Cmd-F, not a
second implementation — and prints one line per batch, the totals, every skip and, with `--target`,
one verdict for that path:

```bash
cargo run --release -p crow-app -- --search-trace ~ .backu --hidden --target ~/.backup_excludes
+4ms batch 1: 1 hits
+9ms batch 2: 11 hits
first_hit=4ms total=4100ms hits=12 scanned=5137 skipped=3
skip ~/node_modules (dev: name "node_modules")
skip ~/Library/SomeFolder (no permission)
target: name hit at 4ms (batch 2)
```

The verdict is one of `name hit at Nms (batch N)`, `content hit line N at Nms`, `not found; under
skipped PATH (REASON)`, `not found; outside DIR`, or `not found; no skipped ancestor, and the name
does not match` (checked with the engine's own `FuzzyMatcher`). `--names` / `--contents` restrict the
match, `--hidden` / `--no-hidden` override Show Hidden Files, `--search-hidden` /
`--no-search-hidden` override `search_includes_hidden` (default on), and `--everything` ignores the
exclusion sets.
It reads `settings.toml` and `exclude.toml` from the config folder; a broken file prints its error and
falls back to the defaults.

The app logs one line when a search starts and one when it finishes, both at `debug`:
`RUST_LOG=crow::search=debug` (the log is `~/Library/Logs/Crow/crow.log`, and stderr when launched
from a terminal). The start line carries the root, scope, matching and effective match, hidden,
everything, the skip label and the query's length — never its text, since queries can be private. The
finish line carries elapsed ms, first-hit ms, scanned, skipped, rows, hits, `names_capped`, `contents_stopped`, `dropped` and any error.

## By hand (each milestone)

Run `scripts/bundle.sh --open` so macOS treats Crow as an app. Rows are grouped by window
region. Committed screenshots come from `scripts/shot.sh` (U11, I2).

For agent-driven app checks, always launch with tips off. Choose **Don’t show tips**
if Crow asks on first launch. To skip the prompt in a fresh test profile, set
`tips = "off"` in its `settings.toml` and put `{"tips_choice":false}` in its
`onboarding.json` under `CROW_STATE_DIR`. Enable tips only for checks that exercise them.

### Band and menus

| Check | How |
| --- | --- |
| Toolbar | Hover Back, Forward, the icons, list, columns and gallery buttons, Go to and the search field: each shows a tooltip with its name and key ("Back to <folder> ⌘["); the same for a path segment, the disk row, Search everything and Cancel during a copy. At the start of history Back is the same icon at 40 %. Icons are drawn, crisp at 1× and 2×; traffic lights sit centred on the band. Light and dark themes. VoiceOver still reads each control's label, not the tooltip. |
| Band, narrow | Shrink the window: the title truncates, then the field shrinks, then Go to loses its label. In search mode the field shrinks but the match control and Done stay whole. |
| Window drag | Drag the band's empty area (the title): the window moves. Drag Back, Forward, either view button, Go to or the search field: the window stays put and the control still works (Back still goes back). Double-click the empty band: the system's titlebar action (zoom unless System Settings says otherwise). Traffic lights stay at (14, 19) and the titlebar looks transparent, at 640 px and wide, light and dark. |
| Menu bar | Open each of File, Edit, View, Go, Window and Help: every item shows the key it has in the list (⌘F, ⇧⌘A, ⌘K, ⇧⌘G, ⌘[ …). Rename shows ↩, Filter This Folder `/` and Quick Look Space. Items without a key are the ones the README names. |
| Menus follow focus | With the list focused, Edit ▸ Filter This Folder, Cut, Copy, Paste and Select All and File ▸ Rename are enabled; Tab to the sidebar: they grey out, Go ▸ Back stays enabled. In search results, Copy, Move to Trash and Quick Look are enabled, Paste is not. With the rename field focused, the list's items (Rename, Quick Look, Filter This Folder, Move to Trash …) grey out; Cut, Copy, Paste, Select All, Undo and Redo act on the text. |
| Menus act | Choose Go ▸ Enclosing Folder, Back, Go to…, Go to Folder…; Edit ▸ Find…, Filter This Folder; View ▸ Show Activity: each does what its key does. File ▸ New Window opens the current folder; ⌘N with no window opens Home. |
| Find Everything | From the list press ⇧⌘A (or Edit ▸ Find Everything…): search opens in the current folder with the rules off; typing finds files inside `node_modules`. With search open, ⇧⌘A from the sidebar flips the rules and returns focus to the query field. |
| Undo in text fields | Trash a file, then rename another item, type, and press ⌘Z: the typing is undone (a run of typing or deleting is one step), ⇧⌘Z redoes it, and the trashed file stays in the Trash. Same in the filter, search and palette fields. Back in the list, ⌘Z restores the file. Settings… with no app for `.toml` files shows an error in the bottom bar. |
| Help search | Help ▸ type "trash": macOS points at File ▸ Move to Trash. |

### Sidebar

| Check | How |
| --- | --- |
| Favorites | Drag a favorite up and down: one insertion line shows the exact target (under a group title for that group's start), the line never shifts rows, and the end of a group is reachable; dropping on a group title still moves the favorite to its start. Drag a folder from the list onto a group title, and between two favorites (from the list and from Finder): the line shows at the row's upper or lower edge with no row fill, and the folder is added there, not moved; over the middle of a favorite the row fills instead and the drop moves into it. Drop a file onto a favorite. Check `sidebar.toml` keeps your comments. Add `icon = "🚀"` and `color = "blue"` to one. |
| Recent | Visit a few folders (and `cd` around with zoxide): they appear under Recent, favorites excluded. |
| Line icons | Favorites show the line folder icon and Recent the clock; a favorite's `icon = "🚀"` still shows the emoji and `color = "blue"` tints its line icon. Locations keeps Finder's disk icon, and the file list its Finder icons. Light and dark, 640 px and wide. |
| Locations | Mount a disk image: it appears without a restart, with free space and a bar; ⏏ ejects it. |
| Sidebar twins | Visit two folders with the same name (e.g. `~/a/git`, `~/b/git`): Recent shows "git a" and "git b" with the hint dimmer; with a long name (the sidebar is a fixed 210 px) the hint truncates and goes before the name does. VoiceOver reads "git, in a". |
| Sidebar follows the cursor | Tab into the sidebar: nothing navigates. Arrow down through favorites: each shows in the center, focus stays in the sidebar (ring moves, the list has no focus). Tab or → (or Return): the list has focus on that folder. With a search open, Tab to the sidebar and arrow: search closes, the folder shows. Back after an arrow walk returns to the folder you were in before the walk. List and column view. |
| Sidebar fills | Open a favorite: only it is filled. Tab into the sidebar and arrow: an accent ring moves, no second fill; Tab out: the ring goes. Drag a file over the current row: the drop highlight shows. Check in a light and a dark theme. |
| Disk row | Locations shows "<n> GB free" right-aligned and a grey bar; fill a disk image past 90 %: the bar turns to the warning color. VoiceOver still reads free and total. |
| Mark letters | With `keymap = "vim"`, favorites with a `key` show the letter on the right; `m x` on another folder moves it; Finder layer shows none. |
| Sidebar Recent walk | ⌥↓ to Recent and arrow through it: the rows keep their order while you move. Tab out: the folder you stayed on counts as a visit; folders you only passed don't. |
| Sidebar groups | Tab into the sidebar: Option-Up/Down jump to the first row of the previous/next group (Favorites groups, Recent, Locations); the center follows. |
| Pane dividers | Drag either divider, release and move away. The highlight clears, and a keyboard-focused divider still shows its highlight. Reopen Crow and check that the left sidebar remembers its width. Double-click the divider and relaunch to check that the default width was saved. |

### Pane

| Check | How |
| --- | --- |
| Column view | Cmd-3: arrows move within a column, Right moves into the selected folder, Left moves back; Left in the leftmost column goes up a folder. Only the focused column's selection is in the accent color. Selecting a file shows its preview as the last column. Cmd-2 returns to the list in the folder you were in. |
| Gallery view | Cmd-4: the rail shows mixed files and folders with a thumbnail or icon, name, size and modified date; Up/Down, Left/Right, Home/End, Shift and Cmd-click match List, and the filter, cursor and selection survive List ↔ Icons ↔ Gallery. Selecting a folder shows its immediate children without reading their contents; an empty folder, a partial sample, an unreadable folder and an iCloud placeholder each show a distinct state. Select a high-resolution landscape image, then add a portrait image to the selection. Images fit their cells without cropping, two labeled cells appear in rail order, and Cmd-I leaves room for both the rail and stacked previews at 640 pt. At a wide size, the rail aligns size and date, and the highlight and click target span the row. Select three entries: the preview follows the cursor and the full selection still drives file actions and Cmd-I's summary. |
| Icon view | Cmd-1: tiles show the direct entries only, never inline-expanded children. Open a folder in List, select a child, then Cmd-1: the cursor moves to the direct folder; Cmd-2 restores the open child. Filter, sort, type-to-select, Return, rename, context menu, Preview and drag payloads match List. Arrows follow the tile grid, Shift extends in display order and Cmd-click toggles. |
| Icon sizes and reflow | View ▸ Icon Size Compact, Standard and Large at 640 px and wide: the tile is fixed per preset, names keep up to two lines, and Compact fits more names. Change size with the cursor near the middle and with it at the bottom: the focused tile stays visible and selected, the filter and cursor survive, and a resize with a drag in flight cancels and restores it. |
| Icon marquee | Drag from empty space: tiles intersecting the rectangle get the inset preview; reverse and they drop out. Cmd-drag keeps the starting selection, Esc restores it, and dropping the pointer past an edge autoscrolls with the rectangle origin fixed. A watcher reorder or tab/focus change cancels and restores. |
| Icon thumbnails | On a mixed-image folder: thumbnails replace the pack/native icon as Quick Look replies; scrolling fast keeps frame time bounded and the cache does not grow without limit. A cloud placeholder keeps its download badge and never requests bytes; an unavailable provider falls back to the pack or native icon. |
| Icon caption | With the Git plugin enabled and `icon_caption = "git.lines-changed"` in one folder, each tile shows a muted `Lines · n` line; Off in View removes the line, and a folder override can differ from the default. Disable or reload the plugin: the slot stays reserved and blank, the configured id survives, and VoiceOver reads the caption. |
| Preview column | Column view: the preview takes the rest of the strip, never narrower than 300 px; with a folder under the cursor it shows the folder icon, name, "Folder · n items" and modified; with no matches the folder you are filtering shows the same summary. ⇧⌘P hides it and the columns keep their widths. |
| Preview pane | Select an image, a PDF, a text file and a folder: thumbnail, text, icon. Tags show for tagged files. |
| Syntax preview | Select a `.rs`, `.toml`, `.json`, `.js`, `.py` and `.sh` file: the preview shows line numbers and coloured keywords, strings, numbers, types and comments. A search hit still shows its excerpt with the hit line washed. Text previews stop at 80 lines and clip; a 100k folder's scroll is unchanged. |
| Columns and tint | Cmd-3 at 1280 and 640: browse columns grow up to 360 px when there is room alongside Info and keep a 196 px minimum. Hover a truncated name to see it in full. The focused column is tinted with the surface role (lighter in dark themes, darker in light ones) at the board's strength; moving the selection never changes a width. |
| Open Terminal Here | Palette `>open terminal`, right-click a folder, a file and empty space: Terminal opens at the folder (a file's parent; empty space the shown folder). A multi-selection disables the menu line. Set `terminal = "iTerm"` (or a missing app): iTerm opens, or the status line names the failure. |
| Table | Drag header dividers to resize; click headers to sort and reverse. |
| List width | Start with Preview open and a wide sidebar. Name has room to show common file names, and the headers align with their rows. Toggle Preview or drag either side divider. As space runs short, metadata columns shrink from the right, then Name can shrink further and whole columns hide. At 640 px no cell runs past the edge. Drag Name's divider to trade space with the next column. |
| Inline folders | In the list, the chevron (›/⌄) opens and closes a folder inline; Right opens a folder indented under it (Right again goes in), Left steps to the containing row, Left again closes it. Option-click is a normal click. Select across levels (Cmd-click or Shift-click a nested row) and copy or trash: every selected row is included. Navigate away and back: expansions are gone. |
| Quick Look | Select a file, press Space. Arrow keys while it is open move it along. |
| Click on empty space | List view: select a few items, click below the last row (or in the gutter beside a row): nothing is selected, ↓ selects the first row. Cmd- or Shift-click there keeps the selection. Column view with three columns open: click the empty part of the first column: the others close and nothing is selected. While renaming, click empty space (or another row): the rename is applied, not left open. |
| Batch rename | Select two or more files and press F2: one sheet previews the old → new names before anything moves. Find & Replace, Add Text (before/after) and Numbering (base, start, padding) each update the preview; Keep extension is on by default. A duplicate or occupied target, a source changed since the preview, or an ancestor/descendant selection keeps Rename disabled or refuses with an error and no disk changes. Apply a swap, return, and use ⌘Z once: every name returns and the cursor follows the mapped row; ⇧⌘Z swaps again. Escape leaves the disk and selection alone. Try the same from results across two folders. Tab through the rules, Keep extension and buttons: Space and Enter activate the focused control, and Enter on Cancel closes without renaming. |
| Context menu | Right-click an unselected row: it alone is selected and a native menu opens at the pointer, with the menu bar's names and keys (⌘⌫ beside Move to Trash, ⌥⌘C beside Copy Path). Right-click inside a multi-selection: the selection stays; Move to Trash trashes all. Ctrl-click does the same as right-click. Right-click empty space: selection clears; New Folder starts its rename; Paste is grey with nothing copied, enabled after ⌘C. Show in Finder reveals the items (on empty space, opens the folder). Esc or a click outside closes the menu without acting. Both in list and column view, with the Vim layer on too (keys shown stay ⌘C etc.). While the menu is open, check whether a running copy's progress keeps updating: AppKit's tracking loop pauses GPUI frames, so it may freeze until the menu closes, then resume. |
| Live updates | With a folder open, create, rename and delete files from Terminal; the list follows without jumping. |
| View settings | Cmd-3 in one folder, then visit another: it keeps its own view; going up in column view stays in columns. Add a `[[rule]]` to `views.toml` with the folder open: it applies at once. Cmd-J explains each value. Set `view_per_folder = false`: Cmd-3, then visit that folder with its own list view and go Back: the window stays in columns, Cmd-T opens columns, `views.toml` is unchanged, Cmd-J says "this window". |
| Vim layer | Set `keymap = "vim"`: hjkl, gg/G, `m a` then `'a` from elsewhere. |
| Typing is not stolen | Rename a file and type `/`, Space and Return: the characters stay in the field (Return commits) and no filter or Quick Look opens. Same in the filter, search and palette fields. |
| Type-to-select query | Type `no` in a folder: the bottom bar shows `no` left of the hints and the selection jumps to the first match. Wait a second: it clears. Type again and press Esc: it clears at once and the row stays selected. Type `/`: the field opens and no prefix shows; type in it: still none. With `keymap = "vim"`, letters move instead and no prefix shows. Check light and dark, 640 px and wide. |
| Return setting | `enter = "rename"` (default): Return opens the name field. Set `enter = "open"`: Return opens the cursor item; F2 still opens the name field in both modes. In both modes Return in the sidebar and in search results still opens. Vim `c w` still renames. Change it in Settings ▸ Behaviour ▸ What Return does: only the `enter` line changes in `settings.toml`, comments survive, and an external edit updates the control. |
| Word moves | In the filter, search and palette fields: ⌥←/⌥→ jump by word (`m5-fix.md` is three words), ⌥⇧←/→ select, ⌥⌫ deletes a word. |

### Filter

| Check | How |
| --- | --- |
| Filter | Press `/` in a folder: an accent-ringed field with `/`, the query, "n of N" and `esc` opens as the list's first row (below the header). Typing narrows the rows and the matching letters turn accent and semibold. Arrows move within matches, Return opens, Esc restores focus and rows. |
| Filter in columns | Cmd-3, move into a middle column, `/`: only that column gets the field and footer; no other column moves or changes width. Left or Right closes the filter. Type a 60-character query: the caret stays visible inside the field. |
| Filter to search | Filter a column, then press Cmd-F (or click the footer): search opens with the same text in that folder, and Esc returns to the unfiltered column. Filter something with no matches: "Nothing in … matches", the button opens search, and the hidden-files note shows when hidden files are off. |
| Filter look | In a light and a dark theme, accent text on the selected row is readable; in the Vim layer, letters typed in the field stay there. VoiceOver reads "Filter <folder>", the count, and the no-match line. |

### Search

| Check | How |
| --- | --- |
| Search field | Click the field: search opens with the cursor in the field; the field spans the band with an accent ring; Done esc on the right. Tab to the results: the ring goes; Tab again to the sidebar, again back to the field (with nothing typed, Tab goes straight to the sidebar). From any of them Cmd-F brings the field back with the query selected, and Esc closes. Esc or Done returns to the folder with keys going to the list. |
| All / Names / Contents | Search a word that's in file names and inside files: Names drops the content lines, Contents drops the name-only rows. Names in Home is fast (no file reads). Switch to This Mac: the control shows Names, faded. Cmd-Shift-M in the field and in the results cycles All → Names → Contents. |
| Search | Cmd-F, search a word in file names and in contents: while results stream the top row stays selected and previewed; contiguous name matches come first with their letters highlighted, scattered fuzzy name hits follow the content rows; a name match is one line, a content-only hit shows its line number and line under the name; the Modified column stays on one line ("Today 14:32", "Yesterday", "4. Nov"). Switch <folder> / Home / This Mac; Return, Cmd-R, Space, drag-out, Cmd-C and Cmd-Delete act on results. The right pane previews the selected result in list and column view, at the hit line. |
| Search scope bar | One bar: "Search", the scope segments, and "Skip dev folders" on the right; click the switch and press Cmd-Shift-A: both flip it and the results change. Enable `media` in `exclude.toml`: the label becomes "Skip dev and media folders" without reopening. The folder segment has a folder glyph; hover it for the full `~` path; VoiceOver reads "Search in <name>, ~/path". Hover Home: "Search Home ⇧⌘H". |
| Search Home | Cmd-F in a folder below Home for a word that only exists elsewhere in your home folder: the empty state shows "Search Home ⇧⌘H"; ⇧⌘H finds it, from the field and from a result row. Edit ▸ Search Home and the palette's "Search Home" do the same; with no search open the menu item opens a search scoped Home. |
| Empty state | A miss offers every way to widen: with skipped folders "3 folders were skipped. Search everything ⇧⌘A"; with a folder scope other than Home "Search Home ⇧⌘H"; with Names or Contents "Search names and contents" (no keycap, since ⇧⌘M cycles rather than picking All). Each button is a button to VoiceOver, with its key spoken. |
| Skipped folders | Search a folder with `node_modules` and a package: the footer says "N folders skipped ›"; click it: groups Excluded (each with the rule that skipped it), Hidden folders, Packages ("package (Search everything looks inside)") and Couldn't read ("no permission"), paths relative to the searched folder, above the footer. A folder Crow can't read adds "Open Privacy & Security…", which opens Full Disk Access in System Settings. With Show Hidden Files off, dot folders appear under Hidden folders and do not hide their contents from Search everything. Esc closes the list first, then search. Search everything enters excluded, hidden and package folders; unreadable ones stay listed. |
| Search hidden files setting | Show Hidden Files off (⇧⌘.), Cmd-F for `zshrc` (or another dotfile): found, dimmed, VoiceOver reads "…, hidden"; the footer counts the skipped dot folders. Turn Settings ▸ Search ▸ Search always includes hidden files off: the dot hits and their skipped folders go, and Search everything leaves them out too; turn it on: they return. With Show Hidden Files on, the setting changes nothing. |
| Default match | Settings ▸ Search ▸ Search matches: set Names, open a new Cmd-F: the panel starts on Names (Contents likewise). Change it while a search is open: that panel keeps its own All/Names/Contents choice. |
| Spotlight | Search This Mac for a name outside the home folder: results stream in, the switch is replaced by "Spotlight: indexed volumes only". |
| Search idle | Cmd-F with nothing typed: the heading follows All / Names / Contents and the left-aligned examples fill the field, switch to This Mac or Home, or flip the switch; the last is plain key text and disappears when no `exclude.toml` set is enabled. The key line follows `keymap = "vim"`. |
| Search errors | Type `/(/`: one red footer line, "Invalid pattern: unclosed group"; no rows, and the body shows "Can't search: Invalid pattern: unclosed group" and suggests removing the slashes. |
| Search themes | FireBox and a light scheme: switch knob, selected segment and match highlights on the selected row stay readable; VoiceOver reads the switch as on/off and the footer once per finished search. |
| Search result cap | Two budgets: 20,000 strong (contiguous) name rows, and 20,000 other rows (content and weak fuzzy names). In `target/fixture-search-200k`, Cmd-F `needle` (All): the footer ends "… more not shown · contents stopped at 20000" in `warning`; replace the query with `file-01-02-00001` without leaving search. The new search puts that exact file first. The `strong_name_after_the_cap_is_kept` test covers late strong names in the same walk. Names `file`: the footer says "First 20000 name matches · refine the search", and typing more narrows. Names `f0` in `target/fixture-100k` fills the weak budget: "20000 results · 80000 more not shown", without claiming contents were stopped. At 640 px the footer stays on one line and ellipsizes; VoiceOver reads it once when the search settles. |
| Search context menu | Right-click or Ctrl-click a result: Open, Reveal in Folder, Quick Look, Copy, Cut, Copy Path, Move to Trash. An unselected row is selected first; a row in a multi-selection keeps it. Each item runs once and the menu closes; the file list and sidebar menus are unchanged. Check in the Vim layer too. |

### M8.5 search finish

Run on a bundle built from the branch, in light and dark, at 640 px and wide.

| Check | Steps |
| --- | --- |
| The user's report | Cmd-F in `~`, Folder, All, hidden on: `.back` and `.backu` both show `~/.backup_excludes` within a moment. Then hidden off: the same. |
| Dot names in Cmd-K | Hidden off: Cmd-K `zshrc` finds it under "Files in …"; `~/ssh` finds `~/.ssh`. |
| Scopes | A miss in a subfolder offers Search Home; ⇧⌘H finds it. This Mac finds a name outside Home. |
| Match modes | All, Names, Contents on one word; a Names miss offers "Search names and contents". |
| Skips | `~` with hidden off: the skipped list groups Excluded, Hidden folders, Packages, Couldn't read; ⇧⌘A lifts the first three. |
| Cap | `target/fixture-search-200k`, `needle`, then an exact file name: top row. |
| VoiceOver | Hidden rows read "…, hidden"; the empty-state buttons read with their keys; the Folder segment reads its path; the footer reads once. |

### Palette and sheets

| Check | How |
| --- | --- |
| Command palette | Cmd-K over a scrim; the folder you are in is not listed, so Cmd-K then Return goes to the one before it; sections Recent / Favorites / In <folder>; two folders with the same name show different paths; matched letters accented in name and path; Recent rows show "2 h ago"-style times; ↵ goes, ⌘↵ opens a tab; Vim j/k move across headers. Keycaps follow the layer: `>` rows show ⇧⌘G-style keys, and with `keymap = "vim"` Copy shows `yy`. |
| Palette ⇥ | ⇥ on a row puts its path and `/` in the field; typing narrows to its subfolders and fuzzy-matches its files (mid-name too, e.g. `ack` finds `back.txt`); ↵ on "this folder" goes there; `.` shows hidden folders. |
| Palette files here | In a folder with matching files and subfolders, type a name without `/`: folders come first, then "Files in <folder>", with the best fuzzy file matches; Return reveals the file selected in the folder. With Show Hidden Files off, an empty ⌘K lists no dot folders; typing `config` lists `.config` after visible matches and dimmed (`.co` ranks it first); dot files come after visible ones and dimmed, or not at all with the setting off. Check FireBox and Solarized Light at 640 px: the dimmed rows stay readable and the "Files in" header aligns with the folder rows. |
| Palette budget | In `target/fixture-100k`, open ⌘K and type `file-0`, `file-00`, `file-000` quickly: no visible lag. With `RUST_LOG=debug`, the log's `palette refresh` lines stay under 16 ms (B2: 12–15 ms with 100k files; the U7 folder budget is under 4 ms with 10,000 subfolders). |
| Palette home row | With Home in Favorites or Recent, type `jul` (your account's first letters): the home row shows and `jul` is highlighted in its name; type `~`: the same row shows with `~` highlighted in the path. Other folder rows rank as before. |
| Palette commands | `>` turns into the Commands token and Return on the empty list never quits (typing `quit` finds Quit Crow); Backspace on an empty field removes it; rows show menu, on/off chip for Show Hidden Files and Show Activity, and keycaps (switch to the Vim layer: keycaps follow). |
| Recently used | Run two commands from the palette, quit, relaunch: `>` lists them under Recently used. Set `palette_recent_commands = false`: the section disappears without a restart, and new runs aren't recorded. |
| Palette themes | FireBox and Solarized Light: token, highlights, chip and keycaps readable, the input and footer dividers visible; narrow window keeps the sheet inside with 16 px margins. |
| Go to Folder | Shift-Cmd-G opens Cmd-K with this folder's path and `/` typed and its subfolders listed; type a few letters, Tab, Return goes there; a file's full path selects it in its folder; a missing path beeps and stays open. Typing a capitalised name (`Docs`) never crashes. |
| Sheets | Open Shift-Cmd-G, Cmd-J, the theme picker, Cmd-K, a permanent delete and Empty Trash: each sits on a darkened window (check a dark and a light theme); clicking outside closes it like Esc (picker reverts, delete and Empty Trash do nothing), clicking inside doesn't. The window can still be dragged. |
| Empty Trash | Put one disposable item in the Trash. ⌘⇧⌫, File ▸ Empty Trash… and palette `>empty trash` all ask "Empty the Trash?"; Cancel/Esc leaves the item, Empty Trash removes it with Finder's empty-trash sound and the status says how many. An empty Trash says "The Trash is already empty." with no sheet or sound. ⌘⇧⌫ does nothing while a filter, rename, search or palette field has focus. A disk image with an item in `.Trashes/<uid>` is emptied too; a locked item is named in the status. |
| Theme picker | Cmd-Option-T, arrow through themes (each shows at once), Esc returns; Return keeps one and writes it to `settings.toml`. Apply to All Apps changes `~/.local/state/themes/current.json`. |

### Bottom bar and tips

| Check | How |
| --- | --- |
| Path bar | Click a folder segment to go there (Back returns); click the selected file's segment to open it. The pencil sits at the end of the path and is always visible; the path is a 32 px pill in the 38 px bar. Hover the path: the pill appears around the path and pencil. Click the empty space or the pencil to edit (⌘L and a double-click still do); the field spans the path and the hints up to the counts' divider, with 12 px padding and the text vertically centered. Drag a segment to Finder: the folder or file arrives, not text and not the selection; the drag does not also navigate. Drop a segment on a pane or sidebar favorite: the folder moves like a row drag. While the path field is open, dragging selects text and starts no file drag. VoiceOver reads the pencil as "Edit path". |
| Explorer path bar | Set `path_bar = "explorer"` (or Settings ▸ Appearance ▸ Path bar) in light and dark, 640 px and wide: the path keeps a visible surface-and-border outline at rest; segments still show their hover fill, navigate and drag. Click the blank field or pencil: the same field opens with the whole path selected, an accent border and a Go chevron left of the hints; type a folder and click the chevron: it goes there exactly as Return does. Return on a file reveals it; Tab completion, a missing path's beep and bar error, Escape's focus return and a click-away all behave as in Crow. Change `path_bar` in a text editor while the field is open: the same text stays and only the chevron/outline decoration changes; after Escape the new mode's breadcrumb is drawn. On a saved-query or tag destination with no selected result, the bar shows its title with no pencil, segments or editor. VoiceOver reads the chevron as "Go". |
| Bottom bar hints | Fresh state dir (`CROW_STATE_DIR=$(mktemp -d)`): idle hint `/ filter` only. Press `/` and Esc three times (or Edit ▸ Filter This Folder): it disappears; relaunch: still gone. Focus the sidebar, a filter, search results: the hint set changes each time. `hints = "always"` brings them back; `"off"` hides all. Vim layer: same, and `m` shows "Mark this folder…". |
| Bottom bar counts | Column view: hover the count, the tooltip names the focused column's folder. Filter: the bar keeps "N items" (the field says "n of N"). Select several files: "n selected · size". |
| Progress and cancel | Copy a large folder to another disk: a 2 px accent line along the bottom bar's top edge and "Copying … · n %" with Cancel esc replace the hints; the path and count stay. Esc or Cancel stops it with nothing half-copied. |
| Bottom bar error | Break `views.toml` while an operation runs: progress and the red token show together, hints are gone; Open opens `views.toml`; fixing it removes the token. |
| Bottom bar, narrow | In a deep folder, shrink the window: the path's middle folds into "…" first (its tooltip names the hidden folders, a click goes to the deepest, a drag drags that folder), then the hint goes; the first segment and the current folder or selection stay visible. With an error and a running copy at minimum width, the path still shows its ends and nothing runs off the right edge. |
| Tips | With a fresh `CROW_STATE_DIR`: the first launch asks "Show tips as you go?" once; Show tips and No tips both close it and write only the `tips` line, and a relaunch does not ask again (deleting `settings.toml` does not either). Esc keeps the default and closes it. Then scroll a 200+ item folder → "Type / to filter this folder by name." bottom right above the bottom bar, in list and column view, FireBox and Solarized Light; Got it closes it and it never returns after relaunch; "Don't show tips" closes it and mutes every later tip. Two zero-match filters in a row → the ⌘F tip; ⇧⌘G to a recent folder → the ⌘K tip. Using the key closes and retires a tip. `hints = "off"` shows none; `tips = "off"` shows none but keeps the hints. Opening Activity closes a tip; a narrow window wraps the text inside 12 px margins, with the last row above the card. |

### Files between apps

| Check | How |
| --- | --- |
| Drag out | Drag one row, then a multiple selection, onto a Finder window and into a Mail message. The image is the same small icon plus name pill inside and outside Crow (right edge at the cursor if the grab point is past it); leaving and re-entering the window keeps that one image, with no clipped frame or second icon; dropping back inside Crow moves the item. List and column rows keep one steady drop target while the pointer rests on them. |
| Drop in | Drag files from Finder onto a Crow list, onto a column, and onto sidebar rows and group titles: the target keeps one steady highlight while the pointer moves within it; same disk moves, other disk copies, Option copies. Drag a row from Crow's own list the same way. Drop onto a path segment, a real folder in search/live/saved/tag results, and a Projects sidebar row. |
| Spring-loaded folders | From Finder, hold a file over a folder row for about 600 ms: the folder opens; keep holding over a folder inside it and it opens too. Release: one job lands in the deepest folder and Back returns to the starting folder with surviving sources selected. Leave the window or press Esc before release: the starting folder, selection and scroll return and no job starts. Repeat with Option, from Crow's own list, from a search result, from a path segment, between two windows, and across volumes; the accepted effect matches the source's offered operations. With motion on, the folder's highlight blinks twice just before it opens (Reduce Motion: one soft dim; motion off: steady). Hold over Back: it steps out of the opened folders, then through Back history; releasing over Back drops nothing and restores the start. Type a few letters in Crow first, leave the pointer over Finder, then drag in: the highlight, spring and drop still work. |
| Trash and Put Back | Cmd-Delete a file in Crow, then use Finder's Put Back on it in the Trash. |
| Finder clipboard | Copy in Crow, paste in Finder; copy in Finder, paste in Crow. Above 1000 selected items the status says "(Crow only)": Crow's own paste works, Finder keeps its previous clipboard. |
| Activity | Cmd-Option-L lists active and queued jobs before recent operations; Pause/Resume works on a large copy, Cancel drops a queued job, and Undo on an older terminal entry works or explains why not. The bar's list icon opens the same panel and shows the queued count. |
| Conflict ownership | Copy two files with an existing target into the current folder, dismiss the conflict sheet with Esc, then reopen it with Resolve in Activity. Tab to Resolve and press Space. In the sheet, Tab and Shift-Tab reach each choice, Apply to all remaining, Decide Later, and Continue. Close the window that first showed the sheet while another window is open. The sheet appears there, and an answer for an older attempt does nothing. |
| Close during an operation | Copy a large folder to another disk, then ⌘W the only window (or click its red button, or ⌘Q): a sheet asks. Esc keeps the copy running; Return stops it and closes (or quits) once it has wound down. With a second window open, closing one does not ask. |
| Git refresh | With Git enabled, open a subfolder with a visible ignored file. Once its styling appears, touch another file in the open folder and then touch a file in a sibling folder in the same worktree. The ignored row and Git status stay steady. Remove its ignore rule and check that the ignored styling clears. Disable Git and check that all remaining Git decorations clear immediately. |

### Command line

| Check | How |
| --- | --- |
| App icon | `./scripts/bundle.sh --open`: Finder, the Dock and ⌘Tab show the icon named by `app_icon` (refined when unset). Change `app_icon` in `settings.toml` to sun, graphite and bone while Crow runs: the Dock icon follows each save; a bad value shows the error in the status line and keeps the icon. Rebundle: Finder shows the new choice. Check the 16 px icon in a Finder list. |
| `crow` shim | `./scripts/bundle.sh --install` installs the app and the `crow` command into `~/.local/bin` (or `CROW_BIN_DIR`). With Crow running, `crow ~/Downloads` opens a new window there, `crow notes.txt` opens its folder with the file selected, and `crow` brings Crow forward. Quit Crow and repeat: the requested path is the first window, not Home. Paths with spaces work. |

### Whole window, themes, VoiceOver

| Check | How |
| --- | --- |
| Whole window | `scripts/shot.sh filter crates/crow-core/src firebox finder 'text:/|text:vi'` and the same with `solarized-light`: compare with `docs/plans/ui-overhaul/img/assembled-filter.png`; then `search crates` with `cmd:f|text:config`. One accent fill on screen (the focused column's cursor, or the field's ring while filtering); band, sidebar, rows, filter field, scope bar and bottom bar at the heights in the plan's table. |
| Live theme | Switch macOS light/dark, or edit a scheme in the theme index: colors fade over about half a second (faster with Reduce Motion). |
| VoiceOver | Cmd-F5: the band reads as a toolbar; the field announces "Search <folder>, Command F"; Done reads "Done, close search, Escape"; rows read name, details and "n of N"; the count reads "18 items in <folder>"; the filter count is announced once typing pauses; the search footer reads once when a search settles; a tip is read once and "Got it" and "Don't show tips" are buttons; the first-launch sheet reads "Show tips as you go?" with Show tips and No tips; an error reads "Error: …" once and Open and Cancel read as buttons; every sheet reads its title. |
| Privacy prompts | Open Desktop, Documents, Downloads: the prompt names Crow. |

### Settings window (M6.5, M9 overhaul)

| Check | How |
| --- | --- |
| Window and categories | ⌘, (or Crow ▸ Settings…) opens one titled Settings window; a second ⌘, brings it forward without losing a valid focus. ⌘W closes it; Esc closes it only when no menu or query is open. The sidebar lists Appearance, Behaviour, Views & Preview, Search, Plugins and Config Files and stays visible while the page scrolls. Choose Behaviour, quit and reopen: the new window starts there. Write `views-preview` to `<state>/settings-category`, relaunch: it starts on Views & Preview. Write garbage: Appearance. It is not a tab and reopening Crow does not reopen it. |
| Geometry | Default 760×760, resizable, minimum 640×420. Check 640 px and a wide window at font size 13 and 24, plus a short window: the sidebar, search field and content scroll independently and no control or pop-up leaves the window. |
| Sidebar and Tab | Tab from search reaches the sidebar, then the enabled body controls, then the recovery Open buttons; ⇧Tab reverses. ↑ ↓ move the sidebar cursor without opening a category; Return or Space opens it. Clicking a row also opens it and keeps focus in the sidebar. With `settings.toml` broken, search and the sidebar remain usable focus targets. |
| Search | ⌘F focuses Search Settings. Type `vim`: Keymap appears under Behaviour and edits `keymap` in place. Type `hidden`: Show hidden files (Views & Preview) and Search always includes hidden files (Search) appear under their headings. Type `icon_pack` or `preview_providers`: the exact-key match ranks first. Type a category (`search`): its controls appear. No match shows the query and Clear Search; Tab reaches Clear Search, Return clears it. Typing a query that hides the focused control, or switching category, commits a field draft and closes an open menu first. Editing `settings.toml` externally keeps the query and updates the shown values. Esc closes a menu, then clears the query, then closes Settings. |
| Appearance | FireBox and Solarized Light: the Theme drop-down opens (click, Space, Return or ↓) a menu over the page, with no scroll box on the page itself: System then the index's themes, dark before light; ↓/↑ previews each in every window, Return or a click keeps it, Esc or a click outside reverts. The menu sits under its button and stays inside the window. The four app-icon tiles show at 64 px; the Dock switches when one is chosen. Icon pack lists Native, local packs and approved plugin packs by name; two packs with the same name show their ids; a saved missing id shows `(not installed)` and stays selected; choosing one changes icons live and writes only `icon_pack`. Path bar offers Crow and Explorer: ←/→ change it live, only `path_bar` changes in the file, a comment on its line survives, and an external edit updates the selection. The Font drop-down starts with System and lists installed families (the menu scrolls); typing opens it at the match; nothing changes until Return or a click. The font-size stepper goes 10–24 with ↑↓ and typed digits. |
| Behaviour | Keymap Finder/Vim, Return Rename/Open, key hints Auto/Always/Off, tips Auto/Off, Remember recent commands, the terminal app, new-window width and height: each changes at once and writes only its line. Type `iTerm` in Terminal app and Return: `terminal = "iTerm"` and Open Terminal Here uses it; clear it and Return: `terminal = "Terminal"`. Motion Auto/Off/Low/Full; Auto's line says which mode it resolves to now and whether Reduce Motion is on. Sounds Crow/Finder/Off; Play plays the trash sound in the chosen set. Width and height are 600–4000; ⌘N uses them. |
| Settings motion | With `motion = "full"`, switching sidebar categories or entering/clearing search fades the content and lifts it 4 px; `"low"` fades only; `"off"` switches in one frame. The sidebar, search field and window size stay still, and Reduce Motion resolves Auto to Low. |
| Views & Preview | Show hidden files, Folder sizes, Show preview and Folders keep their own view write `settings.toml`. Default view List/Columns, Sort by, direction Natural/Ascending/Descending and Folders first write only `[defaults]` in `views.toml`. Choosing a sort key keeps a saved explicit `ascending`; Natural removes it; an absent direction follows the natural direction for the key. External views edits update the controls without a relaunch, and every browser window applies the new defaults. With `view_per_folder = false`, a window's chosen view, a new tab and an open Columns chain stay put while defaults change. |
| Preview providers | Scope starts on Defaults; the editor stays on the scope and target you choose. Defaults edits `[defaults].preview_providers`; an explicit empty list shows "Native only", Clear removes the key. Folder offers Use Current Folder (the folder that invoked Settings; disabled without a browser) and Choose Folder… (cancel leaves the target and file alone; the picker returns focus to the button). The chosen path is shown before any write; a non-UTF-8 path reports that views.toml cannot store it. Path Rule lists rules in on-disk order; repeated patterns show their rule number. Add Provider lists only published Preview descriptors, with titles and ids, and never duplicates a configured id or runs a provider. Move Up, Move Down, Remove, Add and Clear are reachable by Tab and keyboard; an unavailable configured id stays visible and removable. Clearing a mixed rule's list removes only `preview_providers`. A new preview-only rule shows its position after the existing rules, validates the same glob, and wins for matching folders. Inserting or reordering rules externally requires a fresh selection instead of aiming at the new index. |
| Search settings | Search always includes hidden files: find a dot file with Show Hidden Files off, then flip it: open results update at once and only its `search_includes_hidden` line changes. Search matches All/Names/Contents: the next Cmd-F starts in that mode; an open panel keeps its own control. |
| Plugins | Existing inventory and approval, enable/disable, reload and retry flows; a grants error names plugins.toml in the error area and only Reload stays available. |
| Config Files | Edit settings.toml… opens the file (with the old failure message when no app owns `.toml`); Show config folder in Finder opens `~/.config/crow`; `views.toml`, `sidebar.toml`, `exclude.toml` and `plugins.toml` links open their files. |
| Comments survive | Hand-edit a file with comments and a value next to one, then change that value from the window: every other line, comment and blank line is unchanged. If the change is the default, the line is still written. A list edit keeps comments on surviving items. |
| Broken files | Break `settings.toml` (`motion = "slow"`): a row names settings.toml and the line, its writing controls are disabled, and its Open button works. Break `views.toml` and `sidebar.toml` too: all three rows show together and stay reachable in a short window; settings controls stay usable while views controls are disabled. Fix views.toml: only its row clears and its controls re-enable. Fix settings.toml: its row clears. `plugins.toml` errors come from the plugin inventory and keep only Reload usable. |
| External edit | With the window open, change `keymap = "vim"` in a text editor: the segmented control shows Vim without a restart. Delete the line: it shows Finder. Do the same with `[defaults] view`, `sort` and `preview_providers` in views.toml. |
| VoiceOver | The window reads "Settings"; each category row reads its name, position and selected state; the search field reads "Search Settings"; each category heading is a heading. One of each kind: the Show preview switch reads its label and on/off; Motion reads "Motion, Auto, 1 of 4, selected"; an app-icon tile reads its name and selected; the font-size stepper reads its value and 10–24; the Theme drop-down reads as a pop-up button with its value, and its open menu reads its options and the highlighted one; the provider list reads its rows, position and selected state; Move Up, Move Down, Add Provider and the file buttons read as buttons. |
| Palette | ⌘K, type `settings`: Settings opens/focuses the window. Type `edit settings`: Edit settings.toml opens the file. |
| Tab and focus | Tab and ⇧Tab follow the search field, sidebar, visible body and recovery buttons; hidden or disabled controls are skipped and the page scrolls to the focused control. The focus ring is the sidebar's inset ring and never moves the layout. A picker returns focus to the button that opened it. |

### NSSound volume (U10 R3, F7.5)

Set the macOS alert slider low and the output volume normal. With `sound = "finder"` and then
`sound = "crow"`, trash a file and empty the Trash in Crow, then do the same in Finder. If Crow's
sounds are louder than Finder's, report the measurement before changing the sound path: the
root-cause option (`AudioServicesPlaySystemSoundWithCompletion` plus a trimmed cache copy) cannot
fade the trash tail.

## Motion and sound (U10)

`crow --motion-lab` opens a fake folder where every feedback event (trash, move, undo, new
folder, refuse, copy, tip, sheet) can be fired in Off, Low and Full and in the three sound
sets, with slow motion ×5 and the cursor glide. The lab and the app share
`crow_core::motion`, so a duration or look changed there changes both. `p` opens the mock
sheet; `Esc` closes it.

In the bundled app (`./scripts/bundle.sh`), FireBox dark and Solarized Light, each in
`motion = off`, `low`, `full`, sound `finder` and `crow`:

| Check | How |
| --- | --- |
| Trash | ⌘⌫ one file, three files, and a folder of 500 files, in list and column view: the row fades (Low) or its icon becomes a trash and swallows the name while the gap closes (Full); Off removes it in one frame. One sound per operation; afterwards the next row is selected (the previous row if the last row went), and ↓ moves from it. The animation runs smoothly to its end without moving the mouse. |
| Move | Drag a file onto a folder row and onto a sidebar favorite: the source fades, the target row pulses (Full after the fly phase, Low at once), the favorite pulses; Purr in `crow`. |
| New folder | ⇧⌘N: the new folder arrives (rows below slide down in Full), then the rename field opens and takes typing at once. Pop in `crow`. |
| Undo | ⌘Z after a trash: the file arrives; Bottle in `crow`. |
| Refuse | `chflags uchg` a file, then ⌘⌫: Full shakes the row, Low tints it red, Off only beeps; the error line shows. A refused key in a text field (⌥⌫ at the start) shakes/tints the field. |
| Copy | Copy a 2 GB file: progress, the bar fades at the end, Glass in `crow` (only if it ran over a second). |
| Reduce Motion | With `motion = "auto"`, toggle System Settings ▸ Accessibility ▸ Reduce Motion: Full ↔ Low without a relaunch. |
| Sounds off | System Settings ▸ Sound: "Play user interface sound effects" off → no sounds except the alert. Alert volume low → Crow's sounds are as quiet as Finder's (NSSound follows the output volume; Q7). |
| Sheets and tips | Cmd-K, Cmd-J, ⇧⌘G and the theme picker fade in (140 ms Full, 120 Low); a tip rises 8 px and fades in Full, fades in Low. Tooltips still appear without motion. |
| Right pane tabs | Switch Preview, Info and Git Diff (with the Git provider enabled) in List, Columns and search. Full glides the underline and fades the new body in with a 6 px nudge. Low fades in place. Off changes in one frame. The pane stays at its current size and the new tab is active immediately. |
| Folder view switch | In a long folder, select an item partway down and switch List ↔ Columns with Cmd-2 and Cmd-3. Full moves matching visible icons, names and selection fills. Low fades, Off snaps. Try a quick reversal and press Down immediately. Check that automatic per-folder view changes do not start the morph. |
| Column and result reveal | In Columns, select a file and then a folder. Full brings the new child content out from its divider; Low fades it in place. Use Cmd-R on a search result and check the tint pulse is on the destination row, including one well down a long listing. Off changes both instantly. |
| Expand and collapse | List view, a folder with a few items: → or the triangle. Full: the triangle turns, the rows below slide down and the new rows show from the top as the gap opens; ← slides the rows below back up. Low: the new rows fade in, nothing moves. Off: one frame. A folder with hundreds of items opens without a long slide. |
| Hidden items | A folder with dotfiles (Home): hidden files and folders show dimmed (icon, name and every column) in list and column view, in light and dark themes; selected, the fill stays full and the name stays readable; renaming one shows the field at full contrast. |
| Glide | Full, hold ↓: the selection fill follows within one row and settles; ⌘A shows every row selected. Trackpad scroll a long list and a short one that fits: the fill stays on the cursor row and never drifts; the cursor never moves. |
| Lab | `crow --motion-lab` still runs, matches the app, and its sheet (`p`) fades like the app's. |

## Selection and motion (F1)

In the bundle, light and dark, 640 px and wide, list and column view, Vim on and off:

| Check | How |
| --- | --- |
| Delete selects the next row | Trash a middle row, the last row, and a multi-selection (⇧↓ then ⌘⌫): the next row is selected, the previous row when the last row went, and the row after the selection for a block. Delete a file from Terminal while its folder is open: the same, with the row fading (Low) or swallowing then closing the gap (Full). A non-cursor row vanishing leaves the cursor alone; an empty folder leaves nothing selected. |
| Selection islands | Select one row, two adjacent, three adjacent, and two islands separated by an unselected row: one rounded box per contiguous run, square edges between rows, a faint 1 px rule inside a run inset from the island's edges (a delimiter, not a full divider), each island its own box. List and column view, focused (accent) and unfocused (muted). Rows keep their height and position; the cursor glide fill takes the cursor row's rounding. |
| Copy and cut | ⌘C: the copied rows pulse once (no pulse under Off). ⌘X: the cut rows get a dashed outline around the run at full contrast, so they never read as hidden items; the fill stays full. Esc, ⌘Z, a paste, or a new copy/cut clears the outline; a copy in another app clears it within 100 ms. The same in search results and column view. A 100k copy stays responsive. |
| Sidebar fill | Click folders quickly in Full: one fill glides from the old row to the new one; Low fades it in on the new row; Off jumps. The row's text and icons never move. |
| Tip inset | In a long folder, scroll until the `/` tip appears and scroll to the bottom: the last row sits above the card at 640 px and wide, light and dark. "Got it" removes the inset with the card. |
| VoiceOver | Cmd-F5: a selected row reads as selected, the cursor row as active, an unselected row as not selected; the sidebar's current folder reads "current folder" and the ring row is active. |

