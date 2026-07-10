# Flashing_SD_Card

## Introduction

After verifying the generated image, the next step is to flash it onto a microSD card. The Raspberry Pi boots its operating system directly from the SD card, making it the primary storage device for the project.

The generated `.wic` image contains everything required to boot the Raspberry Pi:

- Bootloader
- Linux Kernel
- Device Tree
- Root Filesystem
- Custom RDK-B Packages
- Configuration Files
- Systemd Services

Once flashed, the Raspberry Pi can boot directly into the custom RDK-B Access Point image.

---

# Flashing Workflow

```text
BitBake Build

      │

      ▼

Generated .wic.bz2 Image

      │

      ▼

Verify Image

      │

      ▼

Decompress Image

      │

      ▼

Insert SD Card

      │

      ▼

Flash Image

      │

      ▼

Safely Remove SD Card

      │

      ▼

Insert into Raspberry Pi
```

---

# Requirements

Before flashing, ensure the following are available.

| Component | Requirement |
|-----------|-------------|
| Raspberry Pi 4 | Available |
| microSD Card | Minimum 8 GB (16 GB Recommended) |
| SD Card Reader | USB or Built-in |
| Ubuntu Host PC | For flashing |
| Generated `.wic` Image | Successfully built |

---

# Locate the Generated Image

Navigate to the image directory.

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64
```

List the available images.

```bash
ls -lh
```

Example:

```text
core-image-rdkb-ap-raspberrypi4-64.rootfs.wic.bz2
```

---

# Decompress the Image

If only the compressed image is available, extract it.

```bash
cp core-image-rdkb-ap-raspberrypi4-64-*.rootfs.wic.bz2 image.wic.bz2

bunzip2 image.wic.bz2
```

Verify the extracted file.

```bash
ls -lh image.wic
```

Example output:

```text
image.wic
```

---

# Insert the SD Card

Insert the microSD card into the computer.

Wait a few seconds for Ubuntu to detect the device.

---

# Identify the SD Card

List all storage devices.

```bash
lsblk
```

Example output:

```text
NAME      SIZE

sda       512G

├─sda1

├─sda2

sdb       32G

├─sdb1
```

In this example:

```text
/dev/sdb
```

is the SD card.

> **Important:** Carefully identify the correct device. Flashing the wrong device can permanently erase data from another storage drive.

---

# Unmount Existing Partitions

If Ubuntu automatically mounts the SD card, unmount all partitions before flashing.

Example:

```bash
sudo umount /dev/sdb1
```

Unmount additional partitions if necessary.

```bash
sudo umount /dev/sdb2
```

---

# Flash the Image

Use the `dd` command to write the image.

```bash
sudo dd if=image.wic of=/dev/sdb bs=4M status=progress conv=fsync
```

Where:

| Option | Description |
|----------|-------------|
| if | Input image |
| of | Output SD Card |
| bs=4M | Block size |
| status=progress | Show progress |
| conv=fsync | Flush data before completion |

---

# Wait for Completion

Example output:

```text
245366784 bytes copied...

512000000 bytes copied...

1024000000 bytes copied...
```

After completion:

```text
syncing...

records in

records out
```

---

# Flush Pending Data

Run:

```bash
sync
```

This ensures all buffered data has been written to the SD card.

---

# Remove the SD Card

Safely eject the SD card.

```bash
sudo eject /dev/sdb
```

Now remove the SD card from the computer.

---

# Alternative Method Using Raspberry Pi Imager

Instead of using `dd`, Raspberry Pi Imager can also be used.

Steps:

1. Open Raspberry Pi Imager.
2. Select **Use Custom Image**.
3. Choose the generated `.wic` image.
4. Select the SD card.
5. Click **Write**.
6. Wait for completion.
7. Remove the SD card.

Although this method is simpler, the `dd` command was used throughout this project because it is commonly used in embedded Linux development.

---

# Verifying the Flashed SD Card

Reinsert the SD card.

Check that Ubuntu detects two partitions.

```bash
lsblk
```

Example:

```text
sdb

├── sdb1

└── sdb2
```

This confirms that the image has been written correctly.

---

# Boot Partition Verification

Mount the boot partition.

```bash
sudo mount /dev/sdb1 /mnt
```

List its contents.

```bash
ls /mnt
```

Typical files include:

```text
kernel8.img

config.txt

cmdline.txt

bcm2711-rpi-4-b.dtb
```

Unmount after verification.

```bash
sudo umount /mnt
```

---

# Root Filesystem Verification

Optionally mount the root partition.

```bash
sudo mount /dev/sdb2 /mnt
```

List directories.

```bash
ls /mnt
```

Expected directories:

```text
bin

boot

dev

etc

home

lib

proc

root

usr

var
```

Unmount when finished.

```bash
sudo umount /mnt
```

---

# Flashing Time

Approximate flashing times observed during the project.

| SD Card Speed | Flash Time |
|---------------|------------|
| Class 10 | 3–5 Minutes |
| UHS-I | 2–3 Minutes |
| High-Speed USB Reader | Less than 2 Minutes |

Actual time depends on:

- SD card speed
- USB reader speed
- Host computer performance

---

# Common Flashing Errors

## Wrong Device Selected

Cause:

Incorrect output device specified in the `dd` command.

Solution:

Always verify using:

```bash
lsblk
```

before flashing.

---

## Device Busy

Error:

```text
Device or resource busy
```

Cause:

The SD card is mounted.

Solution:

Unmount all partitions.

```bash
sudo umount /dev/sdb*
```

---

## Permission Denied

Cause:

`dd` requires administrator privileges.

Solution:

```bash
sudo dd ...
```

---

## Corrupted Image

Cause:

Interrupted image extraction or incomplete download.

Solution:

Rebuild the image or decompress it again before flashing.

---

# Commands Used

Navigate to image directory:

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64
```

Extract image:

```bash
bunzip2 image.wic.bz2
```

Identify SD card:

```bash
lsblk
```

Unmount partitions:

```bash
sudo umount /dev/sdb1
```

Flash image:

```bash
sudo dd if=image.wic of=/dev/sdb bs=4M status=progress conv=fsync
```

Flush pending writes:

```bash
sync
```

Safely eject SD card:

```bash
sudo eject /dev/sdb
```

---

# Challenges Faced During the Project

During development, several issues were encountered while preparing the SD card.

## Multiple Hard Links in the Image

Initially, decompressing the original `.wic.bz2` image failed because it had multiple hard links.

Solution:

A copy of the image was created before decompression.

```bash
cp core-image-rdkb-ap-*.wic.bz2 image.wic.bz2

bunzip2 image.wic.bz2
```

---

## Flashing the Wrong Device

To avoid accidentally overwriting the system disk, the target device was always verified using:

```bash
lsblk
```

before executing the `dd` command.

---

## Verifying Before Flashing

Instead of immediately flashing the image, it was mounted and inspected using a loop device. This ensured that all required packages, configuration files, and systemd services were present before deployment.

This verification step significantly reduced debugging time on the Raspberry Pi.

---

# Summary

The final RDK-B image was successfully written to a microSD card using the Linux `dd` utility. Before flashing, the image was decompressed and thoroughly verified to ensure that all required components were included. After writing the image, the SD card was safely removed and prepared for booting on the Raspberry Pi 4. Following these steps ensured a reliable deployment and minimized issues during the initial boot process.