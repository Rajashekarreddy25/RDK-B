# Build Process

## Introduction

The build process is the sequence of operations performed by the Yocto Project and BitBake to generate a complete bootable Linux image. It starts with reading the build configuration, parsing metadata from all layers, resolving package dependencies, compiling source code, generating packages, creating the root filesystem, and finally producing a bootable image.

In this project, the build process was used to generate a customized RDK-B Access Point image for the Raspberry Pi 4. The final image included all required networking packages (`hostapd`, `dnsmasq`, `iw`, `iptables`) along with the custom configuration package (`rdkb-ap-config`).

---

# Overall Build Flow

The complete build flow followed during this project is shown below.

```text
Create Build Environment
        │
        ▼
Configure Yocto
(local.conf & bblayers.conf)
        │
        ▼
Add Custom Layer
(meta-rdkb-ap)
        │
        ▼
Create Image Recipe
(core-image-rdkb-ap.bb)
        │
        ▼
Run BitBake
        │
        ▼
Parse Recipes
        │
        ▼
Resolve Dependencies
        │
        ▼
Download Source Code
        │
        ▼
Compile Packages
        │
        ▼
Generate Packages
        │
        ▼
Create Root Filesystem
        │
        ▼
Generate Bootable Image (.wic)
        │
        ▼
Verify Image
        │
        ▼
Flash SD Card
        │
        ▼
Boot Raspberry Pi
```

---

# Step 1 : Configure the Build Environment

Before starting the build, the Yocto build environment must be initialized.

Navigate to the Poky directory:

```bash
cd ~/rdkb-pi/poky
```

Initialize the build environment:

```bash
source oe-init-build-env
```

This command creates (if not already present) the following build directory:

```text
build/
```

Inside this directory:

```text
build/
├── conf/
├── cache/
├── downloads/
├── sstate-cache/
└── tmp/
```

---

# Step 2 : Configure Build Settings

The main build configuration is stored in:

```text
build/conf/local.conf
```

Important configurations used in this project:

```conf
MACHINE = "raspberrypi4-64"

INIT_MANAGER = "systemd"

DISTRO_FEATURES:append = " systemd"

DISTRO_FEATURES_BACKFILL_CONSIDERED += "sysvinit"

VIRTUAL-RUNTIME_init_manager = "systemd"

VIRTUAL-RUNTIME_initscripts = ""

DISTRO_FEATURES:remove = "sysvinit"
```

Purpose:

- Target Raspberry Pi 4 (64-bit)
- Use `systemd` as the init system
- Disable SysV init scripts

---

# Step 3 : Load Metadata from Layers

BitBake reads all metadata from the layers listed in:

```text
build/conf/bblayers.conf
```

Example:

```text
meta
meta-poky
meta-yocto-bsp
meta-openembedded/meta-oe
meta-openembedded/meta-networking
meta-openembedded/meta-python
meta-raspberrypi
meta-rdkb-ap
```

Each layer contributes recipes and configuration files required for the final image.

---

# Step 4 : Parse Recipes

BitBake scans every recipe (`.bb`) and append file (`.bbappend`) from all enabled layers.

Recipes used in this project included:

```text
core-image-rdkb-ap.bb

rdkb-ap-config.bb

hostapd.bb

dnsmasq.bb

iptables.bb

iw.bb
```

During parsing, BitBake determines:

- Package dependencies
- Build order
- Required tasks
- Installation files

---

# Step 5 : Resolve Dependencies

BitBake automatically resolves package dependencies.

Dependency tree used in this project:

```text
core-image-rdkb-ap
        │
        ├── hostapd
        ├── dnsmasq
        ├── iw
        ├── iptables
        └── rdkb-ap-config
```

The custom package also depends on:

```text
hostapd

dnsmasq
```

Configured using:

```bitbake
RDEPENDS:${PN} += "hostapd dnsmasq"
```

---

# Step 6 : Download Source Code

If required source files are not already available, BitBake downloads them into:

```text
build/downloads/
```

This directory stores source archives for all packages.

Examples:

```text
hostapd

dnsmasq

iptables

busybox

linux kernel
```

Once downloaded, these files are reused for future builds.

---

# Step 7 : Compile Packages

Each package is compiled individually.

Typical sequence:

```text
Fetch Source
        │
        ▼
Extract Source
        │
        ▼
Configure
        │
        ▼
Compile
        │
        ▼
Install
        │
        ▼
Package
```

BitBake executes this process for every package required by the image.

---

# Step 8 : Build the Custom Package

The custom package created for this project is:

```text
rdkb-ap-config
```

This package installs:

```text
/etc/hostapd.conf

/etc/dnsmasq.d/rdkb-ap.conf

/usr/bin/rdkb-network.sh

/lib/systemd/system/rdkb-network.service
```

The package also enables:

```text
rdkb-network.service
```

during image generation.

---

# Step 9 : Generate Root Filesystem

After all packages are compiled, BitBake creates the root filesystem.

Installed packages include:

```text
systemd

hostapd

dnsmasq

iw

iptables

busybox

kernel

rdkb-ap-config
```

The resulting filesystem contains:

```text
/etc

/usr

/lib

/bin

/sbin

/var

/home
```

---

# Step 10 : Generate the Bootable Image

BitBake then generates the final bootable image.

Output directory:

```text
build/tmp/deploy/images/raspberrypi4-64/
```

Generated files:

```text
core-image-rdkb-ap.rootfs.wic.bz2

core-image-rdkb-ap.rootfs.ext3

core-image-rdkb-ap.rootfs.tar.bz2

Image

bcm2711-rpi-4-b.dtb
```

The `.wic.bz2` file is the bootable disk image used for flashing the SD card.

---

# Step 11 : Verify the Build

After a successful build, verify the generated image.

Example:

```bash
ls build/tmp/deploy/images/raspberrypi4-64/
```

Expected output:

```text
core-image-rdkb-ap.rootfs.wic.bz2

core-image-rdkb-ap.rootfs.ext3

Image

bcm2711-rpi-4-b.dtb
```

---

# Step 12 : Inspect the Image

To verify the contents of the generated image:

Copy the image:

```bash
cp core-image-rdkb-ap-*.rootfs.wic.bz2 verify.wic.bz2
```

Extract it:

```bash
bunzip2 verify.wic.bz2
```

Attach it as a loop device:

```bash
sudo losetup -Pf --show verify.wic
```

Example output:

```text
/dev/loop28
```

View partitions:

```bash
lsblk /dev/loop28
```

Mount the root filesystem:

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Verify important files:

```bash
find /mnt/verify-root -name hostapd

find /mnt/verify-root -name dnsmasq

find /mnt/verify-root -name iptables

find /mnt/verify-root -name rdkb-network.sh
```

Verify enabled services:

```bash
ls /mnt/verify-root/etc/systemd/system/multi-user.target.wants
```

---

# Step 13 : Flash the Image

Once verified, the image is flashed onto a microSD card using tools such as:

- Raspberry Pi Imager
- Balena Etcher
- `dd` command (Linux)

After flashing, insert the SD card into the Raspberry Pi and power it on.

---

# Step 14 : Boot and Validate

After booting:

Verify the Raspberry Pi has started correctly.

Check:

- `systemd` is running
- `hostapd` service is active
- `dnsmasq` service is active
- `rdkb-network.service` is active
- Wi-Fi SSID is visible
- Clients receive IP addresses
- Internet access works through NAT

---

# Build Artifacts Generated

The final build generated the following major artifacts:

| Artifact | Purpose |
|----------|----------|
| `Image` | Linux kernel image |
| `*.dtb` | Device Tree Blob |
| `*.wic.bz2` | Bootable Raspberry Pi image |
| `*.ext3` | Root filesystem image |
| `*.tar.bz2` | Root filesystem archive |
| Manifest files | Package information |
| License files | Open-source license details |

---

# Build Process Summary

```text
Initialize Build Environment
        │
        ▼
Configure local.conf
        │
        ▼
Load Layers
        │
        ▼
Parse Recipes
        │
        ▼
Resolve Dependencies
        │
        ▼
Compile Packages
        │
        ▼
Build Custom Package
        │
        ▼
Generate Root Filesystem
        │
        ▼
Generate Bootable Image
        │
        ▼
Verify Image
        │
        ▼
Flash SD Card
        │
        ▼
Boot Raspberry Pi
        │
        ▼
Validate Wi-Fi Access Point
```

---

# Summary

The Yocto build process transformed the project metadata and recipes into a complete bootable Linux image for the Raspberry Pi 4. Through BitBake, all required networking packages and the custom `rdkb-ap-config` package were compiled, packaged, and integrated into the final image. After verification and flashing, the Raspberry Pi successfully booted as a Wi-Fi Access Point, providing DHCP, NAT, and internet connectivity to multiple wireless clients.