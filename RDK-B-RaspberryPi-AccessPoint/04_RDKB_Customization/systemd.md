# systemd

## Introduction

**systemd** is the default initialization (init) system used in modern Linux distributions. It is the first userspace process started by the Linux kernel during boot and is responsible for starting, stopping, and managing all system services.

In this project, systemd plays a critical role in automatically configuring the Raspberry Pi as a Wi-Fi Access Point after every boot. It ensures that the network interface is configured before starting the HostAPD and DNSMasq services.

Without systemd, all services would need to be started manually after each reboot.

---

# What is systemd?

systemd is responsible for:

- System initialization
- Service management
- Process supervision
- Dependency management
- Automatic service startup
- Logging
- Boot sequence management

It replaces the older SysV init system and provides faster and more reliable service management.

---

# Why systemd was Used

The Raspberry Pi Access Point should start automatically whenever the board powers on.

The following tasks must occur automatically:

- Configure the Wi-Fi interface
- Assign the static IP address
- Enable IP forwarding
- Configure NAT
- Start HostAPD
- Start DNSMasq

Instead of running these commands manually, systemd performs them automatically during boot.

---

# Boot Sequence

The overall boot process is shown below.

```text
Power ON

      │

      ▼

Bootloader (U-Boot)

      │

      ▼

Linux Kernel

      │

      ▼

Mount Root Filesystem

      │

      ▼

systemd Starts

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

Broadcast SSID

      │

      ▼

dnsmasq.service

      │

      ▼

DHCP Server Starts

      │

      ▼

Access Point Ready
```

---

# systemd in Yocto

The default Poky distribution originally supports both SysV init and systemd.

For this project, systemd was explicitly enabled.

The following configuration was added to:

```text
build/conf/local.conf
```

```text
INIT_MANAGER = "systemd"

DISTRO_FEATURES:append = " systemd"

DISTRO_FEATURES_BACKFILL_CONSIDERED += "sysvinit"

VIRTUAL-RUNTIME_init_manager = "systemd"

VIRTUAL-RUNTIME_initscripts = ""

DISTRO_FEATURES:remove = "sysvinit"
```

These settings ensure that the generated image uses systemd as the only init system.

---

# Verifying systemd

After configuring Yocto, the following commands were used to verify the build configuration.

```bash
bitbake -e core-image-rdkb-ap | grep "^INIT_MANAGER="
```

Output:

```text
INIT_MANAGER="systemd"
```

---

```bash
bitbake -e core-image-rdkb-ap | grep "^VIRTUAL-RUNTIME_init_manager="
```

Output:

```text
VIRTUAL-RUNTIME_init_manager="systemd"
```

---

```bash
bitbake -e core-image-rdkb-ap | grep "^DISTRO_FEATURES="
```

The output contained:

```text
systemd
```

confirming that systemd support was enabled.

---

# Custom systemd Service

A custom service named:

```text
rdkb-network.service
```

was created.

Its purpose is to configure the Raspberry Pi network before HostAPD and DNSMasq start.

---

# Service Location

```text
/lib/systemd/system/rdkb-network.service
```

---

# Service File

The final service used in this project is shown below.

```ini
[Unit]
Description=RDK-B Network Configuration
Before=hostapd.service dnsmasq.service

[Service]
Type=oneshot
ExecStart=/usr/bin/rdkb-network.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

---

# Understanding the Service

## [Unit]

```ini
[Unit]
```

Defines metadata and dependencies for the service.

---

## Description

```ini
Description=RDK-B Network Configuration
```

Provides a human-readable description.

Displayed when checking service status.

---

## Before

```ini
Before=hostapd.service dnsmasq.service
```

This is one of the most important lines.

It ensures:

```text
Network Configuration

        │

        ▼

HostAPD

        │

        ▼

DNSMasq
```

If HostAPD starts before the network is configured, it cannot initialize the wireless interface correctly.

---

# [Service]

```ini
[Service]
```

Defines how the service should execute.

---

## Type

```ini
Type=oneshot
```

The script executes only once during boot.

After completing successfully, the service exits.

---

## ExecStart

```ini
ExecStart=/usr/bin/rdkb-network.sh
```

Runs the network initialization script.

The script performs:

- Bring up wlan0
- Assign IP address
- Enable IP forwarding
- Configure NAT

---

## RemainAfterExit

```ini
RemainAfterExit=yes
```

Even after the script finishes, systemd considers the service active.

This allows dependency tracking for other services.

---

# [Install]

```ini
WantedBy=multi-user.target
```

Ensures the service starts during normal system boot.

---

# Network Initialization Script

The service executes:

```text
/usr/bin/rdkb-network.sh
```

Final version:

```bash
#!/bin/sh

ip link set wlan0 up

ip addr flush dev wlan0

ip addr add 192.168.10.1/24 dev wlan0

echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
    -m state --state RELATED,ESTABLISHED -j ACCEPT

exit 0
```

---

# Service Installation

The custom BitBake recipe installs the service automatically.

Inside:

```text
rdkb-ap-config.bb
```

```bitbake
inherit systemd

SYSTEMD_SERVICE:${PN} = "rdkb-network.service"

SYSTEMD_AUTO_ENABLE:${PN} = "enable"
```

This automatically enables the service during image creation.

Equivalent Linux command:

```bash
systemctl enable rdkb-network.service
```

No manual intervention is required.

---

# Service Dependency Flow

```text
systemd

      │

      ▼

rdkb-network.service

      │

      ▼

Network Configured

      │

      ▼

hostapd.service

      │

      ▼

SSID Broadcast

      │

      ▼

dnsmasq.service

      │

      ▼

DHCP Server Ready

      │

      ▼

Clients Connect
```

---

# Verifying the Service

Check service status:

```bash
systemctl status rdkb-network.service
```

Expected:

```text
Active: active (exited)
```

Because the service is a **oneshot** service.

---

# Checking Service Enablement

```bash
systemctl is-enabled rdkb-network.service
```

Expected output:

```text
enabled
```

---

# Checking Installed Service

During image verification, the following file was confirmed.

```text
/lib/systemd/system/rdkb-network.service
```

The symbolic link was also verified.

```text
/etc/systemd/system/multi-user.target.wants/
```

This confirmed that the service would automatically start after every boot.

---

# Challenges Encountered

## Missing systemd

Initially, the Yocto image was built using the default init system.

Commands such as:

```bash
systemctl status
```

failed because systemd was not included in the image.

The issue was resolved by enabling systemd in `local.conf` and rebuilding the image.

---

## Initial Service Design

The first implementation attempted to start HostAPD and DNSMasq directly from the custom service using multiple `ExecStart` commands.

Example:

```text
ExecStart=/usr/sbin/hostapd ...

ExecStart=/usr/bin/dnsmasq ...
```

This approach caused service management issues and duplicated functionality already provided by the native systemd services.

The final design simplified the architecture:

- `rdkb-network.service` performs only network configuration.
- `hostapd.service` manages the wireless daemon.
- `dnsmasq.service` manages DHCP and DNS.

This separation improved reliability and made debugging easier.

---

# Advantages of Using systemd

Using systemd provides several advantages:

- Automatic startup after boot
- Dependency management
- Reliable service supervision
- Faster boot process
- Standard Linux service management
- Easy debugging using `systemctl`
- Seamless integration with Yocto

---

# Summary

systemd serves as the initialization and service management framework for the Raspberry Pi Access Point. It ensures that network configuration, wireless services, and DHCP services start automatically in the correct order every time the system boots. By enabling systemd in Yocto and creating a dedicated `rdkb-network.service`, the project achieved a fully automated boot process, eliminating manual configuration and providing a reliable foundation for the custom RDK-B Access Point implementation.