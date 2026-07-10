# dnsmasq

## Introduction

**DNSMasq** is a lightweight DNS forwarder and DHCP server widely used in embedded Linux systems, routers, and access points. It provides automatic IP address assignment (DHCP), DNS forwarding, and network management services for client devices.

In this project, DNSMasq is responsible for assigning IP addresses to devices connected to the Raspberry Pi Wi-Fi Access Point. Without DNSMasq, clients could connect to the Wi-Fi network but would not receive an IP address, making network communication impossible.

---

# What is DNSMasq?

DNSMasq combines multiple networking services into a single lightweight application.

It provides:

- DHCP Server
- DNS Forwarder
- DNS Cache
- DHCP Lease Management

Unlike larger DHCP/DNS servers, DNSMasq is specifically designed for embedded devices with limited resources.

---

# Role in This Project

DNSMasq performs the following functions:

- Assigns IP addresses to Wi-Fi clients
- Provides default gateway information
- Provides DNS server information
- Maintains DHCP leases
- Allows multiple clients to communicate over the network

Without DNSMasq:

- Devices connect to Wi-Fi
- No IP address is assigned
- Clients show **"Connected, No Internet"**
- Internet sharing becomes impossible

---

# DHCP Architecture

```text
            Mobile Phone

                  │

             DHCP Request

                  │

                  ▼

          Raspberry Pi AP

                  │

            DNSMasq Server

                  │

       Assign IP Address

                  │

                  ▼

      192.168.10.10
```

DNSMasq automatically allocates an available IP address whenever a new client joins the Access Point.

---

# Package Installation

DNSMasq is included in the custom image recipe.

```bitbake
IMAGE_INSTALL:append = " \
    dnsmasq \
"
```

BitBake installs the package during image generation.

---

# Configuration File

The project installs the following configuration file:

```text
/etc/dnsmasq.d/rdkb-ap.conf
```

The default DNSMasq service automatically loads every configuration file located inside:

```text
/etc/dnsmasq.d/
```

---

# DNSMasq Configuration

The final configuration used in this project is shown below.

```text
interface=wlan0
bind-interfaces

dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,12h

dhcp-option=3,192.168.10.1
dhcp-option=6,8.8.8.8

log-dhcp
log-queries
```

---

# Configuration Parameters

## interface

```text
interface=wlan0
```

Specifies that DNSMasq listens only on the wireless interface.

Only Wi-Fi clients receive DHCP responses.

---

## bind-interfaces

```text
bind-interfaces
```

Forces DNSMasq to bind specifically to the configured interface.

Advantages:

- Prevents conflicts
- Improves security
- Avoids listening on unwanted interfaces

---

## DHCP Range

```text
dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,12h
```

Defines:

| Parameter | Value |
|-----------|-------|
| Start IP | 192.168.10.10 |
| End IP | 192.168.10.100 |
| Subnet Mask | 255.255.255.0 |
| Lease Time | 12 Hours |

Clients automatically receive addresses within this range.

---

## Default Gateway

```text
dhcp-option=3,192.168.10.1
```

DHCP Option 3 specifies the default gateway.

Every connected client uses:

```text
192.168.10.1
```

as its gateway.

This is the IP address configured on the Raspberry Pi's Wi-Fi interface.

---

## DNS Server

```text
dhcp-option=6,8.8.8.8
```

DHCP Option 6 specifies the DNS server.

In this project:

```text
8.8.8.8
```

(Google Public DNS)

was used.

Clients automatically receive this DNS configuration.

---

## DHCP Logging

```text
log-dhcp
```

Enables DHCP logging.

Useful for debugging:

- New connections
- Lease assignments
- Renewals
- DHCP failures

---

## DNS Query Logging

```text
log-queries
```

Logs DNS requests received from connected devices.

Useful during debugging.

---

# DHCP Address Allocation

```text
Client Connects

        │

        ▼

DHCP Discover

        │

        ▼

DNSMasq Receives Request

        │

        ▼

Assign Available Address

        │

        ▼

192.168.10.10

        │

        ▼

Client Configured
```

---

# Service Management

DNSMasq is managed by systemd.

Service file:

```text
/lib/systemd/system/dnsmasq.service
```

---

# DNSMasq Service

The default service installed by Yocto is:

```ini
[Unit]
Description=DNS forwarder and DHCP server
After=network.target

[Service]
Type=forking
PIDFile=/run/dnsmasq.pid
ExecStartPre=/usr/bin/dnsmasq --test
ExecStart=/usr/bin/dnsmasq -x /run/dnsmasq.pid -7 /etc/dnsmasq.d --local-service

[Install]
WantedBy=multi-user.target
```

The service automatically loads every configuration file inside:

```text
/etc/dnsmasq.d/
```

including:

```text
rdkb-ap.conf
```

---

# Boot Flow

```text
System Boot

        │

        ▼

rdkb-network.service

        │

        ▼

Configure wlan0

        │

        ▼

hostapd.service

        │

        ▼

dnsmasq.service

        │

        ▼

Load DHCP Configuration

        │

        ▼

Start DHCP Server

        │

        ▼

Clients Receive IP Addresses
```

---

# Verifying DNSMasq

Service status:

```bash
systemctl status dnsmasq
```

Expected:

```text
Active: active (running)
```

---

# Checking Process

```bash
ps -ef | grep dnsmasq
```

Expected:

```text
/usr/bin/dnsmasq
```

---

# Checking DHCP Leases

DNSMasq stores active leases.

```bash
cat /var/lib/misc/dnsmasq.leases
```

Example:

```text
1658736240
AA:BB:CC:DD:EE:FF
192.168.10.15
Android
*
```

This shows:

- Lease time
- MAC address
- Assigned IP
- Hostname

---

# Viewing Connected Clients

Another method:

```bash
ip neigh
```

Example:

```text
192.168.10.12 dev wlan0 lladdr xx:xx:xx:xx:xx REACHABLE
```

Displays devices currently communicating with the Raspberry Pi.

---

# Testing Performed

During project testing:

- Android devices received IP addresses.
- Windows laptops received IP addresses.
- Linux laptops received IP addresses.
- Multiple phones connected simultaneously.
- Up to six devices successfully obtained DHCP leases.
- Internet access worked correctly after NAT configuration.

---

# Challenges Encountered

## DHCP Working but No Internet

Initially:

- Wi-Fi connection succeeded.
- DHCP assigned valid IP addresses.

However:

```text
Connected
No Internet
```

appeared on client devices.

The problem was **not** DNSMasq.

The actual issue was missing NAT configuration using iptables.

Once NAT rules were added inside `rdkb-network.sh`, Internet connectivity functioned correctly.

---

## Configuration Location

Initially there was uncertainty regarding the correct configuration path.

The project originally attempted to launch DNSMasq with a custom configuration file.

After verification of the default service, it was found that DNSMasq automatically reads all files placed inside:

```text
/etc/dnsmasq.d/
```

Therefore, the project installs:

```text
rdkb-ap.conf
```

into that directory, allowing the default service to load the configuration without modification.

---

# Advantages of DNSMasq

DNSMasq offers several advantages:

- Lightweight implementation
- Low memory usage
- Fast DHCP response
- Embedded Linux friendly
- Easy configuration
- Automatic lease management
- Reliable integration with systemd

---

# Summary

DNSMasq is responsible for providing DHCP and DNS services to all wireless clients connected to the Raspberry Pi Access Point. It automatically assigns IP addresses, configures the default gateway and DNS server, and manages client leases. By using the default systemd service together with a custom configuration file, DNSMasq integrates seamlessly into the Yocto-built image. Combined with HostAPD and NAT configuration, it enables reliable network connectivity and Internet access for multiple connected devices.