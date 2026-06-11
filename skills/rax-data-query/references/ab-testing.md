# AB Testing Guide

## Overview

RAX captures AB test (Settings) configurations from the device and caches them in a local database. You can search, mock, and verify AB experiments.

## Finding AB Settings

### By Key

```bash
# Fuzzy match on key name
rax ab search --key_pattern "feed"
rax ab search --key_pattern "live_room"
rax ab search --key_pattern "new_feature"
```

### By Value

```bash
# Combine key and value search
rax ab search --key_pattern "feed" --value_pattern "true"
```

### Full Summary

```bash
# All settings (can be large)
rax ab search
```

### Fresh Data

By default, RAX uses cached data from the database. Force a fresh fetch:

```bash
rax ab search --key_pattern "feed" --refresh true
```

**When to use `--refresh`:**
- First time querying (DB may be empty)
- After changing settings on the device
- After app restart
- When cached data seems stale

## Mocking AB Settings

### Complete Workflow

```bash
# 1. Find the setting key
rax ab search --key_pattern "new_feature"
# Output shows: key="new_feature_enabled", value="false"

# 2. Enable mock system (required before any mock takes effect)
rax ab mock-enable true

# 3. Set mock value (positional: <propertyName> <mockValue>)
rax ab mock new_feature_enabled true

# 4. Verify: restart the page or re-trigger the feature on device
# Check if behavior changed

# 5. Clean up when done
rax ab clear           # Remove all mocks
rax ab mock-enable false   # Disable mock system
```

### Mock Multiple Settings

```bash
rax ab mock-enable true
rax ab mock feature_a true
rax ab mock feature_b variant_2
rax ab mock feed_style grid
```

### Mock Value Types

Values are strings. Common patterns:

| Type | Example |
|------|---------|
| Boolean | `"true"`, `"false"` |
| Number | `"42"`, `"1.5"` |
| String | `"variant_a"` |
| JSON | `"{\"key\":\"value\"}"` |

## Lifecycle

```
Mock-enable OFF → Mocks have no effect (even if set)
Mock-enable ON  → All set mocks take effect
ab clear        → Removes all mock values (mock-enable stays ON)
Mock-enable OFF → Back to normal
```

## Tips

- Always enable mock system (`mock-enable true`) before setting mocks
- Always clean up mocks after testing to avoid affecting other debug sessions
- Use `--refresh true` if you changed settings outside of RAX
- AB key naming typically follows patterns: `feature_name_enabled`, `module_config`, `experiment_group`
