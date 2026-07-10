# hostapd

## Introduction

**HostAPD (Host Access Point Daemon)** is an open-source user-space daemon used to configure a Linux system as a Wireless Access Point (AP). It provides IEEE 802.11 Access Point functionality and supports authentication, encryption, and security protocols such as WPA, WPA2, and WPA3.

In this project, HostAPD is responsible for converting the Raspberry Pi 4's built-in Wi-Fi interface (`wlan0`) into a secure wireless Access Point. Without HostAPD, the Raspberry Pi cannot broadcast an SSID or accept wireless client connections.

---

# What is HostAPD?

HostAPD is responsible for:

- Creating the Wireless Access Point
- Broadcasting the SSID
- Managing client authentication
- Enabling WPA/WPA2 security
- Handling wireless encryption
- Managing client associations

It acts as the bridge between the Linux networking stack and the Wi-Fi hardware.

---

# Role in This Project

Within this project, HostAPD performs the following operations:

- Creates the Wi-Fi network
- Broadcasts the SSID **RDKB_AP**
- Authenticates wireless clients
- Encrypts wireless communication
- Allows multiple devices to connect simultaneously

Without HostAPD:

- No SSID appears on nearby devices.
- Mobile phones and laptops cannot detect the Raspberry Pi.
- Wireless communication is impossible.

---

# HostAPD Architecture

```text
                 Mobile Phone
                       │
                 Laptop Computer
                       │
                  IoT Devices
                       │
            -----------------------
                     Wi-Fi
            -----------------------
                       │
                  Raspberry Pi
                       │
                  HostAPD Daemon
                       │
                    wlan0 Driver
                       │
               Raspberry Pi Wi-Fi Chip
```

HostAPD communicates directly with the Linux wireless driver (`nl80211`) to control the Wi-Fi hardware.

---

# Package Installation

HostAPD is added to the image using the custom image recipe.

```bitbake
IMAGE_INSTALL:append = " \
    hostapd \
"
```

During image generation, BitBake automatically installs the HostAPD package into the root filesystem.

---

# Configuration File Location

HostAPD uses the following configuration file:

```text
/etc/hostapd.conf
```

This file is installed by the custom `rdkb-ap-config` package.

---

# HostAPD Configuration

The final configuration used in this project is shown below.

```text
interface=wlan0
driver=nl80211

ssid=RDKB_AP

hw_mode=g
channel=6
country_code=IN

ieee80211n=1

auth_algs=1
ignore_broadcast_ssid=0

wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

---

# Configuration Parameters

## interface

```text
interface=wlan0
```

Specifies the wireless interface that will operate as the Access Point.

In this project:

```text
wlan0
```

was used.

---

## driver

```text
driver=nl80211
```

The `nl80211` driver is the Linux kernel interface used for modern wireless devices.

HostAPD communicates with the Wi-Fi chipset through this interface.

---

## SSID

```text
ssid=RDKB_AP
```

Defines the Wi-Fi network name displayed on client devices.

Example:

```text
RDKB_AP
```

This is the SSID that appeared on mobile phones and laptops during testing.

---

## Hardware Mode

```text
hw_mode=g
```

Specifies the wireless standard.

| Mode | Standard |
|--------|-----------|
| a | 5 GHz |
| b | 2.4 GHz |
| g | 2.4 GHz |
| n | 802.11n |

For this project:

```text
g
```

was selected to maximize compatibility with client devices.

---

## Channel

```text
channel=6
```

Specifies the Wi-Fi operating channel.

Channel 6 was selected because it provides good compatibility and reduced interference in many environments.

---

## Country Code

```text
country_code=IN
```

Defines the regulatory domain.

This enables the correct transmit power and legal channel usage according to Indian wireless regulations.

---

## IEEE 802.11n

```text
ieee80211n=1
```

Enables 802.11n support.

Advantages:

- Higher throughput
- Better range
- Improved performance

---

## Authentication

```text
auth_algs=1
```

Specifies the authentication algorithm.

Value:

```text
1
```

means Open System Authentication.

---

## Broadcast SSID

```text
ignore_broadcast_ssid=0
```

Controls SSID visibility.

Value:

```text
0
```

means:

- SSID is visible
- Devices can discover the network normally

---

## WPA Mode

```text
wpa=2
```

Enables WPA2 security.

This provides encrypted wireless communication.

---

## Password

```text
wpa_passphrase=12345678
```

Defines the Wi-Fi password.

Minimum length:

```text
8 characters
```

---

## Key Management

```text
wpa_key_mgmt=WPA-PSK
```

Specifies WPA Pre-Shared Key authentication.

This is commonly used in home and small-office wireless networks.

---

## Encryption

```text
rsn_pairwise=CCMP
```

Specifies AES (CCMP) encryption.

AES provides stronger security than older TKIP encryption.

---

# HostAPD Service

HostAPD is managed by systemd.

Service file:

```text
/lib/systemd/system/hostapd.service
```

The service starts automatically during boot.

---

# HostAPD Service Configuration

The default service installed by Yocto is:

```ini
[Unit]
Description=Hostapd IEEE 802.11 AP
After=network.target

[Service]
Type=forking
PIDFile=/run/hostapd.pid
ExecStart=/usr/sbin/hostapd /etc/hostapd.conf -P /run/hostapd.pid -B

[Install]
WantedBy=multi-user.target
```

The service launches HostAPD using the configuration installed at:

```text
/etc/hostapd.conf
```

---

# Boot Sequence

```text
System Boot

        │

        ▼

systemd

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

Read /etc/hostapd.conf

        │

        ▼

Broadcast SSID

        │

        ▼

Clients Discover Wi-Fi
```

---

# Verifying HostAPD

After booting the Raspberry Pi, the service can be checked.

```bash
systemctl status hostapd
```

Expected result:

```text
Active: active (running)
```

---

# Checking the Process

```bash
ps -ef | grep hostapd
```

Expected output:

```text
/usr/sbin/hostapd
```

---

# Checking Wi-Fi Interface

```bash
iw dev
```

Expected output:

```text
Interface wlan0
```

This confirms that the wireless interface is available.

---

# Checking Connected Stations

HostAPD provides information about connected wireless clients.

```bash
hostapd_cli all_sta
```

Example:

```text
Station xx:xx:xx:xx:xx:xx
```

This displays all devices currently connected to the Access Point.

---

# Project Testing

During testing:

- SSID appeared successfully.
- Android devices detected the AP.
- Laptops connected successfully.
- Up to six client devices connected simultaneously.
- WPA2 authentication worked correctly.
- Stable wireless communication was maintained.

---

# Challenges Encountered

Several HostAPD-related issues were encountered during development.

## Configuration Path Mismatch

Initially, the configuration file was installed at:

```text
/etc/hostapd/hostapd.conf
```

However, the default `hostapd.service` expected the configuration at:

```text
/etc/hostapd.conf
```

As a result, HostAPD failed to start.

The recipe was updated to install the configuration directly to `/etc/hostapd.conf`, resolving the issue.

---

## Service Management

An early design attempted to launch HostAPD from a custom systemd service using additional `ExecStart` commands.

This caused service management issues and complicated debugging.

The final design retained the default `hostapd.service`, allowing systemd to manage the daemon independently while `rdkb-network.service` handled only network initialization.

---

## Internet Access

HostAPD successfully allowed devices to connect to the Wi-Fi network, but initially connected clients could not access the Internet.

The issue was unrelated to HostAPD itself. The root cause was the absence of Network Address Translation (NAT). After adding `iptables` and configuring masquerading in the network script, Internet connectivity functioned correctly.

---

# Advantages of HostAPD

Using HostAPD offers several benefits:

- Stable wireless Access Point implementation
- WPA2 security support
- Native Linux integration
- Systemd service management
- Support for multiple client devices
- Flexible configuration options
- Widely used in embedded Linux systems

---

# Summary

HostAPD is the core component that enables wireless Access Point functionality on the Raspberry Pi. It manages SSID broadcasting, client authentication, and wireless security while integrating seamlessly with the Linux networking stack. In this project, HostAPD was configured to create the **RDKB_AP** wireless network using WPA2 encryption, allowing multiple clients to connect securely. After resolving configuration path mismatches and integrating the service correctly with systemd, HostAPD operated reliably as the foundation of the custom RDK-B Wi-Fi Access Point.