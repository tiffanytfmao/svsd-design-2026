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

- **Canvas page** (based on Ajia's lo-fi sketch): color and size controls on the left, 6 built-in p5.brush brushes in a bottom tray (pen, marker, pencil, soft, charcoal, spray), and layers on the right with hide, opacity, and delete.
- **Every stroke is saved as data:** `{brush, color, weight, layer, seed, segments, time}`. Replaying a stroke with the same seed redraws it identically.
- **Versions** are groups of strokes. A version closes after a **pause** in drawing, a **time cap**, or a **layer being added or deleted**, whichever comes first. The real rules are a 90s pause and a 10-minute cap. Under **⚙ Mock settings**, "demo timing" shortens these to 4s and 30s so you can see it work quickly.
- **Timeline bar** (like Fusion 360's): play controls (⏮ ◀ ▶ ▶| ⏭) and equal-width version chips that scroll sideways. Each chip shows its most-used tool, and its fill shows how much of the canvas changed. Hovering shows time, stroke count, % of canvas, tools and sizes, and what closed the version.
- **Scrubbing** moves stroke by stroke. The past version shows full-size on the canvas with a "Viewing v3 · stroke 4/8 · Back to now" banner. ▶ plays the whole process as a timelapse.
- **Empty state:** the bar is always visible, and the hint can be turned on or off in Mock settings to compare.

**Placeholders and gaps:** drawing on a past version is blocked (the behavior is still undecided: auto-fork vs. read-only + restore). Past versions show as a flattened image, ignoring layer order and visibility. No pan/zoom, eraser, undo, saving between reloads, projects page, or history page. Export saves only the final PNG.

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
