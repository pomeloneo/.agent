# Network Query Guide (via DB)

## Overview

`rax db search-network` queries network requests from the local database. This is the same data as `rax network search` but accessed through the DB query interface.

## Usage

```bash
rax db search-network --url_pattern "api/feed" --method GET --status_code 200
```

## Available Filters

| Flag | Type | Description |
|------|------|-------------|
| `--url_pattern` | string | URL substring match |
| `--method` | string | HTTP method (GET, POST, PUT, DELETE, PATCH) |
| `--status_code` | number | HTTP status code |
| `--success` | boolean | Success/failure |
| `--session_id` | string | Filter by session |
| `--limit` | number | Max results (default 50) |
| `--offset` | number | Skip N results |
| `--order_by` | string | Sort: `time` (default) or `duration` |
| `--fields` | string | Select specific fields |

## Useful Queries

### Find Failed Requests

```bash
rax db search-network --success false
rax db search-network --status_code 500
rax db search-network --status_code 404
```

### Find Slow Requests

```bash
# Sort by duration (slowest first)
rax db search-network --order_by duration --limit 10
```

### Find Requests by Session

```bash
# List sessions with network data
rax db sessions --has_network_requests true

# Query within session
rax db search-network --session_id <id> --url_pattern "api"
```

### Select Specific Fields

```bash
# Only URL, status, and duration
rax db search-network --url_pattern "api" --fields "url,status,duration"
```

## When to Use DB Query vs Network Search

| Feature | `rax network search` | `rax db search-network` |
|---------|---------------------|------------------------|
| Data source | Same DB | Same DB |
| Session filter | ✅ | ✅ |
| Field selection | ✅ | ✅ |
| Functionality | Identical | Identical |

They are equivalent. Use whichever feels more natural in context.
