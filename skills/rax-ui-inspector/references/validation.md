# UI Validation Patterns

## What Can Be Validated

| Dimension | Method | Example |
|-----------|--------|---------|
| Content present | `ui describe` → grep for text | Product name, price, button text |
| Price format | Match `Rp[\d.,]+` pattern | `Rp989.505` |
| Discount consistency | Compare current/original price with discount label | `50% off` vs `Rp989 / Rp1999` |
| Component completeness | Check Lynx component children | Card has Cover + Title + Price + BuyBtn |
| Element clickable | Check `clickable` flag in describe output | Buy button must be clickable |
| Element visible | Element appears in `ui describe` output | Only visible elements are returned |
| Image loaded | `image` node has non-empty `src` | Product cover has CDN URL |
| Navigation | Describe after tap → check new content | Tab switch shows expected page |
| List content | `rax lynx document` → inspect `list.children` array | All products present in stable order |

## What Cannot Be Validated (Use Screenshots)

- Visual styling (colors, fonts, border radius)
- Image content (does the photo match the product?)
- Animation correctness
- Pixel-perfect layout

## Validation via `ui describe`

### Check required elements exist

```bash
# After navigating to a product detail page:
rax ui describe --no-screenshot | grep -E "购买|立即购买|Buy"
# Expected: at least one result with clickable flag

rax ui describe --no-screenshot | grep "TUXPrice"
# Expected: price component present
```

### Check element ordering

Elements in describe output are sorted top-to-bottom. For a product card:

```
ProductFeedSingleCard (720,1845)    ← container
  Image (300,1804)                  ← cover image (top)
  Text "Product Name" (975,1596)    ← title (below image)
  Text "免费配送" (720,1673)         ← shipping label
  TUXPrice (753,1932)               ← price (below labels)
  Text "购买" (1256,1992)           ← buy button (bottom right)
```

Validate: title.y < price.y < button.y (top to bottom order)

## Validation via Lynx DOM

For deeper structural checks, use `rax lynx document`:

### Check product card structure

```bash
rax lynx document --session <id>
```

A valid ProductFeedSingleCard should contain:
- `index-Cover` → `image` with `src` attribute
- `index-Title` → `text` → `raw-text` with product name
- `index-ShopPrice` → `index-TUXPrice` → nested `text`/`raw-text` with price
- `index-BuyNowBtn` → `text` → `raw-text` with "购买"
- `index-PromotionView` → tags like "免费配送", "50% 折扣"

### Check all products in a list

```bash
# Get DOM, find list node, check children array
rax lynx cdp DOM.enable --params '{"useCompression": false}' --session <id>
rax lynx cdp DOM.getDocumentWithBoxModel --session <id>
```

List children maintain stable array order. Each `index-ProductFeedDoubleCard` or `index-ProductFeedSingleCard` child represents one product. Check:
- Total count matches expected
- Each has required sub-components
- Visible items (`content[1] >= list.content[1]`) have reasonable coordinates
