# dnsmasq_issue

## Introduction

After successfully configuring the Access Point using **hostapd**, wireless devices were able to discover and connect to the SSID. However, they were unable to obtain an IP address automatically.

Although the Wi-Fi connection was established, every connected device displayed **"Obtaining IP Address"** for a long time before eventually reporting **"Couldn't obtain IP address"** or assigning itself an APIPA address (169.254.x.x).

This indicated that the DHCP server was not functioning correctly.

This document describes the issue, investigation process, root cause, and the final solution.

---

# Problem Description

After booting the Raspberry Pi:

- Access Point was visible.
- Clients connected successfully.
- WPA2 authentication succeeded.
- No IP address was assigned.
- Internet was unavailable.

The Access Point existed, but clients could not communicate because they were not assigned valid network parameters.

---

# Symptoms

```text
✔ Raspberry Pi Booted

✔ SSID Visible

✔ Client Connected Successfully

✔ WPA2 Authentication Successful

✘ No IP Address Assigned

✘ DHCP Failed

✘ Internet Not Available
```

---

# Initial Investigation

The first step was to verify whether the **dnsmasq** service was running.

```bash
systemctl status dnsmasq
```

Initial observation:

```text
dnsmasq.service

Active: failed
```

or

```text
dnsmasq.service

Active: inactive
```

This confirmed that the DHCP server was not operating correctly.

---

# Verify Configuration File

Check whether the configuration file existed.

```bash
ls -l /etc/dnsmasq.conf
```

View the configuration.

```bash
cat /etc/dnsmasq.conf
```

Initially, either:

- the configuration file was missing, or
- it contained incorrect settings.

---

# Root Cause Analysis

The investigation revealed that:

- `dnsmasq` package was installed.
- The service was available.
- The required configuration file was either absent or incomplete.

Without a valid DHCP configuration, `dnsmasq` starts without serving IP addresses or may fail to start entirely.

---

# Solution

A custom configuration file named:

```text
dnsmasq.conf
```

was created.

Location:

```text
meta-rdkb-ap/

recipes-connectivity/

rdkb-ap-config/

files/
```

---

# Configuration Used

The final configuration used in this project was:

```text
interface=wlan0

bind-interfaces

dhcp-range=192.168.10.10,192.168.10.100,12h

dhcp-option=3,192.168.10.1

dhcp-option=6,8.8.8.8
```

---

# Configuration Explanation

Interface

```text
interface=wlan0
```

Specifies that DHCP requests should only be served on the wireless interface.

---

Bind Interfaces

```text
bind-interfaces
```

Prevents dnsmasq from listening on unwanted interfaces.

---

DHCP Address Pool

```text
dhcp-range=192.168.10.10,192.168.10.100,12h
```

Allocates addresses between:

```text
192.168.10.10

↓

192.168.10.100
```

Lease Time:

```text
12 Hours
```

---

Gateway

```text
dhcp-option=3,192.168.10.1
```

Provides the Raspberry Pi as the default gateway.

---

DNS Server

```text
dhcp-option=6,8.8.8.8
```

Supplies Google's public DNS server.

---

# Recipe Modification

The BitBake recipe was updated to install the configuration file.

Example:

```bitbake
install -m 0644 ${WORKDIR}/dnsmasq.conf \
${D}${sysconfdir}/dnsmasq.conf
```

---

# Rebuild Process

After updating the configuration:

Clean recipe:

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild image:

```bash
bitbake core-image-rdkb-ap
```

Flash the updated image.

---

# Image Verification

Before flashing, the generated image was mounted.

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Verify configuration.

```bash
cat /mnt/verify-root/etc/dnsmasq.conf
```

The expected configuration was present.

---

# Verification After Boot

Check service status.

```bash
systemctl status dnsmasq
```

Expected Output:

```text
Active: active (running)
```

---

# DHCP Verification

Connect a Wi-Fi client.

Verify assigned IP address.

Expected:

```text
IP Address

192.168.10.x
```

Gateway:

```text
192.168.10.1
```

DNS:

```text
8.8.8.8
```

---

# Network Flow After Fix

```text
Client Connects

        │

        ▼

DHCP Discover

        │

        ▼

dnsmasq Receives Request

        │

        ▼

Assign IP Address

        │

        ▼

Provide Gateway

        │

        ▼

Provide DNS

        │

        ▼

Client Connected
```

---

# Testing Performed

After implementing the fix:

- Android phones received IP addresses automatically.
- Windows laptops obtained DHCP leases.
- Linux systems connected successfully.
- Multiple devices were assigned unique addresses.
- Gateway configuration was correct.
- DNS server information was correctly distributed.

---

# Lessons Learned

The following concepts became clear during debugging:

- Installing `dnsmasq` is not sufficient.
- A correct DHCP configuration is essential.
- Every required configuration file should be included in the custom Yocto layer.
- Verifying generated images before flashing reduces debugging time.
- Service status should always be checked before investigating networking issues.

---

# Best Practices

- Keep DHCP configuration simple and well documented.
- Allocate a reasonable IP address range.
- Define gateway and DNS options explicitly.
- Verify configuration files after every image build.
- Check `journalctl -u dnsmasq` whenever DHCP issues occur.
- Test with multiple client devices.

---

# Final Result

After installing the correct `dnsmasq.conf` into the custom Yocto image:

- The `dnsmasq` service started automatically.
- DHCP requests were processed successfully.
- Wireless clients received valid IP addresses.
- Gateway and DNS information were distributed correctly.
- Multiple devices connected simultaneously without IP conflicts.

Although DHCP was now functioning correctly, Internet access was still unavailable because Network Address Translation (NAT) had not yet been configured. This issue is discussed in the next document.