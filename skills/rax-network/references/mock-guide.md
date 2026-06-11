# Mock Guide

## Response Mock Basics

Mocks intercept matching requests and return a fixed response body.

### Add a Single Mock

```bash
rax network add-mock "/api/user/profile" '{"name":"test_user","level":99}'
```

- First argument: request **path** (substring match)
- Second argument: response **body** (JSON string)

### Add Multiple Mocks (Batch)

```bash
rax network add-mock --items '[
  {"path": "/api/feed", "responseBody": "{\"items\": [], \"hasMore\": false}"},
  {"path": "/api/user/profile", "responseBody": "{\"name\": \"mock_user\"}"},
  {"path": "/api/cart/count", "responseBody": "{\"count\": 3}"}
]'
```

### Remove Mocks

```bash
# Remove single
rax network remove-mock "/api/user/profile"

# Remove multiple
rax network remove-mock --paths '["/api/feed", "/api/user/profile"]'
```

## Common Mock Patterns

### Empty List Response
```bash
rax network add-mock "/api/feed" '{"items":[],"hasMore":false,"cursor":""}'
```

### Error Response
```bash
rax network add-mock "/api/order/submit" '{"code":500,"message":"Server Error"}'
```

### Feature Flag Override
```bash
rax network add-mock "/api/config/feature" '{"new_feature_enabled":true}'
```

## Verification

After adding a mock, verify it works:

1. Add the mock
2. Trigger the request on device (open page, tap button)
3. Check the response in network capture:
   ```bash
   rax network search --url_pattern "user/profile" --limit 1
   ```
4. The response body should match your mock

## Tips

- Path matching is **substring-based**: `"/api/feed"` matches any URL containing `/api/feed`
- If multiple mocks match, all apply — use specific paths to avoid conflicts
- Mocks persist until removed or device disconnects
- Use `rax network remove-mock` to clean up after testing
