# AP_Test

## Introduction

The primary objective of this project was to transform the Raspberry Pi 4 into a wireless Access Point (AP) using a custom-built RDK-B image. After booting the device successfully, the first validation step was to verify whether the Access Point was functioning correctly.

This test ensures that:

- The wireless interface is operational.
- The SSID is being broadcast.
- Clients can detect the network.
- Clients can authenticate successfully.
- Multiple devices can connect simultaneously.

A successful AP test confirms that the wireless subsystem, hostapd, and the custom configurations are functioning as expected.

---

# Objective

The objectives of this test are:

- Verify that the Raspberry Pi starts in AP mode.
- Verify SSID broadcasting.
- Verify WPA2 authentication.
- Verify client association.
- Verify multiple client connections.
- Ensure the AP remains stable under multiple connections.

---

# Test Environment

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| OS | Custom Yocto + RDK-B Image |
| Wireless Interface | wlan0 |
| AP Software | hostapd |
| DHCP Server | dnsmasq |
| Authentication | WPA2-PSK |

---

# Access Point Configuration

The following configuration was used.

| Parameter | Value |
|-----------|-------|
| SSID | RDKB_AP |
| Interface | wlan0 |
| Driver | nl80211 |
| Channel | 6 |
| Band | 2.4 GHz |
| Authentication | WPA2 |
| Password | 12345678 |

---

# Test Workflow

```text
Boot Raspberry Pi

      │

      ▼

Start hostapd

      │

      ▼

Broadcast SSID

      │

      ▼

Scan Using Client Device

      │

      ▼

Connect to AP

      │

      ▼

Authenticate

      │

      ▼

Association Successful

      │

      ▼

Verify Client Connection
```

---

# Test Procedure

## Step 1 – Boot the Raspberry Pi

Power on the Raspberry Pi.

Wait until:

- Linux boots completely.
- systemd starts all services.
- hostapd becomes active.
- dnsmasq starts successfully.

---

## Step 2 – Verify hostapd

Login to the Raspberry Pi.

Check service status.

```bash
systemctl status hostapd
```

Expected output:

```text
Active: active (running)
```

---

## Step 3 – Verify Wireless Interface

Check wlan0.

```bash
ip addr show wlan0
```

Expected:

```text
wlan0

192.168.10.1/24
```

---

## Step 4 – Scan for SSID

Using a mobile phone or laptop:

Open

```text
Wi-Fi Settings
```

Scan available wireless networks.

Expected result:

```text
RDKB_AP
```

should appear in the list.

---

## Step 5 – Connect to AP

Select

```text
RDKB_AP
```

Enter the password.

```text
12345678
```

Expected result:

Connection should be established successfully.

---

## Step 6 – Verify Authentication

hostapd authenticates the client using WPA2.

Successful authentication indicates:

- Correct passphrase
- Successful key exchange
- Client association

---

## Step 7 – Verify Client Association

On Raspberry Pi:

```bash
iw dev wlan0 station dump
```

Example output:

```text
Station xx:xx:xx:xx:xx:xx

inactive time:

rx bytes:

tx bytes:

signal:
```

Each connected client appears as one station.

---

## Step 8 – Verify Connected Clients

Check connected stations.

```bash
iw dev wlan0 station dump
```

Expected:

One entry per connected device.

---

# Multiple Client Test

Several devices were connected simultaneously.

Devices used:

- Android Phone
- Laptop
- Tablet
- Another Smartphone
- Desktop with Wi-Fi Adapter
- Additional Mobile Device

Total devices connected:

```text
6
```

All devices connected successfully without disconnecting.

---

# Verify Using hostapd

Display associated stations.

```bash
hostapd_cli all_sta
```

This command displays:

- Connected MAC Address
- Signal Strength
- TX Rate
- RX Rate
- Connection Time

---

# Verify Service Status

Confirm hostapd is running.

```bash
systemctl status hostapd
```

Expected:

```text
Active: active (running)
```

---

# Verify Logs

Display hostapd logs.

```bash
journalctl -u hostapd
```

Typical messages:

```text
AP-ENABLED

AP-STA-CONNECTED

AP-STA-DISCONNECTED
```

These messages indicate client association and disconnection events.

---

# Expected Results

| Test | Expected Result |
|-------|-----------------|
| Raspberry Pi Boots | Pass |
| hostapd Starts | Pass |
| SSID Broadcast | Pass |
| WPA2 Authentication | Pass |
| Client Association | Pass |
| Multiple Clients | Pass |
| AP Stability | Pass |

---

# Actual Results

| Test | Result |
|-------|---------|
| SSID Visible | Pass |
| Authentication Successful | Pass |
| Device Connected | Pass |
| AP Stable | Pass |
| Six Devices Connected | Pass |

---

# Problems Encountered

## SSID Not Visible

Cause:

hostapd started before wlan0 was configured.

Solution:

Moved interface configuration into:

```text
rdkb-network.sh
```

---

## Authentication Failure

Cause:

Incorrect hostapd configuration.

Solution:

Verified:

```text
SSID

Password

Driver

Channel
```

After rebuilding the image, authentication worked correctly.

---

## AP Service Failed

Cause:

Configuration file missing.

Solution:

Verified:

```text
/etc/hostapd.conf
```

was included in the image.

---

# Commands Used

Check hostapd:

```bash
systemctl status hostapd
```

Check wlan0:

```bash
ip addr show wlan0
```

Display connected stations:

```bash
iw dev wlan0 station dump
```

Display hostapd station information:

```bash
hostapd_cli all_sta
```

View logs:

```bash
journalctl -u hostapd
```

---

# Observations

During testing:

- The SSID became visible immediately after boot.
- Clients detected the AP without delay.
- Authentication completed successfully.
- Association was stable.
- No unexpected disconnects occurred.
- Six client devices remained connected simultaneously.

The AP remained operational throughout the testing period.

---

# Conclusion

The Access Point functionality was successfully validated. The Raspberry Pi automatically started hostapd during boot, broadcast the configured SSID (`RDKB_AP`), authenticated clients using WPA2, and supported simultaneous connections from multiple devices. This confirmed that the wireless Access Point component of the custom RDK-B image was functioning correctly and was ready for subsequent DHCP, NAT, Internet sharing, and performance testing.