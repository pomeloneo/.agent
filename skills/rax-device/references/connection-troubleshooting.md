# Connection Troubleshooting

## Diagnostic Flow

When a device won't connect, follow this order:

### Step 1: Check Daemon

```bash
rax daemon status
```

- If not running → `rax daemon start`
- If running but unresponsive → `rax daemon restart`
- Daemon HTTP endpoint: `http://127.0.0.1:2680`
- PID file: `~/.rax-mcp/daemon.pid`
- Log file: `~/.rax-mcp/daemon.log`

### Step 2: Check Device Visibility

```bash
# For Android
adb devices

# For iOS
# Devices should appear when plugged in via USB
```

If `adb devices` shows nothing:
- USB cable is charge-only (use a data cable)
- USB debugging not enabled on device (Settings → Developer Options → USB Debugging)
- ADB not installed or not in PATH

### Step 3: Scan and Check

```bash
rax device scan
rax device list
```

**Device shows `📱` (USB only) but not `✅` (RAX connected):**
- App is not running → Launch the App
- App doesn't have RAX SDK → SDK integration required
- App is running but RAX service not started → Force-close and relaunch
- Wrong port → Check port table below

**Device doesn't show at all:**
- Re-plug USB cable
- For iOS: tap "Trust" on the device trust dialog
- For Android: re-authorize USB debugging prompt
- Try `rax device scan` again

### Step 4: Check Ports

RAX apps listen on fixed debug ports:

| App | Port |
|-----|------|
| Default/RAX | 2692 |
| TikTok | 2693 |
| Musically | 2694 |
| M2 | 2695 |
| Seller | 2696 |
| Pro | 2697 |
| Whee | 2698 |
| Drama | 2699 |

If port is occupied by another process, connection will fail. Check with:

```bash
# Android: check if port is listening
adb shell "nc -z 127.0.0.1 2693 && echo open || echo closed"
```

### Step 5: Check Logs

```bash
# View daemon log for connection errors
tail -100 ~/.rax-mcp/daemon.log
```

Common errors:
- `ECONNREFUSED` — App not running or port wrong
- `ETIMEDOUT` — Network unreachable (remote connection)
- `ADB connection unusable` — Known issue with `@yume-chan/adb`; daemon will retry with fresh connection

## Platform-Specific Issues

### Android

| Issue | Solution |
|-------|----------|
| `adb devices` shows "unauthorized" | Re-authorize USB debugging on device |
| `adb devices` shows "offline" | Re-plug USB, or `adb kill-server && adb start-server` |
| Connection drops frequently | Use a shorter/better USB cable |
| Remote ADB fails | Ensure `adb tcpip 5555` was run while USB-connected |

### iOS

| Issue | Solution |
|-------|----------|
| Device not detected | Trust the computer on device |
| `lockdownd` errors | Restart `usbmuxd`: `sudo killall usbmuxd` |
| Connection timeout | Ensure device is unlocked |
| idb not found (for UI tools) | Install idb: `brew install idb-companion` |

## Environment Check

Run `rax doctor` to check:
- Node.js version (requires >=20.0.0)
- ADB availability and version
- Daemon status
- USB device detection
- Network connectivity
