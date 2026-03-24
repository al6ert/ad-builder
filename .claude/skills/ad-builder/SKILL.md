---
name: ad-builder
description: >
  Generate professional advertising images using Replicate AI models.
  Use this skill whenever the user wants to create an ad, promotional image, social media
  creative, or any marketing visual — even if they don't say "ad" explicitly.
  Triggers on: "crea un anuncio", "genera una imagen", "make an ad", "create a social post",
  advertising brief with colors and copy, "diseña un banner", "quiero una imagen para
  redes", or any request that includes headline/CTA/brand color combinations.
  When in doubt, use this skill — it handles the full workflow from brief to saved PNG.
---

# Ad Builder

Generate advertising images using Replicate AI, following Meta best practices.

**IMPORTANT**: Output is always a set of image files downloaded from Replicate. Never generate HTML. Every ad requires a hero visual — pure text-only ads are not allowed. The workflow always produces **3 files**: one per aspect ratio (1:1, 4:5, 9:16).

---

## Model

All generation uses a single model:

```
model: "google/nano-banana-pro"
Prefer: "wait"
minimum quality params:
  output_format: "jpg"
  allow_fallback_model: true
```

> `Prefer: "wait"` tells Replicate to hold the connection up to 60 seconds and return the finished result in a single API call. Only fall back to polling with `get_predictions` if the response comes back with `status == "starting"` or `"processing"` (timeout exceeded — then poll).

Each of the 3 variations must pass the correct `aspect_ratio` parameter:

| Variation | aspect_ratio param |
|-----------|--------------------|
| 1:1       | `"1:1"`            |
| 4:5       | `"4:5"`            |
| 9:16      | `"9:16"`           |

---

## Required Inputs

| Field | Notes |
|-------|-------|
| **Título** | Main headline |
| **Subtítulo** | Secondary line (can be empty) |
| **CTA** | Call-to-action text |
| **Color primario** | Hex value |
| **Colores adicionales** | Optional — derive palette if not given |
| **Restricciones de marca** | Optional — font, style, logo rules, layout constraints |

If any required field is missing, ask before proceeding.

---

## Meta Ads Best Practices (apply always)

**Composition**
- Every ad needs a **hero visual** — a compelling image (lifestyle, product, concept, emotion). Pure typographic ads underperform. **Never generate an ad without a hero image.**
- **Text overlay ≤ 15% of image area** — large, concise, punchy. More text = less reach.
- Text only in zones of visual calm: solid color areas, gradients, sky, negative space. Never over busy or noisy areas.
- Clear focal point, depth, negative space.

**Visual hierarchy**
1. Hero visual captures attention first
2. Headline (large, high contrast) communicates value
3. Subheadline adds context
4. CTA closes the action

**Style**
- Emotion > information — show the feeling, not just the feature
- Lifestyle beats product shots
- High contrast, bold type, no clutter

---

## Step 1 — Color Palette

If only a primary color is given, derive a 2–3 color palette:

1. Compute HSL
2. Complementary: hue +180°
3. Analogous: hue ±30°
4. Check contrast ratios — text needs ≥4.5:1 on its background (WCAG AA)
5. Prefer clean neutrals (black, white, near-white) when primary is saturated

Tell the user which palette and why. When explicit colors are given, follow them exactly.

---

## Step 2 — Layout Selection

Each invocation should produce a **different layout**. This encourages creative exploration and prevents repetitive drafts. Unless the brief specifies a layout, **pick one of the archetypes below using variety** — don't default to the same one every time.

All 3 ratio variations share the same layout — adapt proportions per ratio.

### Adaptation rules by ratio

| Ratio | Behavior |
|-------|----------|
| **1:1** | Base reference. Use the percentages as described in each layout. |
| **4:5** | More vertical space. Increase separation between top and bottom text elements by ~10–15%. Hero gets more room. |
| **9:16** | Maximum vertical space. Spread text elements further apart. Keep headline away from top 8% (Stories profile/close UI). Keep CTA above bottom 6% (swipe-up zone). Right-aligned elements must avoid right 10% (Stories reaction buttons). |

---

### Layout archetypes

**A — Lower Third** *(classic, high reliability)*
- All text grouped in the bottom 30–35% of the canvas
- Headline centered, subtitle below, CTA below subtitle
- Hero fills the rest
- Text zone: `bottom 30–35%`
- Text alignment: `center`
- Best for: safe default, any brief, proven thumb-stop

**D — Full Bleed + Floating**
- Hero fills the entire canvas
- All text floated together in the upper-left area where the image is clean
- Headline top, subtitle below, CTA below subtitle — all left-aligned
- Text zone: `top-left, grouped`
- Text alignment: `left`
- Best for: lifestyle images, emotional scenes, Stories

**F — Centered Overlay**
- Hero fills the entire canvas
- All text centered in the middle third of the canvas, stacked vertically
- Headline, subtitle, CTA — all centered
- Text zone: `center third, grouped`
- Text alignment: `center`
- Best for: symmetrical compositions, bold statements, works well in all 3 ratios

**I — Bookend** *(top priority for Stories/Reels)*
- Headline + subtitle grouped at the top, centered
- CTA isolated at the bottom, centered
- Hero visible between both text groups
- Text zone: `top 15% + bottom 10%`
- Text alignment: `center`
- Best for: 9:16 Stories (CTA lands on swipe-up zone), vertical formats, guided reading flow

**J — Z-Read**
- Headline top-left
- Subtitle bottom-right
- CTA below subtitle, bottom-right
- The eye follows a Z pattern across the full image
- Text zone: `top-left + bottom-right`
- Text alignment: `headline left, subtitle + CTA right`
- Best for: maximum hero visibility, 1:1 and 4:5 feed, dynamic compositions

**K — Headline Crown**
- Headline large at the very top, full width, centered
- Subtitle + CTA small at the bottom-left, left-aligned
- Headline dominates visually; CTA closes at the bottom
- Text zone: `top 12% + bottom 15%`
- Text alignment: `headline center, subtitle + CTA left`
- Best for: short punchy headlines, brand awareness, bold typography

**L — Scattered Vertical**
- Each element in a different vertical third of the canvas
- Headline top-left
- Subtitle middle-right
- CTA bottom-center
- Alternating alignment forces the eye to travel the full image
- Text zone: `three separate zones`
- Text alignment: `alternating (left, right, center)`
- Best for: 9:16 (plenty of vertical space to separate), editorial feel, high dwell time

**M — Bottom Dual**
- Bottom area split into two columns
- Headline + subtitle stacked on the left
- CTA as a large button on the right, vertically centered with the text
- Hero fills the top 70–75%
- Text zone: `bottom 25–30%, two columns`
- Text alignment: `headline + subtitle left, CTA right`
- Best for: conversion ads, product launches, breaking the monotony of centered lower thirds

**N — Right Stack**
- All text grouped on the right side, vertically centered
- Headline, subtitle, CTA — all right-aligned, stacked
- Hero subject occupies the left side of the canvas
- Text zone: `right 40%, vertically centered`
- Text alignment: `right`
- Best for: hero subjects facing/pointing left, product images with left emphasis, people looking at camera

**O — CTA Island**
- Headline + subtitle grouped at the top, centered
- CTA isolated as a floating button at the bottom-right corner
- Large separation between text group and CTA creates visual tension
- Text zone: `top 15% centered + bottom-right corner`
- Text alignment: `headline + subtitle center, CTA right`
- Best for: conversion-focused ads, the isolated CTA draws attention as a standalone element

---

### Layout selection guidance

When no layout is specified in the brief, use this priority:

1. **Match the hero subject position** — if the subject is left-heavy, prefer N (right stack) or J (Z-read). If centered, prefer A, F, or I.
2. **Match the ratio** — for 9:16 prefer I (bookend), L (scattered), or O (CTA island). For 1:1 prefer A, J, or M.
3. **Match the goal** — for conversion (CTA matters most), prefer O, M, or I. For awareness (headline matters most), prefer K or F.
4. **Rotate** — don't repeat the same layout within a set of variations for the same brief.

> The layout is a **creative constraint, not a rigid template**. Adapt proportions to serve the brief and the image. The percentages are starting points — shift them if the hero composition demands it.

---

## Step 2.5 — Visual Treatment

After choosing a layout, choose **one option from each treatment dimension** below. These are independent of the layout — any combination works. The goal is to break visual monotony and add art direction to each ad.

---

### Treatment dimensions

#### T1 — Text background

| Option | Prompt language | When to use |
|--------|----------------|-------------|
| **Solid block** | "solid [color] rectangle behind the text, opaque, clean edges" | Safe default. High legibility. Works with any image. |
| **Gradient fade** | "smooth gradient fading from transparent to [color], covering bottom N%" | When the image should bleed into the text zone. Cinematic feel. |
| **Frosted glass** | "translucent frosted glass panel behind the text, blurred background visible through it" | Premium, modern. Great for Stories. Keeps image visible. |
| **Naked** | "text placed directly on the image with no background, high contrast [color] text on clean area of the image" | Maximum impact. Only use when the image has large, calm, uniform areas. |

> **Default**: Gradient fade.

#### T2 — CTA shape

| Option | Prompt language | Personality |
|--------|----------------|-------------|
| **Pill** | "pill-shaped button with fully rounded ends" | Friendly, approachable, tech/SaaS |
| **Sharp rectangle** | "sharp rectangular button with 90° corners" | Bold, urgent, retail, clearance |
| **Rounded rectangle** | "button with slightly rounded corners" | Neutral, safe default |
| **Brush stroke** | "hand-painted brush stroke shape in [color] behind the CTA text, organic and slightly irregular edges" | Creative, artisanal, authentic, NGO |
| **Underline** | "CTA text with a bold underline in [color], no button shape" | Minimal, editorial, luxury |
| **Ghost button** | "CTA text inside a thin [color] border with no fill, transparent background" | Elegant, premium, fashion |

> **Default**: Rounded rectangle.

#### T3 — Edge transition

Only applies to layouts with a distinct text area (A, K, M, and any layout using solid block or gradient background).

| Option | Prompt language | Feel |
|--------|----------------|------|
| **Hard cut** | "clean straight horizontal line separating image from text zone" | Corporate, precise, structured |
| **Gradient** | "soft gradient transition, no visible edge" | Smooth, cinematic |
| **Wave** | "gentle organic wave curve separating the image area from the text area" | Fluid, friendly, wellness/lifestyle |
| **Torn paper** | "rough torn paper edge between image and text area, handmade feel" | Craft, authentic, community, NGO |
| **Diagonal** | "angled diagonal line separating image from text area, dynamic" | Energetic, sporty, startup |

> **Default**: Gradient.

#### T4 — Frame

| Option | Prompt language | When to use |
|--------|----------------|-------------|
| **None** | *(don't mention frames)* | Default. |
| **Thin inset** | "thin white line border inset 20px from all edges" | Editorial, photography |
| **Rounded inset** | "thin rounded rectangle border inset from edges, soft corners" | Premium, cosmetics |
| **Corner marks** | "small L-shaped corner marks at each corner, editorial crop marks style" | Magazine, fashion editorial |
| **Accent line** | "single vertical line accent on the left side, thin, [color]" | Minimal, modern, tech |

---

### Treatment in the prompt

```
TREATMENT:
- Text background: [T1 option]
- CTA shape: [T2 option]
- Edge: [T3 option if applicable]
- Frame: [T4 option if applicable]
```

---

## Step 3 — Visual Concept

Based on the chosen layout, plan the full ad:

- **Audience & emotion**: what moment resonates with them?
- **Scene + typography together**: the prompt must describe both the hero image AND the text elements in a single description
- **Text zones**: describe exactly where in the scene the text will sit (calm area, solid color, gradient)
- **Typography**: specify font weight, color, size hierarchy, CTA treatment
- **Lighting & mood**: bright / warm / dramatic / clean?

**Hero image is mandatory.** Every ad must have a real visual scene, lifestyle photo, or conceptual illustration. If no visual concept is obvious from the brief, invent one that fits the brand and emotion.

---

## Step 4 — Generate (3 variations)

Write a **single base prompt** that describes the complete finished ad: the hero image, the layout zones, and all text elements. Then call the model 3 times in parallel, one per aspect ratio.

**Prompt template:**
```
[aspect_ratio] [photorealistic / editorial / flat design / illustrative] advertising image.
[LAYOUT NAME] layout.

HERO: [subject, setting, action, emotion — what fills the visual area]
LIGHTING: [type and quality of light]
COMPOSITION: [framing, subject position, depth]

TEXT ZONE ([position per layout]): [describe the calm zone — solid color, gradient, etc.]

HEADLINE: "[TITLE]" — very large bold sans-serif, [hex color], [position]. No shadow.
SUBHEADLINE: "[SUBTITLE]" — medium weight, [hex], below headline. Noticeably smaller.
CTA BUTTON: [pill / rectangle] with [hex] background, "[CTA TEXT]" in [hex] bold, [position].

COLOR PALETTE: [hex values]
MOOD: [adjectives]
STYLE: [clean / cinematic / bold / etc. — any brand style restrictions]

TREATMENT:
- Text background: [T1]
- CTA shape: [T2]
- Edge: [T3 if applicable]
- Frame: [T4 if applicable]

No logo. High quality. Print-ready.
```

Adapt the prompt for each ratio:
- **1:1**: base prompt
- **4:5**: same prompt, note "4:5 vertical format — increase separation between text elements by 10–15%"
- **9:16**: same prompt, note "9:16 Stories format — keep headline below top 8%, CTA above bottom 6%, right elements inside left 90%"

Call the model 3 times (in parallel if possible), passing `aspect_ratio` accordingly.

---

## Step 5 — Evaluate

For each of the 3 outputs:

- [ ] Hero image is compelling and relevant (not a text-only image)
- [ ] Text is legible — high contrast, no shadow unless intentional
- [ ] Text ≤ 15% of image area
- [ ] Visual hierarchy correct (image → headline → subheadline → CTA)
- [ ] CTA visually distinct
- [ ] Colors match brief
- [ ] No clutter
- [ ] Layout adapts correctly to the ratio (9:16 safe zones respected)

---

## Step 6 — Save

```bash
OUTDIR="output/$(date +%Y%m%d%H%M)"
mkdir -p "$OUTDIR"
curl -L "[url_1:1]"  -o "$OUTDIR/ad_1x1.jpg"
curl -L "[url_4:5]"  -o "$OUTDIR/ad_4x5.jpg"
curl -L "[url_9:16]" -o "$OUTDIR/ad_9x16.jpg"
```

Report:
```
✅ 3 variaciones guardadas en output/YYYYMMDDHHMM/
   · ad_1x1.jpg   — feed cuadrado
   · ad_4x5.jpg   — feed vertical
   · ad_9x16.jpg  — Stories / Reels
```
