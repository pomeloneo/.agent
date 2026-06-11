# Capture Workflow

## Complete Capture & Search Guide

### Step 1: Enable Capture BEFORE the Action

Capture only records requests that occur **after** it's enabled. Always start capture first.

```bash
rax network capture network
```

### Step 2: Perform the Action

Do the action on the device that triggers the request (open a page, tap a button, pull to refresh, etc.).

### Step 3: Search for the Request

Start with a broad search, then narrow down:

```bash
# Broad: just a URL substring
rax network search --url_pattern "feed"

# Narrow: add method filter
rax network search --url_pattern "feed" --method POST

# More specific: add status
rax network search --url_pattern "api/v1/feed" --method GET --status_code 200
```

### Step 4: Get Details

Use `--fields` to include extra data (request/response bodies and headers):

```bash
# Include response body in results
rax network search --url_pattern "feed" --fields "response_body"

# Include full request/response data
rax network search --url_pattern "feed" --fields "request_body,response_body,request_headers,response_headers"
```

### Step 5: Sort and Page

```bash
# Slowest requests first
rax network search --url_pattern "feed" --order_by duration

# Most recent first (default)
rax network search --url_pattern "feed" --order_by time

# Pagination
rax network search --url_pattern "feed" --limit 20 --offset 0   # Page 1
rax network search --url_pattern "feed" --limit 20 --offset 20  # Page 2
```

## Troubleshooting: Can't Find the Request

### Problem: No results at all

1. **Capture not enabled:**
   ```bash
   # Check — if you get no output, capture was off
   rax network capture network
   ```

2. **Action happened before capture started:**
   Re-enable capture, then repeat the action.

3. **Wrong device selected:**
   ```bash
   rax device list   # Check which device is active
   ```

### Problem: Too many results, can't find the right one

1. **Use a more specific URL pattern:**
   ```bash
   # Too broad
   rax network search --url_pattern "api"
   # Better
   rax network search --url_pattern "api/v2/feed/recommend"
   ```

2. **Filter by method:**
   ```bash
   rax network search --url_pattern "feed" --method POST
   ```

3. **Filter by status code:**
   ```bash
   # Only failures
   rax network search --url_pattern "feed" --success false
   # Only 500 errors
   rax network search --url_pattern "feed" --status_code 500
   ```

4. **Filter by session:**
   ```bash
   # List sessions to find the right one
   rax db sessions --has_network_requests true
   # Search within that session
   rax network search --url_pattern "feed" --session_id <id>
   ```

### Problem: Request exists but not captured

Some requests are not captured by RAX:
- WebSocket connections (only HTTP)
- Requests made before RAX SDK initializes
- Requests through custom network stacks that bypass the SDK interceptor
