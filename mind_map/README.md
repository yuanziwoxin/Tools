# MindMap · 思维导图工具

[简体中文](README.zh-CN.md) ｜ **English**

**One HTML file is the entire mind-mapping tool. Double-click it — no install, no sign-up, no network.**

<p align="center">
  <img src="screenshots/mindmap-hero.en.png" width="900" alt="MindMap main window: horizontal mind-map layout"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/single--file-zero--dependency-2f6fed" alt="single file, zero dependencies"/>
  &nbsp;
  <img src="https://img.shields.io/badge/offline-ready-16a34a" alt="works offline"/>
  &nbsp;
  <img src="https://img.shields.io/badge/export-12%20formats-f59e0b" alt="12 export formats"/>
  &nbsp;
  <img src="https://img.shields.io/badge/layout-mind%20%7C%20org-0ea5a4" alt="two layouts"/>
  &nbsp;
  <img src="https://img.shields.io/badge/tests-38%20suites%20%2F%202850%20assertions-7c3aed" alt="test suite scale"/>
  &nbsp;
  <img src="https://img.shields.io/badge/UI-bilingual%20%C2%B7%20%E4%B8%AD%E8%8B%B1%E5%8F%8C%E8%AF%AD-0f9d8f" alt="bilingual interface"/>
</p>

Most mind-mapping tools want an account, a desktop client, a yearly subscription — or they keep your map on someone else's server. This one is **a single ~493 KB HTML file**: double-click to run, works with the network unplugged, and the map lives on your own disk.

> **Sharing is the install**: send the HTML file to anyone; they double-click it and hold exactly the same full tool — view it, edit it, export from it. No package, no login, no server.
>
> **Try it online**: [mindmap-tool-30044.app.workbuddy.host](https://mindmap-tool-30044.app.workbuddy.host/?sharecode=UQCAJVJB6M) (for a quick look; download and open the file locally for real work so the data stays with you)

---

## Why this one

No install, no network, no trace, no format lock-in — all four, at once:

| Trait | How |
| --- | --- |
| **No install** | One HTML file; double-click and it is a complete application |
| **No network** | Zero CDNs, zero frameworks, zero remote fonts — works offline |
| **No trace** | No account, no analytics, no cloud sync |
| **No lock-in** | Common outlines and XMind come in; 12 formats go out |

---

## The whole picture

From "text I already wrote" to "a document I can hand over" — every step in between is covered.

<p align="center">
  <img src="docs/feature-map.en.svg" width="900" alt="Capability map: input → edit &amp; organize → 12 outputs, on top of three layers of data safety"/>
</p>

---

## What it does

### Input — turn existing text into structure

`Import document`, or just drop a file onto the window. `.md` `.markdown` `.txt` `.json` `.xmind` `.opml` are read directly; Word and PDF work by copy-paste.

Structures it recognizes:

- **Markdown headings** `#` `##` `###` …
- **Numbered outlines** `1.` / `1.1.1` / `一、` / `（一）` / `第 X 章`
- **Indentation and bullets** `-` `*` `•`
- **Arrow flows** written vertically (`A` ↓ `B` ↓ `C`); `↓` `↓↓` `→` `->` `=>` all count, arrows are dropped
- **Tables** `| a | b |`, first column as the node, the rest as children

The preview is a **checkable item tree**: untick an entry and its whole subtree greys out, the footer counts what you excluded, and the confirm button tracks the number of nodes that will really be created.

To extend one node instead of replacing the map: select it, press `Ctrl+V` with text on the clipboard, and a subtree grows underneath.

### Editing — restructure without delete-and-retype

- **Drag to reparent**: drop a node onto another and the whole subtree moves (with cycle detection)
- **Keyboard all the way**: `Tab` child, `Enter` sibling, `F2` rename, arrows to walk, `Alt+↑↓` to reorder
- **Multi-select**: `Ctrl/⌘ + click` to add, `Shift`-drag for a rubber band, then delete / colour / fold / badge the batch
- **Batch text cleanup**: prefix and suffix, strip leading numbering, trim, full-width ↔ half-width, split by delimiter, merge nodes — one undo step for the whole batch
- **Readable undo history**: a sidebar listing what each step did (`delete "Phase 2" and its 1 child`, `batch replace 12 occurrences`); click a row to jump there, and step forward again
- **Notes, semantic badges, node links**: any node can carry multi-line notes, one of ⭐❗❓✅🔒, or a URL / local path — all three land in all 12 exports

### Organizing — usable when the map gets big

Big maps don't hurt because you can't draw them; they hurt because you can't find, move or see them.

- **Outline sidebar**: click to locate, double-click to rename, drag a row to reorder or re-parent, `Alt+↑↓` to nudge
- **Minimap**: the whole map redrawn live in the corner; click to jump
- **Statistics**: level distribution, branch share, badge summary, and last render time — so "is it slow?" becomes a number
- **Search** `Ctrl+F`: highlighted hits, "only matches" mode, **notes are searched too**, case / whole-word / regex, plus batch replace
- **Two layouts**: **horizontal** (mind map, default) and **vertical** (org chart, reporting lines read top-down). Switching changes no data, arrow keys and fold dots follow, and exports use the current layout

<p align="center">
  <img src="screenshots/mindmap-org.en.png" width="780" alt="Vertical (org chart) layout: levels share a centre line, reporting reads top-down"/>
</p>

### Output — 12 formats, notes and badges included

The **Export ▾** panel groups formats by purpose — **Images / Office / Data & interchange** — and the first nine have number shortcuts `1`–`9`.

<p align="center">
  <img src="screenshots/mindmap-export.en.png" width="720" alt="Export panel: three groups, 12 formats"/>
</p>

| Format | Typical use | How notes travel |
| --- | --- | --- |
| **PNG** / **SVG** / **PDF** | Documents, slides, sharp at any zoom | Numbered badge + a note list under the map |
| **PPT · one slide** | The map as vector shapes on one slide, still recolourable | A small grey line inside the shape |
| **PPT · paged outline** | Cover + one page per top-level branch | A small grey line after the node |
| **Word** | Numbered headings by level, keep writing | A grey paragraph one level in |
| **Excel** | One row per node with level / parent / full path | A dedicated "Note" column |
| **Markdown** | A standard `#` outline for notes apps | A `> 备注：…` blockquote — **re-imports onto the node** |
| **XMind** / **OPML** | Continue in XMind / Mubu / Workflowy / Feishu | `notes` / `_note` fields |
| **Standalone page** | Double-click to open, complete **and still editable** | Included in the data |
| **Map data `.json`** | The main backup format; open it later to continue | Included in the data |

One line to remember: **images / SVG / PDF / PPT·one slide** export "what you see on screen"; every other format exports **all nodes**.

---

## Quick start

1. Download `mind_map.html` and keep it in a fixed folder
2. **Double-click it** (or press `Ctrl+O` in a browser and pick it)
3. The first screen offers four ways in

<p align="center">
  <img src="screenshots/mindmap-welcome.en.png" width="780" alt="Welcome screen: from a document / blank canvas / open an existing map / start from a template"/>
</p>

| Entry | When to use it |
| --- | --- |
| **From a document** | You already have notes, an outline or minutes to convert |
| **Blank canvas** | Think by writing, structure as you go |
| **Open an existing map** | Continue a `.json`, or open XMind / OPML |
| **Start from a template** | Reading notes / meeting minutes / project plan skeletons |

Recent versions of Edge, Chrome, Firefox or Safari.

---

## Interface language

The interface is **bilingual (Chinese / English)** — one click on `EN` / `中文` in the top-right corner. The button shows *what it will switch to*, so you never have to guess the current language.

- **Chinese users** get Chinese on first open, with nothing to configure.
- **English users** click `EN` once: the toolbar, menus, context menu, all 14 panels, every toast, and the shortcut cheat sheet switch over. Exported Markdown / Word / Excel / PPT / standalone pages follow too, table headers and captions included.
- **Your own content never changes.** Node text, notes, links and the map names you typed stay exactly as written, however often you switch. Switching back to Chinese restores every label byte-for-byte — originals are kept on the DOM instead of reverse-looked-up, so an ambiguous word can never leak into the wrong panel.
- Language is a **preference**: remembered locally, never written into the map data, and it never pushes an undo step.
- On a still-untouched canvas the central topic follows the language too (“中心主题” ↔ “Central topic”).
- Every screenshot in this file was taken with the UI **set to English** (`screenshots/*.en.png`); the Chinese README uses the Chinese ones, so the pictures always match the language you are reading.

---

## Phone and tablet

Below 760 px the shell changes instead of squeezing the desktop UI:

- The top bar keeps **☰ + title**; everything else slides in from the left in a drawer that scrolls to the very bottom
- The **FAB "＋"** opens quick actions — child / sibling / edit / delete children / delete node — so no physical keyboard is needed
- The canvas owns every gesture: one-finger pan, pinch zoom, long-press menu; long-press no longer triggers the system "copy / search / translate" strip
- Drawer and panel are overlays, so they **never make the header taller** and the canvas keeps the full height

---

## Appearance

Changes are remembered automatically.

- **9 themes** (Day, Minimal white, Fresh mint, Warm paper, Business navy, Ink wash, Premium grey, Night, Midnight blue)
- **8 palettes** · **4 backgrounds** (solid / dots / grid / ruled)
- **6 font stacks** (Segoe UI + Microsoft YaHei by default) · **4 sizes** · **3 weights**
- **5 node widths**, including **auto** (short nodes tighten, long ones grow; layout and exports follow)

Want a different mood: Fresh mint + Candy macaron for something young; Business navy + Corporate navy for something formal; Warm paper + Earth warm for long sessions. **Reset appearance** restores the factory look without touching content.

---

## Data safety

For a local tool the real fear isn't a crash — it's opening it and finding the map gone. So there are three layers:

| Layer | Where | Protects against |
| --- | --- | --- |
| ① **Browser storage** | localStorage, isolated per tab | Closed tab, crash, restart |
| ① plus **history snapshots** | Same storage, up to 10 full trees | Accidental delete or mess — even after closing |
| ② **Auto-backup folder** | The `auto_backup` folder you choose, real HTML files | Switching browsers, clearing cache, deletion |
| ③ **Manual export** | `.json` / `.html` / other formats | Sharing, archiving, moving machines |

A few deliberate decisions:

- **Tabs never bleed into each other**: every tab owns a slot, new pages start blank, and nobody inherits a map someone is still editing
- **Backups never overwrite good data**: when the node count drops by more than half, a timestamped copy is written instead of clobbering the last good one
- **Undo is capped twice**: 80 steps *and* 6 MB, always keeping at least one
- **A full storage rescues itself first**: expired slots are cleaned according to the 7-day rule and the write retried — never touching your own slot or slots still in use

---

## Under the hood

**10,258 lines, ~582 KB, zero external dependencies.** A few things worth pointing at:

**Layout**
A hand-written tidy-tree (left→right) and a vertical org chart (top→bottom) share one coordinate convention (`x` = node's left edge, `y` = vertical centre), so positioning, bounds, fit-to-window, minimap and rubber-band selection all run from a single implementation. Collapsed subtrees never enter layout.

**Rendering**
The main path stays **synchronous** — 60+ call sites read results right after `render()`. Throttling exists only on the high-frequency input path: `renderSoon()` coalesces same-frame requests with `requestAnimationFrame`. Nodes mount through a `DocumentFragment`, and a real-Chrome `MutationObserver` measured DOM mutation batches per render dropping from **1 + 607 to 2**. During a drag only `transform` and the selection box change; a 10-frame pan measured **0** mutation batches.

**File formats**
Office and XMind files are **hand-written OOXML / ZIP** (an internal store + CRC32 writer, docx / xlsx / pptx parts and relationships, XMind's `content.json`). Reading `.xmind` uses a **hand-written DEFLATE decompressor** (dynamic and static Huffman) — no third-party library. OPML goes through the browser's `DOMParser`; the subtle part is that newlines inside attribute values must be encoded as `&#10;`, or XML whitespace collapsing silently eats multi-line notes.

**Consistency**
The search switches (`Aa` / whole word / regex) are not three matchers: one `findCompile()` turns "keyword + switches" into a single `{test, replace}` shared by search and replace, so they can't drift. An invalid regex degrades to literal matching and marks the input, instead of throwing.

**How it is verified**
jsdom loads the real page and runs end-to-end interaction tests — **38 suites, 2850 assertions** — then real Chrome headless covers what jsdom cannot: computed styles and true layout, real `PointerEvent` pinch zoom with anchor maths, real `elementFromPoint` hit testing (proving the mobile drawer isn't swallowed by the scrim), minimap canvas pixels, and genuinely filling localStorage to 5 MB to exercise quota recovery. Exports are unpacked and checked independently with Python `zipfile` and `python-pptx`.

Every fix also gets a **counter-proof**: temporarily restore the old behaviour and confirm the same assertions **really turn red** — otherwise the test had no teeth (23 falsification scripts, each verifying the restored file with sha1), plus 21 real-Chrome probes for what jsdom cannot measure. One command: `python _test/runall.py`.

---

## Project layout

```
mind_map.html            ← everything lives in this one file (double-click to run)
docs/USAGE.md            ← full user guide (English)
docs/USAGE.zh-CN.md      ← full user guide (简体中文)
docs/feature-map.en.svg  ← capability map (English)
docs/feature-map.svg     ← capability map (中文)
docs/移动端适配改造方案.md ← mobile roadmap and design notes (中文)
screenshots/             ← README images: *.png (Chinese UI) / *.en.png (English UI)
CHANGELOG.md             ← release notes, newest first (English)
CHANGELOG.zh-CN.md       ← release notes, newest first (中文)
优化分析报告.md           ← per-version analysis, measurements and decision records (中文)
_test/                   ← test sources: jsdom suites, Chrome probes, falsification scripts
_test/_artifacts/        ← their output (snapshots, probe pages, exports, profiles) — git-ignored
.gitignore               ← keeps local data and test output out of the repo
```

Nothing in `_test/_artifacts/` is needed to run the tests: every snapshot, probe page and
exported sample there is regenerated on the next run, so the folder can be deleted at any time.

---

## FAQ

**Can Word / PDF files be imported directly?**
Not as files. Select all, copy, and paste into the import box — recognition is identical to `.md` / `.txt`.

**Windows says "Your Internet security settings prevented these files from being opened"?**
That is Windows tagging files that came through a browser download, nothing to do with the file. The export panel's "Save with the Save As dialog" writes straight to disk without the tag. For an existing file: right-click → Properties → tick **Unblock**.

**Several tabs open — do they interfere?**
No. Each tab has its own storage slot and starts blank, and each reload restores its own map.

**Can several people collaborate?**
No — it is a local single-user tool. Collaboration means exporting `.html` or `.json` and sending it over.

**How big can a map get?**
600+ nodes stays fluid in testing. Beyond a thousand, collapse to 2–3 levels before editing or exporting.

---

**Full documentation: [docs/USAGE.md](docs/USAGE.md)** — every export format in detail, arrow flows, import preview ticking, workspace tabs, appearance combinations, all shortcuts, and more FAQ.
