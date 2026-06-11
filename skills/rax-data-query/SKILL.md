---
name: rax-data-query
description: |
  Query and manage app data — AB testing, event tracking, Bridge logs, router logs, and session management.

  Trigger Scenarios:
  - User wants to check or mock AB test configurations
  - User needs to verify event tracking (埋点)
  - User wants to inspect Bridge (JSB) call history
  - User needs to trace router navigation
  - User mentions "AB", "实验", "settings", "埋点", "event", "bridge", "router", "日志"
---

# RAX Data Query

Guide users through AB test management, event tracking verification, Bridge call inspection, and router log analysis using the RAX CLI.

## Prerequisites

- Device connected via RAX (`rax device list` shows `✅`)
- Relevant capture types enabled before performing actions
- CLI: `rax`

## Workflows

### 1. AB Test Configuration

**Find and inspect AB settings:**

```bash
# Search by key (fuzzy match)
rax ab search --key_pattern "live"

# Search by value
rax ab search --key_pattern "feed" --value_pattern "true"

# Force refresh from device (bypass DB cache)
rax ab search --key_pattern "live" --refresh true
```

**Mock an AB setting:**

```bash
# Step 1: Find the key
rax ab search --key_pattern "new_feature"

# Step 2: Enable mock system
rax ab mock-enable true

# Step 3: Set mock value (positional: <propertyName> <mockValue>)
rax ab mock new_feature_enabled true

# Step 4: Verify on device (restart page or re-trigger)
# ... user checks behavior ...

# Step 5: Clean up
rax ab clear
rax ab mock-enable false
```

See [AB Testing Guide](references/ab-testing.md) for detailed patterns.

### 2. Event Tracking Verification (埋点)

```bash
# Step 1: Enable event log capture
rax network capture app_log

# Step 2: Perform the action on device

# Step 3: Search for the event
rax db search-events --event_name "product_click"

# Fuzzy search by event name pattern
rax db search-events --event_name_pattern "click"

# Filter by session
rax db search-events --event_name_pattern "click" --session_id <id>
```

See [Event Tracking Guide](references/event-tracking.md) for verification workflows.

### 3. Bridge (JSB) Call Inspection

```bash
# Step 1: Enable Bridge capture
rax network capture bridge

# Step 2: Perform the action on device

# Step 3: Search Bridge calls
rax db search-bridge --method "getUserInfo"

# Fuzzy search
rax db search-bridge --method_pattern "user"

# Filter by status
rax db search-bridge --method_pattern "user" --success true
```

See [Bridge Logs Guide](references/bridge-logs.md) for debugging JSB issues.

### 4. Router Log Analysis

```bash
# Step 1: Enable router capture
rax network capture router

# Step 2: Navigate in the app

# Step 3: Search router logs
rax db search-router --url_pattern "product/detail"

# Search by page name
rax db search-router --page_name "FeedPage"
```

### 5. Session Management

Sessions group captured data by time period. Use sessions to isolate data from specific test runs.

```bash
# List sessions
rax db sessions

# Filter sessions with specific data types
rax db sessions --has_network_requests true
rax db sessions --has_router_logs true
rax db sessions --has_app_log true

# Filter by device
rax db sessions --device_id <id>

# Paginate (default limit 100)
rax db sessions --limit 20 --offset 0

# Clean old data (--yes to skip confirmation)
rax db clean --days 7
rax db clean --days 7 --yes
```

### 6. RAX Logs

```bash
# Enable RAX log capture
rax network capture rax

# Search RAX logs
rax db search-rax --content_pattern "error"
rax db search-rax --log_type "error"
```

## Troubleshooting

```
No data found?
├── Is capture enabled for the data type?
│   ├── Events (埋点) → rax network capture app_log
│   ├── Bridge (JSB) → rax network capture bridge
│   ├── Router → rax network capture router
│   └── RAX logs → rax network capture rax
│   └── All at once → rax network capture network,bridge,app_log,router,rax
├── Was capture started before the action?
├── Is the search pattern correct? → Try a shorter/broader pattern
├── Is the session correct? → rax db sessions
└── Is old data cleaned? → rax db clean --days 30
```

## Command Reference

### AB Settings

| Command | Description |
|---------|-------------|
| `rax ab search` | Search AB settings |
| `rax ab mock <propertyName> <mockValue>` | Mock a setting |
| `rax ab mock-enable <enable>` | Enable/disable mock system (true/false) |
| `rax ab clear` | Clear all mocks |

### Database Queries

| Command | Description |
|---------|-------------|
| `rax db sessions` | List capture sessions |
| `rax db search-network` | Search network requests (same as `rax network search`) |
| `rax db search-events` | Search event tracking logs |
| `rax db search-bridge` | Search Bridge (JSB) call logs |
| `rax db search-router` | Search router navigation logs |
| `rax db search-rax` | Search RAX internal logs |
| `rax db clean` | Clean old captured data |

### Common Flags

| Flag | Commands | Description |
|------|----------|-------------|
| `--session_id` | All search commands | Filter by session |
| `--limit` | All search commands | Max results (default varies) |
| `--offset` | All search commands | Skip N results |

## References

- [AB Testing Guide](references/ab-testing.md) — Search, mock, verify, and clean up AB experiments
- [Event Tracking Guide](references/event-tracking.md) — Verify event tracking implementation
- [Network Query Guide](references/network-query.md) — Advanced network request queries via DB
- [Bridge Logs Guide](references/bridge-logs.md) — Debug JSB call issues
