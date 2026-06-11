# Weak Network Simulation

## Purpose

Simulate slow network conditions to test loading states, timeouts, retry logic, and user experience under poor connectivity.

## Basic Usage

```bash
# Add 3-second delay to feed API
rax network add-weak "/api/feed" 3000
```

## Batch Configuration

```bash
rax network add-weak --items '[
  {"id": "1", "path": "/api/feed", "desc": "Feed loading", "weakNetworkDelay": 3000},
  {"id": "2", "path": "/api/search", "desc": "Search delay", "weakNetworkDelay": 5000},
  {"id": "3", "path": "/api/image", "desc": "Image slow load", "weakNetworkDelay": 8000}
]'
```

Fields:
- `id` — Unique identifier for the rule
- `path` — URL path to match (substring)
- `desc` — Description for reference
- `weakNetworkDelay` — Delay in milliseconds

## Remove Rules

```bash
# Remove single
rax network remove-weak "/api/feed"

# Remove multiple
rax network remove-weak --paths '["/api/feed", "/api/search"]'
```

## Testing Strategies

### Loading State Verification
```bash
# Add enough delay to see loading UI (3-5 seconds)
rax network add-weak "/api/feed" 4000
# → Open page → Verify skeleton/spinner appears
# → Wait → Verify content loads correctly
```

### Timeout Testing
```bash
# Add delay longer than app's timeout threshold
rax network add-weak "/api/order" 30000
# → Trigger request → Verify timeout error handling
```

### Sequential Request Testing
```bash
# Delay only the second request in a chain
rax network add-weak "/api/detail" 5000
# → Open list → Tap item → Verify detail page handles delay
```

## Tips

- Delay is added on top of actual network time
- Remove rules after testing to avoid confusing subsequent debug sessions
- Combine with `rax device screenshot` to capture loading states
