# Rewind: proof of concept

> **This branch is a throwaway demo, not the main project.**
> It was built on 2026-10-17 to test how a canvas page with a Fusion 360–style history timeline could work, and whether p5.brush can handle live drawing.
> Design decisions and the real build happen elsewhere (see `main`). Don't build on this branch; borrow from it.

Rewind is version control for digital art, made for the SVSD 2026 design competition. It records an artist's process so they can rewind it, explore it, and share the story of how a piece was made.

## Run it

```bash
python3 -m http.server 5173 --directory prototype
```

Then open http://localhost:5173.

## What the demo shows

- **Canvas page, layout C1:** the white canvas fills the window and everything floats over it: back + project name (top left), Export (top right), a tool rail on the left (6 built-in p5.brush brushes plus an eraser as round buttons, a color swatch with 8 default colors, and size and opacity sliders), and a layers panel on the right with hide, opacity, and delete.
- **Visual system:** white floating panels, dark purple (#20255D) text and icons, Rose 50 segments, tan selection with a purple ring, an orange playhead, and M PLUS Rounded 1c type. The CSS variables use the same names as the Figma variables (e.g. `--color-bg-segment-selected`).
- **Every stroke is saved as data:** `{brush, color, opacity, weight, layer, seed, segments, time}`. Replaying a stroke with the same seed redraws it identically. Eraser strokes are recorded the same way.
- **Versions** are groups of strokes. A version closes after a **pause** in drawing, a **time cap**, or a **layer being added or deleted**, whichever comes first. The real rules are a 90s pause and a 10-minute cap. Under **⚙ mock settings**, "demo timing" shortens these to 4s and 30s so you can see it work quickly.
- **Timeline pill** (like Fusion 360's): a play button and equal-width version segments that scroll sideways, each showing its most-used tool. Segments after the playhead turn white with a dashed outline. The pill fades while you draw and comes back when the pen lifts.
- **Scrubbing** moves stroke by stroke, and the past version shows full-size on the canvas. One label above the playhead shows "v3 · Oct 17 · 8 min · +12%" (on desktop, hovering a segment previews it in white). A "Back to now" pill appears above the bar while you're viewing the past. ▶ plays the whole process as a timelapse.

**Placeholders and gaps:** drawing on a past version is blocked (the behavior is still undecided: auto-fork vs. read-only + restore). Past versions show as a flattened image, ignoring layer order and visibility. No pan/zoom, undo, saving between reloads, projects page, or history page. Export saves only the final PNG. The canvas is sized once at load.

## What we learned about p5.brush (2.2.3, with p5 2.3.3)

- Brush strokes are buffered and only written at the **end of each frame** (p5's `postdraw`), to whichever target is loaded at that moment. Calling `brush.load()` to switch back mid-frame **throws away** the pending strokes.
- `p5.Framebuffer` targets didn't receive brush strokes in testing, even though the docs list them. **`createGraphics(w, h, WEBGL)` works.**
- So in each frame: do all regular p5 drawing first, then load **one** brush target and make the brush calls last. They appear on the next frame.
- Each WEBGL graphics is its own WebGL context (browsers allow ~16), so layers are capped at 8 here.
- Drawing short `brush.line()` segments on every pointer move is fast enough for live drawing, even though the library's author says it's meant for high-res artwork rather than real-time use.
- Redrawing textured strokes is slow, so scrubbing starts from a **snapshot saved at the end of each version** and only redraws the strokes after it.
- In p5 2.x, `redraw()` is async and returns a Promise.

## Files

- `prototype/index.html`, `style.css`, `app.js`: the demo
- `prototype/lib/`: p5 2.3.3 and p5.brush 2.2.3 (MIT, see `LICENSE-p5.brush.md`)
