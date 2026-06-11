# Layout Verification

Verify spacing, alignment, sizing, and overlap using `box_model` data.

## Getting Layout Data

```bash
rax lynx cdp DOM.enable --params '{"useCompression": false}' --session <id>
rax lynx cdp DOM.getDocumentWithBoxModel --session <id>
```

Each node's `box_model`:

```json
{
  "width": 352,
  "height": 128,
  "content": [16, 428, 368, 428, 368, 556, 16, 556],
  "padding": [16, 428, 368, 428, 368, 556, 16, 556],
  "border":  [16, 428, 368, 428, 368, 556, 16, 556],
  "margin":  [8, 420, 376, 420, 376, 564, 8, 564]
}
```

`content`/`padding`/`border`/`margin` are quad arrays: `[x1,y1, x2,y2, x3,y3, x4,y4]` (top-left, top-right, bottom-right, bottom-left).

## Spacing Between Elements

### Vertical gap between siblings

```
Element A: content bottom = content[5]  (y of bottom-left corner)
Element B: content top = content[1]     (y of top-left corner)
Vertical gap = B.content[1] - A.content[5]
```

### Horizontal gap

```
Element A: content right = content[2]  (x of top-right corner)
Element B: content left = content[0]   (x of top-left corner)
Horizontal gap = B.content[0] - A.content[2]
```

### Padding check

```
Padding left   = content[0] - padding[0]   (should be 0 if no padding)
Padding top    = content[1] - padding[1]
Padding right  = padding[2] - content[2]
Padding bottom = padding[5] - content[5]
```

## Alignment Checks

### Left-aligned siblings

All siblings should have the same `content[0]` (x of top-left).

```
Child 1: content[0] = 16
Child 2: content[0] = 16
Child 3: content[0] = 16
→ All left-aligned ✓
```

### Center-aligned

```
Element center_x = content[0] + width / 2
Parent center_x = parent.content[0] + parent.width / 2
Offset = element_center_x - parent_center_x
→ |offset| < 1dp? Centered ✓
```

### Two-column grid alignment

```
Left column:  content[0] = 16, width = 168
Right column: content[0] = 200, width = 168
Gap = 200 - (16 + 168) = 16dp
→ Column gap consistent? ✓
→ Both columns same width? ✓
```

## Overlap Detection

Two siblings overlap if their content quads intersect:

```
A: [x1=0, y1=100, x2=384, y2=200]
B: [x1=0, y1=150, x2=384, y2=250]

Overlap_x = min(A.x2, B.x2) - max(A.x1, B.x1) = min(384,384) - max(0,0) = 384
Overlap_y = min(A.y_bottom, B.y_top_bottom) - max(A.y_top, B.y_top) = 200 - 150 = 50

If overlap_x > 0 AND overlap_y > 0 → overlapping by 50dp vertically
```

## Size Constraints

### Minimum tap target

Accessibility guideline: interactive elements should be at least 44×44dp.

```
Button: width=66dp, height=28dp
→ height 28dp < 44dp minimum ⚠️
```

### Image aspect ratio

```
Image: width=128dp, height=128dp
→ 1:1 ratio? ✓ (square product image)

Image: width=352dp, height=128dp
→ 2.75:1 ratio? Match design spec?
```

### Text container sizing

```
Text node: width=216dp
Computed font-size: 14px, text: "Very Long Product Name That Might Be Cut Off"
Approximate text width: ~320dp (at 14px)
→ Text overflows container → check text-overflow / max-lines in computed-style
```
