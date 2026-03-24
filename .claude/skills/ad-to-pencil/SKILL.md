---
name: ad-to-pencil
description: >
  Convert ad-builder generated images into an editable Pencil (.pen) file.
  Analyzes the 3 aspect ratio images (1:1, 4:5, 9:16) using Gemini 3 Pro vision
  for precise color/layout extraction, generates clean images for EACH visual
  layer (background, decorative elements) using nano-banana-pro, then
  reconstructs them as a layered .pen file with 3 artboards.
  Uses Pencil vector primitives (rectangles, polygons, gradients) for geometric
  and decorative elements instead of generating images. Only generates images
  for photographic or complex illustration content.
  Triggers on: "convierte a pencil", "pasa a pencil", "crea el .pen",
  "convert to pencil", "make it editable", "reconstruye en pencil",
  "genera el fichero pen", or any request to turn ad images into an editable
  Pencil design file.
---

# Ad to Pencil

Convert ad-builder output images into an editable Pencil (.pen) file with 3 artboards (1:1, 4:5, 9:16).

---

## 1 — Design mindset

You are a designer rebuilding a flat image as editable layers. Every decision follows one question: **"What tool gives the most control to the person editing this file?"**

- **Editable > baked.** A gradient fill is editable (the user can change colors, angle, stops). A JPG of a gradient is not.
- **Vector > raster** for anything geometric, typographic, or color-based.
- **Raster only** for photographic content and complex hand-drawn illustration that cannot be expressed with simple shapes.
- **Overlap > stacking.** Real designs have layers that overlap. Use `layout:"none"` on artboards so every element is free-positioned.
- **Images ALWAYS in their own layer.** Never use an image fill directly on the artboard frame. Always create a child frame for the image so it can be moved, scaled, and rotated independently. The artboard with `clip:true` acts as a mask.

### Pencil capabilities (use these, don't generate images for them)

| Capability | How |
|-----------|-----|
| Solid color | `fill: "#HEX"` |
| Gradient (linear, radial, angular) | `fill: {type:"gradient", gradientType:"linear", rotation:DEG, colors:[{color:"#RRGGBBAA",position:0}, ...]}` |
| Mesh gradient | `fill: {type:"mesh_gradient", columns:N, rows:N, colors:[...], points:[...]}` — complex multi-point gradients |
| Alpha/transparency in colors | Use 8-digit hex: `#00000000` = fully transparent, `#000000FF` = opaque. Works in fills AND gradient stops. |
| Rounded corners | `cornerRadius: N` or `[topLeft, topRight, bottomRight, bottomLeft]` |
| Borders/strokes | `stroke: {fill:"#HEX", thickness:N, align:"inside\|center\|outside"}` |
| Dashed strokes | `stroke: {fill:"#HEX", thickness:N, dashPattern:[dash, gap]}` |
| Background blur (frosted glass) | `effect: {type:"background_blur", radius:N}` combined with semi-transparent fill |
| Drop shadows | `effect: {type:"shadow", shadowType:"outer", offset:{x:0,y:4}, blur:12, spread:0, color:"#00000044"}` |
| Inner shadows | `effect: {type:"shadow", shadowType:"inner", ...}` |
| Layer blur | `effect: {type:"blur", radius:N}` — blurs entire layer content |
| Multiple effects | `effect: [{type:"shadow",...}, {type:"blur",...}]` — array of effects |
| Rotation | `rotation: DEGREES` (counter-clockwise) on any node |
| Opacity | `opacity: 0.0-1.0` on any entity |
| Rectangles | `{type:"rectangle", ...}` |
| Ellipses | `{type:"ellipse", ...}` with optional `innerRadius`, `startAngle`, `sweepAngle` for arcs/rings |
| Polygons (regular) | `{type:"polygon", polygonCount:N, cornerRadius:R}` — regular N-sided polygon |
| Paths (bezier curves) | `{type:"path", geometry:"M0,0 C...", fillRule:"nonzero\|evenodd"}` — SVG-like curves and arbitrary shapes |
| Clipping | `clip:true` on frames — children are clipped to the frame bounds |
| Image fills | `fill:{type:"image", url:"./relative.jpg", mode:"fill\|fit\|stretch"}` |
| **Blend modes** | `blendMode` on ANY fill (color, gradient, image): `"multiply"`, `"overlay"`, `"screen"`, `"softLight"`, `"colorBurn"`, `"darken"`, `"hardLight"`, `"difference"`, `"exclusion"`, `"hue"`, `"saturation"`, `"color"`, `"luminosity"`, etc. |
| **Multiple fills** | `fill: [fill1, fill2, ...]` — stack multiple fills on a single node. Each can have its own blendMode and opacity. |
| **Image fill opacity** | `fill:{type:"image", url:"...", mode:"fill", opacity:0.7}` — semi-transparent images |
| Flip | `flipX: true` or `flipY: true` on any entity |

### Pro design tricks with blend modes

| Effect | How |
|--------|-----|
| Color tint on photo | Stack a color fill with `blendMode:"multiply"` over an image fill |
| Warm/cool tone | `fill:[{type:"image",...}, {type:"color", color:"#FF660033", blendMode:"overlay"}]` |
| Darken photo for text legibility | `fill:[{type:"image",...}, {type:"color", color:"#00000066", blendMode:"multiply"}]` |
| Desaturate | Use `blendMode:"saturation"` with a gray color fill |
| Duotone effect | `blendMode:"color"` with desired tint color |
| Lighten/brighten | `blendMode:"screen"` with light color |
| High contrast | `blendMode:"hardLight"` or `"softLight"` |

### Multiple fills for complex effects

A single node can have **multiple stacked fills**, each with its own blend mode and opacity. This avoids needing extra overlay layers:

```json
"fill": [
  {"type": "image", "url": "./photo.jpg", "mode": "fill"},
  {"type": "color", "color": "#D7211E44", "blendMode": "multiply"},
  {"type": "gradient", "gradientType": "linear", "rotation": 180,
   "colors": [{"color": "#00000000", "position": 0}, {"color": "#000000CC", "position": 1}]}
]
```

This creates: photo + red color tint + gradient darkening at bottom — all in one layer.

### What Pencil CANNOT do (limitations to work around)

| Limitation | Workaround |
|-----------|-----------|
| No raster transparency (JPG has no alpha) | Use Pencil shapes/gradients for anything needing transparency. Only use JPG for opaque raster content (photos, illustrations). |
| No raster masks (arbitrary shape masking) | Use `clip:true` on frames for rectangular masks, or path/polygon nodes with fill for shaped masks. |
| Polygon type is regular polygons only | For arbitrary closed shapes, use `type:"path"` with SVG geometry instead. |

---

## 2 — The layer decision tree

For every visual element in the ad, walk this tree:

```
Is it photographic content or complex illustration?
├── YES → Generate clean image with nano-banana-pro → layer_*.jpg
│         Place in its own frame with clip:true
│         Consider: can blend modes enhance it? (tint, darken, overlay)
└── NO → Can it be expressed with a single Pencil fill (solid, gradient)?
    ├── YES → Use fill property (with blendMode if needed)
    └── NO → Can it be expressed with simple Pencil geometry (rectangle, ellipse)?
        ├── YES → Use rectangle/ellipse nodes (with rotation, cornerRadius if needed)
        └── NO → Can it be expressed with a path (SVG bezier curves)?
            ├── YES → Use path node with SVG geometry
            └── NO → Generate image as last resort (on matching bg color, flat/graphic style)
```

**Always prefer higher in the tree.** Vector shapes are editable, scalable, and don't have transparency issues.

### Structural rule: images always in their own layer

**NEVER** apply an image fill directly on the artboard frame. Always create a separate child frame:

```json
// WRONG — image baked into artboard, can't move/scale independently
{"type":"frame", "name":"Ad 1:1", "fill":{"type":"image","url":"./photo.jpg","mode":"fill"}, "children":[...]}

// CORRECT — image in its own layer, artboard is the mask
{"type":"frame", "name":"Ad 1:1", "clip":true, "fill":"#000000", "layout":"none", "children":[
  {"type":"frame", "name":"layer-photo", "x":0, "y":0, "width":1080, "height":1080,
   "fill":{"type":"image","url":"./photo.jpg","mode":"fill"}},
  ...
]}
```

The artboard with `clip:true` acts as a mask — nothing visually escapes its bounds. Each image layer can be moved, scaled, rotated independently by the designer.

---

## 3 — Analyze source images

### 3.1 — Locate & upload

```bash
IMGDIR="output/$(ls -t output/ | head -1)"
ABS_IMGDIR=$(cd "$IMGDIR" && pwd)
```

Upload to Replicate for Gemini:
```bash
URL=$(curl -s -X POST "https://api.replicate.com/v1/files" \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -F "content=@$ABS_IMGDIR/ad_1x1.jpg;type=image/jpeg" \
  -F "filename=ad_1x1.jpg" | python3 -c "import json,sys; print(json.load(sys.stdin)['urls']['get'])")
```

### 3.2 — Gemini analysis

For each image, call `mcp__replicate__create_models_predictions` with `google/gemini-3-pro`, `Prefer: "wait"`, `temperature: 0.1`.

**Prompt:**

```
You are a senior designer decomposing this advertising image (WIDTH×HEIGHT px) into editable layers for a vector design tool.

For each visual element, decide whether it should be:
- A raster image (photos, complex illustrations only)
- A vector shape in the design tool (geometry, gradients, fills, shapes)

The design tool supports: solid fills, linear/radial/angular/mesh gradients with alpha stops (#RRGGBBAA), rectangles, ellipses, paths (SVG bezier geometry for arbitrary shapes), rounded corners, rotation, clipping, background blur, drop shadows (inner/outer), blend modes (multiply, overlay, screen, softLight, hardLight, colorBurn, darken, difference, exclusion, hue, saturation, color, luminosity), multiple stacked fills per layer, opacity per fill and per entity.

Return JSON:

{
  "layers": [
    {
      "id": "slug",
      "description": "What this layer shows",
      "position": {"x":0, "y":0, "w":1080, "h":700},
      "z_index": 0,
      "implementation": "raster | solid-fill | gradient-fill | rectangle | ellipse | path | frame-with-blur | multi-fill",
      "implementation_detail": {
        // For raster: {"prompt": "description for image generation without text"}
        // For solid-fill: {"color": "#HEX"}
        // For gradient-fill: {"gradientType": "linear", "rotation": 180, "colors": [{"color":"#RRGGBBAA","position":0}, ...]}
        // For rectangle: {"color": "#HEX", "rotation": 0, "cornerRadius": 0}
        // For ellipse: {"color": "#HEX"}
        // For path: {"color": "#HEX", "svg": "M0,0 C..."}
        // For frame-with-blur: {"fill": "#FFFFFFCC", "blur_radius": 20, "cornerRadius": 16}
        // For multi-fill: {"fills": [{"type":"image","url":"raster","mode":"fill"}, {"type":"color","color":"#D7211E44","blendMode":"multiply"}]}
      },
      "blend_mode": "normal",
      "opacity": 1.0
    }
  ],
  "text_elements": [
    {
      "role": "headline | subtitle | cta-label",
      "content": "exact text with \\n for breaks",
      "bounds": {"x":48, "y":56, "w":984, "h":120},
      "font": {"family_type": "geometric-sans|neo-grotesque|grotesque-sans|humanist-sans|rounded-sans|condensed-sans|slab-serif|serif|display", "weight": "bold", "size_px": 52, "line_height_ratio": 1.1},
      "color": "#HEX",
      "alignment": "left|center|right"
    }
  ],
  "cta_container": {
    "type": "pill | rounded-rect | sharp-rect | ghost | organic-shape | underline | none",
    "bounds": {"x":200, "y":900, "w":680, "h":80},
    "fill": "#HEX or null",
    "stroke": {"color": "#HEX", "width": 2} or null,
    "cornerRadius": 38,
    "implementation": "rectangle | path | frame",
    "implementation_detail": {}
  },
  "colors": {
    "dominant_bg": "#HEX", "secondary_bg": "#HEX",
    "headline_color": "#HEX", "subtitle_color": "#HEX",
    "cta_bg": "#HEX", "cta_text": "#HEX"
  }
}

IMPORTANT:
- Gradient transitions between a photo and a solid zone are NOT a separate "zone" — they are a gradient-fill layer that OVERLAPS the photo. Position the gradient so it starts 30-50% into the photo area.
- Irregular edges (torn, wavy, diagonal cuts) between zones should be path nodes with SVG geometry in the destination zone's color.
- Organic button shapes (brush strokes, blob shapes, irregular containers) should be path nodes, not raster images.
- Geometric patterns (stripes, grids, chevrons) should be multiple rotated rectangles, not raster images.
- Text is NEVER part of an image layer. Always a separate text element.
- CTA containers that are non-rectangular (organic, brush-like) should use path geometry.
- Consider where blend modes would improve the design (e.g., multiply overlay to darken a photo zone for text legibility, overlay for color tinting).
- Measure positions in pixels. Be precise.
```

### 3.3 — Parse & run in parallel

Run all 3 predictions in parallel. Concatenate output chunks: `"".join(prediction["output"])`.

---

## 4 — Homogenize across ratios

### Text
All 3 images should show the same text. Pick the clearest version.

### Colors
Average colors within ~10 RGB units. Use median for outliers.

### Typography

| family_type | fontFamily |
|-------------|-----------|
| geometric-sans | `"Montserrat"` |
| neo-grotesque | `"Inter"` |
| grotesque-sans | `"DM Sans"` |
| humanist-sans | `"Source Sans 3"` |
| rounded-sans | `"Nunito"` |
| condensed-sans | `"Barlow Condensed"` |
| slab-serif | `"Roboto Slab"` |
| serif | `"Merriweather"` |
| display | `"Montserrat"` |

### Layers
Match layers by description across the 3 ratios. Build unified list with per-ratio positions.

---

## 5 — Generate raster layers

Only for layers where `implementation: "raster"`:

```
mcp__replicate__create_models_predictions(
  model_owner: "google", model_name: "nano-banana-pro", Prefer: "wait",
  input: {
    prompt: "<from implementation_detail.prompt> No text, no words, no letters, no typography, no watermarks.",
    aspect_ratio: "4:5",  // or match layer shape
    output_format: "jpg"
  })
```

Download: `curl -sL "<output_url>" -o "$ABS_IMGDIR/layer_<id>.jpg"`

Verify with Read tool: no text in image, subject matches.

---

## 6 — Build the .pen file

### Structure

```json
{
  "version": "2.6",
  "variables": {
    "primary": {"type":"color","value":"#HEX"},
    "headlineColor": {"type":"color","value":"#HEX"},
    "subtitleColor": {"type":"color","value":"#HEX"},
    "ctaBg": {"type":"color","value":"#HEX"},
    "ctaText": {"type":"color","value":"#HEX"}
  },
  "children": [
    {"type":"frame","name":"Ad 1:1","x":0,"y":0,"width":1080,"height":1080,"clip":true,"layout":"none","children":[...]},
    {"type":"frame","name":"Ad 4:5","x":1180,"y":0,"width":1080,"height":1350,"clip":true,"layout":"none","children":[...]},
    {"type":"frame","name":"Ad 9:16","x":2360,"y":0,"width":1080,"height":1920,"clip":true,"layout":"none","children":[...]}
  ]
}
```

### Critical rules for artboard construction

1. **Artboard = mask.** Every artboard frame MUST have `clip:true` and `layout:"none"`. The artboard clips everything inside.
2. **Images in their own layer.** Every image MUST be a child frame, never the artboard's own fill. This lets the designer move, scale, rotate the image independently.
3. **Artboard fill = fallback color.** The artboard's `fill` should be the dominant background color (what shows if the image layer is moved/hidden).
4. **Layer order = z-order.** Children are rendered in order — first child is furthest back.
5. **Overlapping layers create transitions.** A gradient frame overlapping a photo frame creates a smooth fade. Position gradients to start 30-50% into the photo area.

### Building each artboard's children

Walk through the unified layer list, sorted by z_index. For each layer, build the appropriate Pencil node using the implementation type:

**`raster`** → frame with image fill (in its own layer):
```json
{"type":"frame","name":"layer-NAME","x":X,"y":Y,"width":W,"height":H,"clip":true,
 "fill":{"type":"image","url":"./layer_ID.jpg","mode":"fill"}}
```

**`raster` with color tint** → frame with multiple fills:
```json
{"type":"frame","name":"layer-NAME","x":X,"y":Y,"width":W,"height":H,"clip":true,
 "fill":[
   {"type":"image","url":"./layer_ID.jpg","mode":"fill"},
   {"type":"color","color":"#D7211E33","blendMode":"multiply"}
 ]}
```

**`solid-fill`** → frame with solid fill:
```json
{"type":"frame","name":"zone-NAME","x":X,"y":Y,"width":W,"height":H,"fill":"#HEX"}
```

**`gradient-fill`** → frame with gradient:
```json
{"type":"frame","name":"gradient-NAME","x":X,"y":Y,"width":W,"height":H,
 "fill":{"type":"gradient","gradientType":"linear","rotation":ROT,
  "colors":[{"color":"#RRGGBBAA","position":0},{"color":"#RRGGBBAA","position":1}]}}
```

**`rectangle`** → rectangle node:
```json
{"type":"rectangle","name":"deco-NAME","x":X,"y":Y,"width":W,"height":H,
 "fill":"#HEX","rotation":DEG,"cornerRadius":R}
```

**`ellipse`** → ellipse node:
```json
{"type":"ellipse","name":"deco-NAME","x":X,"y":Y,"width":W,"height":H,"fill":"#HEX"}
```

**`path`** → path node (for arbitrary shapes: organic buttons, torn edges, wavy lines, irregular shapes):
```json
{"type":"path","name":"shape-NAME","x":X,"y":Y,"width":W,"height":H,
 "fill":"#HEX","geometry":"M0,0 C10,20 30,40 50,50 Z"}
```

**`frame-with-blur`** → frame with effect:
```json
{"type":"frame","name":"glass-NAME","x":X,"y":Y,"width":W,"height":H,
 "fill":"#FFFFFFCC","cornerRadius":R,
 "effect":{"type":"background_blur","radius":BLUR}}
```

**`multi-fill`** → frame with stacked fills and blend modes:
```json
{"type":"frame","name":"layer-NAME","x":X,"y":Y,"width":W,"height":H,"clip":true,
 "fill":[
   {"type":"image","url":"./photo.jpg","mode":"fill"},
   {"type":"color","color":"#00000066","blendMode":"multiply"},
   {"type":"gradient","gradientType":"linear","rotation":180,
    "colors":[{"color":"#00000000","position":0},{"color":"#000000CC","position":1}]}
 ]}
```

**Text elements** → text nodes at the end (highest z):
```json
{"type":"text","name":"headline","x":X,"y":Y,"width":W,
 "fill":"$headlineColor","textGrowth":"fixed-width",
 "content":"Text","lineHeight":LH,"fontFamily":"Inter","fontSize":SIZE,
 "fontWeight":"bold","textAlign":"center"}
```

**CTA** → depends on cta_container.type:
- `pill`/`rounded-rect`/`sharp-rect`: frame with fill, cornerRadius, padding, child text
- `ghost`: frame with stroke (no fill), cornerRadius, padding, child text
- `organic-shape`: path node (z behind) + text node (z above), no wrapper frame. Use SVG geometry for the organic shape.
- `underline`: text node + thin rectangle below it
- `none`: just a text node

### Shadow effects on elements

When the design uses shadows:
```json
{"type":"frame","name":"card","x":X,"y":Y,"width":W,"height":H,
 "fill":"#FFFFFF","cornerRadius":12,
 "effect":{"type":"shadow","shadowType":"outer","offset":{"x":0,"y":4},"blur":16,"spread":0,"color":"#00000022"}}
```

### Write to disk

Use the Write tool to save `$IMGDIR/ad.pen`.

---

## 7 — Verify

1. `mcp__pencil__open_document(filePathOrTemplate: "$ABS_IMGDIR/ad.pen")`
2. `mcp__pencil__get_editor_state(include_schema: false)` — get artboard IDs
3. Take screenshots of each artboard
4. Read original images with Read tool
5. Compare

### Verification

For each artboard, compare the screenshot against the original image:

- **Layout fidelity**: zones, proportions, and positioning match
- **Color accuracy**: fills, text colors, CTA colors match the original
- **Transitions**: gradient fades are smooth (no hard cuts where originals have fades)
- **Decorative elements**: vector shapes render correctly (stripes, organic shapes, irregular edges)
- **Text**: content, size, weight, alignment, and position are correct
- **CTA**: shape, color, and text match the original
- **Image layers**: photos/illustrations render (not blank) — each in its own layer frame
- **Legibility**: text has sufficient contrast and background treatment
- **Blend modes**: tints and overlays look correct
- **Layer independence**: images are in separate frames (not baked into artboard fill)

Fix issues with `mcp__pencil__batch_design` U() operations, or rewrite the .pen file.

---

## 8 — Export & report

```
mcp__pencil__export_nodes(filePath: "<path>", nodeIds: [...], outputDir: "$ABS_IMGDIR", format: "png", scale: 2)
```

Report summary of artboards, variables, image layers, vector layers, and blend modes used.
