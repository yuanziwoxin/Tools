# MindMap · 思维导图工具 · Changelog

[简体中文](CHANGELOG.zh-CN.md) ｜ **English**

> Single-file, zero-dependency tool: `mind_map.html` — double-click and run.
> Newest first. Every round ships the same four steps: a dedicated jsdom suite → a counter-proof (restore the old behaviour, prove the tests turn red) → full regression → a real-Chrome headless recheck.
>
> Notes: for a compact record the English file gives the highlights of every release; [`CHANGELOG.zh-CN.md`](CHANGELOG.zh-CN.md) is the full, item-by-item original. Release numbers correspond one to one.
>
> Naming: the product is **思维导图工具** in Chinese and **MindMap** in English; the file name is **`mind_map.html`**. v39 renamed the old `思维导图工具.html` to `mind_map.html`; v40 settled the Chinese and English product names.

---

---

> **Planned, not implemented · 2026-11-07 (round 63)**: the font-styling overhaul for node text, boundary titles and group-frame titles has finished a **read-only assessment with three prototypes**. Option B ("plain-text projection + inline runs") is recommended, split into P1-P3; see [docs/文字样式改造方案.md](docs/文字样式改造方案.md) for the design and four prototype figures. **No product code changed** and no verification round was run. The same round created the repository's first baseline commit (v49-v55 plus docs, screenshots and test sources). The online release is still v48.

---

## v55 · 2026-11-06 (a zoomed-in preview drags as one piece; frame edges step around image-grown cards)

Two reports: "after zooming the hover preview far enough, holding the left button moves the canvas but not the picture", and "after inserting an image, the enlarged node covers the boundary or group frame lines".

**Why the picture stopped moving:** `#imghover` still had `max-width:min(760px,64vw)` while its `<img>` had no cap, so a 1.5× zoom made the image overflow the panel. v54 had set `#imghover img{pointer-events:none}` to steer wheel events to the panel, so real `elementFromPoint` in the overflow area hit `HEADER`, `btn-hist` or `#stage` — the press never reached the preview layer and only the canvas panned. (The first diagnostic sampled `y=-44`, outside the viewport and hitting `null`; clamping to the real viewport exposed it.) The panel is now `width:max-content; max-width:none` so it always wraps the whole image; the image is `pointer-events:auto` and `draggable="false"`; `imgHoverFit()` still caps the first frame to the viewport. Pointer-anchored zoom, the 140ms dismissal grace and `hovPinned` are unchanged.

**Which frames actually got covered:** not ordinary ones. A node's own boundary (`BD_PAD=20`) and a normal group frame (`GB_PAD=36`) keep their gap, and adding an image after drawing a frame re-renders it. Across a nine-case matrix only one shape reproduced: a **cross-branch group frame**, whose bounding rectangle necessarily cuts through the middle branch's cards (the geometry conflict already noted in v48). An image grows a card from 46px to 138px, making the collision much more likely; the minimal case is `s7` (group `A2+C`, image on `B1`). The user chose "**expand the frame to wrap the card**" rather than splitting the frame per branch or refusing the group.

**Fix — expand only, never shrink, from one shared geometry source:** `FRAME_EDGE_BAND=4` and `FRAME_KEEP_M=8`; `frameOuterRect()` → `keepOutFrameRect()` pushes any edge outward past cards crossing that edge band (plus 8px), iterating up to 6 passes. `bdFrameBox()` / `gbFrameBox()` are now the single source used by on-screen frames, PNG/SVG exports, relationship endpoints, label-occlusion rects and summary bracket outer edges — so no consumer can draw a different shape. `s7` grows from `656×414` to `656×488`, and real Chrome asserts `__NO_SHRINK()` for every frame. One regression was caught first: routing the summary's **sibling span** through `frameBox()` pulled unrelated branch cards into the bracket and made the v49 counter-proof go falsely green, so the span keeps `contentRect() + framePadOf()` and only the outer edges use the shared boxes.

Verified with `_test/v55.js` (46), `_falsify_v55.py` (10 counter-proofs, all bite), `_chrome_v55.py` (46 assertions: six real wheel notches, real hit testing in the old overflow area, synchronous preview+canvas drag, nine image/layout/width scenarios with zero overlap and a reproducible overlap when avoidance is disabled), `_falsify_chrome_v55.py` (2 real-Chrome counter-proofs), and the full run: **49 suites / 3770 assertions / 0 failures**. Older falsifications and Chrome probes spot-checked green. Product: **13,493 lines / 774,479 bytes (~756 KB)**, sha1 `25fe31a6d173`; the EN glossary is still 1011 entries.

---

---

## v54 · 2026-11-05 (wheel zoom no longer makes the hover preview vanish)

Report: "dragging the hover preview now moves both the big picture and the canvas behind it, but a few wheel notches make the big picture disappear."

**Root cause, measured in real Chrome** (CDP real wheel events plus real `elementFromPoint` hit testing): every notch of `imgHoverZoom()` called `imgHoverPlace(th)` and re-anchored the panel to the **thumbnail**. As the panel grew it was re-centred vertically and, once it hit `max-width:760px`, flipped to the other side in one jump (x 609 → 260 at 1280px wide — 349px). The pointer fell out of the panel after **one** zoom-in notch on the right edge, or three zoom-out notches at the bottom. The old `mouseleave` handler then called `imgHoverHide()` immediately; subsequent wheel events landed on `#stage` and panned the canvas while the zoom factor froze. A second trap: `#imghover img` is `pointer-events:none` and overflows the width-capped panel, so anchoring on the `<img>` rect still leaves the pointer on non-hittable area — the anchor has to be the **panel** rect, which is what actually receives events.

**Fix:** `imgHoverZoom(f, clientX, clientY)` measures the panel rect before and after resizing and shifts it by a pure-geometry `hovAnchorShift()` (fractions clamped to `[0,1]`), keeping the pointer at the same relative spot inside the panel; it never re-anchors to the thumbnail. `mouseleave` now uses the 140ms grace and rechecks `hovPointerInside()` (panel **or** overflowing image) before hiding; the wheel cancels any pending dismissal; capture-phase `mousemove`/`pointermove` track the pointer; `hovPinned` stops the async high-resolution swap from yanking a zoomed/dragged panel back. Zoom still clamps to `[0.2, 8]`, `render()` still dismisses immediately, sub-3px still counts as a click, and dragging still pans the canvas.

Verified with `_test/v54.js` (51), `_falsify_v54.py` (10 counter-proofs, all bite), `_chrome_v54.py` (36 assertions over four real-Chrome scenarios that all failed before), `_falsify_chrome_v54.py` (3 real-Chrome counter-proofs), and the full run: **48 suites / 3724 assertions / 0 failures**. Older Chrome probes (v47 / v50 / v51 / v52 / v53) spot-checked green. Two cases of "false green" were caught and fixed in the tests first: four jsdom assertions shared a guard or measured the unaffected `dx`, and the Chrome probe read top-level `let tx/ty` as `window.tx/ty` (always 0), making the canvas-pan check vacuous.

---

---

## v53 · 2026-11-05 (relationships stop knotting; hover preview is draggable)

Two reports: "the relationship lines are knotting now" and "hovering an image preview, holding the left button and moving does not move the preview or the canvas behind it".

**Why the lines knotted** (measured, not guessed): `linkCurve()` opened the two control points along the chord by `lead = 0.3·chord + 0.55·|bend|`. A cubic Bezier's control polygon `P0 → M1 → M2 → P3` has to run forward; the gap between the two control points along the chord is `chord − 2·lead`. Once v51 let `bend` reach ±900, dragging past roughly 150 turned that gap negative — the polygon folds back and the curve loops into a knot. Sampled over 320 combinations of chord × bend × shift: **242 self-intersecting before, 0 after**. Sweeping the ratio, 1.0× chord is still clean and 1.2× always crosses, so 0.42 leaves a two-times margin.

**Fix:** `lead` is now a fixed fraction of the chord — `clamp(chord × 0.3, 6, chord × 0.42)` — with the cap held below half the chord. A large bend now produces a long, narrow detour: narrow, but never doubling back. `bend` and `shift` still clamp at ±900, so the avoidance range is untouched (at bend=900 the arc apex sits 675px off the chord midpoint).

**Hover preview dragging:** the hover layer had no pointer listeners at all; v52 only did the full-size overlay. Now holding the preview (or its caption) drags it, and the canvas behind pans by the same distance; under 3px it still counts as a click and opens the full view; a brief `mouseleave` during a drag no longer dismisses it; right-click is ignored; view changes never enter the undo stack or mark the document dirty.

**A real defect found on the way:** the first cut stored the preview's offset from its anchor, so a wheel zoom would keep it in place. Real Chrome measured a 170px jump — the canvas and the thumbnail move together during a drag, so the offset relative to the thumbnail does not change, and adding it again double-counted. The offset was removed; positioning is computed from the anchor only.

Verified with `_test/v53.js` (38), `_falsify_v53.py` (8 counter-proofs, all bite), `_chrome_v53.py` (23, including self-intersection detection on the **rendered SVG paths** by arc-length sampling), and the full run: **47 suites / 3673 assertions / 0 failures**.

---

---

## v52 · 2026-11-05 (dragging in the image preview moves the map too)

User: "when previewing an image, holding the mouse on the previewed image and moving should drag the whole mind map and the preview image — that makes the picture easier to look at."

**Why it did not drag:** the overlay's pan only wrote `scrollLeft / scrollTop` on `#iv-wrap`. Once the image is fitted to the window there is no overflow, no scrollbar, and `scrollLeft` stays 0 — the pointer moved, the picture did not. On top of that the backdrop was a 44% dark wash, so even a moved canvas would have been invisible.

**Three changes:**
- **Lighter backdrop** — `#imgview` drops to `rgba(16,24,40,.16)`, and to `.06` while dragging, so the map behind stays visible; every other dialog keeps the 44% wash;
- **Both sides move** — the image keeps using `scrollLeft` when there is overflow, and whatever the scroll cannot absorb (image smaller than the frame, or already at an edge) is covered by a temporary `ivOff` transform capped at 35% of the frame; the canvas accumulates the same delta into `tx / ty`, so image and map travel the same direction by the same distance;
- **Clean edges** — zoom / fit / close reset the offset (otherwise it leaks into the zoom anchor or survives into the next open); the frame's empty area is draggable too; right-click is ignored; and a pan never touches the undo stack or marks the document dirty.

Also fixed: the drag delta used to be "two `worldPt()` calls subtracted", which depends on the canvas rect, `tx`, `ty` and `scale` — any of them changing mid-drag (scrollbar appearing, overlay closing, wheel zoom) ate or invented a whole chunk of movement. The basis is now pinned at pointer-down.

Verified by `_test/imgpan.js` (33 assertions), `_falsify_v52.py` (7 fuses), and a real-Chrome probe that checks the no-scrollbar path and the zoomed-in scrollbar path separately.

---

## v51 · 2026-11-04 (link dragging opens up to ±900; short links become visible)

Two user reports: "the relationship line can only be dragged a little, so even at the limit it cannot get out of the way of frames or labels"; and "when the two endpoints are close, the line barely shows, let alone drags".

**Measured root cause** — geometry was clamped, not just stiff: on a 20px chord, dragging `bend` from 100 to 900 moved the curve's midpoint by 28.5px at most, because of the soft saturation `kCap = max(44, chord × 0.85)`; on a 210px chord, `bend = 300` and `bend = 900` were nearly identical. Short links were drawn with a 6px automatic bulge, so the whole line hid inside the 20px gap.

**Fix:** `bend` goes into the geometry unchanged (hard clamp at `RLK_BEND_MAX = 900`); dragging becomes two-dimensional (normal `bend` + tangential `shift`, where `0` is a legal value that serializes, undoes and exports); a `lead` term spreads the control points so a large arc does not tie itself in a knot; short links are lifted out of the gap by `RLK_LIFT` and the emptier side is chosen automatically; the label-avoidance list now includes frames and group boxes; and the drag basis is pinned at pointer-down.

Verified by `rlink51.js` (57 assertions), `rlink.js` (192), and 12 fuses in `_falsify_v51.py`.

---

## v50 · 2026-11-04 (originals kept: two copies, IndexedDB, inline-on-export)

User picked the architecture discussed in the previous round: "leave a lightweight copy on the node for layout and thumbnails, put the original in IndexedDB for zoom and export, then inline it on export — without losing single-file sharing."

**Why zoom used to blur:** v47 kept exactly **one** image. On insert it was downsampled to a 1280px long edge and re-encoded under 400 KB, and that same copy fed the thumbnail, hover preview, full-size overlay, PNG, SVG and PDF. "100%" therefore showed an already-shrunk bitmap. Even a 1×1 GIF was unconditionally re-encoded to JPEG. Simply storing the original in the document is not an option either: `serialize(root)` is the whole tree as JSON, `pushUndo()` snapshots it on every step, and every tab has its own slot — one 4 MB photo would be duplicated N times and burst the 5 MB localStorage quota.

**Two copies, two jobs:**
- the node keeps a **lightweight copy** (320px long edge / 96 KB cap) for layout and thumbnails, plus four short fields `oid` / `ow` / `oh` / `os`;
- the **original is kept byte-for-byte** (no re-encode, no quality loss, GIF frames intact) in an `imgOrigMem` map and in IndexedDB under the fixed name `IMG_DB='mindmap-studio-orig-v1'`;
- small images that already fit (≤ 320px, ≤ 96 KB, valid MIME) keep their original bytes as the lightweight copy — zero re-encoding;
- `imgOk()` stays the single gate. Old v47–v49 images without `oid` still load; an invalid `oid` is dropped as a field rather than rejecting the whole image.

**Display and exports use the original:** `render()` stays synchronous (`imgBestSync()` shows whatever is at hand first); async work happens only at insert, overlay upgrades and export boundaries. `openImgView()` / `imgHoverShow()` then swap in the original via `imgBest()`, `ivScaleTo()` computes **100% from the original's pixels** instead of the thumbnail's, and `imgXSrc()` makes PNG / SVG decode the original too.

**Single-file sharing, five boundaries:** standalone `.html` inlines originals into `IMG_DATA`; map `.json` / workspace `.json` gain an `images` table; imports and exported pages call `imgBootInline()` to restore originals into the **receiving** browser's IndexedDB (malicious entries are filtered by MIME and `oid`); exported pages change `STORE_KEY` but **never `IMG_DATA`'s fixed `IMG_DB`**, and keep the `/*<<IMG_DATA_END>>*/` marker so a page exported from an exported page still works; and the exporter strips the runtime `src` of `#iv-img` / `#ih-img` plus the `#iv-info` / `#ih-cap` captions, otherwise the currently open overlay would inline the original a second time and the previous user's caption would travel with the file. **Every export takes `const doc = serialize(root)` synchronously before any `await`** — putting it after the await let "Back up .json and start fresh" swap in an empty canvas mid-flight and export an empty map (`run.js` caught it); `exportJson()` / `exportHtml()` / `bkWriteNow()` all follow this rule, guarded by a static assertion.

`imgGc()` runs 8s after startup and only reclaims originals referenced by no localStorage slot, undo/redo entry, current canvas or current session — an original that undo could still bring back is never deleted early. Browsers without IndexedDB degrade gracefully: the in-memory original still works, it just doesn't survive a reload.

Verified: new `_test/imgstore.js` **91 checks** (with a fake IndexedDB whose transactions commit on `setTimeout(0)`, matching real "oncomplete after all requests succeed"); `_falsify_v50.py` **9/9 bolts bite**; full regression **44 suites / 3545 checks / 0 failures**; `i18n.js` 75 green with the EN glossary going 1007 → **1009**; new `_chrome_img50.py` uses **CDP with real waiting and two brand-new Chrome profiles** (IndexedDB transactions never settle under `--virtual-time-budget`) — **37 green** (29 source + 8 receiver), including hard proof that a fresh profile opening the exported page still zooms into the original bytes; `_falsify_chrome_v50.py` **6/6 bolts bite** on real Chrome too. Size **13,200 lines / 754,087 bytes**, sha1 `9f604b892bb6`.

> Online is still v48. v49 / v50 are only in the local release copy and **not published** — that waits for an explicit "publish".

---

---

## v49 · 2026-11-04 (summary brackets no longer sit inside frames)

User: "summary brackets overlap the frame lines too, please fix."

v48 fixed **frame-to-frame** crossings; this round is **bracket-to-frame**. A real-Chrome probe sampled 101 points along each summary bracket in 12 scenes and counted points inside a frame, points within 3px of a frame edge, and summary topic cards crossed by a frame edge. Before the fix, eight scenes had the entire bracket sitting on top of a frame's tinted background (101/101 points), and the topic card was crossed whenever the covered siblings belonged to a group frame.

**Cause:** `placeSumOne()` still used the v42 rule — `subtreeRect` (cards only) plus a padding that recognised only a boundary on the covered siblings themselves. v48 introduced `contentRect()` (cards + every frame in the subtree) and `framePadOf()`, but this call site was never updated. A parent boundary, a group frame, and a boundary on a grandchild were all invisible to the bracket.

**Fix, in one function:** the bracket **span** now uses `contentRect(c) + framePadOf(c)`; the parent boundary and any group frame contribute a separate **outer edge**, computed with the very same `contentRect(p)` / `groupRect(gid)` geometry the renderer uses; and that outer edge feeds only `bx` / `by`, never the span — otherwise a bracket meant for two siblings would grow to wrap the whole branch. With no frames at all the numbers are bit-for-bit identical to before (scenes with images included), so nothing moves.

Verified: `summary.js` 92 → **105**; six counter-proof bolts all bite on jsdom **and** on the new real-Chrome probe (6/6); full regression **43 suites / 3454 checks / 0 failures**; new `_chrome_sum49.py` **53 green**. Other geometry probes rechecked: summary 29 / boundary 21 / groupframe 40 / img 48. Docs and screenshots synced. The local release copy is updated but **not published** — that still waits for an explicit "publish".

---

## v48 · 2026-11-04 (frames no longer cross: the block a node occupies always includes its own frame)

Two sentences from the user: ① frames overlap right now; ② after inserting an image they must not overlap either.

The second one is not a footnote, it is the **acceptance criterion**: it rules out "just make the padding bigger",
because any frame placed by a hard-coded offset will overlap again the moment an image makes a card taller.
So this round changed no numbers at all — it changed the **coordinate system**.

**There were four problems, not one.** A throwaway diagnostic measured the real geometry in 23 scenarios
(frame-to-frame and frame-to-card) and sorted hits into nested / crossing / frame-over-card. Before: 5 crossings. After: 0.

| Problem | Measured | Cause |
| --- | --- | --- |
| Parent boundary × child group frame | crossing **246 × 118** (mind) / **468 × 77** (org) | the parent frame was drawn from `subtreeRect()`, which only sees cards, not the frames on its children — so it ran straight through the child group frame |
| Two group frames in the org layout | overlapped **8px** horizontally | `orgMeasure()` only added `BD_PAD` for boundaries, leaving the group frame's 36px unaccounted; adjacent spans 64 apart cannot hold 72 |
| Boundary × neighbouring group frame | fine without images, **overlapped 12px with them** | sibling spacing decided "is there a frame on this side" from the `__gbs` marker, which drifted from the real frame edge; a taller card amplified the error |
| Two grouped siblings' own frames | nested 485 × 118 | **not a problem** — a group is one frame by definition. Kept as its own category so it would not be "fixed" along with the rest |

**The fix is one sentence**: the block a node occupies in the layout always includes its own frame, and drawing a frame uses
the *content box*, which always includes the frames inside the subtree.

- New **`contentRect(n)`** = card + **every frame in the subtree** (recursing through `framePadOf(child)`), so a parent frame
  wraps child frames by construction instead of relying on sibling padding — that padding only ever helped the neighbouring branch.
  `subtreeRect(n)` stays: it covers the card layer and is still used by tests and call sites.
- New **`framePadOf(n)`** (the outermost frame's padding for that node) and **`padSide(n, sameGb)`** — a **one-sided** padding.
  `sameGb` means the side is adjacent to a group member, and then `GB_PAD` is not added: adding it on both sides pushes
  grouped siblings 72px apart (card gap 66 → 138).
- `layout()`'s block = content + `pT` + `pB`; between siblings there is only `Math.max(GAP_Y, FRAME_CLEAR)` left to matter.
- Org layout: `orgContent(n)` covers only the card / the row of children (**not its own frame**); the frame padding is added
  by the **parent** in `orgKids()`, per side. Measuring (`orgMeasure`) and placing (`place`) must share one function.
  Row pitch = `max(ORG_GAP_Y, FRAME_CLEAR + what the row above spills down + what the row below spills up)`.
- **`markGbTree()` / `__gbs` / `__gbT` / `gbOut` are retired** — "remember, for every ancestor, whether something below has a frame"
  duplicates the new block-carries-its-frame rule and the two disagree.
- **Images need no special case**: insert one into a framed node and the card grows from 46px to 138px; the parent frame,
  the group frame and the neighbouring branch all re-flow. Height comes from the layout, never from a constant.

**Four traps hit on the way**

- **Symmetric padding pushes grouped siblings apart** (66 → 138). Fixed by skipping `GB_PAD` on the side that faces the group.
- **`GB_PAD` counted twice**: `+ framePadOf(n) * 2` in `orgMeasure` stacked on top of `padSide` in `orgKids`, which pushed the two
  org group frames too far apart. Splitting out `orgContent(n)` (no frame of its own) leaves the padding added exactly once, by the parent.
- **A killed counter-proof leaves a screw removed.** The product file is mutated before each round; on SIGTERM (tool timeout) the
  `finally` block never runs, and the next run treats the broken file as the baseline. Hit for real this round — the script now
  restores the original from a signal handler before exiting.
- **A full-width colon in an assertion name truncates it** — the harness splits names on `：`, so the assertion can never match and the
  screw looks like it does not bite. Assertion names avoid it.

**Tests**

- `boundary.js` **106 → 116 assertions**: a new section [13] "frames do not cross" (parent frame wraps the child group frame,
  a gap between the two layers, neighbouring branches do not overlap, the edge gap is exactly `FRAME_CLEAR`, four nested layers
  only nest), plus three static guards rewritten for the new coordinate system.
- `groupframe.js` **94 → 98**: new section [7.5] (grouped siblings are not pushed apart, the group frame's bottom edge sits
  `GB_PAD` below its last member's subtree, it does not crush the card of the next branch, two org group frames do not overlap horizontally).
- New counter-proof `_test/_falsify_v48.py` — **6 screws**, each verified to turn the listed assertions red, with a sha1 check after every restore.
- New real-Chrome probe `_test/_chrome_gap48.py` — **21 assertions**, including four that are re-run **after inserting images**,
  "frame edges are exactly `GAP_X` apart" in the org layout, and "a frame never eats a click". The same six screws were replayed
  against the browser probe: **6/6 bite**.
- Full regression **43 suites / 3441 assertions / 0 failures**; product **12,886 lines / 719 KB**.

**Upgrade notes**: none. Save format, `localStorage` key and preference keys are untouched; existing boundaries and group frames
simply re-flow when opened.

---

## v47 · 2026-11-04 (node images: insert, images and text together, thumbnail + hover to see the original + zoom)

Three sentences from the user: ① let a node carry an image; ② a node may hold an image *and* text, or only one of them;
③ show it small on the canvas, preview the original on hover, and zoom.

**① Inserting.** Three new right-click items: **Insert image… / Replace image…** (the label switches when an image is already there),
**View full size**, **Remove image** (the last two are greyed out without one). It goes through the system file picker
(`<input type="file" accept="image/*">`), then **downsamples and re-encodes immediately** instead of storing the original.

- Why it has to: single file + works offline means the image must be inlined as a data URL, while `localStorage` holds 5 MB and
  the undo stack is capped at 6 MB — a 4 MB phone photo would blow the save file up on the spot. Three gates:
  `IMG_MAX_PX = 1280` (long edge), `IMG_CAP = 400 KB` (data URL length), `IMG_FILE_MAX = 24 MB` (not even read).
- Re-encoding branches: **opaque → white background then JPEG**, quality stepping down (1 / 0.82 / 0.68 / 0.55 …) until it fits 400 KB;
  **has transparency → pixels only, same format** (turning a PNG icon or screenshot into JPEG paints a white box behind it).
  `imgHasAlpha()` decides on a 64px sample.
- **Only inline image data URLs are accepted** (`IMG_MIME_RE`). `http` / `file` links are refused — accepting them would break
  the "single file" and "works offline" promises at the same time.

**② Image + text, or either alone — expressed with an empty string, not a toggle.**

- `n.img = { src, w, h, n }` is an **ordinary node field**: copying a subtree, cloning, pasting, undo, saving and every export
  pick it up automatically, with no reference bookkeeping anywhere (which is exactly why it is not a side table of attachments).
- With an image the node gets `.hasimg` and `flex-direction:column`; the thumbnail is inserted as the **first child**, text below it.
- No text → `n.text === ''` → `.txt` is an empty element → `.node.hasimg .txt:empty{display:none}` collapses it;
  no image → no `.thumb` is created at all. So "image + text / image only / text only" needs **no special cases**.
- In text contexts (outline, copy as text, Markdown, drag ghost, `aria-label`, node tooltip) an image-only node would render as blank,
  so everything goes through `nodeLabel(n)`, which falls back to **“（图片）”**.

**③ Small on canvas, original on hover, zoomable.**

- The canvas only ever draws a 96px-tall thumbnail. `IMG_THUMB_H = 96` / `IMG_PAD_Y = 8` / `IMG_PAD_X = 12` / `IMG_GAP = 6`
  are **one set of numbers shared by CSS and the PNG / SVG exports** (computed in `imgThumbBox()`), so screen and export agree.
- **Hover preview**: a panel of the original flies out beside the thumbnail (`#imghover`, fixed, `z-index:72`), wheel zooms
  (clamped to 0.2–8), click opens the full view. Leaving grants **140ms** of grace (enough to move the pointer into the panel);
  moving into the panel itself keeps it open; `render()` closes it first thing — thumbnails are rebuilt on every render,
  so the listener can only be delegated on `#nodes`.
- **Full-size overlay `#imgview`**: − / ＋ / Fit / Actual size; the wheel zooms **anchored at the pointer** (whatever sits under
  the cursor stays put), and dragging pans. Zoom changes the `<img>` width/height (**not a transform**) — 800% stays sharp and
  the scrollbars are real.
- **Export**: PNG awaits `imgPreload(flat)` first (a broken image gives up after 2.5s so the button never hangs), then `ctx.drawImage`
  into a rounded thumbnail box; SVG uses `<image … xlink:href="dataURL">` with **only `xlink` written once** (writing `href` too
  stores the whole image twice) and the namespace declared on the root. Text exports (TXT / Markdown / OPML…) **deliberately carry
  no image**, only the “（图片）” placeholder.

Two knock-on numbers: `#imgview` takes the overlay count from **15 to 16** (synced in `o5o6.js`, `boundary.js`, `summary.js`,
`groupframe.js`, `rlink.js`, `_chrome_o5o6.py`), and `pushUndo` call sites go **43 → 45** (`imgApply` and `imgDel`, both named).

Verification: jsdom `img.js` **88 assertions** (8 sections: data model / rendering / hover preview / full-size overlay /
menu and actions / export / English UI / zero JS errors); counter-proof `_falsify_v47.py` **14 rounds, all biting**
(remote URLs, oversized src, build dropping the image, serialize dropping w/h, `:empty`, thumbnail position, the 140ms grace,
moving into the preview, closing on render, zoom clamping, undo, `nodeLabel`, SVG `<image>`, the xlink namespace);
full regression **43 suites / 3430 assertions / 0 failures**; real Chrome **48 assertions** (real computed styles and geometry,
the thumbnail still hit-testing as `.thumb` at 0.6 zoom, a real file picker taking 2400×1200 down to 1280×640,
18,048 red pixels on a real canvas versus 0 once the image is removed, SVG through a strict parser).
Product: **12,849 lines / 717 KB**, EN glossary **976 → 1007 entries**.

---

## v46 · 2026-11-04 (relationships: short ones stay visible, dragging tracks the pointer, no more knots; frames stop hugging their neighbours)

Two sentences from the user: one about relationship lines, one about frames.

**① "Dragging is stiff, and when the two ends are close the line and even the arrowhead disappear — so there is nothing to grab. Sometimes the line knots itself."** Four separate faults stacked together.

- **The endpoints were solved against the wrong edge.** `rayExit()` used to take "the edge facing outward", which is actually the *exit* point when the target rect sits ahead of the ray. With one end nested inside the other (a boundary or group frame wrapping the other node) the two ends were solved from opposite sides, so the whole curve ran underneath the cards — which reads as "no line". Rewritten as a **slab** test: t0 for entering, t1 for leaving, `t = t0 >= 0 ? t0 : t1` — where the first stretch of empty space begins. Plus a **nesting rule**: when one rect contains the other, the direction comes from the *inner rect's edges* (not its centre) — whichever of the four gaps is smallest. Measured after: sibling chord 68 → 20, nested stub 54 → 20.
- **On short chords the dashes and the arrowhead ate each other.** A 7/5 dash pattern on 20px leaves a single dot; a 10px arrowhead swallows the line. Below 30px the line is now **solid**, and the arrowhead scales with the chord (`AL = clamp(len*0.28, 5.5, 10)`).
- **Knots.** Two independent causes: the automatic curvature had a 16px floor (a 28px gap bulging 16px is literally a loop), and a second line between the same pair of endpoints got the *same* sign as the first. Curvature now follows the chord (`clamp(chord*0.22, 6, 40)`), and the second line of the same **unordered endpoint pair** is forced to the opposite side — counted per pair, not per index, so a third relationship in between does not confuse it.
- **Dragging too far used to wrap into a loop** → **soft saturation**: up to 0.7 × chord the bend is applied verbatim (1:1 with the pointer), beyond that it converges exponentially to at most 1 × chord. The first attempt used `tanh`, which bit at tiny offsets (30px of pointer travel produced 24.9px) and felt gritty.
- **Nothing to grab** → **the label is the handle**: `pointerdown` starts the drag (and must *not* call `preventDefault`, or double-click-to-rename dies). Whichever part of the curve you grab tracks the pointer 1:1 (measured error ≤ 4px).
- **The label covered the whole line** → for short links (`len < 90`) the label is parked 28px beyond an endpoint first (open space outside the frames), then falls back to the curve.

**② "The frame lines sit too close to the nodes or to other frames."** The spacing became one rule instead of patches:

- `BD_PAD` **14 → 20**, new `FRAME_CLEAR = 20` (frame edge to somebody else's card or frame edge), `GB_PAD = BD_PAD + 16 = 36` (must fit the 21px title band), org layout `ORG_GAP_Y` **44 → 52** (after 20px of padding on both sides, 44 left only 4px for the frame line itself).
- The sibling allowance is now one explainable expression: a boundary's padding is already inside the height `layout()` returns (the block edge *is* the frame edge), so it adds nothing; a group frame is a background layer and does not enter the block, so only the extra part is added; **two siblings in the same group add nothing** (they already live in one frame). The `GB_GAP` constant is gone.

Also: the real-Chrome i18n probe still expected 14 overlays (v45 shipped a 15th, the relationship dialog); the boundary probe's gap lower bound now reads `FRAME_CLEAR` instead of a hard-coded 24.

Verification: jsdom `rlink.js` **192 assertions** (179 → 192), `boundary.js` 109, `groupframe.js` **94** (93 → 94); five counter-proofs all biting — `_falsify_v46.py` **10 rounds**, `_falsify_v45.py` 14, `_falsify_rlink.py` 13, `_falsify_boundary.py` 3, `_falsify_groupframe.py` 7. Full regression **42 suites / 3342 assertions / 0 failures**; real Chrome **25 groups green** (rlink 57 / boundary 21 / groupframe 40 / summary 29 / o5o6 82 / i18n 37 …). Product: **12,252 lines / 687 KB**.

---

## v45 · 2026-11-03 (relationships: make them visible, let them get out of the way, sane defaults)

The relationship feature shipped one release ago; half a day of real use produced three notes. None of them was a missing capability — all three were "it is drawn, but it does not work well".

**① Too thin, too faint.** It was a 1.6px solid line in `#6b7a90` at 0.85 opacity, which all but disappears next to the structure edges. Three things changed together: **dashed** (7 on, 5 off), **2.2px** (thicker than a node card's 1.4px border, thinner than a frame's 2.4px), and **`#33405a`** — two shades darker — at full opacity, with a slightly larger arrowhead. Screen, PNG and SVG follow the same numbers; changing only one is the kind of miss you notice in the exported file. Dark themes get their own colour (`#9aabc4`): `#33405a` on `#0f1420` is invisible, and that is the other half of "visible".

**② Nodes, boundaries, group frames and summaries cover the line.** The curve is painted beneath the nodes — a promise, it never covers text — so wherever it crosses a card that stretch simply vanishes. Rather than move the layer (which would cover text), **the line now listens to you**: grab it and drag it into open space.

- One stored number, `lk.bend` (an offset along the normal, signed for either side); the geometry is still the same Bézier — so screen, PNG, SVG, save files and undo all follow for free. No second geometry path.
- A 2.2px dashed line is impossible to grab, so each one carries a **transparent 16px hit band** (`pointer-events:stroke`). It only picks up pointer events in open space; over a card you still hit the card (the curve stays beneath the nodes, unchanged).
- One drag is **one** undo step (the undo entry is a snapshot, so it must be taken before the first real change); dragging redraws only the curves and labels, never the canvas, and never pans the view.

**③ The default start and end were arbitrary.** The old rule took the start from `selected` — which is the node clicked **last** — so clicking A and then Ctrl-clicking B silently swapped the two ends. Now it follows the order the user asked for: **node → summary → boundary → group frame**. Each selected object contributes its identities to a pool; after a stable sort the first becomes the start and the next distinct one becomes the end. The dropdown is grouped in the same order.

Also fixed: an ordering assertion that could not fail before — with no summary on the canvas, "summary before boundary" and "boundary before summary" produce identical lists.

Verification: jsdom `rlink.js` **179 assertions**, new counter-proof `_falsify_v45.py` with **14 rounds all biting**, full regression **42 suites / 3327 assertions / 0 failures**, real Chrome **57 assertions** (including real hit-testing and a real drag), plus boundary 21 / groupframe 40 / summary 29 / o5o6 82. Product: **12,086 lines / 677 KB**.

---

## v44 · 2026-11-02 (three reported bugs fixed; relationships added)

Three debts from the previous rounds, then one new feature.

**① A group frame's border was hidden by node cards.** The frame's padding is wider than the gap between siblings (12), so its top edge inevitably reached into the card above — and since the frame is painted beneath the nodes, that stretch of dashes disappeared. The fix is to **make room**: a group frame now adds a gap between neighbouring branches, so the gap is always `padding + 2` and the dashes run through empty space. The padding itself is 26px — enough for the 21px title band plus the 2.6px border (it was 22 until the title grew a heavier frame). The decision has to propagate down the subtree (when a member is a first child, the frame's top reaches into the grandparent's level), hence a `__gbT` mark. The trade-off: adding or removing a frame shifts the map vertically; **no node moves horizontally**.

**② Frames were not prominent enough.** New scheme: darken the line, tint the inside. `frameEdge(col)` = the frame colour darkened 45%; the inside carries the same colour at 20% (boundaries) / 24% (group frames); stroke widths 2.4 / 2.6 with a denser dash (6/4). Screen, PNG and SVG all follow. **The first attempt used 14% / 7.5% and was called out as “still too faint”; 32% and 38% were turned down too — 45% is the version that finally reads as a region.**

**③ Summaries lost their meaning in XMind.** Verified before touching code: the summary's **text** was always there in `content.json` (as an ordinary child topic); what was missing was the **semantics** — no `summaries` array. Now it is attached to the parent topic the way XMind does it: `summaries:[{id, class:'summary', range:'(a,b)', topicId}]`. Open the file and the bracket and the summary level are back.

**④ The two frame titles were not the same size.** A boundary title sat on the border at 11.5px, a group frame title inside its top edge at 11px — two components that had drifted apart. They are now **one component**: same size, same line height, same padding, same radius, same border, same background source; only the anchor differs (a boundary label rides the border, a group label sits inside the top edge — each for its own occlusion reason). The group frame's padding grew 22 → 26px so that band always fits a 21px title plus a 2.6px border.

**⑤ Summary topics looked like every other node.** A summary topic is a real node, but it is also a *conclusion* — it should be findable at a glance. It is now **pill-shaped** (fully rounded, and no vertical colour bar on the left), carries a **heavier tint** of the branch colour, uses the **solid** branch colour for its border instead of a faded one, and is **bold**. Every one of those is relative to a plain node in the same branch colour (the test flips `sum` off the same node and compares), so it survives a theme change. Screen, PNG and SVG share the same rule.

**New: relationships.** Pick any two objects on the canvas — **node / boundary / summary / group frame** — right-click **Add relationship**, and a labelled curve appears between them. The division of labour with edges is sharp: edges draw structure, a relationship draws **the extra thing you want to say**.

- Stored as `root.links = [{id, t, a:{k,id}, b:{k,id}}]`, four endpoint kinds.
- Nodes get a stable id for the first time: `n.uid`, **only issued when a node is actually an endpoint** (no change to serialised size elsewhere); copy / paste / clone strip it, otherwise a duplicate would fight the original over the same endpoint.
- One clean-up pass: `normLinks()` at the top of `render()` — endpoint gone, group gone, self-link, duplicate id, all dropped.
- Geometry: `rectEdgePt()` for the edge intersection plus a cubic Bézier that bows along the normal (sign alternating by index), with an arrowhead at the far end.
- **Layering**: the curve sits beneath the nodes (it never covers text), the label above them (so it can be clicked).
- **The label dodges**: a midpoint often lands on the middle branch's card, which would cover someone else's text. It now slides along the curve to the first spot that covers no card — shared by all four outputs.
- XMind: native `relationships` on the sheet. A group frame is not a topic, so it lands on its **first member** — better to point at one node than to lose the relationship.
- Text formats (MD / Word / Excel / OPML / PPT outline) deliberately drop them: a linear outline has nowhere to put "between two objects".

Two things found on the way: `.lk-tip` was scoped to `#link` and `.rlk-row` had no styling at all (the new panel would have been a cramped mess); and the "strip uid on paste" test originally called `stripUid` itself — that verifies the test, not the product. It now goes through the real `pasteSubtree` entry, which is the only reason the counter-proof bites.

Verification: jsdom `rlink.js` **131 assertions**, **13 counter-proof rounds all biting** (including regressions for all three bugs), full regression **42 suites / 3279 assertions / 0 failures**, real Chrome **37 assertions**. Overlays 14 → 15; `o5o6` / `boundary` / `summary` / `groupframe` counts updated with it.

---

## v43 · 2026-11-02 (group frames: wrap the selected nodes into one group)

**A boundary wraps one branch and its subtree; a group frame wraps whatever you selected** — across branches, or just a few siblings. Select two or more nodes, right-click **Group the selected nodes**, and one dashed frame appears around them (radius 18, label centred on the top edge).

The three frames divide the work cleanly: a **boundary**'s range comes from that node itself; a **summary**'s range is a consecutive run and it grows a topic of its own; a **group frame**'s range is your multi-selection — it adds no node and changes no structure.

**It is stored differently from a summary, deliberately**: group definitions live on the root as `groups:[{id,t,c}]`, and a member node only carries `gb:<id>`. Copy/paste a subtree and that `gb` points at a group which is not there, so the normalisation pass drops it — "pasted nodes carry no group" needs no extra line of code.

**The one that matters most: a group frame takes no part in layout.** It is a background layer under the lines and nodes, so not a single coordinate moves when you add one (both the real-Chrome probe and the counter-proof guard this: add one padding line to `layout()` and the assertion goes red). Reserving space would re-flow the whole tree, and a cross-branch group has nowhere to reserve it; with the frame painted behind, nodes cover the line and the eye still reads it as "these belong together".

Other deliberate edges:

- A node belongs to one group; **delete down to a single member and the group dissolves** (a group of one means nothing);
- **A node already in another group is not offered for a new group** — one mis-click would otherwise wipe that group's title and colour;
- Fold a member, cut at depth N, or turn on "hits only" and the frame shrinks to what is still visible; hide every member and it is not drawn at all;
- The frame body is click-through (`pointer-events:none`); only the label is clickable.

**Exports**: drawn in PNG / SVG / PDF (title included), kept in JSON (`groups` and `gb`), deliberately dropped from text formats (MD / Word / Excel / OPML / PPT outline). XMind now gets a native `<boundary>`: a group frame maps to one when its members are a consecutive run of siblings under the same parent, and **single-node boundaries are wired up too** (the range degrades to a single index, wrapping that branch and its subtree) — which closes the XMind part of P1.5.

Verification: jsdom `groupframe.js` **85 assertions**, **7 counter-proof rounds all biting**, full regression **41 suites / 3120 assertions / 0 failures**, real Chrome **38 assertions**.

---

---

## v42 · 2026-11-02 (summaries: one bracket over several siblings)

**A summary covers several siblings; a boundary wraps one branch.** Select two or more siblings under the same parent and **Add summary** draws a bracket down their right side plus a **summary topic** that states what they have in common. Everything in between counts — pick the 1st and 3rd and the 2nd comes along, same as XMind.

**The summary topic is a real node**, not a label. It lives in `children` with one extra marker:

```js
{ text: 'Shared by both', sum: { a: 0, b: 1 } }   // covers children 0 and 1
```

That single decision buys select, double-click editing, colouring, folding, copy / clone / paste, undo and every export traversal for free — none of those paths knows a summary exists. **It can carry sub-nodes of its own** (the call was made up front, rather than shipping an "add child is disabled here" version and removing the guards later): pick five things, collapse them into one sentence, then break that sentence into actions.

**Layout gives it a patch of its own; the column model is untouched.** A summary sits *between* two levels, so `colX`/`colW` cannot express it — it and its whole subtree are excluded from the column-width pass (`__inSum`), and `placeSums()` derives the x from "right edge of what it covers + 10". Its sub-tree then lays itself out from that point with a local column array. Verified the hard way: the jsdom stub measures summary nodes at 420 px against 260 for everything else, so the moment they are counted into `colW` the `colX` assertion goes red.

**One bracket geometry, three renderers.** `__br` is computed once per summary and consumed by the on-screen `sumPaths()`, the PNG `paintSumPaths()` and the SVG `<g data-sums="1">` group. Frames and brackets both draw below the lines and the nodes; the bracket never takes a click (real Chrome `elementFromPoint` proves it falls through to the stage).

**The range is an index, not a reference** — nodes have no ids. So it has to survive edits: inserting *inside* the range grows it, inserting *before* shifts it, inserting right after the tail leaves it alone (a new sibling after the last covered node is outside the bracket, like XMind). Deleting shifts it back, and deleting every covered node removes the summary. All of that funnels through `clampTree()` at the top of `render()` — one call site instead of 15, so no structural edit can forget it.

**Where it travels.** PNG / SVG / PDF draw it; the map data `.json` keeps it; Markdown / Word / Excel / OPML / the PPT paged outline export it **as an ordinary child** — a summary has content, so an outline that dropped it would lose information. XMind gets it as a sub-topic (not yet as XMind's native summary marker). A boundary deliberately does *not* wrap a summary: the frame skips `sum` children, because two nested brackets around the same thing read as nonsense.

**Deliberate edges.** A summary cannot be dragged under another parent, cannot be moved up or down, and cannot take a sibling. Its position comes from the nodes it covers.

**Verification.** New jsdom suite `summary.js` with **77 assertions** (column model untouched, bracket hugging the covered subtree, the range growing / shifting / staying put on insert, shrinking on delete, clamping out-of-range data from a foreign file, sub-nodes hanging off the summary, fold and depth-limit behaviour, the menu's three items greying out correctly, undo names carrying context, the summary's three deliberate "cannot"s, org layout, and SVG / PNG / Markdown output); **6 counter-proofs** — remove the `colW` skip → the column assertion turns red; remove the `__inSum` marker → the sub-tree falls back into the global column; remove the insert-time growth → a node added mid-range is no longer covered; remove the `clampTree()` call → out-of-range data survives; remove `sumPaths()` → the bracket disappears while the nodes stay; remove the bounding-box skip → the boundary swallows the summary. Each round restores the file and checks sha1. Full regression **40 suites, 3035 assertions, 0 failures**; a new 23rd real-Chrome probe group with **25 assertions** (real geometry: the bracket clears every node rectangle and sits between the two levels, the summary's centre lands between the first and last covered node, folding narrows the bracket, org layout doesn't collide, and `getImageData` proving the PNG really draws the bracket's spine and its leader line).

**Docs**: new section 3.11 in both guides, plus a sixth screenshot pair (`screenshots/mindmap-summary.png` / `.en.png`) generated by a new `sm` scene in `_test/_shot.py`.

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

## v41 · 2026-11-01 (boundaries: group a branch with a dashed frame)

**A boundary is decoration carried by a node, not a node.** Put it on a branch and it wraps that node plus every **visible** descendant. Right-click a node for **Add boundary / Edit boundary title / Remove boundary**, and the five swatches beside **Frame** set its colour (clicking one also adds the frame if there is none). Double-clicking the title edits it in place — Enter saves, Esc reverts.

Because it hangs off the node, everything else follows for free: copy, clone, paste, reorder and delete all carry the frame along, and deleting the node removes the frame with nothing left dangling. Fold the branch, cut it with "show up to level N", or filter to search hits and the frame tightens to the visible part — not a special case, just the bounding box being recomputed. Nested frames are allowed (a child may carry its own inside its parent's).

**Layout only gives way; the column model is untouched.** A framed branch reserves `BD_PAD = 14` above and below, and when two adjacent branches are both framed the gap between them grows by another 14 — measured at **26 px** (GAP_Y 12 + 14). Without that second rule two dashed borders sat 12 px apart and read as one blob; the prototype caught it. `colX[]` / `colW[]` never change: `GAP_X` is 64, far more than 14, so a frame's right edge reaches into the gutter without ever touching the next column. The vertical (org chart) layout only gives way horizontally — its 44 px row gap already swallows the 28 px a frame needs, so `orgRowY()` needs no change. That is the whole reason a boundary could land cheaply: it needs *space*, not a new column.

**It never eats a click.** `#frames` sits before `#edges` in the DOM, so frames are drawn below the lines and below the nodes; the frame body is `pointer-events:none` and only its title is clickable (clicking it selects the owning node). Real-Chrome `elementFromPoint` proves all three: the title is hittable, the node centre still hits the node, and the empty area inside a frame falls through to the canvas.

**Where it travels.** PNG / SVG / PDF (printed from the screen) draw it, and the map data `.json` keeps it. Markdown / Word / Excel / OPML / the PPT paged outline deliberately omit it — a boundary is a visual grouping and an outline has nowhere to put it (XMind doesn't export them either). XMind export does not carry it yet; that is the next round.

**Data shape.** `{ text: 'branch A', bd: { t: 'optional title', c: '#7F77DD' } }`. `build()` accepts it and `serialize()` writes it, so old maps simply lack the field and old files open unchanged. Colour is validated as a 6-digit hex before it is ever written into a `style`, and titles are trimmed and capped at 60 characters. Known and accepted: a file saved by v41 and re-saved by an older version loses its frames, exactly as `link` / `mark` did when they were introduced.

**Verification.** New jsdom suite `boundary.js` with **107 assertions** (bounding box vs. `subtreeRect`, the 26 px gap, fold and depth-cut shrinking, nesting, field sanitising, serialize round-trip, copy / clone / paste / move, undo names carrying context, in-place title editing with Enter / Esc, the colour swatch, PNG+SVG sharing one geometry (`bdExportFrames`), text exports deliberately carrying nothing, accessibility roles, and the org layout); **3 counter-proofs** — remove the sibling gap → the 26 px assertion turns red while "both frames drawn" stays green; remove the `renderFrames()` call → the on-screen assertions turn red while the data assertions stay green; remove `bd` from `serialize()` → storage and paste lose the frame while adding one still works. Each round restores the file and checks sha1. Full regression **39 suites, 2957 assertions, 0 failures**; a new 22nd real-Chrome probe group with **21 assertions** (real hit testing, real 26 px gap, no clipping of the title, org layout, and `getImageData` proving the PNG really draws the border, the 6 % tint and the title).

**Also fixed:** the frame title's background follows `--stage-bg` instead of a hard-coded white, so it does not show up as a white blob on the dark themes.

**Docs**: new section 3.10 in both guides, plus a fifth screenshot pair (`screenshots/mindmap-boundary.png` / `.en.png`) generated by a new `bd` scene in `_test/_shot.py`.

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
