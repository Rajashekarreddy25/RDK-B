# Missing_systemd

## Introduction

During the development of the RDK-B Raspberry Pi Access Point, one of the first major runtime issues was that the custom network initialization script did not execute automatically after the Raspberry Pi booted.

Although the image built successfully and booted without errors, the Wi-Fi Access Point was not configured because the initialization script (`rdkb-network.sh`) was never executed.

This issue was traced back to the absence of a custom **systemd service** responsible for starting the network configuration script during system startup.

This document explains the issue, investigation process, root cause, and final solution.

---

# Problem Description

After flashing the generated Yocto image and booting the Raspberry Pi:

- Linux booted successfully.
- Ethernet interface was detected.
- Wi-Fi hardware was detected.
- hostapd package was installed.
- dnsmasq package was installed.
- No Access Point was created.

The Raspberry Pi booted normally, but clients could not discover the configured SSID.

---

# Symptoms

The following observations were made after booting:

```text
✔ Linux Booted

✔ Ethernet Interface Available

✔ Wireless Interface Detected

✘ SSID Not Visible

✘ wlan0 Not Configured

✘ Static IP Missing

✘ Internet Sharing Disabled
```

---

# Initial Investigation

The first step was to verify whether the required packages had been installed correctly.

Check hostapd:

```bash
systemctl status hostapd
```

Check dnsmasq:

```bash
systemctl status dnsmasq
```

Both services existed.

Next, verify the Wi-Fi interface.

```bash
ip addr
```

Result:

```text
wlan0
```

The interface existed but did not have the expected IP address:

```text
192.168.10.1/24
```

---

# Manual Verification

The network script was executed manually.

```bash
/usr/bin/rdkb-network.sh
```

Immediately after running the script:

```text
wlan0 received IP address

Packet forwarding enabled

NAT rules created
```

The Access Point began functioning correctly.

This confirmed that the script itself was correct.

---

# Root Cause Analysis

Since manual execution worked correctly, the problem was not:

- hostapd
- dnsmasq
- iptables
- network configuration

Instead, the issue was that nothing executed the script automatically during boot.

The build included the script:

```text
/usr/bin/rdkb-network.sh
```

but there was no service responsible for launching it.

---

# Solution

A custom **systemd service** was created.

File:

```text
rdkb-network.service
```

Location:

```text
meta-rdkb-ap/

recipes-connectivity/

rdkb-ap-config/

files/
```

---

# Service File

```ini
[Unit]
Description=RDK-B Network Initialization
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/rdkb-network.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

---

# Recipe Modification

The recipe was updated to install the service.

Example:

```bitbake
SYSTEMD_SERVICE:${PN} = "rdkb-network.service"
```

and

```bitbake
inherit systemd
```

The service file was copied into:

```text
/lib/systemd/system/
```

---

# Enable Service During Installation

Inside the recipe:

```bitbake
SYSTEMD_AUTO_ENABLE = "enable"
```

This automatically created the symbolic link:

```text
/etc/systemd/system/

multi-user.target.wants/

rdkb-network.service
```

---

# Rebuild Process

After updating the recipe:

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild image:

```bash
bitbake core-image-rdkb-ap
```

Flash SD card.

Boot Raspberry Pi.

---

# Verification

Check service.

```bash
systemctl status rdkb-network.service
```

Expected:

```text
Loaded: loaded

Active: active (exited)
```

Verify symbolic link.

```bash
ls -l /etc/systemd/system/multi-user.target.wants
```

Output:

```text
rdkb-network.service
```

---

# Boot Sequence After Fix

```text
Power ON

      │

      ▼

Linux Boot

      │

      ▼

systemd Starts

      │

      ▼

rdkb-network.service

      │

      ▼

rdkb-network.sh

      │

      ▼

Configure wlan0

      │

      ▼

Enable IP Forwarding

      │

      ▼

Configure NAT

      │

      ▼

Access Point Ready
```

---

# Lessons Learned

During this issue, the following important concepts were learned:

- Installing a script does not guarantee its execution.
- System services are responsible for starting applications automatically during boot.
- Yocto recipes must explicitly register services with systemd.
- `SYSTEMD_AUTO_ENABLE` simplifies automatic service activation.
- Verifying symbolic links under `multi-user.target.wants` confirms whether a service is enabled.

---

# Best Practices

- Create a dedicated systemd service for every custom startup script.
- Always enable the service during image creation instead of enabling it manually after boot.
- Verify service status immediately after boot.
- Keep startup scripts modular and executable.
- Use `journalctl` to diagnose service failures.

---

# Final Result

After implementing the custom `rdkb-network.service`, the network initialization script executed automatically during every boot.

As a result:

- Wi-Fi interface was configured automatically.
- Static IP address was assigned.
- Packet forwarding was enabled.
- NAT rules were applied.
- The Access Point became operational without requiring any manual intervention.

This resolved the first major runtime issue encountered during the development of the RDK-B Raspberry Pi Access Point project.