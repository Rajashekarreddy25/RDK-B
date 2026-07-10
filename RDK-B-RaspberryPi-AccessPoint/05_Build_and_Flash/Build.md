# Build

## Introduction

Once the custom RDK-B layer, image recipe, and all supporting packages were created, the next stage of the project was to build the complete Linux image using the Yocto Project.

The build process combines:

- Yocto Build System
- Poky Distribution
- Raspberry Pi BSP
- Custom RDK-B Layer
- Required Packages
- Custom Recipes

to generate a bootable image for the Raspberry Pi 4.

The final output is a compressed `.wic.bz2` image that can be flashed directly onto an SD card.

---

# Build Workflow

```text
Configure Build Environment

        │

        ▼

Parse BitBake Recipes

        │

        ▼

Resolve Dependencies

        │

        ▼

Compile Source Code

        │

        ▼

Package Binaries

        │

        ▼

Generate Root Filesystem

        │

        ▼

Create Boot Image

        │

        ▼

Generate WIC Image

        │

        ▼

Compress Image (.wic.bz2)
```

---

# Project Build Environment

The build environment used for this project is summarized below.

| Component | Version |
|-----------|----------|
| Host OS | Ubuntu 22.04 LTS |
| Yocto Release | Kirkstone |
| Build System | Poky |
| Machine | raspberrypi4-64 |
| Architecture | AArch64 |
| Image Recipe | core-image-rdkb-ap |
| Target Board | Raspberry Pi 4 Model B |

---

# Step 1 – Navigate to the Build Directory

Move to the Poky build directory.

```bash
cd ~/rdkb-pi/poky/build
```

This directory contains:

- `conf/`
- `tmp/`
- `downloads/`
- `sstate-cache/`

---

# Step 2 – Verify Build Configuration

Before building, confirm the target machine.

```bash
cat conf/local.conf | grep MACHINE
```

Expected output:

```text
MACHINE = "raspberrypi4-64"
```

---

Verify the build layers.

```bash
bitbake-layers show-layers
```

Example output:

```text
meta
meta-poky
meta-yocto-bsp
meta-openembedded
meta-networking
meta-python
meta-raspberrypi
meta-rdkb-ap
```

---

# Step 3 – Verify the Image Recipe

Confirm that the custom image recipe is available.

```bash
bitbake-layers show-recipes | grep core-image-rdkb-ap
```

Expected output:

```text
core-image-rdkb-ap
```

---

# Step 4 – Start the Build

Build the complete operating system.

```bash
bitbake core-image-rdkb-ap
```

BitBake performs multiple stages during the build.

---

# Stage 1 – Parsing Recipes

BitBake first reads all metadata.

Example:

```text
Parsing recipes...
```

During this stage it reads:

- Image recipe
- Package recipes
- Layer configuration
- Machine configuration
- Dependencies

---

# Stage 2 – Dependency Resolution

BitBake determines the dependency tree.

Example:

```text
Resolving dependencies...
```

It identifies:

```text
core-image-rdkb-ap

↓

hostapd

↓

libnl

↓

openssl

↓

kernel headers
```

Every required dependency is added automatically.

---

# Stage 3 – Fetch Source Code

If source code is unavailable locally, BitBake downloads it.

Example:

```text
Fetching source...
```

Downloaded sources are stored in:

```text
downloads/
```

---

# Stage 4 – Configure

Each package is configured.

Example:

```text
Configuring hostapd

Configuring dnsmasq

Configuring busybox
```

Configuration prepares the source code for compilation.

---

# Stage 5 – Compilation

BitBake compiles each package.

Example:

```text
Compiling...

hostapd

dnsmasq

iptables

iw

systemd
```

Compiled binaries are stored temporarily.

---

# Stage 6 – Packaging

After compilation, packages are created.

Example:

```text
Packaging hostapd...

Packaging dnsmasq...

Packaging iptables...
```

Packages are converted into installable formats.

---

# Stage 7 – Root Filesystem Generation

Next, BitBake assembles the root filesystem.

```text
Creating Root Filesystem...
```

The root filesystem includes:

```text
/bin

/usr

/lib

/etc

/home

/var
```

Custom files from `rdkb-ap-config` are also installed.

Examples:

```text
/etc/hostapd.conf

/etc/dnsmasq.d/rdkb-ap.conf

/usr/bin/rdkb-network.sh

/lib/systemd/system/rdkb-network.service
```

---

# Stage 8 – Boot Image Creation

The Raspberry Pi boot partition is generated.

Contents include:

```text
kernel8.img

config.txt

cmdline.txt

bcm2711-rpi-4-b.dtb
```

---

# Stage 9 – WIC Image Creation

Finally, Yocto creates a complete disk image.

Example:

```text
Creating WIC image...
```

The WIC image contains:

```text
Boot Partition

+

Root Filesystem

+

Partition Table
```

---

# Stage 10 – Image Compression

The generated image is compressed.

Output:

```text
core-image-rdkb-ap-raspberrypi4-64.rootfs.wic.bz2
```

Compression reduces storage requirements and speeds up file transfer.

---

# Successful Build Output

A successful build typically ends with:

```text
NOTE: Tasks Summary:

Attempted XXXX tasks

All succeeded.
```

This confirms that the image was generated successfully.

---

# Build Time

Approximate build durations observed during the project:

| Build Type | Approximate Time |
|------------|------------------|
| First Build | 2–4 Hours |
| Incremental Build | 5–20 Minutes |
| Rebuild After Small Changes | 2–10 Minutes |

The exact duration depends on:

- CPU
- RAM
- SSD performance
- Internet speed
- Number of modified recipes

---

# Incremental Build

One advantage of BitBake is incremental compilation.

After modifying only one recipe, BitBake rebuilds only the affected components.

Example:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

Only the modified package is rebuilt.

This significantly reduces build time.

---

# Cleaning a Package

To rebuild a single package:

```bash
bitbake -c clean rdkb-ap-config
```

To completely remove compiled output:

```bash
bitbake -c cleansstate rdkb-ap-config
```

---

# Common Build Commands

Build image:

```bash
bitbake core-image-rdkb-ap
```

Clean recipe:

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild recipe:

```bash
bitbake rdkb-ap-config
```

Show environment:

```bash
bitbake -e core-image-rdkb-ap
```

List available recipes:

```bash
bitbake-layers show-recipes
```

Show layers:

```bash
bitbake-layers show-layers
```

---

# Output Directory

After a successful build, the generated images are located in:

```text
poky/build/tmp/deploy/images/raspberrypi4-64/
```

Example contents:

```text
core-image-rdkb-ap.wic.bz2

core-image-rdkb-ap.bmap

Image

kernel8.img

config.txt

bcm2711-rpi-4-b.dtb
```

---

# Build Verification

Confirm that the image exists.

```bash
cd ~/rdkb-pi/poky/build/tmp/deploy/images/raspberrypi4-64

ls -lh
```

Expected:

```text
core-image-rdkb-ap-raspberrypi4-64.rootfs.wic.bz2
```

The presence of this file confirms that the build completed successfully.

---

# Challenges Faced During Build

Several build-related issues were encountered while developing this project.

## Package Conflict

The build initially failed because both `hostapd` and the custom recipe attempted to install `/etc/hostapd.conf`.

Solution:

- Removed the duplicate installation from the custom recipe.
- Allowed the original `hostapd` package to install the configuration file.

---

## Missing iptables

The system booted correctly, but Internet sharing did not work.

Cause:

`iptables` was not included in the image.

Solution:

Added it to the image recipe.

```bitbake
IMAGE_INSTALL += "iptables"
```

---

## Recipe Not Updating

After modifying `rdkb-network.sh`, the generated image still contained the old version.

Solution:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

This forced the updated recipe to be rebuilt.

---

# Summary

Building the RDK-B image involved combining the Yocto build system, Raspberry Pi BSP, and the custom `meta-rdkb-ap` layer into a single bootable Linux image. BitBake automatically parsed recipes, resolved dependencies, compiled packages, generated the root filesystem, and produced a compressed `.wic.bz2` image ready for deployment. Incremental builds significantly reduced development time, while careful verification of build outputs ensured that all custom configurations were correctly included in the final image.