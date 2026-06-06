# Generated Illustrations in Editable PPT Decks

Use this reference when a PPT needs premium visuals but must remain editable.

## Lesson captured

For PPT requests, the user's default expectation is **high-quality + editable**. Editable does not mean every pixel must be a PowerPoint primitive: generated illustrations are acceptable, especially for hero pages, conceptual openers, section breaks, and abstract visuals where crude shape drawings would lower perceived quality.

The correction from the embodied-AI deck: a hand-built robot out of PPT shapes looked too crude. A higher-quality generated / illustration asset made the cover feel more professional while keeping the title, subtitle, pills, and explanatory content editable.

## Rule of thumb

- Keep **information** editable: titles, body copy, labels, numbers, charts, tables, timelines, diagrams, callouts.
- Use generated/static images for **visual atmosphere**: hero robot, conceptual AI agent, background texture, product-like illustration, photo-real scene, cinematic cover element.
- Do not flatten whole slides into images unless explicitly requested.
- Do not use low-effort native-shape character drawings as premium hero visuals.

## Preferred workflow

1. Choose deck visual language first, usually via `popular-web-designs` tokens.
2. Identify slides that benefit from non-editable visual assets:
   - cover hero visual
   - section divider illustration
   - conceptual metaphor image
   - product screenshot / environment photo
3. Generate or source a high-quality image asset.
   - Prefer GPT-image-style generation when available, e.g. user may call it `gptimage2` / `image2`.
   - If the image generator is unavailable, be explicit and either use a high-quality deterministic illustration fallback or continue with editable vector/icon composition that is not pretending to be generated.
4. Insert the image as a separate image object, never as a full-slide flattening layer unless it is only a background.
5. Layer editable text/shapes above or beside it.
6. Verify editability and image use:
   - count slides
   - extract expected text with `python-pptx` or PPTX XML
   - count `<p:sp>` editable shapes and `<p:pic>` pictures
   - ensure picture count is intentional, e.g. one hero image on slide 1, not one screenshot per slide
   - inspect text bounds where possible

## Prompt pattern for hero illustrations

```text
Premium dark-mode 3D illustration for a PowerPoint cover about [topic].
Subject: [robot / agent / conceptual object].
Style: clean product-marketing, high-end AI startup aesthetic, [chosen design system] inspired.
Palette: near-black background, violet/cyan accent glow, subtle particles/holographic rings.
Composition: 16:9, subject on the right, ample negative space on the left for editable title text.
Constraints: no text, no letters, no watermark, no logo, no UI labels.
```

## PPT insertion pattern

```javascript
slide.addImage({
  path: "/path/to/hero.png",
  x: 5.55, y: 0.45, w: 3.8, h: 3.0,
  transparency: 0,
});
// Keep these editable:
slide.addText("Deck title", { x: 0.5, y: 1.0, w: 5.0, h: 0.6, fontSize: 36 });
slide.addText("Subtitle", { x: 0.5, y: 1.8, w: 5.0, h: 0.3, fontSize: 15 });
```

## Pitfalls

- **Crude native shape mascot:** If a hero character/robot looks like a quick diagram, replace it with a generated illustration or a more abstract premium visual.
- **Full-slide screenshot deck:** Generated images should enhance slides, not replace editable information architecture.
- **Text in generated images:** Avoid; generated text is usually unreliable and non-editable. Use PowerPoint text boxes.
- **Unverified generator availability:** Check the actual image generation backend before promising to use it. If unavailable, report the fallback honestly.
- **Too many images:** A deck with one picture per slide may still be editable in text, but can feel less maintainable. Prefer images for high-impact visuals only.
