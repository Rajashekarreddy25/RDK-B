# dnsmasq_Debugging

## Introduction

`dnsmasq` is a lightweight DNS forwarder and DHCP server widely used in embedded Linux systems. In this project, `dnsmasq` was responsible for automatically assigning IP addresses to wireless clients connected to the Raspberry Pi Access Point.

Without `dnsmasq`, wireless clients could connect to the Access Point but would not receive valid IP addresses, preventing communication with the Raspberry Pi or the Internet.

During the implementation of this project, several DHCP-related issues were encountered and resolved. This document describes the debugging process, commands used, problems faced, and the final solutions.

---

# Role of dnsmasq in the Project

```text
                 Internet
                     │
                     │
                 Ethernet
                   (eth0)
                     │
                     ▼
            Raspberry Pi 4
         Custom Yocto Image
                     │
             hostapd Service
                     │
              Wi-Fi Clients
                     │
                     ▼
             dnsmasq Service
                     │
          Assigns IP Addresses
                     │
                     ▼
         192.168.10.10 - 192.168.10.100
```

`dnsmasq` performs the following functions:

- DHCP Server
- DNS Forwarder
- Gateway Information
- DNS Configuration
- Lease Management

---

# DHCP Configuration File

Configuration file used in this project:

```text
/etc/dnsmasq.d/rdkb-ap.conf
```

Contents:

```text
interface=wlan0

dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,24h

dhcp-option=3,192.168.10.1

dhcp-option=6,8.8.8.8
```

---

# Debugging Workflow

```text
Client Connects

        │

        ▼

Gets IP Address?

        │

 ┌──────┴──────┐

 │             │

YES            NO

 │             │

 ▼             ▼

Internet?   Check dnsmasq

 │             │

 ▼             ▼

Success    View Logs

               │

               ▼

      Verify Configuration

               │

               ▼

        Restart Service

               │

               ▼

         Verify Again
```

---

# Step 1 – Verify Service Status

Check whether `dnsmasq` is running.

```bash
systemctl status dnsmasq
```

Expected:

```text
Active: active (running)
```

---

# Step 2 – Verify Configuration File

Display DHCP configuration.

```bash
cat /etc/dnsmasq.d/rdkb-ap.conf
```

Verify:

- Interface
- DHCP Range
- Gateway
- DNS Server

---

# Step 3 – View Service Logs

Display logs.

```bash
journalctl -u dnsmasq
```

The logs help identify:

- Configuration syntax errors
- Port conflicts
- DHCP failures
- Interface issues

---

# Step 4 – Restart Service

After modifying configuration:

```bash
systemctl restart dnsmasq
```

Verify:

```bash
systemctl status dnsmasq
```

---

# Step 5 – Verify DHCP Leases

Display current DHCP leases.

```bash
cat /var/lib/misc/dnsmasq.leases
```

Example:

```text
192.168.10.10

192.168.10.11

192.168.10.12
```

Each entry contains:

- Lease Time
- MAC Address
- IP Address
- Hostname

---

# Issue 1 – Client Did Not Receive IP Address

## Symptoms

- Device connected to Wi-Fi.
- Stuck on:

```text
Obtaining IP Address...
```

- Eventually disconnected.

---

## Cause

`dnsmasq` service was not running.

---

## Debugging

Check:

```bash
systemctl status dnsmasq
```

View logs.

```bash
journalctl -u dnsmasq
```

---

## Solution

Restart service.

```bash
systemctl restart dnsmasq
```

---

## Result

Clients immediately received valid IP addresses.

---

# Issue 2 – Invalid DHCP Configuration

## Symptoms

`dnsmasq` failed to start.

---

## Cause

Incorrect syntax inside:

```text
rdkb-ap.conf
```

---

## Debugging

Test configuration.

```bash
dnsmasq --test
```

Expected:

```text
syntax check OK
```

---

## Solution

Correct the configuration.

Restart service.

---

# Issue 3 – Wrong DHCP Range

## Symptoms

Clients received incorrect addresses.

Example:

```text
169.254.x.x
```

---

## Cause

Invalid DHCP range.

---

## Debugging

Display configuration.

```bash
cat /etc/dnsmasq.d/rdkb-ap.conf
```

Verify:

```text
dhcp-range=
```

---

## Solution

Configure:

```text
dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,24h
```

Restart service.

---

# Issue 4 – Configuration Changes Not Applied

## Symptoms

Changes made to:

```text
rdkb-ap.conf
```

were not reflected after rebuilding.

---

## Cause

BitBake reused cached artifacts.

---

## Solution

Clean recipe.

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild.

```bash
bitbake core-image-rdkb-ap
```

Verify generated image before flashing.

---

# Issue 5 – Verify dnsmasq in Image

Before flashing, verify that `dnsmasq` exists.

```bash
find /mnt/verify-root -name dnsmasq
```

Expected:

```text
/usr/bin/dnsmasq
```

Verify configuration.

```bash
ls /mnt/verify-root/etc/dnsmasq.d
```

Expected:

```text
rdkb-ap.conf
```

Verify service.

```bash
ls /mnt/verify-root/lib/systemd/system
```

Expected:

```text
dnsmasq.service
```

---

# Monitoring DHCP Clients

Display leases.

```bash
cat /var/lib/misc/dnsmasq.leases
```

Example:

```text
MAC Address

IP Address

Hostname
```

This allows identification of every connected client.

---

# Useful dnsmasq Commands

Check service.

```bash
systemctl status dnsmasq
```

Restart service.

```bash
systemctl restart dnsmasq
```

Enable service.

```bash
systemctl enable dnsmasq
```

View logs.

```bash
journalctl -u dnsmasq
```

Test configuration.

```bash
dnsmasq --test
```

View leases.

```bash
cat /var/lib/misc/dnsmasq.leases
```

---

# Actual Debugging Performed

During the project, the following validations were performed:

- Verified `dnsmasq` package was installed inside the image.
- Confirmed `rdkb-ap.conf` existed under `/etc/dnsmasq.d`.
- Verified `dnsmasq.service` was installed.
- Ensured the service started automatically after boot.
- Confirmed DHCP addresses were assigned correctly.
- Verified lease generation for all connected clients.
- Tested six simultaneous wireless devices.

---

# Lessons Learned

While debugging `dnsmasq`, several important lessons were learned:

- Always verify the DHCP configuration before rebooting.
- Use `dnsmasq --test` to detect syntax errors.
- Confirm that the DHCP range matches the network configuration.
- Verify image contents before flashing.
- Monitor lease files to identify connected clients.
- Ensure `dnsmasq` starts after the wireless interface is available.

---

# Conclusion

`dnsmasq` was responsible for providing automatic IP address assignment and DNS forwarding in the custom RDK-B Access Point implementation. Through careful verification of service status, configuration files, DHCP leases, and runtime logs, all DHCP-related issues were successfully resolved. The final implementation reliably assigned unique IP addresses to all connected wireless devices and worked seamlessly with `hostapd` and `iptables` to provide complete network connectivity. Understanding the operation and debugging techniques of `dnsmasq` was essential in achieving a stable and functional embedded networking solution.