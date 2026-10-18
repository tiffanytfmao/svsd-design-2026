# Rewind — Decision Log

Design decisions for Rewind (version control for digital art), SVSD 2026.
Each entry: the question, options we weighed, what we chose, and why. These double as material for the video's "exploration" section.

## Open items

**Ask Ajia & Isabella**
- [ ] Drawing on a past version: auto-fork, or read-only until "Fork from here"? (Q2)
- [ ] Onion-skin overlay when previewing past versions? (Q2)
- [ ] Instead of a hint on the empty timeline, should a tutorial popup explain the timeline the first time someone opens a project? (Q2)

**Prototype placeholders (decide, then update `prototype/`)**
- [ ] Drawing on a past version is blocked, with a notice that the behavior is undecided (auto-fork vs. read-only + restore)
- [ ] Scrubbing shows a flattened image: past versions ignore layer order, visibility, and opacity, and a deleted layer stays visible in the version where it was deleted

**TBD**
- [ ] Main storytelling export: timelapse with chapters, a process board, or something else? (Q3)
- [ ] Manual "marks" on versions (from Ajia's sketch): keep them? They tie into chaptered timelapses (Q3)

---

## Q1. What counts as a version?

**Decided:** 2026-10-17

### Options considered

| Trigger | Pro | Con |
|---|---|---|
| Fixed time interval (e.g. every 15 min) | Simple, predictable | Uneven: 15 min of detail shading ≠ 15 min of blocking in |
| Edit size (% of canvas changed) | Tracks "something happened" | Threshold is arbitrary; a tiny but important detail (e.g. eyes) would be missed |
| Pause in drawing (idle) | Pauses are when artists step back and look — natural reflection points | A long unbroken session becomes one giant version |
| Structural event (layer added / deleted) | Clear, explainable | Some artists rarely touch layers |
| Manual mark | Artist chooses the story beats | Extra work; must stay optional |

### Decision

A version closes on **whichever comes first**:
1. a **pause** in drawing (~90s idle — tune during testing)
2. **10 minutes** of active drawing
3. a **layer being added or deleted**

### Recorded, but not used to define versions

These are captured per version as **metadata** — shown in the history UI, never used as triggers:
- **Edit size** — how much of the canvas changed. Can drive visual weight (e.g. node size/outline) so big moves stand out without us having to define "significant."
- **Tools used** — which brushes, brush settings (size, opacity, color).
- **Transforms** (move/scale/rotate) — *probably out of scope*; log if we build them.

### Why

- Fixed time alone is uneven, and "significant edit" has no good threshold. Pauses track how artists actually work, which fits Rewind's goal of encouraging reflection.
- Separating *what defines a version* from *what we display about it* means edit size and tools add to the story without making version boundaries hard to explain.

### Implications

- **Data model:** record every stroke (needed for timelapse playback), and snapshot the canvas at each version boundary. Gives three zoom levels: session → version → stroke.
- **Demo:** 10-min versions are slow to generate on camera. Add a debug time multiplier or a seeded sample history.

---

## Q2. Where does history live, and how is it structured?

**Decided (part 1):** 2026-10-17

### Part 1 — Two surfaces

1. **Timeline bar on the canvas page** (inspired by Fusion 360's bottom timeline). Always visible under the drawing; drag a playhead forwards/backwards through the process without leaving the canvas.
2. **A separate, detailed history page** for deeper browsing (structure: see part 2).

**Why:** History should be one drag away while drawing, not a separate mode you have to remember to open. Fusion's timeline proves a linear, draggable history is learnable; the detail page holds anything too dense for a single bar.

**Timeline bar details (proposed):**
- Segments = versions (from Q1). Each shows icons for the tools used (Q1 metadata), like Fusion's feature icons.
- Edit size can show as segment intensity/height.
- Because every stroke is recorded, dragging *within* a segment scrubs stroke-by-stroke, so it feels smooth rather than jumping between versions.
- The bar is linear, so it shows **the current branch only**. Fork points get a small notch; branches themselves live on the detail page.

**Decided: Segment width = equal.**

```
│ ✎ │ ✎ │ ▧ │ ✎ │ ✂ │ ✎ │
└───┴───┴───┴───┴───┴───┘
 v1  v2  v3  v4  v5  v6
 hover → "v3 · 8 min · +12% canvas"
```

| Option | Trade-off |
|---|---|
| **Equal width** | ✅ Easy to click, and long histories stay readable. Duration and edit size show on hover instead |
| Width = time spent | Shows where effort went, but short versions get too small to grab |
| Width = edit size | Highlights big moves, but squeezes careful detail work |

**Decided: Long histories scroll sideways.**

| Option | Trade-off |
|---|---|
| **Horizontal scroll** | ✅ Simple, familiar (what Fusion does), and every version stays one click away. You can't see the whole project at once; the history page covers that |
| Group by session | Each session reads like a chapter, but adds expand/collapse to design and build |
| Pinch / zoom | Flexible, but harder to build and to discover |

**Decided (for now): A past version shows on the full canvas.**

```
┌─ Viewing v3 · Oct 17 · [back to now] ─┐
│                                       │
│          (canvas shows v3)            │
│                                       │
└───────────────────────────────────────┘
│ ✎ │ ✎ │▲▧ │ ✎ │ ✂ │ ✎ │
```

| Option | Trade-off |
|---|---|
| **On the canvas** | ✅ Feels like rewinding, and shows the drawing at full size. A banner keeps "back to now" one tap away |
| Thumbnail popup | Your current work never leaves the screen, but the preview is too small to judge detail |
| Canvas + onion skin | Before and after at once. Good for comparing, but cluttered on busy drawings |

**Decided: The timeline bar is always visible, even on a brand-new project.** This matches Fusion 360, which shows its timeline from the start (empty track plus playback controls, with feature icons added as you work).

**Decided: No hint.** We mocked up a one-line hint ("Your process is being saved. Drag here later to rewind.") and didn't like it. The empty bar stays as it is, like Fusion's.

**⏸ Tabled — ask Ajia & Isabella:** should a **tutorial popup** explain the timeline instead, for example the first time someone opens a project?

**⏸ Tabled — ask Ajia & Isabella:** should we add an **onion-skin** overlay (the current version faintly on top of the past one) to the canvas preview, either as a toggle or by default?

### Part 2 — Detail page structure: layers or forks as rows?

**Decided:** **C — forks are the main graph; layers live in a selected-version panel.**

```
main ──●──●──●──●──●──● (current)
           \
fork 1      ●──●──● (alt background)

[selected version ▸]
 ├ lineart   ✎ pen 4px
 ├ color     ✎ brush 20px  (new)
 └ bg        👁 hidden
```

| Option | Why not / why |
|---|---|
| A. Rows = layers (Ajia's sketch) | Shows build-up well, but mostly duplicates the timeline bar, and forks have no home |
| B. Rows = forks only | Loses the per-layer detail |
| **C. Forks + layer panel** | ✅ Shows what the bar can't (exploration), keeps Ajia's per-layer breakdown as the inspector |
| D. No forks | Simplest, but drops the "exploration" story — Rewind's main differentiator vs. Procreate timelapse |

**Why:** The timeline bar already tells the linear story, so the detail page should show branching exploration. Most projects never fork, so the default stays a clean single line.

### ⏸ Tabled — ask Ajia & Isabella

What happens if you **draw while the playhead is rolled back**?

Shortlist (Tiffany is OK with either):
- **Auto-fork:** drawing on an old version quietly starts a new branch from there, with a toast like "Started a new branch from v3." Nothing is lost, and forks appear naturally. Risk: branches the artist didn't mean to create.
- **Read-only + Restore:** past versions are view-only (canvas dimmed) until you tap "Fork from here." More explicit, but it adds a step and stops the artist mid-stroke.

Ruled out:
- **Discard the future** (like undo): destroys history, which contradicts the premise of Rewind.
- **Insert, Fusion-style:** later pixel strokes would replay over the inserted work. Confusing, and hard to build.

---

## Q3. What can an artist export to share their process?

**Decided (for now):** three export options:
1. **Timelapse** video
2. **Branch version as an image** (a still of any version/fork)
3. **Final image**

**⏸ TBD:** which one is the *main* storytelling output. Ideas we discussed:
- **Timelapse with chapters:** pauses briefly at key moments (marked versions, forks, new layers) with a short caption like "tried 3 backgrounds." A format people already post, plus Rewind's reflection. Ajia's manual marks would fit here.
- **Process board:** a static layout of key versions and branches, like a design-process slide. Good for portfolios.

---

## Q4. Canvas page layout

**Decided:** 2026-10-17. **C1: everything floats over a full-screen canvas, and the timeline is a wide floating bar that stays open.**

```
┌──────────────────────────────────────────────┐
│ [← Project]                       [Export]   │
│ ┌──┐                              ┌────────┐ │
│ │✎ │                              │ Layers │ │
│ │🖌│          (canvas, full       │ [ L2 ] │ │
│ │✒ │           screen)            │ [ L1 ] │ │
│ │⌫ │                              └────────┘ │
│ │● │                                         │
│ └──┘                                         │
│ ┌──────────────────────────────────────────┐ │
│ │ ▶ ▢▢▢▢▢▢▢ timeline        ⤢ Open history │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

| Option | Trade-off |
|---|---|
| A. Ajia's sketch + timeline | Familiar, but brushes and the timeline both sit at the bottom and compete; the timeline reads as another toolbar |
| B. Timeline docked alone at bottom, tools in fixed side panels | Clear hierarchy, but fixed panels take space from the drawing |
| C. Floating panels, timeline as a collapsible pill | Most canvas space, but a hover-to-expand pill hides history and breaks the "always visible" decision (Q2) |
| **C1. Floating panels, wide floating timeline that stays open** | ✅ The drawing is the focus, and history stays visible. It covers the bottom of the canvas |
| C2. Floating tools, timeline docked as its own strip | Never covers the art and reads like a video editor, but takes ~50px of height from the canvas |

**Why:** The drawing is the main point. History should always be visible, but it shouldn't take space from the canvas.

**Order of importance:** canvas, then timeline, then brushes and settings, then layers, then project navigation and export.

**To explore in mockups:** how the floating bar avoids getting in the way of drawing:
- Keep it slim (~32–40px) and partly see-through.
- Fade it while the pen is down and bring it back when the pen lifts. It dims but never hides, so it stays "always visible."
- Allow panning, so artwork under the bar can always be reached.

---

## Q5. Which device do we design for?

**Decided:** 2026-10-17. **Design for tablet first. The demo will probably be on desktop.**

**Why:** Most digital artists draw on a tablet with a stylus, so a tablet-first design supports the story that Rewind is made for artists. The demo video will probably be recorded on a laptop, so everything also has to work with a mouse.

**Implications**
- **Design frame:** iPad landscape (e.g. 1180×820).
- **Touch targets:** at least 44px for every button, segment, and layer row.
- **No hover-only features.** Hover can preview something on desktop, but every piece of information must also be reachable by touch.
- **Input:** the pencil draws; two-finger drag and pinch pan and zoom. On desktop, the mouse draws and space + drag pans.
- **Demo:** record in a browser window sized to the tablet's proportions, so it looks like the tablet design.

**Decided: Combine the tooltip and the past-version banner into one playhead label.** Hover tooltips don't work on touch. One label sits above the playhead whenever it's on a past version (on any device): "v3 · Oct 17 · 8 min · +12%." It shows information only, with no buttons.
- **On the latest version:** no label.
- **While dragging:** the label follows the playhead, placed above it so a finger doesn't cover it.
- **After letting go on a past version:** the label stays.
- **Desktop hover:** shows the same label in gray as a preview, without moving the playhead.

**Working choice, not final: "Back to now" is a separate button, floating above the bar on the right**, shown only while viewing the past. We also considered putting it inside the bar at the right end, like a video player's "jump to live." That keeps the canvas clearer, but it makes the space for versions narrower.

---

## Q6. Visual system

**Decided:** 2026-10-17. **Soft and app-like: a white canvas, white floating panels, a neutral UI, and color reserved for the artwork.** Set up in the svsd-designs Figma file as variables and text styles.

**References:** the radial tool palette with the "Thickness" card, Paper by FiftyThree, a Japanese design-system board (rounded type, a black pill button with a yellow dot), and soft pastel phone UI. What they share: floating light panels with soft shadows, round buttons, one dark color for "selected," and color used for content instead of UI.

### How we got here

| Direction | Why not / why |
|---|---|
| A. Synth panel (patterned, saturated segments; monospace labels) | Most "art x tech," but loud next to the drawing, and it clashes with the artist's own colors |
| B. Paper studio (flat cream panels, tan accents) | Calm, but read as a website more than an app |
| B2. Soft product (floating cards, shadows, round buttons) | Felt right, but the first version had cream panels and failed contrast |
| B2 contrast pass (tan panels, outlined segments, purple selected everywhere) | Passed WCAG, but the tan panels and heavy outlines lost the softness |
| **B2 middle ground** | ✅ Keeps the soft look and fixes everything that failed |

### Rules

- **The canvas is pure white.** It's the artist's paper.
- **Panels are white too**, separated from the canvas by a faint purple hairline (16% opacity) and a soft shadow.
- **Text and icons are dark purple** (#20255D), never black. Muted text is #5E6189 (4.6:1 on white).
- **Selected brush and play button:** filled dark purple, with white icons.
- **Selected version:** tan fill (#CCB67D) with a 2px dark purple ring, so it doesn't rely on tan alone.
- **Default segments, active layer row, slider track:** Rose 50 (#E5D4CA).
- **Future versions and hidden layers:** dashed outline (#9C98A6) and muted text. **Never use opacity to show state**, since faded content fails contrast.
- **Playhead:** orange (#D54F1F, 4.2:1 on white), so "where you are" stands out from both tan and purple.
- **Accent dot:** tan, used on the "Back to now" pill.
- **Saturated palette colors** (orange, teal, dark green, magenta, and others) are the **default brush swatches**, not UI colors.
- **Type:** M PLUS Rounded 1c (called **"Rounded Mplus 1c"** in Figma). Medium for labels and headings, regular for body text.
- **Shapes:** fully round buttons and pills; panels 22px radius; segments 12px.

### Contrast (WCAG)

| Pair | Ratio |
|---|---|
| Dark purple text on white | 14.2:1 |
| Muted text on white | 4.6:1 |
| Dark purple icon on Rose 50 | ~10:1 |
| Dark purple on tan | 7.1:1 |
| Orange playhead on white | 4.2:1 |
| Dashed outline on white | 2.8:1 (decorative; the icon inside carries the meaning) |

### In Figma (svsd-designs)

- **Variables, Primitives collection (17):** the palette, plus white, `dark-purple-muted` (#5E6189), `outline-grey` (#9C98A6), and `dark-purple-a16` (the panel hairline). Hidden from property pickers.
- **Variables, Color collection (29):** names describe their use (`color/bg/canvas`, `color/bg/segment-selected`, `color/text/muted`, `color/border/panel`, `color/accent/playhead`, `color/swatch/1–8`, and so on). Each points to a primitive.
- **Variables, Shape collection (12):** radii (segment and layer row 12, panel 22, full 999), spacing (4/8/12/16/24), and sizes (touch target 44, segment 44, timeline height 58).
- **Every variable has a CSS name** matching its Figma name, e.g. `var(--color-bg-canvas)`. The prototype's CSS still uses its older names (`--ink`, `--paper`, and so on).
- **Text styles:** Rewind/Title 20, Heading 17, Body 15, Label 15, Caption 13, Caption strong 13.
- **Effect style:** Rewind/Panel shadow (two soft dark purple shadows).
- The original 13 color styles were left as they were.

---

## T1. Drawing engine

**Decided:** 2026-10-17. **p5.brush with its built-in brushes**, following [draw-your-attention](https://draw-your-attention.vercel.app/). We won't design custom brushes.

| Option | Trade-off |
|---|---|
| **p5.brush, built-in brushes** | ✅ Natural textures (pen, marker, spray, charcoal, 2B/HB/2H pencils) with no brush design work. The example draws live by calling `brush.line(prevX, prevY, x, y)` on every drag event, so short segments seem fast enough for live drawing even though the author warns it's meant for high-res artwork |
| Canvas 2D + perfect-freehand + our own stamp brushes | Fast, replays exactly, but brush tuning takes time and it looks plainer |
| Fork Klecks (Kleki's engine, MIT) | Brushes and layers come free, but its history stores pixel tiles in a straight undo list: no stroke data, no forks. We'd be rebuilding history inside a ~600-file codebase |

**Implications**
- **Stroke record:** `{brush, color, weight, layer, seed, segments: [[x1,y1,x2,y2], ...], t}`. Replaying = calling `brush.line` again with the same seed.
- **Scrubbing:** redrawing textured strokes is slow, so rely on the snapshot taken at each version (Q1) and only redraw the strokes after it.
- **Layers:** one WEBGL `p5.Graphics` per layer, using `brush.load(pg)`. Tested 2026-10-17, see "Gotchas" below.
- **Version:** pinned to **p5 2.3.3 + p5.brush 2.2.3**, stored locally in `prototype/lib/`.

**Gotchas from building the prototype (p5.brush 2.2.3)**
- Brush strokes are buffered and only written at the **end of the frame** (p5's `postdraw`), to whichever target is loaded at that moment. Calling `brush.load()` to switch back mid-frame **throws away** the pending strokes.
- `p5.Framebuffer` targets didn't receive brush strokes in testing (with p5 2.2 or 2.3), even though the docs list them. `createGraphics(w, h, WEBGL)` works.
- So in each frame: do all regular p5 drawing first, then load **one** brush target and make the brush calls last. They appear on the next frame.
- Each WEBGL graphics is its own WebGL context (browsers allow ~16), so the prototype caps layers at 8.
- In p5 2.x, `redraw()` is async and returns a Promise.

---

## P1. Canvas page + timeline bar prototype

**Built:** 2026-10-17, in `prototype/`. A clickable HTML mockup that also tests p5.brush.

**Run it**
```
python3 -m http.server 5173 --directory prototype
```
Then open http://localhost:5173. In Claude Code, the `prototype` entry in `.claude/launch.json` does the same.

**What it covers**
- **Canvas page:** color + size rail (left), 6 built-in p5.brush brushes in a tray (bottom), layers with hide, opacity, and delete (right), following Ajia's lo-fi sketch.
- **Timeline bar:** Fusion-style transport (⏮ ◀ ▶ ▶| ⏭) and equal-width version chips that scroll sideways. Each chip shows its most-used tool icon, and its fill shows edit size relative to the biggest version. Hovering shows time, stroke count, % of canvas, tools and sizes, and what closed the version.
- **Versions** close after a pause, a time cap, or a layer add/delete. Under **⚙ Mock settings**, "demo timing" shortens these to a 4s pause and a 30s cap, so you can see it working quickly.
- **Scrubbing** moves stroke by stroke inside a version, and the past is shown full-size on the canvas with a "Viewing v3 · stroke 4/8 · Back to now" banner (Q2). ▶ plays the whole process as a timelapse.
- **Empty state:** the bar is always visible. The hint can be turned on or off in Mock settings, so both versions can be compared (Q2).
- **Not built:** pan/zoom, eraser, undo, saving between reloads, the projects page, the history page. Export saves only the final PNG.
