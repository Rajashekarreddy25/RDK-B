# Project Architecture

## 1. Introduction

A software architecture describes how different hardware and software components interact to accomplish a specific task. In this project, a Raspberry Pi 4 running a custom Yocto-based Embedded Linux distribution functions as a Wireless Access Point (AP). The architecture integrates multiple networking components such as **hostapd**, **dnsmasq**, **systemd**, and **iptables** to provide wireless connectivity and Internet sharing.

The complete system can be divided into five major layers:

- Hardware Layer
- Boot Layer
- Operating System Layer
- Networking Layer
- Client Layer

Each layer performs a specific role and communicates with adjacent layers to provide the final functionality.

---

# 2. Overall Project Architecture

```text
                        +----------------------+
                        |      Internet        |
                        +----------+-----------+
                                   |
                                   |
                            Ethernet (eth0)
                                   |
                     +-------------+--------------+
                     | Raspberry Pi 4             |
                     |----------------------------|
                     | Yocto Linux Image          |
                     |                            |
                     | systemd                    |
                     | hostapd                    |
                     | dnsmasq                    |
                     | iptables                   |
                     | Linux Kernel               |
                     +-------------+--------------+
                                   |
                              Wi-Fi (wlan0)
                                   |
              +--------------------+--------------------+
              |                    |                    |
          Mobile Phone         Laptop             IoT Device
```

The Raspberry Pi receives Internet through its Ethernet interface and shares it with wireless clients through its Wi-Fi interface.

---

# 3. Hardware Architecture

The hardware used in this project consists of:

```text
               Ubuntu Build Host
                       |
               Build Yocto Image
                       |
                 Bootable SD Card
                       |
               Raspberry Pi 4
              /               \
        Ethernet           Wi-Fi
            |                 |
      Internet Router     Wireless Clients
```

### Components

| Component | Purpose |
|------------|---------|
| Ubuntu Host | Builds Yocto Image |
| SD Card | Stores Linux Image |
| Raspberry Pi 4 | Target Hardware |
| Ethernet | Internet Connection |
| Wi-Fi | Access Point |
| Client Devices | Connect to AP |

---

# 4. Software Stack

The software stack consists of multiple layers.

```text
+----------------------------------+
| User Applications                |
+----------------------------------+
| hostapd                          |
| dnsmasq                          |
| iptables                         |
+----------------------------------+
| systemd                          |
+----------------------------------+
| Linux Kernel                     |
+----------------------------------+
| Raspberry Pi Hardware            |
+----------------------------------+
```

Each layer depends on the services provided by the layer below it.

---

# 5. Yocto Architecture

The Linux image is generated using the Yocto Project.

```text
meta-rdkb-ap
        |
meta-raspberrypi
        |
meta-openembedded
        |
Poky
        |
BitBake
        |
Linux Image
```

### Description

**Poky**

Provides the reference build system.

**BitBake**

Compiles every package according to its recipe.

**meta-openembedded**

Provides additional software packages.

**meta-raspberrypi**

Adds Raspberry Pi hardware support.

**meta-rdkb-ap**

Contains all project-specific customizations.

---

# 6. Boot Architecture

The Raspberry Pi follows the boot sequence below.

```text
Power ON
     |
     v
Boot ROM
     |
     v
Raspberry Pi Firmware
     |
     v
Linux Kernel
     |
     v
Root Filesystem
     |
     v
systemd
     |
     v
rdkb-network.service
     |
     +----------------------+
     |                      |
hostapd               dnsmasq
     |
     v
Wireless AP Ready
```

---

# 7. Linux Boot Flow

The operating system initialization follows this sequence.

```text
Power

↓

Bootloader

↓

Linux Kernel

↓

Initialize CPU

↓

Initialize RAM

↓

Initialize Device Drivers

↓

Mount Root Filesystem

↓

Start systemd

↓

Start Network Services

↓

hostapd

↓

dnsmasq

↓

Access Point Ready
```

---

# 8. Network Architecture

The network topology is shown below.

```text
                Internet
                    |
                    |
              Router (DHCP)
                    |
             192.168.1.x
                    |
                 eth0
                    |
      +---------------------------+
      | Raspberry Pi              |
      |                           |
      | eth0  -> Internet         |
      | wlan0 -> Access Point     |
      +-------------+-------------+
                    |
           192.168.10.1
                    |
         DHCP Server (dnsmasq)
                    |
      +------+------+------+
      |             |      |
   Phone        Laptop   Tablet
```

---

# 9. Access Point Architecture

```text
              Wireless Clients
                     |
             WPA2 Authentication
                     |
                hostapd
                     |
             Linux Wireless Driver
                     |
                 wlan0 Interface
                     |
               Raspberry Pi Wi-Fi
```

hostapd is responsible for broadcasting the SSID and authenticating wireless clients.

---

# 10. DHCP Architecture

```text
Client

↓

DHCP Discover

↓

dnsmasq

↓

DHCP Offer

↓

Client Request

↓

DHCP ACK

↓

Client Receives IP
```

dnsmasq automatically assigns IP addresses from the configured DHCP pool.

---

# 11. NAT Architecture

```text
Wireless Client

↓

192.168.10.x

↓

wlan0

↓

iptables

↓

eth0

↓

Internet
```

iptables translates private IP addresses into the Raspberry Pi's Ethernet address, allowing wireless clients to access the Internet.

---

# 12. Packet Flow

When a client accesses a website, packets flow through the system as follows:

```text
Phone

↓

Wi-Fi

↓

hostapd

↓

Linux Kernel

↓

iptables

↓

eth0

↓

Internet

↓

Website

↓

Internet

↓

eth0

↓

iptables

↓

wlan0

↓

Phone
```

---

# 13. systemd Service Flow

systemd manages service startup during boot.

```text
systemd

↓

rdkb-network.service

↓

Configure wlan0

↓

Enable IP Forwarding

↓

Configure iptables

↓

hostapd.service

↓

dnsmasq.service

↓

Access Point Ready
```

---

# 14. File Organization

```text
meta-rdkb-ap

├── conf
│
├── recipes-core
│      └── images
│
├── recipes-connectivity
│      ├── hostapd
│      └── rdkb-ap-config
│
└── files
```

---

# 15. Component Responsibilities

| Component | Responsibility |
|------------|----------------|
| Yocto | Build Linux Image |
| BitBake | Compile Packages |
| Raspberry Pi BSP | Hardware Support |
| Linux Kernel | Operating System Core |
| systemd | Service Manager |
| hostapd | Wireless Access Point |
| dnsmasq | DHCP Server |
| iptables | NAT Configuration |
| rdkb-network.sh | Configure Network |
| rdkb-network.service | Automatic Startup |

---

# 16. Advantages of This Architecture

The implemented architecture provides several advantages:

- Modular design
- Easy customization
- Small Linux image
- Automatic boot configuration
- Reliable service management
- Multiple client support
- Easy debugging
- Reproducible builds
- Open-source software stack
- Scalable architecture

---

# 17. Limitations

Although the project successfully implements the core functionality of an Access Point, it is not a complete RDK-B broadband gateway.

The following features are not implemented:

- CCSP Framework
- TR-181 Data Model
- Web Management Interface
- Remote Device Management
- WPA3 Authentication
- IPv6 Routing
- Firewall Rules
- Captive Portal
- Mesh Networking

These can be added in future enhancements.

---

# 18. Summary

This project architecture combines the flexibility of the Yocto Project with the networking capabilities of Linux to create a lightweight Embedded Linux distribution capable of functioning as a Wireless Access Point. The layered design ensures that each software component has a clearly defined responsibility, making the system easier to understand, maintain, and extend. The resulting implementation demonstrates the core principles used in broadband gateway software such as RDK-B while remaining simple enough for educational and experimental purposes.