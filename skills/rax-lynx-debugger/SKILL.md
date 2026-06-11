---
name: rax-lynx-debugger
description: |
  Debug Lynx pages on connected devices — DOM inspection, style debugging, script analysis, and page control.

  Trigger Scenarios:
  - User wants to inspect a Lynx page's DOM structure
  - User needs to debug CSS/style issues on a Lynx page
  - User wants to query elements by selector or coordinates
  - User needs to reload or control a Lynx page
  - User mentions "lynx", "DOM", "style", "CDP", "lynx page"
---

# RAX Lynx Page Debugger

Guide users through Lynx page debugging — DOM inspection, style analysis, script debugging, and page control using the RAX CLI.

## Prerequisites

- Device connected via RAX (`rax device list` shows `✅`)
- A Lynx page open on the device
- CLI: `rax`

## Workflows

### 1. Connect to a Lynx Page

```bash
# Step 1: List Lynx clients (debug ports) on device
rax lynx clients

# Step 2: List active sessions (each = a Lynx view)
rax lynx sessions

# Step 3: If multiple sessions, commands default to the latest session
# Use --session <id> to target a specific one
```

### 2. DOM Inspection

```bash
# Get full DOM tree
rax lynx document

# Query by CSS selector
rax lynx query ".product-card"
rax lynx query-all ".product-card"

# Get node at coordinates
rax lynx node-at --x 200 --y 500

# Search DOM (text, CSS selector, XPath)
rax lynx search "Buy Now"

# Get specific node details
rax lynx attributes <node_id>
rax lynx box-model <node_id>
rax lynx inner-text <node_id>

# Scroll node into view
rax lynx scroll-into-view <node_id>
```

See [DOM Inspection Guide](references/dom-inspection.md) for patterns.

### 3. Style Debugging

Three levels of style information:

```bash
# Computed style — final rendered values (most useful)
rax lynx computed-style <node_id>

# Matched styles — which CSS rules matched this node
rax lynx matched-styles <node_id>

# Inline styles — only the style attribute
rax lynx inline-styles <node_id>
```

**Typical style debugging flow:**
1. Find the node: `rax lynx query ".target-element"`
2. Check computed values: `rax lynx computed-style <node_id>`
3. If unexpected, check matched rules: `rax lynx matched-styles <node_id>`
4. Check for inline overrides: `rax lynx inline-styles <node_id>`

See [Style Debugging Guide](references/style-debugging.md) for common issues.

### 4. Script Debugging

```bash
# List loaded scripts
rax lynx scripts

# Get script source code
rax lynx script-source <script_id>
```

### 5. Page Control

```bash
# Take a Lynx-only screenshot
rax lynx screenshot
rax lynx screenshot --output ./lynx-page.png

# Reload the page
rax lynx reload

# Open a URL in Lynx
rax lynx open "lynx://example/page"

# Close current page
rax lynx close

# Simulate tap on Lynx coordinates
rax lynx tap --x 200 --y 500
```

### 6. Raw CDP Commands

For advanced operations not covered by dedicated commands:

```bash
# Send any CDP command (<method> is positional, params via --params flag)
rax lynx cdp DOM.getDocument --params '{"depth": 2}'

# Target specific session
rax lynx cdp Runtime.evaluate --params '{"expression": "1+1"}' --session <id>

# Enable DOM events
rax lynx cdp DOM.enable --params '{"useCompression": false}'

# Get document with box model (no params needed)
rax lynx cdp DOM.getDocumentWithBoxModel
```

See [CDP Reference](references/cdp-reference.md) for supported methods.

### 7. Cache Management

```bash
# Clear Lynx target cache (after switching pages/devices)
rax lynx clear-cache
```

## Troubleshooting

```
No Lynx sessions found?
├── Is a Lynx page actually open? → Open a Lynx page on device
├── Is DevTool enabled in the app? → Check app configuration
├── Stale cache? → rax lynx clear-cache, then retry
├── Multiple clients? → rax lynx clients, select the right one
└── Wrong device? → rax device list, rax device select
```

## Command Reference

| Command | Description |
|---------|-------------|
| `rax lynx clients` | List Lynx debug clients on device |
| `rax lynx sessions` | List active Lynx sessions |
| `rax lynx document [--session <id>]` | Get DOM tree |
| `rax lynx query <selector> [--session <id>]` | Query first matching node |
| `rax lynx query-all <selector> [--session <id>]` | Query all matching nodes |
| `rax lynx attributes <node_id>` | Get node attributes |
| `rax lynx box-model <node_id>` | Get node box model |
| `rax lynx inner-text <node_id>` | Get node text content |
| `rax lynx node-at --x <x> --y <y>` | Get node at coordinates |
| `rax lynx search <query>` | Search DOM (text, CSS, XPath) |
| `rax lynx scroll-into-view <node_id>` | Scroll node into view |
| `rax lynx computed-style <node_id>` | Get computed styles |
| `rax lynx matched-styles <node_id>` | Get matched CSS rules |
| `rax lynx inline-styles <node_id>` | Get inline styles |
| `rax lynx screenshot [--output <path>]` | Take Lynx screenshot |
| `rax lynx reload` | Reload page |
| `rax lynx open <url>` | Open URL in Lynx |
| `rax lynx close` | Close current page |
| `rax lynx tap --x <x> --y <y>` | Simulate tap at Lynx coordinates |
| `rax lynx scripts` | List loaded scripts (enables debugger) |
| `rax lynx script-source <script_id>` | Get script source |
| `rax lynx cdp <method> [--params <json>] [--session <id>]` | Send raw CDP command |
| `rax lynx clear-cache` | Clear Lynx target cache |

## References

- [DOM Inspection Guide](references/dom-inspection.md) — Finding and analyzing DOM nodes
- [Style Debugging Guide](references/style-debugging.md) — Diagnosing style issues
- [CDP Reference](references/cdp-reference.md) — Supported Chrome DevTools Protocol methods
