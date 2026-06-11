---
name: rax-share-fetch
description: |
  Fetch and inspect shared data from RAX share links — network captures, network mocks, events, router logs, bridge logs, AB settings, and more.

  Trigger Scenarios:
  - User provides a RAX share link (URL containing shareId or shareMockId)
  - User wants to view shared network capture details
  - User wants to apply a shared mock configuration
  - User pastes a share URL from rax-app
  - User mentions "share link", "分享链接", "shareId", "shareMockId"
---

# RAX Share Fetch

Retrieve and inspect shared data from RAX share links. Supports all share types from rax-app: network captures, network mocks, events, router logs, bridge logs, AB settings, and AB mocks.

## Prerequisites

- CLI: `rax`
- Network access to `rax.tiktok-row.net`
- Optional: `RAX_TOKEN` for authenticated access (env variable or `~/.rax-mcp/user_config.json`)

## Supported Link Types

| Type | URL pattern | Query param |
|------|-------------|-------------|
| Network Capture | `/log-event-network/network-capture?shareId=xxx` | `shareId` |
| Network Mock | `/log-event-network/network-capture?shareMockId=xxx` | `shareMockId` |
| Event | `/log-event-network/real-time-event?shareId=xxx` | `shareId` |
| Router | `/log-event-network/real-time-router?shareId=xxx` | `shareId` |
| Bridge Call | `/lynx-h5-kit/bdx-bridge?shareId=xxx` | `shareId` |
| Bridge Event | `/lynx-h5-kit/bdx-bridge-event?shareId=xxx` | `shareId` |
| AB Settings | `/device-and-app-info/ab-and-settings?shareId=xxx` | `shareId` |
| AB Mock | `rax://ab_test_and_settings/mock?shareMockId=xxx` | `shareMockId` |

A full share URL is required — bare share IDs are not supported because different share types are stored in different backend tables.

## Workflows

### 1. Fetch and View Shared Data

```bash
# Network capture share
rax share fetch "https://rax.tiktok-row.net/log-event-network/network-capture?shareId=new_xxx"

# Network mock share
rax share fetch "https://rax.tiktok-row.net/log-event-network/network-capture?shareMockId=xxx"

# Router share
rax share fetch "https://rax.tiktok-row.net/log-event-network/real-time-router?shareId=new_xxx"

# Event share
rax share fetch "https://rax.tiktok-row.net/log-event-network/real-time-event?shareId=new_xxx"
```

Output format:
```
Type: network_capture
{"host":"example.com","path":"/api/data","method":"POST","code":200,"requestHeader":{...},"responseBody":{...}}
```

- Type line is always first
- Content is compact JSON
- JSON string fields (responseBody, requestHeader, responseHeader, bizInfo, ttnet) are auto-parsed into objects
- Encrypted data (network capture, network mock) is auto-decrypted
- Internal fields (_id, __bytedoc, userId, shareId) are stripped

### 2. Apply a Shared Mock to Device

After fetching a network mock share, apply it to the connected device:

```bash
# Step 1: Fetch the shared mock
rax share fetch "https://rax.tiktok-row.net/...?shareMockId=xxx"
# Output shows: path, responseBody

# Step 2: Apply the mock (copy path and responseBody from output)
rax network add-mock "/api/shop/v1/homepage" '{"code":0,"data":{...}}'
```

For AB settings mock:
```bash
# Step 1: Fetch
rax share fetch "rax://ab_test_and_settings/mock?shareMockId=xxx"

# Step 2: Apply
rax ab mock-enable true
rax ab mock <key> <value>
```

### 3. Investigate a Shared Network Capture

```bash
# Step 1: Fetch the shared capture
rax share fetch "https://...?shareId=new_xxx"

# Step 2: Compare with live requests
rax network search --url_pattern "/api/shop" --method POST

# Step 3: Check if there's a mock active
rax network search --url_pattern "/api/shop" --fields is_mock
```

## Error Handling

| Error | Cause | Action |
|-------|-------|--------|
| "Share not found" | Invalid or expired share ID | Verify the link with the sharer |
| "Failed to decrypt" | Data format changed | Report to RAX team |
| "Request timeout" | API unreachable | Check network access to rax.tiktok-row.net |
| "url is required" | No URL provided | Pass the full share link as argument |
| "no shareId or shareMockId found" | Bare ID or malformed URL | Use the full share URL from rax-app |

## MCP Tool

The `fetch_share` MCP tool provides the same functionality for AI agents:

```
fetch_share({ url: "<share URL>" })
```

Returns MCP-standard content array:
- `content[0].text`: `Type: <share_type>`
- `content[1].text`: content as compact JSON
