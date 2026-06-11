# Style Debugging Guide

## Three Levels of Style Information

| Level | Command | Shows | Best For |
|-------|---------|-------|----------|
| Computed | `rax lynx computed-style <id>` | Final rendered values | "What does this look like?" |
| Matched | `rax lynx matched-styles <id>` | CSS rules that matched | "Why does it look like this?" |
| Inline | `rax lynx inline-styles <id>` | Only `style=""` attribute | "Is there an inline override?" |

## Debugging Flow

### "Why doesn't this element look right?"

```bash
# 1. Find the element
rax lynx query ".target-element"
# → Returns node_id

# 2. Check what the computed value actually is
rax lynx computed-style <node_id>
# → Look for the unexpected property

# 3. Check which CSS rules are setting it
rax lynx matched-styles <node_id>
# → Find the rule with highest specificity

# 4. Check if there's an inline override
rax lynx inline-styles <node_id>
# → Inline styles win over class-based rules
```

## Common Style Issues

### Wrong Color

```bash
rax lynx computed-style <node_id>
# Look for: color, background-color
# Compare against design spec values
```

### Wrong Size

```bash
rax lynx box-model <node_id>
# Check: width, height
# Check: padding (adds to content size)
# Check: margin (affects spacing)

rax lynx computed-style <node_id>
# Look for: width, height, min-width, max-width, flex properties
```

### Element Not Visible

```bash
rax lynx computed-style <node_id>
# Check: display (none?)
# Check: visibility (hidden?)
# Check: opacity (0?)
# Check: overflow (hidden with 0 dimensions?)

rax lynx box-model <node_id>
# Check: width/height (0?)
# Check: position (off-screen coordinates?)
```

### Unexpected Spacing

```bash
rax lynx box-model <node_id>
# Check: margin (outer spacing)
# Check: padding (inner spacing)
# Compare with sibling elements for consistency
```

### Text Styling

```bash
rax lynx computed-style <node_id>
# Key properties:
# - font-size
# - font-weight
# - color
# - line-height
# - text-align
# - text-overflow (ellipsis?)
# - max-lines / -webkit-line-clamp
```

## Design Token Verification

Check if design tokens are used correctly:

```bash
# Get class names
rax lynx attributes <node_id>
# → class="text-color-TextPrimary background-color-UIPageFlat1"

# Verify computed values match token definitions
rax lynx computed-style <node_id>
# → color: rgba(22, 24, 35, 1)  ← Is this TextPrimary?
```

Hardcoded colors in inline styles instead of design tokens indicate a potential issue.
