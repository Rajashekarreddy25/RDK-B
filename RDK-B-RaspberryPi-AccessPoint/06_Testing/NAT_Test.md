# NAT_Test

## Introduction

Network Address Translation (NAT) enables multiple devices connected to the Raspberry Pi Access Point to access the Internet using a single Ethernet connection. In this project, NAT was implemented using `iptables`, while IPv4 forwarding was enabled through the Linux kernel.

After verifying the Access Point and DHCP functionality, the next step was to ensure that wireless clients could access the Internet through the Raspberry Pi.

This document describes the complete NAT testing procedure, configuration, commands used, observations, and issues encountered during implementation.

---

# Objective

The objectives of this test are:

- Verify IPv4 forwarding.
- Verify NAT configuration.
- Verify Internet access from Wi-Fi clients.
- Verify packet forwarding.
- Verify multiple clients accessing the Internet simultaneously.
- Ensure stable Internet connectivity.

---

# Network Topology

```text
                   Internet
                       │
                       │
                Home Router
                       │
                       │
                   Ethernet
                       │
                  Raspberry Pi
               eth0        wlan0
                 │            │
                 │            │
          Internet       Access Point
                               │
                ┌──────────────┴──────────────┐
                │              │              │
           Mobile Phone     Laptop        Tablet
```

---

# NAT Workflow

```text
Client Sends Packet

        │

        ▼

wlan0

        │

        ▼

Kernel IP Forwarding

        │

        ▼

iptables MASQUERADE

        │

        ▼

eth0

        │

        ▼

Internet

        │

        ▼

Response Returned

        │

        ▼

Client Receives Data
```

---

# NAT Configuration

The NAT configuration was implemented inside:

```text
/usr/bin/rdkb-network.sh
```

The following commands were executed during boot.

Enable IPv4 forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Enable NAT:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Allow forwarding from Wi-Fi to Ethernet:

```bash
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Allow return traffic:

```bash
iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

---

# Test Procedure

## Step 1 – Connect Ethernet

Connect the Raspberry Pi to the home router using an Ethernet cable.

Verify that `eth0` has received an IP address.

```bash
ip addr show eth0
```

Example:

```text
192.168.1.105
```

---

## Step 2 – Verify Internet on Raspberry Pi

Check Internet connectivity.

```bash
ping -c 4 8.8.8.8
```

Expected output:

```text
4 packets transmitted

4 packets received
```

Then verify DNS resolution.

```bash
ping -c 4 google.com
```

Expected output:

```text
PING google.com...
```

---

## Step 3 – Verify IP Forwarding

Check whether IPv4 forwarding is enabled.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected output:

```text
1
```

A value of **1** confirms that packet forwarding is enabled.

---

## Step 4 – Verify NAT Rules

Display the NAT table.

```bash
iptables -t nat -L
```

Expected output:

```text
MASQUERADE
```

Verify forwarding rules.

```bash
iptables -L FORWARD
```

Expected output:

```text
ACCEPT
```

---

## Step 5 – Connect a Wireless Client

Using a mobile phone or laptop:

Connect to:

```text
RDKB_AP
```

Password:

```text
12345678
```

Verify that the client receives an IP address.

Example:

```text
192.168.10.10
```

---

## Step 6 – Test Internet Access

Open a web browser.

Visit:

```text
https://www.google.com
```

Expected result:

The webpage should load successfully.

---

## Step 7 – Test Using Ping

On the client device:

Ping a public IP address.

Example:

```text
8.8.8.8
```

Expected result:

Successful replies.

---

## Step 8 – Test DNS Resolution

Open any website.

Example:

```text
https://github.com
```

Successful loading confirms that DNS forwarding is functioning correctly.

---

# Multiple Client NAT Test

Connect several devices simultaneously.

Devices used:

- Android Phone
- Laptop
- Tablet
- Smartphone
- Desktop Wi-Fi Adapter
- Additional Mobile Device

Total connected devices:

```text
6
```

All devices were able to browse the Internet simultaneously without interruptions.

---

# Commands Used

Verify Ethernet IP:

```bash
ip addr show eth0
```

Verify forwarding:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

View NAT rules:

```bash
iptables -t nat -L
```

View forwarding rules:

```bash
iptables -L FORWARD
```

Check Internet:

```bash
ping -c 4 8.8.8.8
```

Check DNS:

```bash
ping -c 4 google.com
```

---

# Expected Results

| Test | Expected Result |
|-------|-----------------|
| Ethernet Connected | Pass |
| Raspberry Pi Internet | Pass |
| IP Forwarding Enabled | Pass |
| NAT Rule Present | Pass |
| Client Internet Access | Pass |
| DNS Resolution | Pass |
| Multiple Clients Supported | Pass |

---

# Actual Results

| Test | Result |
|-------|---------|
| Ethernet Active | Pass |
| Internet Working | Pass |
| NAT Working | Pass |
| IP Forwarding Enabled | Pass |
| Wi-Fi Clients Access Internet | Pass |
| Six Devices Accessed Internet Simultaneously | Pass |

---

# Problems Encountered

## Clients Connected but No Internet

### Cause

Initially, clients successfully connected to the Access Point and received IP addresses, but Internet access was unavailable.

### Root Cause

The custom network script only configured the wireless interface and assigned the static IP address. It did not enable:

- IPv4 forwarding
- NAT (MASQUERADE)
- Packet forwarding rules

As a result, packets from wireless clients could not be forwarded to the Ethernet interface.

---

## Solution

The following commands were added to:

```text
rdkb-network.sh
```

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

After rebuilding the image, Internet access worked correctly.

---

## Missing iptables

### Cause

Initially, the `iptables` package was not included in the image.

Attempting to execute:

```bash
iptables
```

resulted in:

```text
command not found
```

### Solution

The image recipe was updated.

```bitbake
IMAGE_INSTALL += "iptables"
```

The image was rebuilt, and `iptables` became available.

---

## Cached Network Script

### Cause

After modifying `rdkb-network.sh`, the Raspberry Pi still booted with the previous version of the script.

### Root Cause

BitBake reused cached build artifacts.

### Solution

Clean the recipe and rebuild.

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

The updated script was packaged into the new image.

---

# Observations

During testing:

- NAT rules were automatically applied during boot.
- IPv4 forwarding remained enabled.
- Internet access was stable.
- DNS resolution functioned correctly.
- Multiple devices accessed the Internet simultaneously.
- No packet forwarding failures were observed.

The Raspberry Pi successfully functioned as a wireless router.

---

# Conclusion

The NAT implementation was successfully validated. After enabling IPv4 forwarding and configuring `iptables` MASQUERADE rules, all wireless clients connected to the `RDKB_AP` network were able to access the Internet through the Ethernet interface. The Raspberry Pi reliably forwarded traffic between `wlan0` and `eth0`, allowing multiple devices to browse the Internet simultaneously without any manual configuration after boot.