# DOM Inspection Guide

## Finding Elements

### By CSS Selector

```bash
# Single element
rax lynx query ".product-card"
rax lynx query "#main-content"
rax lynx query "text"

# All matching elements
rax lynx query-all ".product-card"
rax lynx query-all "image"
```

### By Coordinates

When you know where the element is on screen:

```bash
rax lynx node-at --x 200 --y 500
```

Useful after taking a screenshot and identifying a visual element.

### By Text Content

```bash
rax lynx search "Buy Now"
rax lynx search "¥99.00"
```

### By DOM Traversal

```bash
# Get full document tree
rax lynx document

# Target a specific session
rax lynx document --session <id>

# Then inspect specific nodes
rax lynx attributes <node_id>
rax lynx inner-text <node_id>
```

## Analyzing Nodes

### Attributes

```bash
rax lynx attributes <node_id>
```

Returns all attributes: `class`, `style`, `id`, `bindtap`, `src`, `data-*`, etc.

Key things to check:
- `class` — CSS classes applied
- `bindtap` / `catchtap` — Event handlers
- `src` — Image source URL
- `data-*` — Custom data attributes

### Box Model

```bash
rax lynx box-model <node_id>
```

Returns: `width`, `height`, `content`, `padding`, `border`, `margin`.

Use for:
- Checking element dimensions
- Verifying spacing (margin/padding)
- Detecting overlap between elements

### Text Content

```bash
rax lynx inner-text <node_id>
```

Returns the visible text content. Use to verify:
- Correct text displayed
- Text not truncated
- Localization correct

## Common Patterns

### Find a Component by Structure

```bash
# Get document, look for component hierarchy
rax lynx document --session <id>

# Then drill down
rax lynx query ".product-card"          # Find container
rax lynx inner-text <container_id>      # Check text
rax lynx query-all "image"              # Find all images
```

### Verify Element Visibility

```bash
# Get box model to check position
rax lynx box-model <node_id>
# → If x/y is off-screen, element is not visible

# Check computed display/visibility
rax lynx computed-style <node_id>
# → Look for display:none, visibility:hidden, opacity:0
```

### Check Parent-Child Relationship

```bash
# Get full document tree
rax lynx document

# The tree structure shows nesting:
# <view class="container">
#   <view class="card">
#     <text>Product Name</text>
#   </view>
# </view>
```
