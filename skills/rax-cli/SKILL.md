---
name: rax-cli
version: 1.0.0
description: "RAX CLI — AI-powered mobile device debugging tool. Manage device connections, capture & mock network requests, automate UI interactions, inspect Lynx pages, query captured data, and more. Use when user mentions: device debugging, network capture, UI automation, screenshot, AB settings, Lynx inspection, mock network, weak network, app launch, clipboard, sandbox, bridge logs, router logs, event tracking."
metadata:
  requires:
    bins: ["rax"]
  cliHelp: "rax --help"
---

# RAX CLI Skill

RAX CLI is a command-line tool for mobile device debugging. It connects to iOS/Android devices via USB or network, and provides a rich set of debugging capabilities through a background daemon.

## Prerequisites

If RAX CLI is not installed, install it first:

```bash
npm install -g @tiktok-fe/rax-cli --registry=https://bnpm.byted.org --loglevel=error
```

RAX CLI requires a running daemon. The daemon starts automatically on first command. If you encounter connection errors, ensure the daemon is running:

```bash
rax daemon status
rax daemon start
```

## Quick Reference

| Domain | Prefix | What it does |
|--------|--------|--------------|
| Device | `rax device` | Connect, select, screenshot, clipboard |
| App | `rax app` | Get app info, accounts, and switch accounts |
| Network | `rax network` | Capture, search, mock, weak network, rewrite |
| AB Settings | `rax ab` | Search, mock AB experiment settings |
| UI Interaction | `rax ui` | Tap, swipe, input, describe UI, launch/stop apps |
| iOS | `rax ios` | Sandbox, bundle, keychain, notifications, HW buttons |
| Android | `rax android` | SharedPreferences, MMKV, Settings |
| Database | `rax db` | Query captured sessions, network, router, event logs |
| Webhook | `rax webhook` | Register webhooks for real-time events |
| Share | `rax share` | Fetch shared data from RAX share links |
| Lynx | `rax lynx` | DOM inspection, CSS, screenshots, CDP commands |

## Global Flags

All commands support these flags:

| Flag | Description |
|------|-------------|
| `--help`, `-h` | Show help |
| `--json` | JSON output (alias for `--format json`) |
| `--format <fmt>` | Output format: `text` \| `json` \| `table` \| `csv` |
| `--dry-run` | Preview the RPC call without executing |
| `--yes`, `-y` | Skip confirmation for high-risk commands |
| `--device <id>` | Target a specific device |

## Typical Workflow

1. **Connect & select a device**
   ```bash
   rax device list          # See connected devices
   rax device select D1     # Select by tag or ID
   ```

2. **Debug network requests**
   ```bash
   rax network capture network           # Start capturing
   rax network search --url_pattern /api  # Search requests
   rax network add-mock /api/data '{"ok":true}'  # Mock a response
   ```

3. **Automate UI**
   ```bash
   rax ui describe                # Get current UI tree
   rax ui tap 100 200             # Tap at coordinates
   rax device screenshot          # Take screenshot
   ```

4. **Inspect Lynx pages**
   ```bash
   rax lynx sessions              # List active sessions
   rax lynx document              # Get DOM tree
   rax lynx query ".my-class"     # Query selector
   ```

## Command Details

### Device Management

```bash
rax device list                              # List connected devices
rax device scan                              # Rescan USB devices
rax device select <device>                   # Select active device (by tag or ID)
rax device connect <host> [--port 5555]      # Connect remote Android via ADB TCP
rax device connect-rax <id> --ip <ip> --port <port>  # Connect via RAX WebSocket
rax device disconnect                        # Disconnect all
rax device info                              # Get device & app info
rax device screenshot [--output path]        # Take screenshot (saves PNG)
rax device screenshot --base64               # Output base64 string
rax device clipboard [content]               # Read or write clipboard
rax device keep-alive <enable>               # Keep screen awake
rax device open-schema <schema>              # Open schema URL on device
```

### App Info

```bash
rax app info                                 # Get app info via deviceInfo/appInfo
rax app info --json                          # Output raw appInfo JSON
rax app account                              # Get current/loginAccountList account summary
rax app account --json                       # Only customID, nickname, and userID are exposed
rax app account change <userid>              # Change account via deviceInfo/changeSettings
rax app account change <userid> --dry-run    # Preview method and params
```

`rax app account change <userid>` sends this RAX data to `deviceInfo/changeSettings`:

```json
{
  "account": "<userid>"
}
```

Recommended flow: run `rax app account`, copy a `loginAccountList[].userID`, then run `rax app account change <userid>`.

### Network Debugging

```bash
# Capture
rax network capture network                  # Start capturing network requests
rax network capture network,router,event     # Capture multiple types

# Search
rax network search --url_pattern /api/shop   # Search by URL
rax network search --method POST --status_code 200
rax network search --fields request_body,response_body  # Include body data
rax network stats                            # Request statistics

# Mock
rax network add-mock /api/path '{"data":"mocked"}'   # Single mock
rax network add-mock --items '[{"path":"/api","responseBody":"{}"}]'  # Batch
rax network remove-mock /api/path

# Weak Network
rax network add-weak /api/path 3000          # Add 3s delay
rax network remove-weak /api/path

# Rewrite
rax network rewrite --ppe boe                # PPE environment headers
rax network rewrite --items '[{"match":{"host":"a.com"},"rewrite":{"host":"b.com"}}]'
rax network rewrite --clear

# Send
rax network send https://example.com --method POST
```

### AB Settings

```bash
rax ab search                                # List all AB settings
rax ab search --key_pattern feature_flag     # Filter by key
rax ab search --refresh                      # Force re-fetch from device
rax ab mock feature_flag true                # Mock a setting value
rax ab mock-enable true                      # Enable/disable mocking
rax ab clear --yes                           # Clear all mocks (destructive)
```

### UI Interaction

```bash
# WebDriverAgent (iOS automation prerequisite)
rax ui wda init --udid <udid> --team_id <id>
rax ui wda start
rax ui wda stop

# Gestures
rax ui tap 100 200                           # Tap at (100, 200)
rax ui tap 100 200 --duration_ms 1000        # Long press
rax ui swipe 100 500 100 100                 # Swipe up
rax ui input "hello world"                   # Type text
rax ui press back                            # Press back/home key

# UI inspection
rax ui describe                              # Full UI tree
rax ui describe --filter_text "Login"        # Filter elements

# App management
rax ui launch com.example.app                # Launch app
rax ui stop com.example.app                  # Stop app
rax ui apps                                  # List installed apps
rax ui apps --query "tiktok"                 # Search apps
```

### iOS-Specific

```bash
# Sandbox & Bundle
rax ios sandbox                              # Browse app sandbox
rax ios sandbox --path Documents/            # Browse specific path
rax ios sandbox-content Documents/config.json
rax ios add-sandbox Documents/test.txt --content "hello"
rax ios clear-sandbox Documents/cache/ --yes
rax ios bundle                               # Browse app bundle
rax ios bundle-content Info.plist

# System
rax ios user-defaults                        # Get UserDefaults
rax ios keychain                             # Get keychain entries
rax ios notifications                        # Get notifications
rax ios send-notify myEvent --userInfo '{"key":"value"}'

# UI targets & input
rax ios targets                              # List UI targets
rax ios select-target <target>
rax ios press-button home                    # Press HW button
rax ios key-sequence "abc"                   # Send key sequence
rax ios describe-point 100 200               # Describe element at point
```

### Android-Specific

```bash
rax android prefs                            # Get SharedPreferences
rax android prefs --name my_prefs            # Specific file
rax android put-prefs my_prefs key value     # Write SharedPreferences
rax android mmkv                             # Get MMKV info
rax android settings                         # Get Settings
rax android awe-storage                      # Get AweStorage
rax android current-app                      # Current foreground app
```

### Database (Captured Data)

```bash
# Sessions
rax db sessions                              # List capture sessions
rax db sessions --has_network_requests       # Only sessions with network data

# Search captured data
rax db search-network --url_pattern /api     # Network requests
rax db search-router --page_name HomePage    # Router logs
rax db search-events --event_name app_launch # Event/tracking logs
rax db search-bridge --method getUserInfo    # Bridge call logs
rax db search-rax --content_pattern error    # RAX framework logs

# Maintenance
rax db clean --days 7 --yes                  # Clean data older than 7 days
```

### Webhook

```bash
rax webhook add https://my-server.com/hook --events network,router
rax webhook list
rax webhook remove <id>
```

### Share

```bash
# Fetch shared data from a RAX share link
rax share fetch "https://rax.tiktok-row.net/log-event-network/network-capture?shareId=new_xxx"
rax share fetch "https://rax.tiktok-row.net/log-event-network/network-capture?shareMockId=xxx"
rax share fetch "https://rax.tiktok-row.net/log-event-network/real-time-router?shareId=new_xxx"
rax share fetch "https://rax.tiktok-row.net/log-event-network/real-time-event?shareId=new_xxx"
```

Supported types: network capture, network mock, event, router, bridge call, bridge event, AB settings, AB mock. Output: `Type: xxx` line followed by content JSON. JSON fields (responseBody, headers, etc.) are auto-parsed into objects.

### Lynx Page Inspection

```bash
# Sessions & DOM
rax lynx clients                             # List Lynx clients
rax lynx sessions                            # List Lynx sessions
rax lynx document [--session <id>]           # Get DOM document
rax lynx reload                              # Reload page

# DOM queries
rax lynx query ".my-class"                   # querySelector
rax lynx query-all "view"                    # querySelectorAll
rax lynx search "button text"               # Search DOM
rax lynx node-at --x 100 --y 200            # Node at coordinates

# Node details
rax lynx attributes <node_id>               # Get attributes
rax lynx box-model <node_id>                # Get box model
rax lynx inner-text <node_id>               # Get inner text
rax lynx scroll-into-view <node_id>         # Scroll into view

# Styles
rax lynx computed-style <node_id>
rax lynx matched-styles <node_id>
rax lynx inline-styles <node_id>

# Actions
rax lynx tap --x 100 --y 200                # Tap in Lynx coordinates
rax lynx open <url>                          # Open Lynx page
rax lynx close                               # Close current page
rax lynx screenshot [--output path]          # Lynx screenshot

# Debugging
rax lynx scripts                             # List loaded scripts
rax lynx script-source <script_id>           # Get script source
rax lynx cdp <method> [--params '{}']        # Raw CDP command
```

### System Commands

```bash
rax doctor                                   # Check environment & connectivity
rax daemon start|stop|status|restart         # Manage background daemon
rax tui                                      # Interactive dashboard
rax tui --network                            # Network capture TUI
rax tui --log                                # Log viewer TUI
rax upgrade                                  # Upgrade rax-cli to the latest version
rax upgrade --check                          # Check for updates without installing
rax completion [bash|zsh|fish]               # Shell auto-completion script
```

## Tips for AI Agents

1. **Always start with `rax device list`** to see available devices. If no devices, suggest `rax device scan` or USB connection.
2. **Use `--json` flag** when you need to parse output programmatically.
3. **Screenshot after UI actions** — use `rax device screenshot` to verify UI state after taps/swipes.
4. **Network debugging flow**: `capture` → do actions on device → `search` to find requests.
5. **Lynx inspection flow**: `sessions` → `document` → `query` to drill down into DOM.
6. **High-risk commands** (like `ab clear`, `db clean`, `ios clear-sandbox`) will prompt for confirmation. Use `--yes` to skip in automated flows.
7. **Use `--dry-run`** to preview what will be called without executing.
8. **Auto-upgrade**: When a `rax` command fails with an unexpected error, returns an unknown command error, or the CLI lacks a capability you expect it to have, proactively run `rax upgrade` to install the latest version. After upgrading, retry the failed command.
9. **Missing capability**: If the user requests a mobile-device-related feature that doesn't appear in the command list above, **do not assume it is unsupported**. First run `rax upgrade` to get the latest version, then re-check with `rax --help` or the relevant subcommand's `--help`. New capabilities are added frequently — always upgrade before concluding that a feature is unavailable.

## Update Notices (proactive upgrade flow)

There are **two channels** through which a new-version signal can reach you, depending on how you invoked rax. Both are emitted by the same daemon-maintained cache, so they're equally trustworthy — react to whichever one fires first.

### Channel A — `_notice.update` JSON field

The daemon attaches `_notice.update` to RPC responses when a newer version of `@tiktok-fe/rax-cli` is available. You'll see this when:

- **MCP tool calls** (rax MCP server → Claude): the field is part of the tool result JSON. ✅ Primary channel.
- **CLI with `--format json` / `--json`**: the field appears in stdout JSON. ✅
- **CLI default output (`text` / `table` / `csv`)**: ❌ NOT rendered — `formatResult` only emits `content`. Use Channel B instead.

Example shape:

```json
{
  "content": [{ "type": "text", "text": "..." }],
  "_notice": {
    "update": {
      "current": "0.3.19",
      "latest": "0.3.22",
      "command": "rax upgrade"
    }
  }
}
```

The `command` is `rax upgrade` — a single short command that internally
downloads the new CLI, refreshes the rax-cli skill, and restarts the
daemon. Just run it verbatim.

### Channel B — stderr text notice

When you invoke `rax` via Bash with default output (no `--json`), the CLI itself prints a notice to **stderr** on exit. Your Bash tool captures stderr alongside stdout, so you'll see this block somewhere in the combined tool output:

```
[rax-cli] A new version is available: v0.3.19 → v0.3.22
Run: rax upgrade
```

(In your tool output the ANSI color codes may appear as `\x1b[33m...\x1b[0m` — strip mentally; the literal text `[rax-cli] A new version is available` is the reliable marker.)

Treat this exactly like `_notice.update` — the upgrade command is `rax upgrade` in both channels.

### What to do (applies to BOTH channels)

When you see either signal, you MUST:

1. **Finish the user's current task first** — never interrupt mid-operation.
2. **After the task is done**, proactively tell the user a new version is available. Include the exact version jump (e.g. `v0.3.19 → v0.3.22`) and offer to run the upgrade command for them.
3. **Use the `command` field (Channel A) or the `Run:` line (Channel B) verbatim** — it already bundles CLI + Skill in one line.
4. **After upgrading**, remind the user to **exit and reopen the AI Agent** (Claude Code, etc.) so the new Skill / MCP definitions load. Skill changes do not take effect until the Agent restarts.
5. **Mention the upgrade at most once per conversation.** Even if you see the notice on multiple commands (Channel B repeats every CLI invocation; Channel A is deduped per MCP session by the daemon), only surface it to the user once. If the user declines, drop it for the rest of the session.

Example reply after completing a task:

> Done — I've finished listing your devices. Heads up: rax-cli has a new version available (v0.3.19 → v0.3.22). Want me to run the upgrade? It takes one command and I'll remind you to restart this Agent after it finishes.
