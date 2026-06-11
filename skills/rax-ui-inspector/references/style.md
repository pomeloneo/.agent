# Style Verification

Verify colors, fonts, design tokens, and visual properties.

## Getting Style Data

```bash
# All computed properties (actual rendered values)
rax lynx computed-style <nodeId> --session <id>

# Which CSS rules matched this node
rax lynx matched-styles <nodeId> --session <id>

# Inline style only
rax lynx inline-styles <nodeId> --session <id>

# Node attributes (includes class and style)
rax lynx attributes <nodeId> --session <id>
```

## Common Properties to Check

### Typography

| Property | What to verify | Example |
|----------|---------------|---------|
| `font-size` | Matches design spec | `14px`, `16px`, `20px` |
| `font-weight` | Bold where expected | `400` (normal), `600` (semi-bold), `700` (bold) |
| `color` | Correct text color token | `rgba(22, 24, 35, 1)` = TextPrimary |
| `line-height` | Proper line spacing | `20px` for 14px text |
| `text-overflow` | Truncation behavior | `ellipsis` |
| `max-lines` | Max visible lines | `1` (single line), `2` (two lines) |
| `text-align` | Alignment | `left`, `center`, `right` |

### Colors

| Property | What to verify |
|----------|---------------|
| `color` | Text color |
| `background-color` | Background fill |
| `border-color` | Border color |
| `outline-style` | Outline/shadow color |

### Spacing

| Property | What to verify |
|----------|---------------|
| `padding-left/right/top/bottom` | Internal spacing |
| `margin-left/right/top/bottom` | External spacing |
| `gap` | Flex/grid gap |

### Visual

| Property | What to verify |
|----------|---------------|
| `border-radius` | Corner rounding |
| `opacity` | Transparency |
| `overflow` | Content clipping |
| `visibility` | Hidden or visible |

## Design Token Verification

Lynx TUX components use design token classes. Check `class` attribute in node attributes:

```bash
rax lynx attributes <nodeId> --session <id>
```

### Text color tokens

| Class | Expected color |
|-------|---------------|
| `text-color-TextPrimary` | Main text (dark) |
| `text-color-TextSecondary` | Secondary text (gray) |
| `text-color-TextTertiary` | Tertiary text (light gray) |
| `text-color-TextLink` | Link text (blue) |
| `text-color-TextBrandPrimary` | Brand color (red/pink) |

### Background tokens

| Class | Expected color |
|-------|---------------|
| `background-color-UIPageFlat1` | Page background |
| `background-color-UICardFlat1` | Card background |
| `background-color-UISheetBackdrop1` | Sheet overlay |

### Checking for hardcoded values

**Bad:** Inline style with hardcoded color
```
style="color: #161823;"
```

**Good:** Design token via class
```
class="text-color-TextPrimary"
```

To find hardcoded colors, check `inline-styles` for color/background-color properties that should use tokens.

## Theme Consistency

The root node's class indicates theme:

```
class="theme-light direction-ltr"
```

All child elements should use theme-aware tokens. Check that:
- No hardcoded `#ffffff` (should be token that adapts to dark mode)
- No hardcoded `#000000` (should be TextPrimary token)
- Background colors use `UI*` tokens

## RTL Layout

If `direction-ltr` is in root class, verify:
- Text alignment is `left` (not explicitly set to `right`)
- Flex layouts flow left-to-right
- Icons/badges on correct side

For RTL (`direction-rtl`):
- Mirror all horizontal positions
- Text alignment should be `right`
