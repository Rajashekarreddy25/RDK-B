# Output Images

## Introduction

After the successful completion of the Yocto build process, BitBake generates several output files. These files include the Linux kernel, root filesystem, bootable disk images, device tree files, package manifests, and debugging information.

All generated images are stored in the deployment directory and are used for flashing the Raspberry Pi or verifying the build before deployment.

For this project, the final output image contained the customized RDK-B Access Point configuration, including `hostapd`, `dnsmasq`, `iptables`, `iw`, and the custom `rdkb-ap-config` package.

---

# Output Directory

After building the project, all generated files are available in:

```text
poky/build/tmp/deploy/images/raspberrypi4-64/
```

Example:

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64
```

List all generated files:

```bash
ls -lh
```

---

# Typical Output Files

The deployment directory contains several files.

Example:

```text
Image

bcm2711-rpi-4-b.dtb

core-image-rdkb-ap.rootfs.wic.bz2

core-image-rdkb-ap.rootfs.ext3

core-image-rdkb-ap.rootfs.tar.bz2

core-image-rdkb-ap.manifest

core-image-rdkb-ap.testdata.json

core-image-rdkb-ap.wic.bmap
```

Each file has a specific purpose.

---

# Linux Kernel Image

File:

```text
Image
```

Purpose:

- Linux Kernel
- Loaded by the Raspberry Pi bootloader
- Initializes hardware
- Starts the operating system

---

# Device Tree Blob (DTB)

Example:

```text
bcm2711-rpi-4-b.dtb
```

Purpose:

- Describes Raspberry Pi hardware
- CPU
- Memory
- GPIO
- USB
- Wi-Fi
- Ethernet

The kernel reads this file during boot.

---

# Root Filesystem Image

Example:

```text
core-image-rdkb-ap.rootfs.ext3
```

Purpose:

Contains the Linux root filesystem.

Example directories:

```text
/

├── bin

├── etc

├── home

├── lib

├── proc

├── sys

├── usr

├── var
```

---

# TAR Root Filesystem

Example:

```text
core-image-rdkb-ap.rootfs.tar.bz2
```

Purpose:

Compressed archive of the root filesystem.

Useful for:

- Inspection
- Backup
- Deployment
- Container images

---

# Bootable WIC Image

Example:

```text
core-image-rdkb-ap.rootfs.wic.bz2
```

Purpose:

Complete bootable Raspberry Pi disk image.

Contains:

```text
Boot Partition

Root Filesystem

Kernel

Device Tree

Bootloader Files
```

This is the image flashed onto the microSD card.

---

# WIC Image Layout

The `.wic` image contains multiple partitions.

Example:

```text
+--------------------------+
| Boot Partition           |
| FAT32                    |
| Kernel                   |
| DTB                      |
| Boot Files               |
+--------------------------+

| Root Filesystem          |
| EXT4                     |
| Linux OS                 |
| hostapd                  |
| dnsmasq                  |
| systemd                  |
| rdkb-ap-config           |
+--------------------------+
```

---

# Build Manifest

Example:

```text
core-image-rdkb-ap.manifest
```

Purpose:

Lists every package installed inside the image.

Useful for:

- Package verification
- Debugging
- Dependency analysis

Example:

```text
busybox

hostapd

dnsmasq

iw

iptables

systemd

kernel

rdkb-ap-config
```

---

# BMAP File

Example:

```text
core-image-rdkb-ap.wic.bmap
```

Purpose:

Optimizes image flashing.

Instead of writing the entire image, only used blocks are copied.

Benefits:

- Faster flashing
- Reduced write time
- Lower SD card wear

---

# Test Data

Example:

```text
core-image-rdkb-ap.testdata.json
```

Purpose:

Contains metadata related to the build.

Includes:

- Build information
- Package details
- Image information

---

# Viewing Output Files

Navigate to the deployment directory:

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64
```

Display all files:

```bash
ls -lh
```

Display only image files:

```bash
ls *.wic*
```

Example:

```text
core-image-rdkb-ap.rootfs.wic.bz2

core-image-rdkb-ap.wic.bmap
```

---

# Decompressing the WIC Image

The generated image is compressed.

Copy it:

```bash
cp core-image-rdkb-ap-*.rootfs.wic.bz2 image.wic.bz2
```

Extract:

```bash
bunzip2 image.wic.bz2
```

Result:

```text
image.wic
```

---

# Viewing Image Partitions

Attach the image:

```bash
sudo losetup -Pf --show image.wic
```

Example:

```text
/dev/loop28
```

Display partitions:

```bash
lsblk /dev/loop28
```

Example:

```text
loop28

├── loop28p1

└── loop28p2
```

Partition Description:

| Partition | Purpose |
|------------|----------|
| p1 | Boot Partition |
| p2 | Linux Root Filesystem |

---

# Mounting the Root Filesystem

Create a mount point:

```bash
sudo mkdir -p /mnt/verify-root
```

Mount:

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Now the complete filesystem can be inspected.

---

# Verifying Installed Packages

Verify important binaries:

```bash
find /mnt/verify-root -name hostapd

find /mnt/verify-root -name dnsmasq

find /mnt/verify-root -name iptables

find /mnt/verify-root -name systemctl

find /mnt/verify-root -name rdkb-network.sh
```

Expected locations:

```text
/usr/sbin/hostapd

/usr/bin/dnsmasq

/usr/sbin/iptables

/bin/systemctl

/usr/bin/rdkb-network.sh
```

---

# Verifying Configuration Files

Check HostAPD configuration:

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Check dnsmasq configuration:

```bash
cat /mnt/verify-root/etc/dnsmasq.d/rdkb-ap.conf
```

Verify network script:

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

---

# Verifying Systemd Services

Display installed services:

```bash
ls /mnt/verify-root/lib/systemd/system
```

Important services:

```text
hostapd.service

dnsmasq.service

rdkb-network.service
```

Verify enabled services:

```bash
ls /mnt/verify-root/etc/systemd/system/multi-user.target.wants
```

Expected output:

```text
hostapd.service

dnsmasq.service

rdkb-network.service
```

---

# Image Verification Performed in This Project

Before flashing the SD card, the generated image was verified by:

- Decompressing the `.wic.bz2` image
- Attaching it using a loop device
- Mounting the root filesystem
- Verifying required binaries
- Verifying configuration files
- Checking systemd services
- Confirming that networking scripts were present
- Ensuring the final image contained all custom packages

This verification helped identify missing packages or configuration issues before deployment to the Raspberry Pi.

---

# Output Image Summary

| File | Purpose |
|------|----------|
| `Image` | Linux kernel |
| `*.dtb` | Device Tree Blob |
| `*.wic.bz2` | Bootable Raspberry Pi image |
| `*.ext3` | Root filesystem image |
| `*.tar.bz2` | Compressed root filesystem |
| `*.manifest` | Installed package list |
| `*.wic.bmap` | Flash optimization file |
| `*.testdata.json` | Build metadata |

---

# Final Output Flow

```text
BitBake Build
      │
      ▼
Kernel Generated
      │
      ▼
Root Filesystem Created
      │
      ▼
Packages Installed
      │
      ▼
Bootable WIC Image Created
      │
      ▼
Image Verification
      │
      ▼
Flash to microSD Card
      │
      ▼
Boot Raspberry Pi
      │
      ▼
Wi-Fi Access Point Ready
```

---

# Summary

The Yocto build process produced a complete set of deployment artifacts required to boot the Raspberry Pi and run the customized RDK-B Access Point. The most important output was the compressed `.wic.bz2` image, which contained the Linux kernel, boot partition, root filesystem, networking packages, and custom configuration files. Before flashing, the image was thoroughly verified by mounting and inspecting its contents, ensuring that all required components were correctly integrated into the final system.