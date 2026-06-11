# Remote Connection Guide

Two methods to connect to a device over the network.

## Method 1: ADB TCP (Android Only)

Best for: devices on the same WiFi or connected via VPN.

### Setup (One-Time)

While the device is connected via USB:

```bash
# Enable ADB TCP mode on the device
adb tcpip 5555

# Get device IP (Settings → About Phone → IP address)
# Or:
adb shell ip route | grep wlan0
```

You can now unplug the USB cable.

### Connect

```bash
rax device connect <device_ip>
# Example:
rax device connect 192.168.1.100

# Custom port:
rax device connect 192.168.1.100 --port 5555
```

Once connected, the device appears in `rax device list` and works identically to USB — all features available (DB, network capture, etc.).

### Disconnect

```bash
rax device disconnect
```

### Limitations

- Device must remain on the same network
- ADB TCP mode resets after device reboot
- Slightly higher latency than USB
- Only works for Android

## Method 2: RAX WebSocket (Cross-Platform)

Best for: direct connection without ADB, or when ADB is not available.

### Requirements

- Device ID (DID)
- Device IP address
- RAX port number

### Connect

```bash
rax device connect-rax <device_id> --ip <device_ip> --port <rax_port>

# Example:
rax device connect-rax abc123def --ip 10.0.0.50 --port 2693
```

### When to Use

| Scenario | Recommended Method |
|----------|-------------------|
| Android on same WiFi | ADB TCP |
| Android via VPN | ADB TCP |
| iOS remote | RAX WebSocket |
| No ADB available | RAX WebSocket |
| Cloud device | RAX WebSocket |
