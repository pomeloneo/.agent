---
name: rax-device
description: |
  Device connection, management, and troubleshooting for RAX debugging.

  Trigger Scenarios:
  - User wants to connect a device (USB or remote)
  - Device is not showing up or connection fails
  - User needs to switch between multiple devices
  - User asks about RAX daemon status or environment issues
  - User mentions "rax device", "connect", "scan", "device not found"
---

# RAX Device Management

Guide users through device connection, multi-device management, and connection troubleshooting using the RAX CLI.

## Prerequisites

CLI: `rax` (at `<rax-monorepo>/apps/rax-cli/bin/rax.sh`)

## Workflows

### 1. First-Time Setup

Check environment health first, then connect.

```bash
# Step 1: Check environment
rax doctor

# Step 2: Ensure daemon is running
rax daemon status
# If not running:
rax daemon start

# Step 3: Scan for devices
rax device scan

# Step 4: List detected devices
rax device list

# Step 5: Verify connection
rax device info
```

### 2. USB Connection (Standard)

```bash
# Scan USB devices (auto-detects Android & iOS)
rax device scan

# List devices: ✅ = RAX connected, 📱 = USB only
rax device list

# If device shows 📱 but not ✅:
# → App is not running or RAX SDK not integrated
# → Launch the App on device, then re-scan:
rax device scan
```

### 3. Remote Connection

Two modes available. Choose based on the situation:

**ADB TCP (Android, same network/VPN):**
```bash
# Device must have adb tcpip enabled first
rax device connect <device_ip>
# Custom port:
rax device connect <device_ip> --port 5555
```

**RAX WebSocket (direct, no ADB):**
```bash
# Need device_id + ip + port
rax device connect-rax <device_id> --ip <ip> --port <port>
```

See [Remote Connection Guide](references/remote-connection.md) for detailed setup.

### 4. Multi-Device Management

```bash
# List all devices with tags (D1, D2, ...)
rax device list

# Select target device by tag or device_id
rax device select D2

# All subsequent commands operate on selected device
rax device info
rax device screenshot
```

### 5. Quick Actions

```bash
# Screenshot (default saves to ~/.rax-mcp/screenshot/)
rax device screenshot
rax device screenshot --output ./screenshot.png
rax device screenshot --base64

# Clipboard
rax device clipboard                    # Read
rax device clipboard "text to copy"     # Write

# Keep screen awake
rax device keep-alive true

# Open app schema route
rax device open-schema "sslocal://feed"

# Disconnect all devices
rax device disconnect
```

## Troubleshooting

When a user reports connection issues, follow this decision tree:

```
Device not found?
├── Is daemon running? → rax daemon status / rax daemon start
├── Is USB cable connected? → Check physical connection
├── Is App running on device? → Launch App, then rax device scan
├── Is RAX SDK integrated? → Device shows 📱 but not ✅
├── Android ADB issue? → Check adb devices, check USB debugging enabled
├── iOS trust issue? → Trust computer on device, check lockdown
└── Port conflict? → See port table in references
```

See [Connection Troubleshooting](references/connection-troubleshooting.md) for detailed diagnostics.

## Command Reference

| Command | Description |
|---------|-------------|
| `rax doctor` | Check environment and connectivity |
| `rax daemon start/stop/status/restart` | Manage background daemon |
| `rax device scan` | Rescan USB devices |
| `rax device list` | List all detected devices |
| `rax device select <device>` | Select active device (D1/D2/... or device_id) |
| `rax device connect <host>` | Remote connect via ADB TCP |
| `rax device connect-rax <device_id> --ip <ip> --port <port>` | Remote connect via RAX WebSocket |
| `rax device info` | Get device & app info |
| `rax device screenshot [--output <path>] [--base64]` | Take screenshot (default saves to ~/.rax-mcp/screenshot/) |
| `rax device clipboard [content]` | Read/write clipboard |
| `rax device keep-alive <enable>` | Keep screen awake (true/false) |
| `rax device open-schema <schema>` | Open schema URL |
| `rax device disconnect` | Disconnect all devices |

## Global Flags

All commands support:
- `--device <id>` — Target a specific device without selecting
- `--json` — Output as JSON
- `--format <text|json|table|csv>` — Output format

## References

- [Connection Troubleshooting](references/connection-troubleshooting.md) — Diagnose connection failures
- [Remote Connection Guide](references/remote-connection.md) — ADB TCP vs RAX WebSocket setup
- [Multi-Device Guide](references/multi-device.md) — Working with multiple devices
