# Multiple_Clients

## Introduction

One of the important requirements of an Access Point is its ability to support multiple wireless clients simultaneously without affecting network stability or Internet connectivity. After successfully validating the Access Point, DHCP server, and NAT functionality, the Raspberry Pi was tested with multiple client devices connected at the same time.

The purpose of this test was to evaluate whether the Raspberry Pi could:

- Accept multiple wireless connections.
- Assign unique IP addresses to every client.
- Provide Internet connectivity to all connected devices.
- Maintain stable wireless communication.
- Continue operating without crashes or service failures.

This document describes the complete testing procedure, observations, commands used, and the final results.

---

# Objective

The objectives of this test are:

- Verify simultaneous client connectivity.
- Verify DHCP lease allocation for multiple clients.
- Verify Internet access for all clients.
- Verify AP stability under multiple connections.
- Verify hostapd client association.
- Ensure there are no unexpected disconnections.

---

# Test Environment

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| Operating System | Custom Yocto + RDK-B Image |
| Access Point | hostapd |
| DHCP Server | dnsmasq |
| NAT | iptables |
| Wireless Interface | wlan0 |

---

# Client Devices Used

The following devices were connected during testing.

| Device | Connection Status |
|---------|-------------------|
| Android Phone | Connected |
| Laptop | Connected |
| Tablet | Connected |
| Smartphone | Connected |
| Desktop (Wi-Fi Adapter) | Connected |
| Additional Mobile Phone | Connected |

Total Connected Devices:

```text
6
```

---

# Test Workflow

```text
Boot Raspberry Pi

        │

        ▼

Start Access Point

        │

        ▼

Client 1 Connects

        │

        ▼

DHCP IP Assigned

        │

        ▼

Client 2 Connects

        │

        ▼

DHCP IP Assigned

        │

        ▼

...

        │

        ▼

Six Clients Connected

        │

        ▼

Internet Access Verified

        │

        ▼

Connection Stability Verified
```

---

# Test Procedure

## Step 1 – Boot Raspberry Pi

Power on the Raspberry Pi.

Wait until:

- hostapd starts.
- dnsmasq starts.
- rdkb-network.service completes successfully.

---

## Step 2 – Verify Access Point

Check:

```bash
systemctl status hostapd
```

Expected:

```text
Active: active (running)
```

---

## Step 3 – Connect Clients

Connect each device one after another.

For every device:

1. Scan Wi-Fi.
2. Select

```text
RDKB_AP
```

3. Enter password

```text
12345678
```

4. Wait for connection.

Repeat until all six devices are connected.

---

## Step 4 – Verify DHCP Assignment

Each client automatically received an IP address.

Example:

| Device | Assigned IP |
|----------|-------------|
| Phone 1 | 192.168.10.10 |
| Laptop | 192.168.10.11 |
| Tablet | 192.168.10.12 |
| Phone 2 | 192.168.10.13 |
| Desktop | 192.168.10.14 |
| Phone 3 | 192.168.10.15 |

No duplicate IP addresses were observed.

---

## Step 5 – Verify Connected Stations

On Raspberry Pi:

```bash
iw dev wlan0 station dump
```

Expected:

One station entry for every connected client.

Example:

```text
Station xx:xx:xx:xx:xx:01

Station xx:xx:xx:xx:xx:02

Station xx:xx:xx:xx:xx:03

Station xx:xx:xx:xx:xx:04

Station xx:xx:xx:xx:xx:05

Station xx:xx:xx:xx:xx:06
```

---

## Step 6 – Verify DHCP Leases

Display lease file.

```bash
cat /var/lib/misc/dnsmasq.leases
```

Expected:

Six lease entries.

Each client should have:

- MAC Address
- Assigned IP
- Lease Time
- Hostname

---

## Step 7 – Verify Internet Connectivity

Each connected device performed:

- Web browsing
- Google Search
- YouTube access
- Package downloads

All devices accessed the Internet successfully.

---

## Step 8 – Verify Service Stability

Monitor services.

```bash
systemctl status hostapd
```

```bash
systemctl status dnsmasq
```

Expected:

```text
Active: active (running)
```

No service restart occurred during testing.

---

# Commands Used

Check connected stations:

```bash
iw dev wlan0 station dump
```

Display DHCP leases:

```bash
cat /var/lib/misc/dnsmasq.leases
```

Check hostapd:

```bash
systemctl status hostapd
```

Check dnsmasq:

```bash
systemctl status dnsmasq
```

View hostapd logs:

```bash
journalctl -u hostapd
```

---

# Expected Results

| Test | Expected Result |
|-------|-----------------|
| Six Clients Connected | Pass |
| Unique IP Address | Pass |
| DHCP Working | Pass |
| Internet Working | Pass |
| Stable Connections | Pass |
| No Unexpected Disconnects | Pass |

---

# Actual Results

| Test | Result |
|-------|---------|
| Six Clients Connected Successfully | Pass |
| DHCP Assigned Unique IPs | Pass |
| Internet Available to All Devices | Pass |
| hostapd Stable | Pass |
| dnsmasq Stable | Pass |
| NAT Stable | Pass |

---

# Performance Observations

During testing:

- All clients connected within a few seconds.
- DHCP response time was quick.
- No authentication failures occurred.
- Internet browsing remained smooth.
- No packet loss was observed during normal usage.
- The Raspberry Pi remained responsive throughout the test.

---

# Problems Encountered

## Unable to View Connected Devices

Initially, there was a need to determine which clients were currently connected to the Access Point.

### Solution

The following command was used:

```bash
iw dev wlan0 station dump
```

This displayed all associated wireless stations.

---

## Mapping IP Address to Connected Device

The `iw` command only displayed MAC addresses and wireless statistics. To identify which IP address belonged to each device, the DHCP lease file was examined.

```bash
cat /var/lib/misc/dnsmasq.leases
```

By comparing the MAC addresses in the lease file with the output of `iw`, the assigned IP address for each connected client was identified.

---

## Viewing Active Wireless Clients

To verify all associated clients directly through hostapd, the following command was used:

```bash
hostapd_cli all_sta
```

This displayed detailed information about every connected station, including:

- MAC Address
- Signal Strength
- TX/RX Statistics
- Connection Duration

---

# Lessons Learned

This testing phase demonstrated that:

- `hostapd` efficiently manages multiple wireless clients.
- `dnsmasq` successfully allocates unique IP addresses.
- `iptables` correctly forwards traffic for all connected devices.
- The Raspberry Pi 4 hardware is capable of operating as a stable wireless Access Point for multiple simultaneous users.

The combination of these components resulted in a reliable and functional embedded networking solution.

---

# Conclusion

The multiple client connectivity test was successfully completed. A total of **six wireless devices** were connected simultaneously to the `RDKB_AP` Access Point. Each client received a unique IP address through DHCP, accessed the Internet via NAT, and remained connected without interruption throughout the testing period. The Raspberry Pi maintained stable operation, confirming that the custom Yocto-built RDK-B image can reliably support multiple concurrent wireless users.