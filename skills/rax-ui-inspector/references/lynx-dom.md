# Lynx DOM Reference

## DOM Structure

Lynx DOM nodes have this structure:

```json
{
  "nodeId": 186,
  "localName": "text",
  "attributes": ["style", "font-size:12px;", "class", "product-title", "bindtap", "handleClick"],
  "box_model": {
    "width": 216, "height": 17,
    "content": [152, 428, 368, 428, 368, 445, 152, 445]
  },
  "children": [...]
}
```

### Key Fields

| Field | Description |
|-------|-------------|
| `localName` | Element tag or component path. Components have paths like `src/components/ec-shop-badge/index-ECShopBadge` |
| `attributes` | Flat key-value array: `[key1, val1, key2, val2, ...]`. Important keys: `text`, `src`, `class`, `style`, `bindtap`, `idSelector` |
| `box_model.content` | 4-corner quad `[x1,y1, x2,y2, x3,y3, x4,y4]` — viewport-relative coordinates in Lynx dp |
| `box_model.width/height` | Element dimensions in Lynx dp |
| `children` | Child nodes. **Array order is stable** — does not change with scroll |

### Common Node Types

| `localName` | Description |
|-------------|-------------|
| `page` | Root page node. `width`/`height` = viewport size |
| `view` | Generic container (like `<div>`) |
| `text` | Text container. Children are `raw-text` nodes with `text` attribute |
| `raw-text` | Leaf text node. The `text` attribute has the actual string |
| `image` | Image element. The `src` attribute has the URL |
| `list` | Recycling list (like RecyclerView) |
| `scroll-view` | Scrollable container |
| `x-foldview-ng` | Collapsible fold layout |
| `x-overlay` | Overlay/dialog container |
| Component paths | e.g. `index-ProductFeedSingleCard`, `index-BuyNowBtn`, `TUXPrice-TUXPrice` |

## Coordinate System

```
Screen pixels (Android)          Lynx dp
┌──────────────┐ 1440px         ┌──────────┐ 384dp
│              │                │          │
│              │ 3200px         │          │ 853dp
│              │                │          │
└──────────────┘                └──────────┘

scale = screen_width / page_width = 1440 / 384 ≈ 3.75
screen_x = lynx_x × scale
screen_y = lynx_y × scale
```

- `rax ui tap/swipe` uses **screen pixels**
- `rax lynx node-at` uses **Lynx dp**
- `rax ui describe` outputs **screen pixels** (ready for tap)

## Visibility Rules

### Recycled List Items

`list` containers recycle off-screen items. In the DOM:

```
list (y=384, h=760)
  [0] HomeHeader    y=0    ← recycled (y < list.y)
  [1] Wrap          y=0    ← recycled
  [2] ProductCard   y=450  ← VISIBLE (y >= list.y)
  [3] ProductCard   y=720  ← VISIBLE
  [4] ProductCard   y=0    ← recycled
```

**Rule:** Item is visible when `item.content[1] >= list.content[1]`

**Array index is stable** — `children[2]` is always the same product regardless of scroll position. Use index to identify "the Nth item", not coordinates.

### Hidden Dialogs/Overlays

```
x-overlay class="... tux-dialog--close ..."  ← HIDDEN (--close in class)
  dialog content...                           ← entire subtree is hidden
```

**Rule:** `x-overlay` with `--close` or `--hidden` in class name → not displayed.

### Inactive Sessions

Multiple Lynx sessions may exist (e.g., search page behind store page).

**Rule:** Inactive sessions have all `box_model.content` values as `[0,0,0,0,0,0,0,0]`.

## Getting DOM with Coordinates

```bash
# Enable uncompressed output first
rax lynx cdp DOM.enable --params '{"useCompression": false}' --session <id>

# Get full DOM with box_model on every node
rax lynx cdp DOM.getDocumentWithBoxModel --session <id>
```

The response includes `box_model` with `width`, `height`, and `content` quad on each node.
