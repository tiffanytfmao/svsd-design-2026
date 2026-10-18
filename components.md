# Rewind — Canvas Page Components (layout C1)

Inventory of everything on the canvas page. Layout: floating tools and layers over a full-screen canvas, with a wide floating timeline that stays open (see decisions.md, Q4).

**Designed for tablet first** (decisions.md, Q5): iPad landscape frame (1180×820), touch targets at least 44px, and nothing that only works on hover.

## 1. Canvas
- **Canvas surface:** the paper area, with a background color
- **Zoom indicator:** small "100%" readout, tap to reset *(optional)*
- **Cursor:** a brush-size circle that follows the pen *(optional)*

## 2. Project buttons (top left)
- **Back button:** arrow that goes to the projects page
- **Project name:** editable title next to it

## 3. Export (top right)
- **Export button**
- **Export menu:** three options: timelapse, branch version as an image, final image

## 4. Side bar / tool rail (floating, left)
- **Brush button**, one per brush (5–6 total). States: default, hover, selected
- **Eraser button**, with the same states
- **Color swatch:** shows the current color and opens the color picker
- **Color picker popover:** color field, hue slider, recent colors
- **Size slider:** vertical, with a value readout
- **Opacity slider:** vertical, with a value readout
- **Undo / redo buttons** *(optional; may be covered by the timeline)*

## 5. Layers panel (floating, right)
- **Panel header:** "Layers" title, add-layer button, and a button to collapse the panel
- **Layer row**
  - Thumbnail
  - Name (editable)
  - Show/hide toggle (eye icon)
  - Opacity slider (in the row or when the row is expanded)
  - Delete button
  - States: default, active (selected), hidden (dimmed, eye closed), hover
- **Collapsed panel:** a small chip showing the layers icon and how many layers there are

## 6. Timeline bar (floating, bottom)
- **Bar container:** slim and partly see-through. States: default, faded while drawing, empty
- **Play / pause button**
- **Version segment.** States: default, hover, selected, current, after the playhead (grayed), fork notch
- **Playhead:** a draggable line or handle
- **Playhead label** (decisions.md Q5): combines the tooltip and the past-version banner. Information only: "v3 · Oct 17 · 8 min · +12%." Shown above the playhead whenever it's on a past version. On desktop, hovering a segment shows the same label in gray as a preview.
- **Back to now button:** shown only while viewing the past. For now it floats above the bar on the right *(working choice, not final)*.
- **History button:** opens the history page
- **Scrolling:** what the ends of the bar look like when the history is too long to fit

## 7. Messages and overlays
- **Past-version banner:** merged into the timeline's playhead label (see 6)
- **Toast:** "Started a new branch from v3" *(only if we go with auto-fork, which is tabled)*
- **Fork from here button** *(only if we go with read-only + restore, also tabled)*

## 8. Shared pieces (used by everything above)
- **Icon button:** the base for tool, layer, and timeline buttons
- **Floating panel:** the shared container style for the rail, layers panel, timeline, and chips
- **Slider:** used for size, opacity, and layer opacity
- **Tooltip / popover:** one shared style
- **Icons:** brushes, eraser, eye open/closed, trash, plus, play/pause, branch, history, export, arrow
- **Tokens:** colors, type, corner radius, spacing
