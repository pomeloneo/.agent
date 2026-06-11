# Multi-Device Guide

## Working with Multiple Devices

When multiple devices are connected, RAX assigns tags (D1, D2, ...) for easy selection.

### List Devices

```bash
rax device list
```

Output example:
```
D1  ✅  Pixel 7         Android 14   com.ss.android.ugc.trill
D2  📱  iPhone 15 Pro   iOS 17.4     (USB only)
D3  ✅  Galaxy S24      Android 14   com.ss.android.ugc.trill
```

### Select Active Device

```bash
# By tag
rax device select D1

# By device_id
rax device select abc123def456
```

All subsequent commands target the selected device.

### Per-Command Device Override

Use `--device` flag to target a specific device without changing the selection:

```bash
# Screenshot from D2 without switching
rax device screenshot --device D2

# Check info of D3
rax device info --device D3
```

### Typical Multi-Device Workflows

**Compare behavior across devices:**
```bash
rax device select D1
rax device screenshot --output ./pixel.png

rax device select D3
rax device screenshot --output ./galaxy.png
```

**Debug on one, reference another:**
```bash
# Select primary debug target
rax device select D1

# All debug commands go to D1
rax network capture network
rax network search --url_pattern "api/feed"

# Quick check on D3 without switching
rax device info --device D3
```
