# Event Tracking Guide

## Overview

Verify event tracking (埋点) implementation by capturing and searching event logs from the device.

## Verification Workflow

### Step 1: Enable Capture

```bash
rax network capture app_log
```

### Step 2: Perform the Action

Trigger the event on the device — tap a button, view a page, complete a purchase, etc.

### Step 3: Search for the Event

```bash
# Exact event name
rax db search-events --event_name "product_click"

# Fuzzy search
rax db search-events --event_name_pattern "click"

# With session filter
rax db search-events --event_name_pattern "click" --session_id <id>
```

### Step 4: Verify Event Data

Check the returned event data for:
- Event name matches spec
- Required parameters are present
- Parameter values are correct
- Timestamp is reasonable

## Search Tips

### Find All Events for a Feature

```bash
# Search by event name prefix
rax db search-events --event_name_pattern "product_"
# → product_view, product_click, product_add_cart, product_purchase
```

### Filter by Session

Isolate events from a specific test run:

```bash
# List sessions with event data
rax db sessions --has_app_log true

# Search within that session
rax db search-events --event_name_pattern "click" --session_id <id>
```

### Paginate Results

```bash
# First 20
rax db search-events --event_name_pattern "click" --limit 20 --offset 0

# Next 20
rax db search-events --event_name_pattern "click" --limit 20 --offset 20
```

## Common Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| No events found | Capture not enabled | `rax network capture app_log` |
| No events found | Action before capture | Re-enable capture, repeat action |
| Wrong event name | Typo in search | Try broader pattern |
| Missing parameters | SDK not sending them | Check event SDK integration code |
| Duplicate events | Multiple triggers | Check if event fires on correct lifecycle |
