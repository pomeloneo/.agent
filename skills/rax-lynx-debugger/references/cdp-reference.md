# CDP Reference

## Overview

RAX Lynx debugging supports a subset of the Chrome DevTools Protocol (CDP). Use `rax lynx cdp <method> [--params <json>] [--session <id>]` to send raw CDP commands.

Most common CDP operations have dedicated CLI commands (e.g., `rax lynx document` for `DOM.getDocument`). Use raw CDP only for operations without a dedicated command.

## Supported Domains

### DOM

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `DOM.enable` | — | Enable DOM agent |
| `DOM.disable` | — | Disable DOM agent |
| `DOM.getDocument` | `rax lynx document` | Get document root |
| `DOM.getDocumentWithBoxModel` | — | Document + layout info |
| `DOM.querySelector` | `rax lynx query` | CSS selector query |
| `DOM.querySelectorAll` | `rax lynx query-all` | CSS selector query all |
| `DOM.getAttributes` | `rax lynx attributes` | Node attributes |
| `DOM.getBoxModel` | `rax lynx box-model` | Box model info |
| `DOM.innerText` | `rax lynx inner-text` | Text content |
| `DOM.getNodeForLocation` | `rax lynx node-at` | Node at coordinates |
| `DOM.performSearch` | `rax lynx search` | Search DOM |
| `DOM.getSearchResults` | — | Get search results |
| `DOM.discardSearchResults` | — | Discard search |
| `DOM.requestChildNodes` | — | Request child nodes |
| `DOM.scrollIntoViewIfNeeded` | `rax lynx scroll-into-view` | Scroll into view |
| `DOM.setAttributesAsText` | — | Set attributes |
| `DOM.getOuterHTML` | — | Get outer HTML |
| `DOM.getOriginalNodeIndex` | — | Get original index |

### CSS

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `CSS.getComputedStyleForNode` | `rax lynx computed-style` | Computed styles |
| `CSS.getMatchedStylesForNode` | `rax lynx matched-styles` | Matched CSS rules |
| `CSS.getInlineStylesForNode` | `rax lynx inline-styles` | Inline styles |
| `CSS.getBackgroundColors` | — | Background colors |

### Runtime

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `Runtime.enable` | — | Enable runtime |
| `Runtime.disable` | — | Disable runtime |
| `Runtime.evaluate` | — | Evaluate JS expression |
| `Runtime.callFunctionOn` | — | Call function on object |
| `Runtime.getProperties` | — | Get object properties |
| `Runtime.getHeapUsage` | — | Get heap usage |
| `Runtime.compileScript` | — | Compile script |
| `Runtime.runScript` | — | Run compiled script |

### Page

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `Page.reload` | `rax lynx reload` | Reload page |
| `Page.getResourceTree` | — | Get resource tree |
| `Page.getResourceContent` | — | Get resource content |

### Debugger

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `Debugger.getScriptSource` | `rax lynx script-source` | Get script source |

### Input

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `Input.emulateTouchFromMouseEvent` | — | Simulate touch |

### Overlay

| Method | Dedicated Command | Description |
|--------|-------------------|-------------|
| `Overlay.highlightNode` | — | Highlight a node |
| `Overlay.hideHighlight` | — | Hide highlight |

## Usage Examples

```bash
# Evaluate JavaScript
rax lynx cdp Runtime.evaluate --params '{"expression": "lynx.__globalProps"}'

# Get document with full depth
rax lynx cdp DOM.getDocument --params '{"depth": -1}'

# Get DOM with box model (for layout debugging)
rax lynx cdp DOM.enable --params '{"useCompression": false}'
rax lynx cdp DOM.getDocumentWithBoxModel

# Highlight a node (visual debugging)
rax lynx cdp Overlay.highlightNode --params '{"nodeId": 42}'
rax lynx cdp Overlay.hideHighlight

# Get resource tree
rax lynx cdp Page.getResourceTree

# Check heap usage
rax lynx cdp Runtime.getHeapUsage
```

## Tips

- The `<method>` is a positional argument, params go in `--params` flag as JSON string
- Target specific session: `rax lynx cdp DOM.getDocument --session <id>`
- If commands fail, try `rax lynx clear-cache` first
- Methods without params don't need `--params`
