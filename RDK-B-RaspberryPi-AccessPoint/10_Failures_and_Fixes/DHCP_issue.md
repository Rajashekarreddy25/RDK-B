# DHCP_issue

## Introduction

After successfully configuring **hostapd** and **dnsmasq**, the Raspberry Pi was capable of broadcasting the Wi-Fi network and assigning IP addresses to connected devices. However, during development, DHCP did not work correctly on the first few implementations.

Sometimes clients failed to obtain an IP address, while in other cases the DHCP service appeared to be running but clients remained stuck at **"Obtaining IP Address"**.

This document describes the DHCP-related issues encountered during the project, the debugging process followed, and the final solution that resulted in reliable IP address assignment.

---

# Problem Description

During early testing, the following behavior was observed:

- Wi-Fi SSID was visible.
- Devices successfully authenticated.
- DHCP requests timed out.
- Some devices disconnected automatically.
- No IP address was assigned.

Although the Access Point appeared operational, the network was unusable because clients could not obtain valid network configuration.

---

# Symptoms

```text
✔ Raspberry Pi Booted

✔ Wi-Fi SSID Visible

✔ Device Connected

✘ Obtaining IP Address...

✘ DHCP Timeout

✘ No Gateway Assigned

✘ No Internet Access
```

---

# Initial Investigation

The first step was to verify that the wireless interface had been configured correctly.

```bash
ip addr show wlan0
```

Expected Output

```text
inet 192.168.10.1/24
```

Initially, the interface either had:

```text
No IP Address
```

or

```text
Incorrect Network Configuration
```

Since the DHCP server distributes addresses based on the interface configuration, this prevented successful lease assignment.

---

# Verify dnsmasq Service

Check service status.

```bash
systemctl status dnsmasq
```

Expected

```text
Active: active (running)
```

The service was running, but clients still failed to receive IP addresses.

---

# Verify Interface Binding

The configuration was inspected.

```bash
cat /etc/dnsmasq.conf
```

Particular attention was given to:

```text
interface=wlan0
```

Without binding dnsmasq to the wireless interface, DHCP packets would not be processed correctly.

---

# Verify Static IP Configuration

The custom startup script was checked.

```bash
cat /usr/bin/rdkb-network.sh
```

The script contained:

```bash
ip addr flush dev wlan0

ip addr add 192.168.10.1/24 dev wlan0

ip link set wlan0 up
```

Initially, the order of commands was not always correct during testing.

The corrected sequence ensured:

1. Interface enabled.
2. Previous addresses removed.
3. Static IP assigned.

---

# Verify DHCP Requests

System logs were inspected.

```bash
journalctl -u dnsmasq
```

Typical successful messages included:

```text
DHCPDISCOVER

DHCPOFFER

DHCPREQUEST

DHCPACK
```

Before the fix, these messages were either missing or incomplete.

---

# Root Cause Analysis

The investigation revealed several contributing factors:

- `wlan0` was not always configured before `dnsmasq` started.
- The static IP address was sometimes missing.
- Service startup order was inconsistent.
- DHCP depends on a correctly configured interface.

Although the DHCP server itself was functioning, it could not allocate addresses because the wireless interface was not fully initialized.

---

# Solution

The startup sequence was redesigned.

The custom network initialization script:

```text
rdkb-network.sh
```

was updated to configure the interface before DHCP service initialization.

The sequence became:

```text
Bring wlan0 UP

↓

Flush Existing Address

↓

Assign Static IP

↓

Enable Packet Forwarding

↓

Configure NAT

↓

Start Networking Services
```

---

# Verification

After rebuilding and flashing the updated image:

Check interface.

```bash
ip addr show wlan0
```

Output

```text
inet 192.168.10.1/24
```

Verify DHCP service.

```bash
systemctl status dnsmasq
```

Output

```text
Active: active (running)
```

---

# Client Verification

Connect a mobile phone.

Network information:

```text
IP Address

192.168.10.15
```

Gateway

```text
192.168.10.1
```

DNS

```text
8.8.8.8
```

The client successfully received all required network parameters.

---

# DHCP Packet Flow

```text
Client

   │

   ▼

DHCP Discover

   │

   ▼

dnsmasq

   │

   ▼

DHCP Offer

   │

   ▼

Client Request

   │

   ▼

DHCP ACK

   │

   ▼

Client Receives IP Address
```

---

# Testing Performed

The following tests were conducted after the fix:

### Test 1

Android Phone

Result

```text
PASS
```

---

### Test 2

Windows Laptop

Result

```text
PASS
```

---

### Test 3

Ubuntu Laptop

Result

```text
PASS
```

---

### Test 4

Reconnect Client

Result

```text
Existing Lease Reused
```

---

### Test 5

Multiple Devices

Result

```text
Unique IP Address Assigned
```

---

# Commands Used During Debugging

Check interface.

```bash
ip addr
```

Check DHCP service.

```bash
systemctl status dnsmasq
```

Check logs.

```bash
journalctl -u dnsmasq
```

Display routing.

```bash
ip route
```

Display wireless stations.

```bash
iw dev wlan0 station dump
```

---

# Lessons Learned

This issue highlighted several important networking concepts:

- DHCP depends on a correctly configured interface.
- Service startup order is critical.
- The wireless interface must have a static IP before the DHCP server starts.
- System logs are the fastest way to diagnose DHCP failures.
- Testing with multiple client devices helps identify configuration problems early.

---

# Best Practices

- Always configure the wireless interface before starting DHCP services.
- Verify static IP assignment after every boot.
- Monitor `journalctl` logs during development.
- Keep DHCP configuration minimal and well documented.
- Test lease assignment using different operating systems.

---

# Final Result

After correcting the startup sequence and ensuring that `wlan0` was fully configured before the DHCP server became active:

- Every connected client received a valid IP address.
- Gateway and DNS information were distributed correctly.
- DHCP leases were assigned consistently after every reboot.
- Multiple devices connected simultaneously without conflicts.

With DHCP functioning correctly, the next remaining challenge was that connected devices still could not access the Internet. This was traced to missing IP forwarding and NAT configuration, which is covered in the next document: **`Internet_Not_Working.md`**.