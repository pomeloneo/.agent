# Validate Product Card Content

Check that a product card renders correctly with all required elements.

## Using `ui describe`

```bash
rax ui describe --no-screenshot
```

A correctly rendered product card should output elements like:

```
- ProductFeedSingleCard (720,1845)
- View (720,1845) clickable                    ← entire card is tappable
- Image (300,1804)                             ← product cover image
- Text "Adidas Samba OG White..." (975,1596)   ← product title
- View (694,1672) clickable                    ← promotion tag (tappable)
- Image (606,1672)                             ← free shipping icon
- Text "免费配送" (720,1673)                    ← free shipping label
- Text "4% 现金返还奖金" (1000,1673)            ← cashback label
- Text "50% 折扣" (675,1733)                   ← discount tag
- Text "5.0" (855,1751)                        ← rating
- Text "全网销量6037件" (1072,1751)              ← sales count
- TUXPrice (753,1932)                          ← price component
- Text "Rp1.088.505" (753,1932)                ← current price
- Text "购买" (1256,1992)                       ← buy button
- Text "Rp2.199.000" (699,2009)                ← original price
```

## Checklist

### Required elements

| Element | How to check |
|---------|-------------|
| Product image | `Image` node inside card |
| Product title | `Text` with product name string |
| Current price | `Text` matching `Rp[\d.,]+` inside `TUXPrice` |
| Buy button | `Text "购买"` |
| Card clickable | Parent `View` has `clickable` flag |

### Optional but expected

| Element | How to check |
|---------|-------------|
| Original price | Second `Rp` text (strikethrough in UI) |
| Discount label | `Text` matching `\d+%\s*折扣` |
| Free shipping | `Text "免费配送"` |
| Rating | `Text` with numeric value like `"5.0"` or `"4.9"` |
| Sales count | `Text` matching `全网销量\d+件` or `已售出.*件` |

### Price consistency

If both current price and original price exist, and a discount label exists:

```
current = 1088505  (from "Rp1.088.505")
original = 2199000 (from "Rp2.199.000")
discount = 50      (from "50% 折扣")

expected_discount = round((1 - current/original) * 100) = 50
actual_discount = 50
→ PASS (difference within 2%)
```

## Using Lynx DOM for Deep Inspection

```bash
rax lynx cdp DOM.enable --params '{"useCompression": false}' --session <id>
rax lynx cdp DOM.getDocumentWithBoxModel --session <id>
```

Find `index-ProductFeedSingleCard` in the tree and verify its children:

```
index-ProductFeedSingleCard
  view (bindtap=handleProductClick)         ← must have bindtap
    view
      index-Cover
        view
          image (src=https://...)            ← must have src
    view
      index-Title
        view
          text
            raw-text "Product Name"          ← must have text
      index-ShopPrice
        index-TUXPrice
          text → raw-text "Rp" + "1.088.505" ← price parts
      view
        index-ProductAddCartButton           ← cart button
        index-BuyNowBtn                      ← buy button
          text → raw-text "购买"
```
