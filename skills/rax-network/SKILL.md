---
name: rax-network
description: |
  Network debugging with RAX — capture, search, Mock, weak network simulation, and request rewrite.

  Trigger Scenarios:
  - User wants to capture/inspect network requests
  - User needs to Mock an API response
  - User wants to simulate weak network conditions
  - User can't find a specific request in capture results
  - User mentions "network", "抓包", "mock", "弱网", "request rewrite"
---

# RAX Network Debugging

Guide users through network capture, request analysis, Mock setup, weak network simulation, and request rewrite using the RAX CLI.

## Prerequisites

- Device connected via RAX (`rax device list` shows `✅`)
- CLI: `rax`

## Workflows

### 1. Capture & Analyze Requests

**Step-by-step workflow for finding and analyzing network requests:**

```bash
# Step 1: Enable network capture
rax network capture network

# Step 2: Perform the action on device (open page, tap button, etc.)
# ... user performs actions ...

# Step 3: Search captured requests
rax network search --url_pattern "api/feed"

# Step 4: Filter further if needed
rax network search --url_pattern "api/feed" --method GET --status_code 200

# Step 5: Get statistics
rax network stats
```

**Can't find the request?** See [Capture Workflow](references/capture-workflow.md) for troubleshooting.

### 2. Mock API Responses

```bash
# Add a mock (path + response body)
rax network add-mock "/api/user/profile" '{"name":"test","level":5}'

# Batch mock (JSON array)
rax network add-mock --items '[
  {"path": "/api/feed", "responseBody": "{\"items\": []}"},
  {"path": "/api/user", "responseBody": "{\"name\": \"mock\"}"}
]'

# Remove a mock
rax network remove-mock "/api/user/profile"

# Remove multiple
rax network remove-mock --paths '["/api/feed", "/api/user"]'
```

See [Mock Guide](references/mock-guide.md) for patterns and tips.

### 3. Weak Network Simulation

```bash
# Add delay to specific requests
rax network add-weak "/api/feed" 3000

# Batch configuration
rax network add-weak --items '[
  {"id": "1", "path": "/api/feed", "desc": "Feed API delay", "weakNetworkDelay": 3000},
  {"id": "2", "path": "/api/search", "desc": "Search delay", "weakNetworkDelay": 5000}
]'

# Remove weak network mock
rax network remove-weak "/api/feed"
```

See [Weak Network Guide](references/weak-network.md) for testing strategies.

### 4. Request Rewrite

Modify requests in flight — change URL, headers, query params, or body.

```bash
# Redirect API to a different host
rax network rewrite --items '[{
  "match": {"host": "api.example.com", "path": "/v1/feed"},
  "rewrite": {"host": "api-staging.example.com"}
}]'

# Switch to PPE environment (adds x-tt-env and x-use-ppe headers to all requests)
rax network rewrite --ppe boe

# Add custom headers
rax network rewrite --items '[{
  "match": {"path": "/api/feed"},
  "rewrite": {"headers": {"X-Debug": "true"}}
}]'

# Clear all rewrite rules
rax network rewrite --clear
```

See [Request Rewrite Guide](references/request-rewrite.md) for advanced matching patterns.

### 5. Send HTTP Request via Device

```bash
# Send request through device proxy (uses device's network/cookies)
rax network send "https://api.example.com/v1/feed" --method GET
```

### 6. Capture Type Management

The `types` positional argument accepts a comma-separated list of capture types:

```bash
# Single type
rax network capture network

# Multiple types at once (comma-separated, no spaces)
rax network capture network,router,event

# Available types: network, bridge, app_log, router, rax
rax network capture network,bridge,app_log,router,rax
```

## Troubleshooting

```
Can't find the request?
├── Is capture enabled? → rax network capture network
├── Was capture started before the action? → Requests before capture won't show
├── URL pattern too strict? → Try a shorter substring
├── Wrong session? → Check rax db sessions, use --session_id
├── Request uses different domain? → Try without --url_pattern first
└── Request is not HTTP? → WebSocket/custom protocols not captured
```

## Command Reference

| Command | Description |
|---------|-------------|
| `rax network capture <types>` | Toggle capture types (network, bridge, app_log, router, rax) |
| `rax network search` | Search captured network requests |
| `rax network stats` | Request statistics summary |
| `rax network add-mock <path> <responseBody>` | Add response mock (or use `--items` for batch) |
| `rax network remove-mock <path>` | Remove mock (or use `--paths` for batch) |
| `rax network add-weak <path> <delay>` | Add weak network simulation (or use `--items` for batch) |
| `rax network remove-weak <path>` | Remove weak network simulation (or use `--paths` for batch) |
| `rax network send <url>` | Send HTTP request through device |
| `rax network rewrite` | Set/clear request rewrite rules |

### Search Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--url_pattern` | URL substring match | `"api/feed"` |
| `--method` | HTTP method filter | `GET`, `POST` |
| `--status_code` | Status code filter | `200`, `500` |
| `--success` | Success/failure filter | `true`, `false` |
| `--session_id` | Filter by session | (from `rax db sessions`) |
| `--limit` | Max results (default 50) | `100` |
| `--offset` | Skip N results | `50` |
| `--order_by` | Sort by field | `time`, `duration` |
| `--fields` | Extra fields to include | `request_body,response_body,request_headers,response_headers` |

## References

- [Capture Workflow](references/capture-workflow.md) — Step-by-step capture and search techniques
- [Mock Guide](references/mock-guide.md) — Mock patterns, batch operations, verification
- [Weak Network Guide](references/weak-network.md) — Weak network testing strategies
- [Request Rewrite Guide](references/request-rewrite.md) — Advanced matching and rewrite rules
