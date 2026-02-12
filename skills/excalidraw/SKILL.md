---
name: excalidraw
description: Use this skill whenever the user wants to create, edit, or modify Excalidraw diagrams and .excalidraw files. This includes generating new diagrams from scratch, editing existing .excalidraw files, adding or removing elements (shapes, arrows, text, frames), creating flowcharts, architecture diagrams, sequence flows, decision trees, and any visual diagram in Excalidraw format. Triggers include any mention of "excalidraw", ".excalidraw", "diagram", "flowchart", "whiteboard sketch", or requests to visualize processes, systems, or workflows as drawings. Do NOT use for SVG, PNG, Mermaid, or other non-Excalidraw diagram formats.
license: MIT
allowed-tools: [Read, Write, Edit, Bash, Glob]
---

# Excalidraw Diagram Skill

You can create and edit `.excalidraw` files directly as JSON. No CLI needed.

## File Structure

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [ ... ],
  "appState": {
    "gridSize": 20,
    "gridStep": 5,
    "gridModeEnabled": false,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

### `appState` options

- `viewBackgroundColor`: Background color of the canvas (default `"#ffffff"`; use `"transparent"` for no background)
- `theme`: `"light"` (default) or `"dark"`
- `gridModeEnabled`: Set `true` to display a snap grid
- `gridSize` / `gridStep`: Grid cell size and subdivision (only relevant when grid is enabled)

## Element Defaults

Every element needs these base properties. Use these defaults unless overridden:

```json
{
  "version": 1,
  "versionNonce": <random 9-digit integer>,
  "index": "a0",
  "isDeleted": false,
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid",
  "roughness": 2,
  "opacity": 100,
  "angle": 0,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "seed": <random 9-digit integer>,
  "groupIds": [],
  "frameId": null,
  "roundness": null,
  "boundElements": [],
  "updated": <Date.now() timestamp>,
  "link": null,
  "locked": false
}
```

- `index`: Assign sequentially per element — see [Index](#index-element-ordering) for the ordering scheme.
- `seed`: Each element must have a **unique** seed — it controls the randomized hand-drawn rendering pattern. Reusing seeds makes shapes look unnaturally identical.
- `roundness`: This base default is `null` (sharp corners). Override per shape type — see [Shapes](#shapes) for recommended values per type.

## Element IDs

Generate short descriptive IDs for new elements (e.g., `"push_box"`, `"arrow_push_to_merge"`). For bound text labels, append `_label` (e.g., `"push_box_label"`).

## Index (Element Ordering)

Elements use fractional indexing strings for z-order: `"a0"`, `"a1"`, ... `"a9"`, `"aA"`, ... `"aZ"`, `"aa"`, `"ab"`, etc. When adding elements to an existing file, continue from the highest existing index.

## Shapes

### Rectangle, Ellipse, Diamond

```json
{
  "type": "rectangle",
  "id": "my_box",
  "x": 100, "y": 200,
  "width": 200, "height": 80,
  "roundness": { "type": 3 },
  "backgroundColor": "#a5d8ff",
  "fillStyle": "solid",
  "boundElements": [
    { "id": "my_box_label", "type": "text" }
  ]
}
```

- `type`: `"rectangle"`, `"ellipse"`, or `"diamond"`
- `roundness: { "type": 3 }` = rounded corners (use for rectangles by default)
- `roundness: { "type": 2 }` = smooth curves (use for ellipses/diamonds)
- `roundness: null` = sharp corners

### Diamond (decision/gate nodes)

```json
{
  "type": "diamond",
  "id": "gate",
  "x": 600, "y": 150,
  "width": 130, "height": 90,
  "roundness": { "type": 2 },
  "backgroundColor": "#fff3bf",
  "fillStyle": "solid",
  "boundElements": [
    { "id": "gate_label", "type": "text" }
  ]
}
```

Diamonds render as rotated squares. The `width`/`height` define the bounding box — the visible diamond touches the midpoints of each side. Size diamonds ~1.5x larger than you'd expect since the usable interior is smaller.

### Fill styles

- `"solid"` — flat color fill (default, clean look)
- `"hachure"` — diagonal line fill (classic excalidraw hand-drawn feel)
- `"cross-hatch"` — cross-hatched lines (good for "proposed" or "in progress" elements)

### Roughness levels

- `0` = Architect — clean, precise lines
- `1` = Artist — slight wobble
- `2` = Cartoonist — full hand-drawn feel (default)

### Common colors

| Color | Hex | Use for |
|-------|-----|---------|
| Blue | `#a5d8ff` | Standard steps |
| Purple | `#d0bfff` | Tooling / automation |
| Green | `#b2f2bb` | Success / prod deploy |
| Light green | `#c3fae8` | Positive state |
| Orange | `#ffd8a8` | Manual / warning |
| Yellow | `#ffd43b` | Highlighted / attention |
| Light yellow | `#fff3bf` | Decision gates |
| Red bg | `#ffc9c9` | Problem / danger |
| Red stroke | `#ef4444` | Problem border/text |
| Gray text | `#757575` | Annotations |

## Text Elements

### Standalone text (labels, annotations)
```json
{
  "type": "text",
  "id": "my_label",
  "x": 100, "y": 200,
  "width": 0, "height": 0,
  "text": "Some label",
  "fontSize": 16,
  "fontFamily": 5,
  "textAlign": "left",
  "verticalAlign": "top",
  "containerId": null,
  "originalText": "Some label",
  "autoResize": true,
  "lineHeight": 1.25
}
```

Note: `width: 0, height: 0` is fine — excalidraw auto-calculates on first render. However, if other elements (arrows, adjacent shapes) need to be positioned relative to this text, estimate the dimensions: width ≈ `character_count × fontSize × 0.6`, height ≈ `fontSize × lineHeight × line_count`.

### Bound text (label inside a shape)
```json
{
  "type": "text",
  "id": "my_box_label",
  "x": 110, "y": 210,
  "width": 180, "height": 60,
  "text": "Box Label",
  "fontSize": 16,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": "my_box",
  "originalText": "Box Label",
  "autoResize": true,
  "lineHeight": 1.25
}
```

**Critical**: The parent shape's `boundElements` must include `{ "id": "my_box_label", "type": "text" }`.

**Sizing**: Set the bound text's `width` and `height` to the parent shape's dimensions minus ~20px padding per side (e.g., for a 200×80 box, use width ~160, height ~40 for the text).

### Font families
- `5` = Excalidraw default (hand-drawn, current — use this)
- `1` = Virgil (hand-drawn, legacy)
- `2` = Helvetica (clean sans-serif)
- `3` = Cascadia (monospace — good for code/technical labels)

### Font size guidelines
- Title: 28
- Normal label: 16-18
- Annotation/note: 14

## Lines

Plain lines without arrowheads — useful for separators, connectors, and underlines.

```json
{
  "type": "line",
  "id": "separator",
  "x": 100, "y": 300,
  "width": 400, "height": 0,
  "points": [[0, 0], [400, 0]],
  "startArrowhead": null,
  "endArrowhead": null
}
```

- Same point system as arrows: `x, y` is origin, `points` are relative
- Can have multiple points for polylines
- Supports `strokeStyle`: `"solid"`, `"dashed"`, `"dotted"`
- Add `"roundness": { "type": 2 }` for smooth curves through points

## Arrows

```json
{
  "type": "arrow",
  "id": "arrow_a_to_b",
  "x": 300, "y": 240,
  "width": 50, "height": 0,
  "points": [[0,0], [50, 0]],
  "startBinding": {
    "elementId": "box_a",
    "mode": "orbit",
    "fixedPoint": [1, 0.5]
  },
  "endBinding": {
    "elementId": "box_b",
    "mode": "orbit",
    "fixedPoint": [0, 0.5]
  },
  "startArrowhead": null,
  "endArrowhead": "arrow",
  "elbowed": false
}
```

### Arrow position and points
- `x, y` is the starting position of the arrow
- `points` are relative to `x, y`: first point is always `[0, 0]`
- `width` = max x extent, `height` = max y extent of the points

### Arrow bindings
- `fixedPoint: [x, y]` where x,y are 0-1 relative to the target element
  - `[0, 0.5]` = left center
  - `[1, 0.5]` = right center
  - `[0.5, 0]` = top center
  - `[0.5, 1]` = bottom center
- `mode`: `"orbit"` (default) or `"inside"`
- Set to `null` for unbound arrows

**Critical**: The target shape's `boundElements` must include `{ "id": "arrow_a_to_b", "type": "arrow" }`.

### Arrow styles
- `endArrowhead`: `"arrow"` (default), `"triangle"`, `"bar"`, `"dot"`, `null`
- `startArrowhead`: same options (for bidirectional arrows)
- `strokeStyle`: `"solid"`, `"dashed"`, `"dotted"`

### Curved arrows (multi-point)

Use 3+ points with roundness for smooth curves:

```json
{
  "type": "arrow",
  "id": "curved_arrow",
  "x": 300, "y": 240,
  "width": 150, "height": 100,
  "points": [[0, 0], [75, 50], [150, 100]],
  "roundness": { "type": 2 },
  "startBinding": null,
  "endBinding": null,
  "startArrowhead": null,
  "endArrowhead": "arrow",
  "elbowed": false
}
```

The middle point(s) act as control points — the arrow curves smoothly through them.

### Elbowed (right-angle) arrows

Set `"elbowed": true` on an arrow to make it route with right-angle bends. Excalidraw auto-routes the path between bound elements. For most cases, omit `fixedSegments` and let auto-routing handle it:

```json
{
  "type": "arrow",
  "elbowed": true,
  "startBinding": { "elementId": "box_a", "mode": "orbit", "fixedPoint": [1, 0.5] },
  "endBinding": { "elementId": "box_b", "mode": "orbit", "fixedPoint": [0, 0.5] },
  "startArrowhead": null,
  "endArrowhead": "arrow"
}
```

`fixedSegments` is an advanced option that pins specific segments of the path to fixed positions. Each entry has `index` (which segment in the auto-routed path, 0-indexed), `start` and `end` (absolute `[x, y]` coordinates of that segment's endpoints). Prefer auto-routing unless you need precise control.

## Groups

Group elements so they're logically associated. All grouped elements share the same group ID in their `groupIds` array.

```json
[
  { "id": "box_a", "groupIds": ["group_pipeline"] },
  { "id": "box_b", "groupIds": ["group_pipeline"] },
  { "id": "arrow_ab", "groupIds": ["group_pipeline"] }
]
```

- Generate group IDs as descriptive strings (e.g., `"group_pipeline"`, `"group_deploy"`)
- An element can belong to multiple groups (nested grouping)
- Groups don't have their own element entry — they exist only via `groupIds` references
- In excalidraw UI, grouped elements select together on click

## Frames

Named containers that visually group a section of a diagram. Frames render as a labeled rectangle with a title.

```json
{
  "type": "frame",
  "id": "frame_initiative1",
  "x": 50, "y": 50,
  "width": 600, "height": 400,
  "name": "Initiative 1: Feature Branches",
  "roundness": null,
  "boundElements": [],
  "version": 1, "versionNonce": 123456789,
  "isDeleted": false, "fillStyle": "solid",
  "strokeWidth": 2, "strokeStyle": "solid",
  "roughness": 0, "opacity": 100, "angle": 0,
  "strokeColor": "#bbb", "backgroundColor": "transparent",
  "seed": 987654321, "groupIds": [], "frameId": null,
  "updated": <Date.now() timestamp>, "link": null, "locked": false,
  "index": "a0"
}
```

- Child elements point to the frame via `"frameId": "frame_initiative1"`
- Frame `name` renders as a label above the frame
- Use `roughness: 0` for frames (clean lines look better)
- Size the frame to contain all child elements with ~20px padding

## Links

Make elements clickable by setting the `link` property:

```json
{
  "id": "my_box",
  "link": "https://example.com/docs",
  ...
}
```

- Works on any element type
- In excalidraw UI, linked elements show a link icon on hover
- Useful for linking diagram nodes to docs, Jira tickets, etc.

## Opacity

Control transparency with `opacity` (0-100):

- `100` = fully opaque (default)
- `50` = semi-transparent (good for background/context elements)
- `25` = ghost/watermark effect

Useful for "before/after" diagrams — show the old flow at low opacity with the new flow at full opacity on top.

## Element Locking

Prevent accidental edits on finalized elements:

```json
{ "id": "my_box", "locked": true, ... }
```

## Operations

### Creating a new file
1. Build the elements array with all shapes, text labels, and arrows
2. Ensure all bindings are bidirectional (shape references arrow AND arrow references shape)
3. Assign sequential index values starting from `"a0"`
4. Wrap in the file structure and write with the Write tool

### Adding elements to an existing file
1. Read the existing file
2. Find the highest index value: `jq '[.elements[] | .index] | sort | last'`
3. Create new elements with indices after the highest
4. Add new elements to the elements array
5. Update any existing elements' `boundElements` if new arrows connect to them
6. Write the updated file using the Write tool, or use Edit for targeted property changes

### Editing elements
Use the Edit tool to find and replace specific JSON properties in the file.

### Deleting elements
Set `"isDeleted": true` on the element. Don't remove it from the array — excalidraw uses tombstones.

### Cleaning up
Find and mark as deleted:
- Elements with `null` x/y coordinates
- Text elements with `containerId` pointing to a deleted element
- Arrows with bindings pointing to deleted elements
- Use: `jq '[.elements[] | select(.x == null) | select(.isDeleted == false or .isDeleted == null) | .id]'`

### Inspecting a file
List all active elements with positions:
```bash
jq '[.elements[] | select(.isDeleted == false or .isDeleted == null) | {id, type, x, y, width, height, text: (if .type == "text" then .text else null end)} | del(.[] | nulls)]' file.excalidraw
```

## Layout Guidelines

- Standard box size: 180-220w x 70-80h
- Diamond size: 130-150w x 90-110h (larger than boxes due to rotated shape)
- Arrow gap between boxes: 40-60px
- Row spacing: 120-150px
- Keep main flow horizontal, branches vertical
- Total width ~1200px max for readability
- Frame padding: ~20px around contained elements

## Template Patterns

### Flowchart row (evenly spaced boxes with arrows)
```
Box A (x=50)  --arrow(50px gap)-->  Box B (x=300)  --arrow-->  Box C (x=550)
```
Formula: next_x = prev_x + prev_width + arrow_gap(50)

### Fork/join (one input, multiple outputs)
```
            --> Box B (y - 70)
Box A --+
            --> Box C (y + 70)
```
Use two arrows from Box A with different fixedPoints: `[1, 0.3]` and `[1, 0.7]`

### Decision gate
```
Box A --> ◇ Decision --> Box B (yes)
              |
              v
          Box C (no)
```
Diamond with arrows from right (yes) and bottom (no). Add standalone text labels "yes"/"no" near each arrow.

### Before/after overlay
Show old flow at `opacity: 30` with new flow at `opacity: 100` on top. Use `strokeStyle: "dashed"` on removed elements.

## Complete Example

A minimal diagram: two labeled boxes connected by an arrow. Every element includes all required base properties.

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [
    {
      "type": "rectangle",
      "id": "box_a",
      "x": 100,
      "y": 200,
      "width": 200,
      "height": 80,
      "index": "a0",
      "version": 1,
      "versionNonce": 482973651,
      "isDeleted": false,
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 2,
      "opacity": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "#a5d8ff",
      "seed": 129837461,
      "groupIds": [],
      "frameId": null,
      "roundness": { "type": 3 },
      "boundElements": [
        { "id": "box_a_label", "type": "text" },
        { "id": "arrow_a_to_b", "type": "arrow" }
      ],
      "updated": 1700000000000,
      "link": null,
      "locked": false
    },
    {
      "type": "text",
      "id": "box_a_label",
      "x": 120,
      "y": 220,
      "width": 160,
      "height": 40,
      "index": "a1",
      "version": 1,
      "versionNonce": 583920147,
      "isDeleted": false,
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 2,
      "opacity": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "transparent",
      "seed": 294817362,
      "groupIds": [],
      "frameId": null,
      "roundness": null,
      "boundElements": [],
      "updated": 1700000000000,
      "link": null,
      "locked": false,
      "text": "Start",
      "fontSize": 18,
      "fontFamily": 5,
      "textAlign": "center",
      "verticalAlign": "middle",
      "containerId": "box_a",
      "originalText": "Start",
      "autoResize": true,
      "lineHeight": 1.25
    },
    {
      "type": "rectangle",
      "id": "box_b",
      "x": 350,
      "y": 200,
      "width": 200,
      "height": 80,
      "index": "a2",
      "version": 1,
      "versionNonce": 719204583,
      "isDeleted": false,
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 2,
      "opacity": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "#b2f2bb",
      "seed": 847261039,
      "groupIds": [],
      "frameId": null,
      "roundness": { "type": 3 },
      "boundElements": [
        { "id": "box_b_label", "type": "text" },
        { "id": "arrow_a_to_b", "type": "arrow" }
      ],
      "updated": 1700000000000,
      "link": null,
      "locked": false
    },
    {
      "type": "text",
      "id": "box_b_label",
      "x": 370,
      "y": 220,
      "width": 160,
      "height": 40,
      "index": "a3",
      "version": 1,
      "versionNonce": 204719583,
      "isDeleted": false,
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 2,
      "opacity": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "transparent",
      "seed": 591037284,
      "groupIds": [],
      "frameId": null,
      "roundness": null,
      "boundElements": [],
      "updated": 1700000000000,
      "link": null,
      "locked": false,
      "text": "End",
      "fontSize": 18,
      "fontFamily": 5,
      "textAlign": "center",
      "verticalAlign": "middle",
      "containerId": "box_b",
      "originalText": "End",
      "autoResize": true,
      "lineHeight": 1.25
    },
    {
      "type": "arrow",
      "id": "arrow_a_to_b",
      "x": 300,
      "y": 240,
      "width": 50,
      "height": 0,
      "index": "a4",
      "version": 1,
      "versionNonce": 382047195,
      "isDeleted": false,
      "fillStyle": "solid",
      "strokeWidth": 2,
      "strokeStyle": "solid",
      "roughness": 2,
      "opacity": 100,
      "angle": 0,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "transparent",
      "seed": 738291046,
      "groupIds": [],
      "frameId": null,
      "roundness": { "type": 2 },
      "boundElements": [],
      "updated": 1700000000000,
      "link": null,
      "locked": false,
      "points": [[0, 0], [50, 0]],
      "startBinding": {
        "elementId": "box_a",
        "mode": "orbit",
        "fixedPoint": [1, 0.5]
      },
      "endBinding": {
        "elementId": "box_b",
        "mode": "orbit",
        "fixedPoint": [0, 0.5]
      },
      "startArrowhead": null,
      "endArrowhead": "arrow",
      "elbowed": false
    }
  ],
  "appState": {
    "gridSize": 20,
    "gridStep": 5,
    "gridModeEnabled": false,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

Note the bidirectional bindings: both `box_a` and `box_b` list `arrow_a_to_b` in their `boundElements`, and the arrow's `startBinding`/`endBinding` reference each box. Both text elements have `containerId` pointing to their parent, and each parent's `boundElements` includes the text. Every element has a unique `seed` and a sequential `index`.

## Checklist Before Writing

1. Every shape with a label has a matching text element with `containerId`
2. Every shape with a label has `boundElements` including the text element
3. Every arrow binding target has `boundElements` including the arrow
4. All `originalText` matches `text`
5. All indices are sequential and unique
6. No `null` coordinates on any active element
7. Groups: all elements in a group share the same `groupIds` entry
8. Frames: all child elements have `frameId` pointing to the frame

## Instructions

When asked to create or edit an excalidraw diagram:
1. If editing, READ the existing file first
2. Plan the layout with coordinates before writing
3. Build complete, valid JSON — don't skip properties
4. Double-check all bindings are bidirectional
5. Write the file directly using Write or Edit tools
