# Image_Verification

## Introduction

Building the image successfully does not guarantee that all the required files and configurations have been included correctly. Before flashing the image onto the Raspberry Pi, it is important to verify the contents of the generated image.

Image verification allows developers to inspect the filesystem without booting the Raspberry Pi. This helps identify missing packages, incorrect configurations, or installation errors early in the development process.

During this project, the generated image was mounted on the Ubuntu host system using a loop device. The filesystem was then inspected to ensure that all required files were present.

---

# Why Image Verification is Important

Image verification helps to:

- Confirm that the image was generated successfully.
- Verify that required packages are installed.
- Ensure configuration files exist.
- Check systemd services.
- Validate custom scripts.
- Detect packaging errors before flashing.

Skipping this step can result in unnecessary flashing and debugging on the target hardware.

---

# Verification Workflow

```text
BitBake Build

      │

      ▼

Generated .wic.bz2 Image

      │

      ▼

Decompress Image

      │

      ▼

Create Loop Device

      │

      ▼

Mount Root Filesystem

      │

      ▼

Verify Files

      │

      ▼

Flash to SD Card
```

---

# Generated Image

After the build completed successfully, the image was available at:

```text
~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64/
```

List the generated files.

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64

ls -lh
```

Example output:

```text
core-image-rdkb-ap-raspberrypi4-64.rootfs.wic.bz2

core-image-rdkb-ap.wic.bz2

core-image-rdkb-ap.wic.bmap
```

---

# Create a Working Copy

Instead of modifying the original image, create a copy.

```bash
cp core-image-rdkb-ap-raspberrypi4-64-*.rootfs.wic.bz2 verify.wic.bz2
```

This ensures the original build artifact remains unchanged.

---

# Decompress the Image

Extract the compressed image.

```bash
bunzip2 verify.wic.bz2
```

After extraction:

```bash
ls -lh verify.wic
```

Example:

```text
verify.wic
```

---

# Create a Loop Device

Attach the image to a loop device.

```bash
sudo losetup -Pf --show verify.wic
```

Example output:

```text
/dev/loop28
```

---

# Verify Partitions

Display the partitions contained in the image.

```bash
lsblk /dev/loop28
```

Example output:

```text
loop28
├── loop28p1
└── loop28p2
```

Partition details:

| Partition | Purpose |
|-----------|----------|
| loop28p1 | Boot Partition |
| loop28p2 | Root Filesystem |

---

# Mount the Root Filesystem

Create a mount point.

```bash
sudo mkdir -p /mnt/verify-root
```

Mount the root filesystem.

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Now the complete Linux filesystem can be inspected.

---

# Verify hostapd

Search for the HostAPD binary.

```bash
find /mnt/verify-root -name hostapd
```

Expected output:

```text
/usr/sbin/hostapd
```

This confirms that HostAPD has been installed successfully.

---

# Verify dnsmasq

Locate the DNSMasq executable.

```bash
find /mnt/verify-root -name dnsmasq
```

Expected output:

```text
/usr/bin/dnsmasq
```

---

# Verify iptables

Locate the iptables binary.

```bash
find /mnt/verify-root -name iptables
```

Expected output:

```text
/usr/sbin/iptables
```

This confirms that NAT support is available in the image.

---

# Verify systemctl

Check whether systemd is present.

```bash
find /mnt/verify-root -name systemctl
```

Expected output:

```text
/bin/systemctl
```

This confirms that systemd is included.

---

# Verify Custom Network Script

Search for the custom network initialization script.

```bash
find /mnt/verify-root -name rdkb-network.sh
```

Expected output:

```text
/usr/bin/rdkb-network.sh
```

---

# Verify Network Service

Locate the custom systemd service.

```bash
find /mnt/verify-root -name rdkb-network.service
```

Expected output:

```text
/lib/systemd/system/rdkb-network.service
```

---

# Verify HostAPD Configuration

Check that the HostAPD configuration file exists.

```bash
ls -l /mnt/verify-root/etc/hostapd.conf
```

Example output:

```text
/etc/hostapd.conf
```

Display the contents.

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Example:

```text
interface=wlan0
driver=nl80211

ssid=RDKB_AP

hw_mode=g
channel=6

country_code=IN

ieee80211n=1

auth_algs=1

wpa=2
wpa_passphrase=12345678
```

This confirms that the wireless configuration was packaged correctly.

---

# Verify DNSMasq Configuration

Check the DNSMasq configuration directory.

```bash
ls -l /mnt/verify-root/etc/dnsmasq.d
```

Expected output:

```text
rdkb-ap.conf
```

Display the configuration.

```bash
cat /mnt/verify-root/etc/dnsmasq.d/rdkb-ap.conf
```

Typical configuration:

```text
interface=wlan0

dhcp-range=192.168.10.10,192.168.10.100,12h
```

---

# Verify systemd Service Files

List the important services.

```bash
ls -l /mnt/verify-root/lib/systemd/system | grep -E "hostapd|dnsmasq|rdkb"
```

Expected output:

```text
dnsmasq.service

hostapd.service

rdkb-network.service
```

This confirms that all required services are installed.

---

# Verify Enabled Services

Check which services will start automatically.

```bash
ls -l /mnt/verify-root/etc/systemd/system/multi-user.target.wants
```

Expected output:

```text
hostapd.service

dnsmasq.service

rdkb-network.service
```

These symbolic links indicate that the services are enabled during boot.

---

# Verify Network Script Contents

Display the installed network script.

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Verify that it contains:

- Interface configuration
- Static IP assignment
- IPv4 forwarding
- NAT rules
- Forwarding rules

This confirms that the latest version of the script has been packaged.

---

# Verify Package Installation

The mounted image confirmed that the following components were installed successfully.

| Component | Verification |
|-----------|--------------|
| hostapd | ✓ Present |
| dnsmasq | ✓ Present |
| systemd | ✓ Present |
| iptables | ✓ Present |
| rdkb-network.sh | ✓ Present |
| hostapd.conf | ✓ Present |
| dnsmasq Configuration | ✓ Present |
| rdkb-network.service | ✓ Present |

---

# Unmount the Image

After verification, unmount the filesystem.

```bash
sudo umount /mnt/verify-root
```

---

# Remove the Loop Device

Detach the loop device.

```bash
sudo losetup -d /dev/loop28
```

Replace `/dev/loop28` with the loop device assigned on your system.

---

# Challenges Faced During Verification

Several issues were identified during image verification while developing this project.

## Missing iptables

Initially, searching the mounted filesystem showed that `iptables` was not installed.

Cause:

The package had not been added to the image recipe.

Solution:

```bitbake
IMAGE_INSTALL += "iptables"
```

After rebuilding the image, the binary appeared in:

```text
/usr/sbin/iptables
```

---

## Old Version of Network Script

Even after modifying `rdkb-network.sh`, the mounted image still contained the previous version.

Cause:

BitBake reused cached build artifacts.

Solution:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

The rebuilt image contained the updated script.

---

## Missing Configuration Files

Early versions of the image lacked some configuration files because they were not installed correctly in the custom recipe.

Verification of the mounted filesystem helped identify these issues before flashing the SD card.

---

# Summary

Image verification was an essential step in this project. By mounting the generated `.wic` image and inspecting its filesystem, all required binaries, configuration files, systemd services, and custom scripts were confirmed before deployment. This process reduced debugging time on the Raspberry Pi and ensured that the final image contained the complete software stack required to operate as a wireless Access Point.