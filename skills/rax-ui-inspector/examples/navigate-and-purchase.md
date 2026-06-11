# Navigate and Purchase a Product

Find a specific product and complete the purchase flow.

## Step 1: Connect to Device

```bash
rax device scan
rax device list
# Expected: RAX: ✅
```

## Step 2: Describe Current Screen

```bash
rax ui describe --no-screenshot
```

Look for tab navigation elements (e.g., `"首页"`, `"商品"`) and their coordinates.

## Step 3: Navigate to Home Tab

```bash
# From describe output: Text "首页" (208,1358) clickable
rax ui tap 208 1358
```

Wait for page load:
```bash
sleep 2
```

## Step 4: Find Target Product

```bash
rax ui describe --no-screenshot | grep -i "nike\|adidas\|puma"
```

If the product is not visible, scroll down:

```bash
rax ui swipe 720 2400 720 800 300
sleep 1
rax ui describe --no-screenshot | grep -i "nike"
```

Repeat scroll + describe until the target appears.

## Step 5: Tap the Product

```bash
# From describe output: Text "Nike P-6000..." (975,1596)
# Use the parent clickable View's coordinates
rax ui tap 975 1596
sleep 3
```

## Step 6: Verify Product Detail Page

```bash
rax ui describe --no-screenshot | grep -E "购买|立即购买|Buy|价格|Rp"
```

Expected output includes price and buy button.

## Step 7: Tap Buy Button

```bash
# From describe output: Text "立即购买" (1024,3046)
rax ui tap 1024 3046
sleep 2
```

## Step 8: Verify Purchase Panel

```bash
rax ui describe
```

Check the screenshot — should show size selection, quantity picker, and confirm button.

## Tips

- **Use content to identify products**, not position. Product feeds are dynamic.
- **Always wait** after tap/navigation (`sleep 2-3`) before describing.
- **Check both** Lynx and native elements — product detail pages may be native (path starts with `a1-b1`) while list pages are Lynx (path starts with `L1-b1`).
