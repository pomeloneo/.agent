# Request Rewrite Guide

## Overview

Request rewrite modifies HTTP requests in flight before they reach the server. Unlike Mocks (which replace responses), rewrite changes the request itself — URL, headers, query params, or body.

## Basic Usage

### Redirect to a Different Host

```bash
rax network rewrite --items '[{
  "match": {"host": "api.example.com"},
  "rewrite": {"host": "api-staging.example.com"}
}]'
```

### Add/Override Headers

```bash
rax network rewrite --items '[{
  "match": {"path": "/api/feed"},
  "rewrite": {
    "headers": {
      "X-Debug": "true",
      "X-AB-Override": "experiment_v2"
    }
  }
}]'
```

### Modify Query Parameters

```bash
rax network rewrite --items '[{
  "match": {"path": "/api/search"},
  "rewrite": {
    "queryParams": {"debug": "1", "region": "US"},
    "deleteQueryParams": ["cache_key"]
  }
}]'
```

### Switch Protocol

```bash
rax network rewrite --items '[{
  "match": {"host": "api.example.com", "protocol": "https"},
  "rewrite": {"protocol": "http"}
}]'
```

### PPE Environment

PPE mode adds `x-tt-env=<env>` and `x-use-ppe=1` headers to all requests:

```bash
rax network rewrite --ppe boe
```

### Clear All Rules

```bash
rax network rewrite --clear
```

## Match Conditions

| Field | Type | Description |
|-------|------|-------------|
| `protocol` | string | Match protocol (http/https) |
| `host` | string | Match host (exact, LIKE, regex) |
| `port` | number | Match port |
| `path` | string | Match path (exact, LIKE, regex) |
| `queryParams` | object | Match query parameters |
| `headers` | object | Match request headers |

Host and path matching support:
- **Exact match**: `"api.example.com"`
- **LIKE match**: `"%example%"` (SQL-style wildcards)
- **Regex match**: depends on device SDK implementation

## Rewrite Targets

| Field | Type | Description |
|-------|------|-------------|
| `protocol` | string | New protocol |
| `host` | string | New host |
| `port` | number | New port |
| `path` | string | New path |
| `queryParams` | object | Add/override query params |
| `deleteQueryParams` | array | Remove query params by key |
| `headers` | object | Add/override headers |
| `deleteHeaders` | array | Remove headers by key |
| `requestBody` | string | Replace request body |

## Multiple Rules

```bash
rax network rewrite --items '[
  {
    "match": {"host": "api.example.com"},
    "rewrite": {"host": "api-staging.example.com"}
  },
  {
    "match": {"path": "/api/feed"},
    "rewrite": {"headers": {"X-Debug": "true"}}
  }
]'
```

Rules are evaluated in order. All matching rules apply.

## Common Scenarios

| Scenario | Match | Rewrite |
|----------|-------|---------|
| Redirect to staging | `host: "api.example.com"` | `host: "staging.example.com"` |
| Force HTTP | `protocol: "https"` | `protocol: "http"` |
| Add debug header | `path: "/api"` | `headers: {"X-Debug": "1"}` |
| Override region | `path: "/api/feed"` | `queryParams: {"region": "JP"}` |
| Remove tracking params | `path: "/api"` | `deleteQueryParams: ["trace_id"]` |

## Tips

- Rewrite rules persist until cleared or device disconnects
- Use `rax network rewrite --clear` to clean up
- Combine with `rax network search` to verify rewritten requests
- When passing `--items`, do NOT also pass `--clear`
