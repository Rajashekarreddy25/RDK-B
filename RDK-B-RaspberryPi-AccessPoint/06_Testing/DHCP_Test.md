# DHCP_Test

## Introduction

The Dynamic Host Configuration Protocol (DHCP) is responsible for automatically assigning IP addresses to wireless clients connected to the Access Point. In this project, `dnsmasq` was configured as the DHCP server.

Without DHCP, every client would require manual IP configuration, making the Access Point difficult to use. Therefore, verifying DHCP functionality was an essential step after confirming that the Access Point was operational.

This document describes the complete DHCP testing procedure, expected behavior, commands used, observations, and issues encountered during development.

---

# Objective

The objectives of this test are:

- Verify that the DHCP server starts automatically.
- Verify automatic IP address assignment.
- Verify gateway assignment.
- Verify DNS configuration.
- Verify lease generation.
- Verify multiple client support.
- Ensure clients receive unique IP addresses.

---

# Test Environment

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| Operating System | Custom Yocto + RDK-B Image |
| DHCP Server | dnsmasq |
| Interface | wlan0 |
| Gateway Address | 192.168.10.1 |
| DHCP Range | 192.168.10.10 – 192.168.10.100 |

---

# DHCP Configuration

The DHCP server was configured using:

```text
/etc/dnsmasq.d/rdkb-ap.conf
```

Configuration:

```text
interface=wlan0

dhcp-range=192.168.10.10,192.168.10.100,12h
```

Configuration Details

| Parameter | Value |
|-----------|-------|
| Interface | wlan0 |
| Starting IP | 192.168.10.10 |
| Ending IP | 192.168.10.100 |
| Lease Time | 12 Hours |

---

# DHCP Testing Workflow

```text
Boot Raspberry Pi

      │

      ▼

Start dnsmasq

      │

      ▼

Client Connects to AP

      │

      ▼

DHCP Discover

      │

      ▼

DHCP Offer

      │

      ▼

DHCP Request

      │

      ▼

DHCP Acknowledgement

      │

      ▼

IP Assigned
```

---

# Test Procedure

## Step 1 – Boot the Raspberry Pi

Power on the Raspberry Pi.

Wait until all services start successfully.

---

## Step 2 – Verify dnsmasq Service

Check service status.

```bash
systemctl status dnsmasq
```

Expected output:

```text
Active: active (running)
```

---

## Step 3 – Connect a Client

Using a mobile phone or laptop:

- Enable Wi-Fi.
- Connect to:

```text
RDKB_AP
```

Password:

```text
12345678
```

---

## Step 4 – Verify Assigned IP Address

On the connected client, open the Wi-Fi network details.

Expected information:

```text
IP Address:

192.168.10.x
```

Example:

```text
192.168.10.12
```

---

## Step 5 – Verify Gateway

Expected gateway:

```text
192.168.10.1
```

---

## Step 6 – Verify DNS Server

Expected DNS server:

```text
192.168.10.1
```

---

## Step 7 – Verify DHCP Lease

On Raspberry Pi:

```bash
cat /var/lib/misc/dnsmasq.leases
```

Example output:

```text
1720000000
74:4d:28:xx:xx:xx
192.168.10.10
android-device
*
```

Each connected client receives one lease entry.

---

# Verify Multiple DHCP Clients

Connect multiple wireless devices.

Example devices:

- Android Phone
- Laptop
- Tablet
- Smartphone
- Desktop Wi-Fi Adapter
- Another Mobile Device

Verify leases.

```bash
cat /var/lib/misc/dnsmasq.leases
```

Expected:

```text
192.168.10.10

192.168.10.11

192.168.10.12

192.168.10.13

192.168.10.14

192.168.10.15
```

Each client should receive a unique IP address.

---

# Verify Connected Clients

Check associated wireless stations.

```bash
iw dev wlan0 station dump
```

Compare the number of stations with the DHCP lease entries.

The number of associated stations should match the number of DHCP leases.

---

# Verify dnsmasq Logs

Display service logs.

```bash
journalctl -u dnsmasq
```

Typical messages:

```text
DHCPDISCOVER

DHCPOFFER

DHCPREQUEST

DHCPACK
```

These messages confirm successful DHCP negotiation.

---

# Expected Results

| Test | Expected Result |
|-------|-----------------|
| dnsmasq Running | Pass |
| DHCP Offer | Pass |
| DHCP ACK | Pass |
| IP Address Assigned | Pass |
| Gateway Assigned | Pass |
| DNS Assigned | Pass |
| Multiple Clients Supported | Pass |

---

# Actual Results

| Test | Result |
|-------|---------|
| Service Started | Pass |
| Client Received IP | Pass |
| Gateway Correct | Pass |
| DNS Correct | Pass |
| Lease Generated | Pass |
| Six Devices Received IP Addresses | Pass |

---

# Commands Used

Check dnsmasq service:

```bash
systemctl status dnsmasq
```

View DHCP leases:

```bash
cat /var/lib/misc/dnsmasq.leases
```

View logs:

```bash
journalctl -u dnsmasq
```

Check connected stations:

```bash
iw dev wlan0 station dump
```

---

# Problems Encountered

## Client Connected but No IP Address

### Cause

The `dnsmasq` service was not running.

### Solution

Verified the service status:

```bash
systemctl status dnsmasq
```

Restarted the service:

```bash
systemctl restart dnsmasq
```

---

## Incorrect DHCP Configuration

### Cause

Errors in the DHCP configuration file.

### Solution

Verified:

```text
/etc/dnsmasq.d/rdkb-ap.conf
```

Ensured that:

```text
interface=wlan0

dhcp-range=192.168.10.10,192.168.10.100,12h
```

was configured correctly.

---

## No Lease File Generated

### Cause

The client had not completed the DHCP handshake.

### Solution

Disconnected and reconnected the client.

Verified the lease file again:

```bash
cat /var/lib/misc/dnsmasq.leases
```

---

# Observations

During testing:

- All clients received unique IP addresses.
- DHCP leases were generated correctly.
- Gateway and DNS settings were assigned automatically.
- No duplicate IP addresses were observed.
- Lease allocation was consistent even with six connected devices.

The DHCP server remained stable throughout the testing process.

---

# Conclusion

The DHCP functionality was successfully validated using `dnsmasq`. Every client connecting to the `RDKB_AP` wireless network automatically received a unique IP address, gateway, and DNS configuration without manual intervention. Lease generation and DHCP negotiation completed successfully for all connected devices, confirming that the Access Point could reliably provide network configuration services to multiple clients simultaneously.