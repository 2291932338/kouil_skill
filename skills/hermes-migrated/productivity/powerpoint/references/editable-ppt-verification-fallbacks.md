# Editable PPT Verification Fallbacks

Use these checks when the normal visual/content QA stack is incomplete (for example, LibreOffice, Poppler, or `markitdown` is unavailable). They do not replace real slide rendering, but they catch common failure modes before delivery.

## Direct PPTX zip inspection

```bash
PPTX="/path/to/output.pptx"
ls -lh "$PPTX"
unzip -l "$PPTX" | awk '/ppt\/slides\/slide[0-9]+\.xml/{c++} /ppt\/media\//{m++} END{print "slides=" c; print "media=" (m?m:0)}'
python - <<'PY'
import zipfile, re
p = '/path/to/output.pptx'
with zipfile.ZipFile(p) as z:
    slides = sorted(
        [n for n in z.namelist() if re.match(r'ppt/slides/slide\d+\.xml$', n)],
        key=lambda x: int(re.search(r'(\d+)', x).group(1))
    )
    print('slide_count', len(slides))
    text = ''
    editable_shapes = 0
    pictures = 0
    for n in slides:
        data = z.read(n).decode('utf-8', errors='ignore')
        text += re.sub(r'<[^>]+>', ' ', data) + '\n'
        editable_shapes += data.count('<p:sp>')
        pictures += data.count('<p:pic>')
    print('editable_shapes', editable_shapes, 'pictures', pictures)
    for kw in ['具身智能', '感知', '行动']:
        print(kw, kw in text)
PY
```

Interpretation:
- `slide_count` should match the requested deck length.
- `editable_shapes` should be non-trivial for editable decks; a deck with mostly full-slide screenshots will have many pictures and few text/shape objects.
- Expected keywords should appear in XML text, proving content is not entirely flattened into images.

## python-pptx text and bounds check

If `python-pptx` is installed:

```bash
python - <<'PY'
from pptx import Presentation
p = '/path/to/output.pptx'
prs = Presentation(p)
print('slides', len(prs.slides))
for i, slide in enumerate(prs.slides, 1):
    texts = []
    for shape in slide.shapes:
        if hasattr(shape, 'text') and shape.text.strip():
            texts.append(shape.text.strip().replace('\n', ' / '))
    print(i, len(texts), ' | '.join(texts[:4]))

errors = []
for si, slide in enumerate(prs.slides, 1):
    for shape in slide.shapes:
        if hasattr(shape, 'text') and shape.text.strip():
            x = shape.left / 914400
            y = shape.top / 914400
            w = shape.width / 914400
            h = shape.height / 914400
            if x < -0.01 or y < -0.01 or x + w > 10.05 or y + h > 5.68:
                errors.append((si, shape.text[:20], x, y, w, h))
print('bounds_errors', len(errors))
print(errors[:5])
PY
```

## Rough contact sheet when rendering is unavailable

A PIL-generated contact sheet from extracted text is not a faithful visual render, especially for Chinese fonts, but it can verify the overall number of slides and that a deck is not empty. Treat it as a fallback only; do not use it as proof of typography quality.
