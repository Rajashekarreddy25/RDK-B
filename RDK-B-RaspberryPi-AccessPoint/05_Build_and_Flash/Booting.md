# Booting

## Introduction

After successfully flashing the custom RDK-B image onto the microSD card, the next step is to boot the Raspberry Pi 4. During the boot process, the Raspberry Pi loads the bootloader, Linux kernel, root filesystem, and starts all required system services. The custom configurations added during the Yocto build are also initialized.

The objective of this phase is to verify that the Raspberry Pi boots successfully and automatically starts the Access Point (AP) services without requiring any manual configuration.

---

# Boot Sequence Overview

```text
Power ON

      │

      ▼

Boot ROM

      │

      ▼

Read Boot Files from SD Card

      │

      ▼

Load Bootloader

      │

      ▼

Load Linux Kernel

      │

      ▼

Kernel Initialization

      │

      ▼

Mount Root Filesystem

      │

      ▼

Start systemd

      │

      ▼

Start System Services

      │

      ▼

Execute rdkb-network.service

      │

      ▼

Configure wlan0

      │

      ▼

Start hostapd

      │

      ▼

Start dnsmasq

      │

      ▼

Access Point Ready
```

---

# Hardware Setup

The following hardware was used for testing.

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| RAM | 4 GB |
| Storage | 16 GB microSD Card |
| Power Supply | 5V / 3A USB-C Adapter |
| Network | Ethernet Cable Connected |
| Wireless | Built-in Wi-Fi Interface |

---

# Insert the SD Card

Insert the flashed microSD card into the Raspberry Pi.

Ensure that:

- The SD card is fully inserted.
- The Ethernet cable is connected (for Internet access).
- The power supply is not connected yet.

---

# Connect Required Peripherals

For the initial boot, connect:

- HDMI Monitor
- USB Keyboard (Optional)
- Ethernet Cable
- Power Adapter

These peripherals help monitor the first boot and diagnose issues if necessary.

---

# Power On the Raspberry Pi

Connect the USB-C power supply.

Immediately after power is applied:

- Red LED turns ON (Power)
- Green LED starts blinking (SD Card Activity)

This indicates that the Raspberry Pi has started reading the boot files.

---

# Bootloader Execution

The Raspberry Pi Boot ROM performs the following tasks:

1. Detects the SD card.
2. Reads the boot partition.
3. Loads the firmware.
4. Loads the bootloader.
5. Starts the Linux kernel.

Required boot files include:

```text
bootcode.bin

start4.elf

fixup4.dat

config.txt

cmdline.txt

kernel8.img
```

---

# Linux Kernel Initialization

After the bootloader finishes, the Linux kernel begins execution.

The kernel initializes:

- CPU
- RAM
- USB Controller
- Ethernet Controller
- SD Card Interface
- Wi-Fi Chipset
- Bluetooth Controller
- Device Drivers

Kernel messages appear on the console during this stage.

---

# Root Filesystem Mounting

The kernel mounts the root filesystem located on the second partition of the SD card.

Typical directories loaded include:

```text
/

├── bin

├── boot

├── dev

├── etc

├── home

├── lib

├── proc

├── root

├── sys

├── usr

└── var
```

---

# systemd Initialization

Once the root filesystem is mounted, the kernel starts the first userspace process.

```text
systemd
```

systemd becomes Process ID 1 (PID 1).

Its responsibilities include:

- Starting system services
- Managing dependencies
- Launching networking
- Starting custom services
- Handling system shutdown

---

# Automatic Service Startup

During boot, systemd automatically starts the required services.

The important services for this project are:

```text
rdkb-network.service

↓

hostapd.service

↓

dnsmasq.service
```

These services were enabled during the image build process.

---

# Network Configuration

The custom network initialization script is executed by:

```text
rdkb-network.service
```

The script performs the following actions:

- Enables the wireless interface.
- Assigns a static IP address.
- Enables IPv4 forwarding.
- Configures NAT using iptables.
- Enables packet forwarding.

This step prepares the Raspberry Pi to function as a wireless router.

---

# Starting hostapd

After the network is configured, hostapd starts automatically.

Responsibilities:

- Enable Access Point mode.
- Configure wireless parameters.
- Broadcast the SSID.
- Handle WPA2 authentication.

Configured parameters:

| Parameter | Value |
|-----------|-------|
| Interface | wlan0 |
| SSID | RDKB_AP |
| Channel | 6 |
| Security | WPA2 |
| Password | 12345678 |

Once hostapd starts successfully, nearby devices can detect the wireless network.

---

# Starting dnsmasq

After hostapd is running, dnsmasq starts.

Responsibilities:

- DHCP Server
- DNS Forwarder
- IP Address Allocation

Clients connecting to the AP automatically receive an IP address.

Example:

```text
192.168.10.10

192.168.10.11

192.168.10.12
```

---

# Internet Sharing

The custom network script enables Internet sharing.

Traffic flow:

```text
Wi-Fi Client

      │

      ▼

wlan0

      │

      ▼

iptables NAT

      │

      ▼

eth0

      │

      ▼

Internet
```

This allows connected wireless devices to access the Internet through the Ethernet connection.

---

# Successful Boot Indicators

A successful boot can be confirmed by checking the following.

| Verification | Expected Result |
|--------------|-----------------|
| Power LED | ON |
| Activity LED | Blinks During Boot |
| Linux Login Prompt | Available |
| SSID Visible | Yes |
| DHCP Working | Yes |
| Internet Sharing | Yes |
| SSH Accessible | Yes |

---

# Verifying Running Services

Login to the Raspberry Pi and verify the status of each service.

Check hostapd:

```bash
systemctl status hostapd
```

Check dnsmasq:

```bash
systemctl status dnsmasq
```

Check custom network service:

```bash
systemctl status rdkb-network
```

All services should display:

```text
Active: active (running)
```

---

# Verify Wireless Interface

Check the wireless interface.

```bash
ip addr show wlan0
```

Expected:

```text
192.168.10.1/24
```

---

# Verify IP Forwarding

Confirm that IPv4 forwarding is enabled.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected output:

```text
1
```

---

# Verify NAT Rules

Display the configured NAT rules.

```bash
iptables -t nat -L
```

Expected:

```text
MASQUERADE
```

---

# Connect a Wireless Device

Using a mobile phone or laptop:

1. Scan available Wi-Fi networks.
2. Locate:

```text
RDKB_AP
```

3. Connect using:

```text
Password:

12345678
```

After successful authentication:

- DHCP assigns an IP address.
- Internet access becomes available through the Ethernet connection.

---

# Challenges Faced During Boot

Several issues were encountered while bringing up the system.

## hostapd Started but No SSID Visible

Cause:

The wireless interface was not configured before hostapd started.

Solution:

The interface initialization was moved to the custom network script.

---

## Devices Connected but No Internet

Cause:

IP forwarding and NAT were missing.

Solution:

Added the following to `rdkb-network.sh`:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

After rebuilding the image, Internet access worked correctly.

---

## Old Network Script Loaded

Cause:

BitBake used cached build artifacts.

Solution:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

This ensured the updated script was included in the image.

---

# Final Boot Result

After completing all modifications and resolving the issues encountered during development, the Raspberry Pi successfully booted into the custom RDK-B image.

The system automatically:

- Booted Linux.
- Started systemd.
- Configured the wireless interface.
- Enabled the Access Point.
- Assigned IP addresses through DHCP.
- Shared Internet access using NAT.
- Allowed multiple wireless clients to connect simultaneously.

No manual configuration was required after power-on.

---

# Summary

The boot process verified the successful integration of the Yocto-built RDK-B image with the Raspberry Pi hardware. All custom services started automatically, the wireless Access Point became available immediately after boot, and connected clients received IP addresses with Internet access through Ethernet. This confirmed that the image was fully functional and ready for further testing and validation.