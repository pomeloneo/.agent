# Bridge (JSB) Logs Guide

## Overview

Bridge logs capture JavaScript Bridge calls between the app's JS runtime and native code. Use this to debug JSB communication issues.

## Capture & Search

### Enable Capture

```bash
rax network capture bridge
```

### Search Bridge Calls

```bash
# Exact method name
rax db search-bridge --method "getUserInfo"

# Fuzzy search
rax db search-bridge --method_pattern "user"

# Filter by success
rax db search-bridge --method_pattern "user" --success true

# Filter by status code
rax db search-bridge --method_pattern "user" --status_code 0

# By session
rax db search-bridge --method_pattern "user" --session_id <id>

# Paginate
rax db search-bridge --method_pattern "user" --limit 20 --offset 0
```

## Debugging JSB Issues

### Call Fails (success=false)

```bash
# Find failing calls
rax db search-bridge --success false

# Check specific method
rax db search-bridge --method "payOrder" --success false
```

Look at the response data for error codes/messages.

### Call Never Returns

If a JSB call seems to hang:
1. Check if the call was actually made: `rax db search-bridge --method "targetMethod"`
2. If found with no response: native implementation may be blocking
3. If not found: JS code may not be reaching the call

### Unexpected Response

```bash
# Find the call and examine the response
rax db search-bridge --method "getConfig" --limit 5
```

Check response fields match the expected interface.

## Common Bridge Methods

Bridge method names are app-specific. Common patterns:
- `getUserInfo`, `getDeviceInfo` — Device/user data
- `openPage`, `closePage` — Navigation
- `showToast`, `showDialog` — UI feedback
- `setStorage`, `getStorage` — Persistent storage
- `trackEvent` — Analytics
