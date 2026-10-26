# MindMap · 思维导图工具 · Changelog

[简体中文](CHANGELOG.zh-CN.md) ｜ **English**

> Single-file, zero-dependency tool: `mind_map.html` — double-click and run.
> Newest first. Every round ships the same four steps: a dedicated jsdom suite → a counter-proof (restore the old behaviour, prove the tests turn red) → full regression → a real-Chrome headless recheck.
>
> Notes: for a compact record the English file gives the highlights of every release; [`CHANGELOG.zh-CN.md`](CHANGELOG.zh-CN.md) is the full, item-by-item original. Release numbers correspond one to one.
>
> Naming: the product is **思维导图工具** in Chinese and **MindMap** in English; the file name is **`mind_map.html`**. v39 renamed the old `思维导图工具.html` to `mind_map.html`; v40 settled the Chinese and English product names.

---

## v40 · 2026-10-25 (internationalisation: switchable Chinese / English UI)

**Names settled.** The product is **思维导图工具** in Chinese, **MindMap** in English, and the file stays **`mind_map.html`**. The toolbar `<h1>` and the window / tab title follow the current language; XMind metadata `creator.name` is always `MindMap` (a machine-read identifier, not UI text).

**The tool itself is now bilingual, switched by one button.** Chinese is the default; international users click once to get English.

- New `EN` / `中文` button at the far right of the toolbar. The label says **what you will get when you click it** (the Gmail / GitHub convention), not the current language.
- Translated: the toolbar and every dropdown item, the context menu (33 entries), all 14 panels (titles, descriptions, buttons), every toast (including undo step names), the 31-row shortcut sheet, template cards and skeleton previews, the empty-canvas hint, the import preview summary, the export panel, the auto-backup panel, workspaces, version history and the steps panel.
- **Never translated**: node text on the canvas, node notes, node links, your own map / file names, and anything you pasted. A skip list covers `#nodes`, `#ol-body`, `#tpl-grid`, `#paste` and friends, so chrome moves and user content does not.
- **Switching back to Chinese restores byte for byte.** The first translation stores the original on the node (`__zh` / `__zhAttr`) and switching back puts it straight back instead of reverse-looking-up the table — otherwise a word like `标准` (both a font size and a width) or the four separate `清除` buttons would cross over after one round trip.
- **Nine full-HTML entries on a separate table.** Where a sentence wraps `<b>` / `<br>` / `<code>`, translating node by node breaks the phrase and English word order will not join up, so those nine use `data-i18n-html` whole-fragment replacement (first translation stored in `__zhHtml`).
- **English plurals.** A value may be written `singular||plural` and the segment is chosen by whether the count variable is 1 (`已删除 1 个子节点` → `Deleted 1 child node`).
- **Language is a preference, not data.** It lives in `savePrefs()` as `lang`, never in `serialize()`, and never pushes an undo entry; `normalizePrefs` validates with `/^(zh|en)$/` and falls back to `zh`; the choice survives a reload.
- **A untouched canvas follows the language.** For a brand-new user who has not done anything yet (empty undo stack, a single node, no note / link / mark) the central topic switches between `中心主题` and `Central topic`. The moment any edit happens it counts as user content and is never touched again.
- **Exports follow the language too.** Column headers in Markdown / Word / Excel / PPT and standalone HTML (`层级 / 编号 / 节点 / …`), the `备注：` and `已完成 N / M` labels, file-name suffixes, and the OPML / XMind fallback names all come out in English when the UI is English.

**Vocabulary engineering.** 905 EN entries + 9 whole-HTML entries. A hand-written state-machine lexer (comments, single and double quotes, template strings with nested `${}`, regex literals) pulled 926 candidates out of 9112 lines, deduplicated to 814, split into 21 JSON shards by area, and `_i18n_build.py` merges them, checks for duplicates and for "value equals key", then injects them at the `/*<<EN>>*/` marker. **No regex scanning of the source** — the file contains regexes with quoted character classes such as `/^[a-z]:[\\/]/i`, and a regex-based scan swallows the following block into a string (the first attempt produced 38 phantom entries).

**Four real defects fixed on the way** (the first two belong to i18n itself; the last two are older ones that only surfaced while shooting the English screenshots).
1. The skip list excluded `textarea` wholesale, so its `placeholder` was never translated → it now skips the subtree but still translates the element's own attributes.
2. The shortcut panel builds itself once and guards on `data-built`, so it did not redraw after a language switch → `applyLang` clears the flag and rebuilds.
3. **The export panel ran the format name and its description together**: both are inline `span`s, which Chinese survives on glyphs alone but English reads as `Image .pngHigh-resolution bitmap…` → both are `display:block` now, one line each.
4. **Short labels were misread as clipped text**: `.txt` shrinks to its content, so a two-letter node like `QA` is only 20 px wide; measuring at that same 20 px wrapped it to two lines and stuck the bottom-left `⋯` marker on a label that plainly fits (English is full of `QA` / `R&D` / `KPI`) → `txtClipped()` now returns false outright for single-line text with no horizontal overflow.

**Verification.** New jsdom suite `i18n.js` with **71 assertions** (table integrity / `tr`-`trf` semantics / per-panel scan for leftover Chinese / user content untouched / round-trip does not cross over / untouched canvas / preference persistence / survives reload / English does not burst buttons); **4 counter-proofs** (`tr` always returns the source → 21 red; the two-layer "user content is never translated" guard removed → 2 red; the empty-undo-stack gate removed → 1 red; `savePrefs` stops recording `lang` → 4 red), each followed by a sha1 check that the file was restored; full regression **38 suites, 2850 assertions, 0 failures**; a new 21st real-Chrome probe group with **37 assertions** (no toolbar overflow, no clipped labels, real `elementFromPoint` hits, template cards and the shortcut table do not overlap, three switch round-trips leave no residue, still exactly 14 overlays, export descriptions sit under their format name, short labels are not misread as clipped while genuinely long text still keeps its marker). Two more counter-proofs (`_falsify_fmtstack.py`, `_falsify_clipshort.py`) keep the last two fixes honest.

**Screenshots now come in two languages.** `_test/_shot.py` takes a `lang` argument (`python _test/_shot.py [zh|en|both]`) and writes the English set as `screenshots/*.en.png`, matching the `feature-map.en.svg` naming. The English README and guide reference only that set, the Chinese ones keep the originals — so the UI language in a picture always matches the language of the document around it. The sample tree is translated too: node text is user content that the tool deliberately never rewrites, so an English screenshot needs English sample data. (The capture also hides the bottom-left guide and does a final `render()` after fonts settle, so the shot shows the settled layout rather than the first frame.)

---

## v39 · 2026-10-25 (rename to mind_map, bilingual docs, restructured guide)

**Rename.** `思维导图工具.html` → **`mind_map.html`**. The product name is now `mind_map` everywhere: window title, toolbar heading, tab title (`mind_map · <root node>`), XMind metadata (`creator.name`), and the "choose this folder" hint in the auto-backup panel. All 92 test/probe references follow the new file name. The storage keys (`mindmap-studio-v1`) are deliberately untouched, so existing local maps are inherited instead of "disappearing".

**Bilingual documentation.** Every reader-facing document now has a Chinese and an English edition, each with a language switch on its second line:

| Chinese | English |
| --- | --- |
| `README.zh-CN.md` | `README.md` (default) |
| `docs/USAGE.zh-CN.md` | `docs/USAGE.md` |
| `CHANGELOG.zh-CN.md` | `CHANGELOG.md` |
| `docs/feature-map.svg` | `docs/feature-map.en.svg` |

**User guide restructured** (`docs/USAGE.zh-CN.md` / `docs/USAGE.md`). The old guide mixed editing, viewing and saving in single chapters and had real defects:

- `## 二之二、从模板开始` — a hand-made "chapter two and a half"; templates now live in **2.3** under Getting started;
- the canvas navigation table was cut in half by a paragraph, so "fit to window / 100% / clear selection" rendered as prose instead of table rows;
- the outline, minimap, statistics, search and batch tools were all under "Editing nodes", and node links / workspace were under "Saving and backups";
- one cell contained an unescaped `|`, breaking the table;
- the technical chapter still quoted 8865 lines / 489376 bytes and 36 suites / 2719 assertions.

New shape — one H1, eleven numbered H2 chapters, `x.y` H3 subsections, no skipping, no "chapter two and a half":

```
1 Overview            5 View & navigation   9 Keyboard & touch
2 Getting started     6 Appearance         10 FAQ
3 Editing             7 Export             11 Implementation notes
4 Finding & organizing 8 Saving & recovery
```

Content moved to where a reader would look for it: notes / badges / links into **Editing**; search / replace / cleanup / outline / statistics into **Finding & organizing**; full screen / layouts / minimap into **View**; storage / snapshots / undo history / workspace / backup into **Saving**. Every section is reachable from a table-of-contents table with per-chapter summaries.

**README rebuilt for the international audience**: English default with a Chinese edition, switcher on line 2, capability map in the matching language, "why this one" table, feature groups, quick start, mobile section, data safety, implementation notes, project layout, FAQ.

**Verification**: full regression after the rename — 37 suites, 2779 assertions, 0 failures. No product logic changed beyond the five name strings.

---

## v38-doc · 2026-10-25 (README rebuilt as a GitHub front page; full guide split out)

No product code touched — documentation structure and screenshots only.

- `README.md` went from 807 lines / 72 KB / 67 headings to **236 lines / 15 KB** with a strict `H1 ×1 + H2 + H3` hierarchy: one-line pitch → badges → hero image → "why this one" → capability map → four feature groups → quick start → mobile → appearance → data safety → implementation → project layout → selected FAQ.
- The old manual was preserved in full as `docs/USAGE.md` (800 lines).
- Screenshots re-shot in real Chrome: framing gained a `maxs` scale cap (a hard-coded `Math.min(…, 1)` refused to fill the frame), bottom padding 18 → 72 (leaf nodes were being cropped because the node-height fallback 40 was below the real height), and the sample tree went from 15 to 9 leaves (tree height 742 px against 416 px of usable height forced scale 0.56, blurring the text).
- `docs/feature-map.svg` gained the mobile work from v36–v38.

---

## v38 · 2026-10-25 (round 40 — root causes of the three phone issues)

The user reported that the v37 fixes "still don't work". Each one turned out to be **fixed in the wrong place**:

1. **Drawer bottom buttons invisible and unscrollable** — a stacking-context inversion, not a missing `overflow`. `#navdrawer` is a child of `<header>` (`z-index:20`), so its own `z-index:90` only applied inside that context and painted at 20, while `#navscrim` (a child of `<body>`, `z-index:85`) covered the whole header and swallowed every touch. Scrim dropped to **19**; drawer height switched to `100dvh`; the FAB (57) is hidden while the drawer is open.
2. **Long-press still showed the system "copy / search / select all / translate" strip** — the selection ban lived in `body.touch` + `@media(max-width:760px)`, and some Android kernels report neither, so both breakpoints missed. The ban is now in the **base style, unconditional**, covering `#stage` and all descendants with `-webkit-` prefixes plus `-webkit-touch-callout`, with `selectstart` interception as a second line of defence and `textarea` exempted.
3. **Context menu cut off at the bottom** — `calc(100vh - 16px)` counted the address bar, and `showCtx` could compute `top = 8` when the menu was taller than the viewport. Now `100dvh`, with `maxHeight` pinned from `innerHeight - 16` before measuring, and `<kbd>` hidden on phones.
4. **Empty-canvas hint on mobile** — the previous round hid `.hint` (the shortcut guide); the sentence the user meant was `#empty-tip`. On narrow screens only the central topic now remains.

Verification: jsdom 60/0, counter-proofs 21/21 red with sha1 restore, full regression **2779/0**, real Chrome 55/0 — including `elementFromPoint` proving the drawer (not the scrim) receives the touch, which a `scrollTop` assignment could never detect.

---

## v37 · 2026-10-26 (round 39 — three phone issues + FAB quick actions)

1. Drawer items were being squeezed: it is a `flex-direction:column` container, so `.btn-group` / `.divider` shrank to the container height and no scrollbar ever appeared. Fixed with `flex:0 0 auto`, `overscroll-behavior:contain` and safe-area padding.
2. Native selection strip: `user-select:none` + `-webkit-touch-callout:none` on the canvas and nodes, plus clearing any live selection in `showCtx()`.
3. Context menu: `max-height` + `overflow-y:auto`, so the last items stay reachable.
4. **FAB became a quick-actions panel**: child / sibling / edit / delete children / delete node, disabled states computed from the selection, closed by scrim tap or `Esc`.

Verification: jsdom 43/0, counter-proofs 11/11 red, full regression 2762/0, real Chrome 37/0.

---

## v36 · 2026-10-26 (round 38 — mobile navigation skeleton)

Diagnosis: the canvas was already touch-ready (C1); the reason phones "couldn't open it" was that the **shell never collapsed** — 42 toolbar controls folded into a very tall header and `syncHeader()` wrote that height into `stage.top`, squeezing the canvas away.

- Desktop keeps `display:contents`, so `<nav>` still merges into the header and nothing changes;
- Below 760 px the nav becomes a **fixed left drawer** that never grows the header (no `syncHeader()` call, verified with a spy);
- `☰` toggles the drawer, with scrim, `Esc`, focus trap and `aria-expanded`;
- Bottom-right **FAB** solves "no physical keyboard";
- Fixed a real bug found on the way: the drawer's click handler had no `mnav` guard, so any toolbar button bubbled into `closeNav()` and stole focus to `☰` (desktop a11y regressions in o5o6/n7).

Verification: jsdom 28/0, counter-proofs 10/10 red, full regression 2747/0, real Chrome 26/0.

---

## v35 · 2026-10-20 (correction to v35)

The user corrected the default: arrow-flow text now generates **sibling steps by default** (checkbox off) and only chains when "↓ Chain arrow flow" is ticked. The choice is sticky and also applies to paste.

## v35 · 2026-10-19 (round 37 — arrow-flow text)

A column of steps separated by `↓` / `->` / `=>` becomes a subtree: arrow-only lines are dropped (including `- ↓`, `1. ↓` and repeats), inline arrows such as "temperature ↓" stay untouched, headings keep their `#`-based structure, and tables / code blocks / notes are never affected.

## v34 · 2026-10-19 (round 36 — node links, N3)

A dedicated `link` field for URLs and local paths. `Ctrl+click` remains "add to selection"; links open via the chain badge or `Alt+click`. Local paths are copied (browsers may not open `file://`) with the reason explained. Dangerous schemes (`javascript:` / `data:`) are refused at storage, opening and every export. Links become real hyperlinks in Markdown / Word / HTML and are preserved in Excel, PPT, XMind, OPML, PNG, SVG and print.

## v33 · 2026-10-18 (round 35 — undo history panel, N7)

`Ctrl+Z` only knows "one step back". The panel lays out the history with human-readable names ("colour 3 nodes", "batch replace 12 occurrences"), and clicking a row equals pressing undo that many times — verified against the step-by-step path. 28 `pushUndo` call sites now all carry names, enforced by a static assertion.

## v32 · 2026-10-18 (round 34 — workspace packaging, C5)

The 24 per-tab slots always existed but had no UI. Workspace lists them, switches between them, saves one separately, deletes one, and exports/imports the whole set as one `.json`. Import rules: add-only, never evict, always use the current timestamp.

## v31 · 2026-10-18 (round 33 — vertical layout, C3)

A layout dropdown: horizontal (mind map, default) / vertical (org chart). Both layouts share one coordinate convention, so positioning, bounds, fit-to-window, minimap, rubber band and all four export paths needed no branching. Vertical: rows use the tallest node of the level, siblings align on a centre line, parents centre over their first and last child.

## v30 · 2026-10-18 (round 32 — touch and tablets, C1)

Pointer Events migration: one-finger pan, pinch zoom anchored at the finger midpoint, long-press menu, double-tap rename, drag reparenting with a wider threshold, and `touch-action:none`. The mouse path is untouched (`pointerType === 'mouse'` returns early).

## v29 · 2026-10-18 (round 31 — node completion state, N2)

A completion flag propagated through `serialize` and all 12 exports, with a mapping table defined before implementation.

## v28 · 2026-10-16 (round 30 — data-safety hardening, C4)

Distinguishes "storage full" from "storage unavailable" and self-rescues once by cleaning expired slots; undo stack capped at 80 steps **and** 6 MB; history snapshots (up to 10, 3-minute interval); slot TTL and pruning.

## v27 · 2026-10-15 (round 29 — shortcut sheet and accessibility, O5/O6)

A grouped shortcut sheet (29 entries, six groups) with a browser-conflict review and aliases for risky keys; dialog semantics, focus trapping, `aria-modal`, and `Esc` handling across all 14 overlays.

## v25 · 2026-10-14 (round 28 — organizing pack, N4/N6/N5)

Outline drag reordering, the batch text tool, and semantic badges — all in the "organizing" flow, one regression pass.

## v24 · 2026-10-14 (rounds 25–26)

First two batches from the analysis report: O1+O4+O7 then N1+O2 — import preview, prompt levels, and related polish.

## v23 · 2026-10-13 (round 24)

Six batches in one pass: A6+A7+A8 → B4 → B5+B8 → B6+B7 → B9 → C2 — export panel grouping, search options, outline and minimap work, notes (C2).

## v22 · 2026-10-07

Fixes: underscores inside words (`run_id`) are no longer eaten by the italic rule; table and inline-cleanup edge cases.

## v21 · 2026-10-07

`↑↓` wraps around within a level; paste text onto a selected node to grow a subtree.

## v20 · 2026-10-07

Depth, trunk-only and expand-all unified into one state; `+` on a truncated node expands only that branch.

## v19 · 2026-10-01

"Reset appearance" button; dropdowns release focus after change so the wheel can't alter them by accident.

## v18 · 2026-10-01

`↑↓` crosses parents within a level; highlight colour propagates to descendants and to edges; darker PNG borders.

## v17 · 2026-10-01

`Tab` / `Enter` swapped to the common convention (`Tab` child, `Enter` sibling); panels take `Tab` / `Enter` while open.

## v16 · 2026-09-30

Search (`Ctrl+F`, highlight, cycling, auto-expand, "only matches"), keyboard navigation, per-node colours, undo/redo.

## v15 · 2026-09-30

Optimization analysis document produced (analysis only): 5 experience items (A) and 3 advanced items (B).

## v14 · 2026-09-30

Typography rework verified by pixel hashing: 6 fonts × 4 sizes × 3 weights genuinely take effect; weights collapsed to 400 / 700; `data-fs` renamed to `data-nodefs` (a name clash kept the toolbar hidden).

## v13 · 2026-09-29

Unified fonts and weights, Segoe UI + YaHei by default, font dropdown; `saveAs` blob handling fixed for jsdom.

## v12 · 2026-09-29

Exports go through the File System Access "Save As" path so Windows no longer marks downloaded files as "from the Internet"; PNG grabs the handle before `toBlob`.

## v11 · 2026-09-29

True immersive full screen: the toolbar hides too, revealed by moving the pointer to the top 8 px; asynchronous `fullscreenchange` handling avoids flicker.

## v10 · 2026-09-29

Double-click editing no longer loses the caret: auto backup stopped committing mid-edit.

## v9 · 2026-09-29

Per-page canvas isolation via `slotKey(slotId)` — several open pages no longer bleed into each other.

## v8 · 2026-09-28

Full screen, collapsible guide, drag-to-reparent; fixed dangling edges on truncated nodes.

## v7 · 2026-09-28

Auto backup into an `auto_backup/` folder as real HTML files, using the File System Access API with the handle kept in IndexedDB.

## v6 · 2026-09-27

PPT "one slide", depth dropdown, clear canvas with confirmation and undo; themes / backgrounds / palettes expanded to 9 / 4 / 8.

## v5 · 2026-09-27

Five fixes: PPT one-slide entry, auto-backup button position, toolbar order, `.xmind` opening, theme expansion; `README.md` added.

## v4 · 2026-09-27

Export to HTML / PNG / MD / PDF / Word / Excel / XMind, then PPT — hand-written ZIP store + CRC32, OOXML parts, XMind format; PNG downsampling; PDF through `print()`.

## v3 · 2026-09-27

The general-purpose mind-mapping tool: generate from a document / blank canvas / delete all children; node CRUD; `parseDoc` (Markdown headings, numbered outlines, Chinese outlines, list indentation, line-by-line fallback).

## v2 · 2026-09-27

First chapter notes: node CRUD, three bug fixes.

## v1 · 2026-09-26

Chapter 1 study notes turned into a mind map — the starting artifact.
