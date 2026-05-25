---
name: powerpoint
description: "Create, read, edit .pptx decks, slides, notes, templates."
license: Proprietary. LICENSE.txt has complete terms
---

# Powerpoint Skill

## When to use

Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions "deck," "slides," "presentation," or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill.

## Quick Reference

| Task | Guide |
|------|-------|
| Read/analyze content | `python -m markitdown presentation.pptx` |
| Edit or create from template | Read [editing.md](editing.md) |
| Create from scratch | Read [pptxgenjs.md](pptxgenjs.md) |
| High-quality editable PPT with web-level design | Use the workflow below: combine `popular-web-designs` design tokens with `pptxgenjs.md` editable PowerPoint primitives |
| Generated illustrations in editable decks | See `references/generated-illustrations-for-editable-ppt.md`: use high-quality generated visuals for hero/concept slides when native shapes look crude, while keeping text/data editable |
| QA when rendering/extraction tools are missing | See `references/editable-ppt-verification-fallbacks.md` for zip/XML and `python-pptx` checks |

---

## High-Quality Editable PPT Workflow

Use this workflow for **every PPT request by default**, even if the user does not explicitly say “high-quality” or “editable.” The baseline deliverable is a polished, visually distinctive deck that remains fully editable in PowerPoint. The goal is **not** to export static website screenshots into slides; the goal is to translate a high-quality web design system into editable PPT objects: text boxes, shapes, charts, tables, icons, and images.

### 1. Pick a design system before writing slide code

If the user asks for a premium / high-quality / beautiful / branded PPT and does not provide a template, pair this skill with `popular-web-designs`:

1. Choose a design system that matches the topic and audience.
   - AI / developer / infra: `linear.app`, `vercel`, `stripe`, `supabase`, `cursor`, `raycast`
   - Product / startup / pitch: `stripe`, `framer`, `apple`, `airbnb`, `webflow`
   - Enterprise / data-heavy: `ibm`, `hashicorp`, `sentry`, `clickhouse`, `coinbase`
   - Premium dark visual: `linear.app`, `superhuman`, `revolut`, `bmw`, `spacex`
2. Load the selected design template:
   ```text
   skill_view(name="popular-web-designs", file_path="templates/<site>.md")
   ```
3. Extract the reusable visual language:
   - Color tokens: background, surface, text, muted text, accent, border
   - Typography: heading/body/mono font families, weights, letter spacing
   - Layout rhythm: margins, grid, card radius, shadows, spacing scale
   - Component patterns: cards, pills, code blocks, dashboards, hero sections, comparison blocks

### 2. Translate web tokens into editable PowerPoint primitives

Map web design concepts to PptxGenJS objects:

| Web design token/component | Editable PPT implementation |
|---|---|
| `background`, gradient, hero surface | Slide background color or generated background image only when necessary |
| Cards / panels | `addShape(RECTANGLE)` or `ROUNDED_RECTANGLE` with fill, line, shadow |
| Headings / body text | `addText` with `fontFace`, `fontSize`, `bold`, `color`, `charSpacing` |
| Buttons / pills / tags | Rounded rectangles + editable text |
| Metric tiles | Large editable numbers + small labels + optional icon |
| Icons | `react-icons` → SVG/PNG inserted as separate movable image objects |
| Charts | Native `addChart` whenever data should remain chart-editable |
| Tables | Native `addTable` whenever tabular data should remain editable |
| Code blocks | Monospace text in editable text boxes on dark/light panels |
| Website screenshots | Use sparingly; only for literal product screenshots, not for general layout |
| Generated hero illustrations | Acceptable as non-editable image assets when they materially improve visual quality; keep surrounding titles, labels, pills, diagrams, charts, and explanatory content editable |

### 3. Preserve editability as a hard requirement

Default to editable elements. Avoid flattening whole slides into a single image unless the user explicitly wants a static poster-like slide.

Editable-first rules:

- Use `addText` for all titles, labels, bullets, callouts, captions, and speaker-facing content.
- Use native shapes for backgrounds, dividers, cards, badges, arrows, timelines, and diagrams.
- Use native charts/tables for data that users may later change.
- Use images only for photos, logos, icons, screenshots, illustrations, textures, or complex backgrounds.
- Use high-quality generated illustrations for hero pages or conceptual visuals when native shapes would look crude; insert them as image assets while keeping the slide’s text and information architecture editable.
- Never create the whole slide as HTML/CSS screenshot if the requested deliverable is an editable `.pptx`.
- If a gradient is central to the design, generate only the gradient background as an image, then layer editable text/shapes/charts above it.

### 4. Build a mini design system in code

Before generating slides, define constants and helper functions so every slide feels like one coherent deck:

```javascript
const pptxgen = require("pptxgenjs");
const pptx = new pptxgen();
pptx.layout = "LAYOUT_16x9";
pptx.author = "Hermes Agent";
pptx.subject = "Editable high-quality PowerPoint";
pptx.title = "Presentation";
pptx.company = "";
pptx.lang = "zh-CN";
pptx.theme = {
  headFontFace: "Aptos Display",
  bodyFontFace: "Aptos",
  lang: "zh-CN"
};

const C = {
  bg: "0B1020",
  surface: "111827",
  surface2: "1F2937",
  text: "F8FAFC",
  muted: "94A3B8",
  accent: "7C3AED",
  accent2: "22D3EE",
  border: "334155",
  white: "FFFFFF"
};
const L = { W: 10, H: 5.625, m: 0.48, gap: 0.22 };
const shadow = () => ({ type: "outer", color: "000000", opacity: 0.16, blur: 8, offset: 2, angle: 45 });

function addTitle(slide, title, subtitle) {
  slide.addText(title, { x: L.m, y: 0.34, w: 7.1, h: 0.42, margin: 0, fontFace: "Aptos Display", fontSize: 24, bold: true, color: C.text, breakLine: false, fit: "shrink" });
  if (subtitle) slide.addText(subtitle, { x: L.m, y: 0.82, w: 7.8, h: 0.3, margin: 0, fontSize: 9.5, color: C.muted, fit: "shrink" });
}

function card(slide, x, y, w, h, opts = {}) {
  slide.addShape(pptx.ShapeType.roundRect, {
    x, y, w, h,
    rectRadius: 0.08,
    fill: { color: opts.fill || C.surface },
    line: { color: opts.line || C.border, transparency: 35, width: 0.7 },
    shadow: opts.shadow === false ? undefined : shadow()
  });
}
```

Adjust the constants from the chosen `popular-web-designs` template instead of defaulting to generic blue/white slides.

### 5. Recommended slide patterns

Use varied, design-system-inspired layouts rather than repeating title + bullets:

- **Hero / title:** large editorial heading, subtitle, pill metadata, abstract shape or product screenshot.
- **Executive summary:** 3–4 metric cards with short insight labels.
- **Problem / opportunity:** asymmetric two-column layout with visual callout panel.
- **Framework / process:** numbered horizontal timeline or vertical step cards.
- **Comparison:** two or three editable columns with strong color coding.
- **Data slide:** one native chart plus two insight callouts; avoid chart-only slides.
- **Architecture / workflow:** editable shapes and connectors; avoid baked-in diagram screenshots.
- **Roadmap:** swimlane or timeline with editable milestone cards.
- **Closing:** bold statement, next steps, contact / appendix cue.

### 6. QA for both visual quality and editability

In addition to the required QA below:

1. Render the `.pptx` to slide images and visually inspect spacing, contrast, alignment, overflow, and layout polish.
2. Run text extraction with `python -m markitdown output.pptx` to confirm content is present as real text, not flattened images.
3. If `markitdown`, LibreOffice, or Poppler are unavailable, verify editability by inspecting the `.pptx` zip directly and/or with `python-pptx`: count slides, extract text from `ppt/slides/slide*.xml`, check for expected keywords, count editable shapes (`<p:sp>`) versus pictures (`<p:pic>`), and check text box bounds.
4. If possible, inspect the PPTX media folder after unpacking. A high-quality editable deck should not contain one giant full-slide image per slide unless explicitly intended.
5. Verify that charts/tables/diagrams expected to be editable are implemented with native PPT objects.

---

## Reading Content

```bash
# Text extraction
python -m markitdown presentation.pptx

# Visual overview
python scripts/thumbnail.py presentation.pptx

# Raw XML
python scripts/office/unpack.py presentation.pptx unpacked/
```

---

## Editing Workflow

**Read [editing.md](editing.md) for full details.**

1. Analyze template with `thumbnail.py`
2. Unpack → manipulate slides → edit content → clean → pack

---

## Creating from Scratch

**Read [pptxgenjs.md](pptxgenjs.md) for full details.**

Use when no template or reference presentation is available.

---

## Design Ideas

**Don't create boring slides.** Plain bullets on a white background won't impress anyone. Consider ideas from this list for each slide.

### Before Starting

- **Pick a bold, content-informed color palette**: The palette should feel designed for THIS topic. If swapping your colors into a completely different presentation would still "work," you haven't made specific enough choices.
- **Dominance over equality**: One color should dominate (60-70% visual weight), with 1-2 supporting tones and one sharp accent. Never give all colors equal weight.
- **Dark/light contrast**: Dark backgrounds for title + conclusion slides, light for content ("sandwich" structure). Or commit to dark throughout for a premium feel.
- **Commit to a visual motif**: Pick ONE distinctive element and repeat it — rounded image frames, icons in colored circles, thick single-side borders. Carry it across every slide.

### Color Palettes

Choose colors that match your topic — don't default to generic blue. Use these palettes as inspiration:

| Theme | Primary | Secondary | Accent |
|-------|---------|-----------|--------|
| **Midnight Executive** | `1E2761` (navy) | `CADCFC` (ice blue) | `FFFFFF` (white) |
| **Forest & Moss** | `2C5F2D` (forest) | `97BC62` (moss) | `F5F5F5` (cream) |
| **Coral Energy** | `F96167` (coral) | `F9E795` (gold) | `2F3C7E` (navy) |
| **Warm Terracotta** | `B85042` (terracotta) | `E7E8D1` (sand) | `A7BEAE` (sage) |
| **Ocean Gradient** | `065A82` (deep blue) | `1C7293` (teal) | `21295C` (midnight) |
| **Charcoal Minimal** | `36454F` (charcoal) | `F2F2F2` (off-white) | `212121` (black) |
| **Teal Trust** | `028090` (teal) | `00A896` (seafoam) | `02C39A` (mint) |
| **Berry & Cream** | `6D2E46` (berry) | `A26769` (dusty rose) | `ECE2D0` (cream) |
| **Sage Calm** | `84B59F` (sage) | `69A297` (eucalyptus) | `50808E` (slate) |
| **Cherry Bold** | `990011` (cherry) | `FCF6F5` (off-white) | `2F3C7E` (navy) |

### For Each Slide

**Every slide needs a visual element** — image, chart, icon, or shape. Text-only slides are forgettable.

**Layout options:**
- Two-column (text left, illustration on right)
- Icon + text rows (icon in colored circle, bold header, description below)
- 2x2 or 2x3 grid (image on one side, grid of content blocks on other)
- Half-bleed image (full left or right side) with content overlay

**Data display:**
- Large stat callouts (big numbers 60-72pt with small labels below)
- Comparison columns (before/after, pros/cons, side-by-side options)
- Timeline or process flow (numbered steps, arrows)

**Visual polish:**
- Icons in small colored circles next to section headers
- Italic accent text for key stats or taglines

### Typography

**Choose an interesting font pairing** — don't default to Arial. Pick a header font with personality and pair it with a clean body font.

| Header Font | Body Font |
|-------------|-----------|
| Georgia | Calibri |
| Arial Black | Arial |
| Calibri | Calibri Light |
| Cambria | Calibri |
| Trebuchet MS | Calibri |
| Impact | Arial |
| Palatino | Garamond |
| Consolas | Calibri |

| Element | Size |
|---------|------|
| Slide title | 36-44pt bold |
| Section header | 20-24pt bold |
| Body text | 14-16pt |
| Captions | 10-12pt muted |

### Spacing

- 0.5" minimum margins
- 0.3-0.5" between content blocks
- Leave breathing room—don't fill every inch

### Avoid (Common Mistakes)

- **Don't repeat the same layout** — vary columns, cards, and callouts across slides
- **Don't center body text** — left-align paragraphs and lists; center only titles
- **Don't skimp on size contrast** — titles need 36pt+ to stand out from 14-16pt body
- **Don't default to blue** — pick colors that reflect the specific topic
- **Don't mix spacing randomly** — choose 0.3" or 0.5" gaps and use consistently
- **Don't style one slide and leave the rest plain** — commit fully or keep it simple throughout
- **Don't create text-only slides** — add images, icons, charts, or visual elements; avoid plain title + bullets
- **Don't forget text box padding** — when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
- **Don't use low-contrast elements** — icons AND text need strong contrast against the background; avoid light text on light backgrounds or dark text on dark backgrounds
- **NEVER use accent lines under titles** — these are a hallmark of AI-generated slides; use whitespace or background color instead

---

## QA (Required)

**Assume there are problems. Your job is to find them.**

Your first render is almost never correct. Approach QA as a bug hunt, not a confirmation step. If you found zero issues on first inspection, you weren't looking hard enough.

### Content QA

```bash
python -m markitdown output.pptx
```

Check for missing content, typos, wrong order.

**When using templates, check for leftover placeholder text:**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

### Visual QA

**⚠️ USE SUBAGENTS** — even for 2-3 slides. You've been staring at the code and will see what you expect, not what's there. Subagents have fresh eyes.

Convert slides to images (see [Converting to Images](#converting-to-images)), then use this prompt:

```
Visually inspect these slides. Assume there are issues — find them.

Look for:
- Overlapping elements (text through shapes, lines through words, stacked elements)
- Text overflow or cut off at edges/box boundaries
- Decorative lines positioned for single-line text but title wrapped to two lines
- Source citations or footers colliding with content above
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
- Uneven gaps (large empty area in one place, cramped in another)
- Insufficient margin from slide edges (< 0.5")
- Columns or similar elements not aligned consistently
- Low-contrast text (e.g., light gray text on cream-colored background)
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
- Text boxes too narrow causing excessive wrapping
- Leftover placeholder content

For each slide, list issues or areas of concern, even if minor.

Read and analyze these images:
1. /path/to/slide-01.jpg (Expected: [brief description])
2. /path/to/slide-02.jpg (Expected: [brief description])

Report ALL issues found, including minor ones.
```

### Verification Loop

1. Generate slides → Convert to images → Inspect
2. **List issues found** (if none found, look again more critically)
3. Fix issues
4. **Re-verify affected slides** — one fix often creates another problem
5. Repeat until a full pass reveals no new issues

**Do not declare success until you've completed at least one fix-and-verify cycle.**

---

## Converting to Images

Convert presentations to individual slide images for visual inspection:

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

This creates `slide-01.jpg`, `slide-02.jpg`, etc.

To re-render specific slides after fixes:

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

---

## Dependencies

- `pip install "markitdown[pptx]"` - text extraction
- `pip install Pillow` - thumbnail grids
- `npm install -g pptxgenjs` - creating from scratch
- LibreOffice (`soffice`) - PDF conversion (auto-configured for sandboxed environments via `scripts/office/soffice.py`)
- Poppler (`pdftoppm`) - PDF to images
