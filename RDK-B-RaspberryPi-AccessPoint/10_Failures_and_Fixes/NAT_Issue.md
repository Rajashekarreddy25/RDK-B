# NAT_Issue

## Introduction

One of the most significant issues encountered during the implementation of the RDK-B Raspberry Pi Access Point project was related to **Network Address Translation (NAT)**.

Initially, the Access Point appeared to function correctly:

- The Raspberry Pi booted successfully.
- The SSID was visible.
- Clients connected successfully.
- DHCP assigned IP addresses.
- Gateway and DNS were configured correctly.

However, Internet access was still unavailable.

Further investigation revealed that the problem was not the NAT rules themselves, but that the **`iptables` package was completely missing from the generated Yocto image**. Since `iptables` was absent, the NAT commands inside the custom startup script failed silently, preventing Internet sharing.

This document describes the complete debugging process, image verification, root cause, and final solution.

---

# Problem Description

After completing the Access Point setup:

```text
✔ Raspberry Pi Booted

✔ SSID Visible

✔ DHCP Working

✔ Clients Connected

✔ IP Address Assigned

✘ No Internet Access
```

The custom script contained NAT commands:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

but Internet sharing did not work.

---

# Initial Investigation

The first step was to verify whether `iptables` existed on the Raspberry Pi.

```bash
which iptables
```

Output:

```text
Command not found
```

Another verification:

```bash
iptables --version
```

Output:

```text
iptables: command not found
```

This confirmed that the executable was missing.

---

# Verify Startup Script

The custom network script was inspected.

```bash
cat /usr/bin/rdkb-network.sh
```

The script already contained:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

The script itself was correct.

The missing executable prevented these commands from running.

---

# Root Cause Analysis

The problem was not:

- hostapd
- dnsmasq
- DHCP
- systemd
- Startup script

Instead, the required package:

```text
iptables
```

had never been installed into the Yocto image.

As a result:

```text
rdkb-network.sh

↓

iptables command executed

↓

Executable missing

↓

NAT not configured

↓

No Internet
```

---

# Verify Image Recipe

The custom image recipe was inspected.

Example:

```bash
nano core-image-rdkb-ap.bb
```

IMAGE_INSTALL initially contained:

```bitbake
IMAGE_INSTALL += "\
hostapd \
dnsmasq \
iw \
rdkb-ap-config \
"
```

The following package was missing:

```text
iptables
```

---

# Solution

The image recipe was updated.

```bitbake
IMAGE_INSTALL += "\
hostapd \
dnsmasq \
iw \
iptables \
rdkb-ap-config \
"
```

This ensured that the firewall utility would be installed into every generated image.

---

# Rebuild Process

Clean build.

```bash
bitbake -c clean core-image-rdkb-ap
```

Rebuild image.

```bash
bitbake core-image-rdkb-ap
```

The build completed successfully.

---

# Image Verification

Before flashing the SD card, the generated image was verified.

Navigate to the deploy directory.

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64
```

Copy the generated image.

```bash
cp core-image-rdkb-ap-*.rootfs.wic.bz2 verify.wic.bz2
```

Extract the image.

```bash
bunzip2 verify.wic.bz2
```

Attach the image to a loop device.

```bash
sudo losetup -Pf --show verify.wic
```

Output:

```text
/dev/loop28
```

Create a mount point.

```bash
sudo mkdir -p /mnt/verify-root
```

Mount the root filesystem.

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

---

# Verify iptables Installation

Search for the executable.

```bash
find /mnt/verify-root -name iptables
```

Output:

```text
/mnt/verify-root/usr/sbin/iptables
```

This confirmed that the package had been successfully installed.

---

# Verify Network Script

Inspect the installed startup script.

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

The script contained:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

Everything required for Internet sharing was now present.

---

# Verify systemd Service

Check that the service was enabled inside the image.

```bash
ls -l /mnt/verify-root/etc/systemd/system/multi-user.target.wants
```

Output included:

```text
hostapd.service

dnsmasq.service

iptables.service

rdkb-network.service
```

This confirmed that all required services would start automatically after boot.

---

# Flash Updated Image

After verification:

Unmount the filesystem.

```bash
sudo umount /mnt/verify-root
```

Detach the loop device.

```bash
sudo losetup -d /dev/loop28
```

Flash the updated image to the SD card.

Boot the Raspberry Pi.

---

# Verification After Boot

Verify `iptables`.

```bash
which iptables
```

Output:

```text
/usr/sbin/iptables
```

Display NAT rules.

```bash
iptables -t nat -L
```

Expected Output:

```text
Chain POSTROUTING

MASQUERADE
```

Display forwarding rules.

```bash
iptables -L
```

Expected:

```text
ACCEPT
```

---

# Network Flow After Fix

```text
Client

      │

      ▼

wlan0

      │

      ▼

Packet Forwarding

      │

      ▼

iptables NAT

      │

      ▼

eth0

      │

      ▼

Internet Router

      │

      ▼

Internet
```

---

# Testing Performed

After rebuilding the image:

### Test 1

Android Phone

```text
PASS
```

Internet working.

---

### Test 2

Windows Laptop

```text
PASS
```

Internet working.

---

### Test 3

Ubuntu Laptop

```text
PASS
```

Internet working.

---

### Test 4

Multiple Devices

```text
6 Devices Connected

PASS
```

All devices accessed the Internet simultaneously.

---

### Test 5

Reboot

```text
PASS
```

NAT rules applied automatically after startup.

---

# Commands Used During Debugging

Verify installed package.

```bash
which iptables
```

Search image contents.

```bash
find /mnt/verify-root -name iptables
```

Mount image.

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Display startup script.

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Display enabled services.

```bash
ls -l /mnt/verify-root/etc/systemd/system/multi-user.target.wants
```

Display NAT rules.

```bash
iptables -t nat -L
```

---

# Lessons Learned

This issue provided several important insights:

- A successful Yocto build does not guarantee that all required packages are included.
- Image verification before flashing can save hours of debugging.
- Missing runtime utilities can cause startup scripts to fail silently.
- `find`, `losetup`, and `mount` are extremely useful for inspecting generated images.
- Always verify `IMAGE_INSTALL` before rebuilding.
- Internet sharing depends on both correct configuration **and** the presence of required packages.

---

# Best Practices

- Verify image contents before flashing to hardware.
- Confirm critical executables (`iptables`, `hostapd`, `dnsmasq`) exist in the root filesystem.
- Keep networking utilities explicitly listed in `IMAGE_INSTALL`.
- Inspect startup scripts inside the generated image.
- Test generated images before deploying to hardware.
- Document every missing package encountered during development.

---

# Final Result

After adding the **`iptables`** package to `IMAGE_INSTALL`, rebuilding the Yocto image, and verifying its presence before flashing:

- The `iptables` executable was included in the final image.
- NAT rules were successfully applied during system startup.
- Packet forwarding operated correctly.
- Wireless clients accessed the Internet through the Ethernet connection.
- Up to **six client devices** connected simultaneously and shared the Internet without issues.

This issue represented the final major networking challenge encountered during the implementation of the RDK-B Raspberry Pi Access Point project. Resolving it completed the full Internet sharing functionality of the custom embedded Linux system.