---
name: rax-ui-inspector
description: Inspect and verify Lynx page UI for development. Use when you need to check layout correctness (spacing, alignment, overlap), style accuracy (colors, font sizes, design tokens), content completeness, or interaction behavior on connected devices.
---

# RAX UI Inspector

Verify Lynx page UI implementation against design specs. Check layout, styling, content, and interaction — not just "does the element exist", but "is it correct".

## Usage

CLI: `rax` (at `<rax-monorepo>/apps/rax-cli/bin/rax.sh`).

```bash
rax device scan          # Connect device
rax ui describe          # See what's on screen
```

## What You Can Verify

### 1. Layout (Spacing, Alignment, Overlap)

Use `box_model` from DOM to check element dimensions and positions.

```bash
# Get DOM with coordinates
rax lynx cdp DOM.enable --params '{"useCompression": false}' --session <id>
rax lynx cdp DOM.getDocumentWithBoxModel --session <id>
```

Each node has `box_model.{width, height, content, padding, border, margin}`.

**Check spacing between siblings:**
```
Element A: content y_bottom = 445
Element B: content y_top = 465
Gap = 465 - 445 = 20dp → matches 20dp design spec? ✓
```

**Check alignment:**
```
Left column card:  content x = 16dp
Right column card: content x = 200dp
Column gap = 200 - 16 - card_width → consistent? ✓
```

**Check overlap:**
```
Element A bounds: [0, 100, 384, 200]
Element B bounds: [0, 150, 384, 250]
Overlap = 200 - 150 = 50dp → unintended overlap? ✗
```

See [Layout Verification](references/layout.md) for detailed patterns.

### 2. Styling (Colors, Font Sizes, Design Tokens)

Use `computed-style` and `matched-styles` to check actual rendered values.

```bash
rax lynx computed-style <nodeId> --session <id>
rax lynx matched-styles <nodeId> --session <id>
rax lynx inline-styles <nodeId> --session <id>
```

**Check font size:**
```bash
rax lynx computed-style <nodeId> --session <id>
# → { "name": "font-size", "value": "14px" }
# Design spec says 14px? ✓
```

**Check color:**
```bash
rax lynx computed-style <nodeId> --session <id>
# → { "name": "color", "value": "rgba(22, 24, 35, 1)" }
# Is this the correct TextPrimary token? ✓
```

**Check design token usage (via class names):**
```
attributes: ["class", "text-color-TextPrimary background-color-UIPageFlat1"]
# Uses design system tokens? ✓
# Hardcoded color in style instead of token? ✗
```

See [Style Verification](references/style.md) for common property checks.

### 3. Content (Text, Images, Truncation)

Use `ui describe` for visible text, or DOM for full content.

```bash
rax ui describe --no-screenshot    # Visible text + coordinates
rax lynx inner-text <nodeId>       # Get node's full text content
rax lynx attributes <nodeId>       # Check image src, alt text
```

**Check text truncation:**
```
Text node: width=216dp, content "Adidas Samba OG White Night Indigo Gum"
Container width: 216dp
→ Text fits? Or is overflow:hidden cutting it off?
Check computed-style for text-overflow, overflow, max-lines
```

**Check image:**
```
image node attributes: src="https://..."
→ src is valid CDN URL (not placeholder)? ✓
→ box_model width:height ratio matches design? ✓
```

### 4. Component Structure

Use DOM tree to verify component hierarchy.

```bash
rax lynx document --session <id>
```

**Check required children exist:**
```
ProductFeedSingleCard
  ├── index-Cover → image (has src?)
  ├── index-Title → text → raw-text (has content?)
  ├── index-ShopPrice → TUXPrice (has price?)
  ├── index-BuyNowBtn → text "购买"
  └── index-PromotionView → tags
```

**Check component naming follows convention:**
```
localName: "src/components/product-feed/index-ProductFeedSingleCard"
→ Follows naming convention? ✓
```

### 5. Interaction & Accessibility

```bash
rax ui describe --no-screenshot    # Check clickable/scrollable flags
rax lynx attributes <nodeId>       # Check bindtap, aria-label
```

**Check tap target size:**
```
Button box_model: width=66dp, height=28dp
Minimum tap target: 44×44dp (accessibility guideline)
→ Too small? ⚠️
```

**Check event binding:**
```
attributes: bindtap="handleBuy"
→ Has tap handler? ✓
```

### 6. Visibility & Rendering

```bash
rax ui describe    # Only returns visible elements + screenshot
```

**What `ui describe` filters out:**
- Recycled list items (scrolled past)
- Hidden dialogs (`x-overlay` with `--close` class)
- Off-screen elements
- Inactive Lynx sessions

If an expected element is missing from describe output, it's not visible to the user.

## Command Reference

| Command | What it gives you |
|---------|-------------------|
| `rax ui describe` | Visible elements with text, coords, clickable state |
| `rax lynx document` | Full DOM tree (all nodes, including off-screen) |
| `rax lynx cdp DOM.getDocumentWithBoxModel` | DOM + layout coordinates per node |
| `rax lynx computed-style <id>` | Actual rendered CSS property values |
| `rax lynx matched-styles <id>` | Which CSS rules/selectors matched |
| `rax lynx inline-styles <id>` | Inline style attribute |
| `rax lynx attributes <id>` | All node attributes (class, style, bindtap, src...) |
| `rax lynx inner-text <id>` | Visible text content of a node |
| `rax lynx box-model <id>` | Box model for a specific node |
| `rax lynx screenshot` | Lynx-only screenshot |
| `rax lynx query-all <sel>` | CSS selector query |

## References

- [Layout Verification](references/layout.md) — Spacing, alignment, overlap checks
- [Style Verification](references/style.md) — Colors, fonts, design token checks
- [Lynx DOM Reference](references/lynx-dom.md) — DOM structure, coordinates, visibility rules
- [Validation Patterns](references/validation.md) — Content and structural checks

## Examples

- [Verify Product Card](examples/validate-product-card.md) — Full checklist for product card UI
- [Navigate and Purchase](examples/navigate-and-purchase.md) — Interaction flow testing
