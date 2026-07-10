# BitBake

## Introduction

BitBake is the build engine of the Yocto Project. It is responsible for parsing recipes, resolving dependencies, executing build tasks, and generating the final Linux image.

It functions similarly to the `make` utility but is specifically designed for embedded Linux development. BitBake reads metadata from Yocto layers and executes tasks in the correct order to produce packages, root filesystems, kernels, and bootable images.

In this project, BitBake was used to build a custom RDK-B Raspberry Pi Access Point image by compiling all required packages and integrating the custom configuration layer.

---

# What is BitBake?

BitBake is a task execution engine.

Its responsibilities include:

- Reading recipes (`.bb` files)
- Reading append files (`.bbappend`)
- Resolving package dependencies
- Executing build tasks
- Managing shared state (sstate cache)
- Creating software packages
- Building bootable images

---

# BitBake Workflow

The overall workflow of BitBake is shown below.

```text
Recipes (.bb)
        │
        ▼
BitBake Parser
        │
        ▼
Dependency Resolver
        │
        ▼
Task Scheduler
        │
        ▼
Execute Tasks
        │
        ▼
Packages
        │
        ▼
Root Filesystem
        │
        ▼
Linux Image (.wic)
```

---

# BitBake Recipes

BitBake builds software using recipe files.

Recipe extension:

```text
.bb
```

Example:

```text
core-image-rdkb-ap.bb
```

Each recipe contains:

- Package information
- Source location
- Dependencies
- Build instructions
- Installation steps

---

# Recipe Example

A simplified version of the image recipe used in this project:

```bitbake
SUMMARY = "RDK-B Raspberry Pi Access Point Image"

LICENSE = "MIT"

require recipes-core/images/core-image-base.bb

IMAGE_INSTALL:append = " \
    hostapd \
    dnsmasq \
    iw \
    iptables \
    rdkb-ap-config \
"
```

This recipe tells BitBake to create a custom image containing:

- hostapd
- dnsmasq
- iw
- iptables
- rdkb-ap-config

---

# BitBake Tasks

Every recipe is divided into multiple tasks.

Some common tasks are:

| Task | Description |
|------|-------------|
| `do_fetch` | Download source code |
| `do_unpack` | Extract source files |
| `do_patch` | Apply patches |
| `do_configure` | Configure package |
| `do_compile` | Compile source code |
| `do_install` | Install files into staging area |
| `do_package` | Create installable packages |
| `do_rootfs` | Generate root filesystem |
| `do_image` | Create bootable image |
| `do_image_wic` | Generate `.wic` disk image |

---

# Viewing Available Tasks

To display all tasks for a recipe:

```bash
bitbake -c listtasks core-image-rdkb-ap
```

Example output:

```text
do_fetch
do_unpack
do_patch
do_configure
do_compile
do_install
do_package
do_rootfs
do_image
do_image_wic
do_image_complete
```

---

# Common BitBake Commands

## Build an Image

```bash
bitbake core-image-rdkb-ap
```

This command builds the complete Linux image.

---

## Build a Single Package

Example:

```bash
bitbake hostapd
```

---

## Clean Build Files

```bash
bitbake -c clean core-image-rdkb-ap
```

Removes the recipe's work directory while keeping downloaded sources.

---

## Clean Everything

```bash
bitbake -c cleansstate core-image-rdkb-ap
```

Removes:

- Build output
- Shared state cache

This forces a complete rebuild.

---

## Build from Scratch

```bash
bitbake -c cleanall hostapd
```

Removes:

- Build artifacts
- Downloaded source files

---

# BitBake Environment Variables

BitBake stores many configuration variables.

To inspect them:

```bash
bitbake -e core-image-rdkb-ap
```

Examples:

Display image packages:

```bash
bitbake -e core-image-rdkb-ap | grep IMAGE_INSTALL
```

Output:

```text
IMAGE_INSTALL="
hostapd
dnsmasq
iw
iptables
rdkb-ap-config
"
```

---

Display init manager:

```bash
bitbake -e core-image-rdkb-ap | grep INIT_MANAGER
```

Output:

```text
INIT_MANAGER="systemd"
```

---

Display distribution features:

```bash
bitbake -e core-image-rdkb-ap | grep DISTRO_FEATURES
```

Output:

```text
DISTRO_FEATURES="systemd wifi ipv4 ipv6 ..."
```

---

# Dependency Resolution

BitBake automatically resolves package dependencies.

Example:

```text
core-image-rdkb-ap
        │
        ├── hostapd
        ├── dnsmasq
        ├── iw
        ├── iptables
        └── rdkb-ap-config
```

The custom package (`rdkb-ap-config`) also depends on:

```text
hostapd

dnsmasq
```

This relationship is defined in the recipe:

```bitbake
RDEPENDS:${PN} += "hostapd dnsmasq"
```

---

# Incremental Builds

BitBake rebuilds only the components that have changed.

Example:

- Modify `hostapd.conf`
- Rebuild image

Only the affected package and image are rebuilt, reducing build time.

---

# Shared State Cache (sstate)

BitBake stores compiled output in the Shared State Cache.

Benefits:

- Faster rebuilds
- Reuse of previously compiled packages
- Reduced build time

Typical build output:

```text
Sstate summary:
Wanted 10
Current 1650
Missed 5
```

Meaning:

- **Wanted** – Packages requested
- **Current** – Already available in cache
- **Missed** – Packages that need rebuilding

---

# BitBake Build Process

The sequence followed during image generation:

```text
Read Configuration
        │
        ▼
Parse Recipes
        │
        ▼
Resolve Dependencies
        │
        ▼
Download Sources
        │
        ▼
Compile Packages
        │
        ▼
Package Software
        │
        ▼
Create Root Filesystem
        │
        ▼
Generate Bootable Image
```

---

# Build Output Location

After a successful build, BitBake stores the generated files in:

```text
poky/build/tmp/deploy/images/raspberrypi4-64/
```

Typical outputs include:

```text
core-image-rdkb-ap.rootfs.wic.bz2

core-image-rdkb-ap.rootfs.ext3

core-image-rdkb-ap.rootfs.tar.bz2

Image

bcm2711-rpi-4-b.dtb
```

---

# Useful BitBake Commands Used in This Project

Display layers:

```bash
bitbake-layers show-layers
```

List image tasks:

```bash
bitbake -c listtasks core-image-rdkb-ap
```

Check image packages:

```bash
bitbake -e core-image-rdkb-ap | grep IMAGE_INSTALL
```

Verify systemd:

```bash
bitbake -e core-image-rdkb-ap | grep INIT_MANAGER
```

Verify distribution features:

```bash
bitbake -e core-image-rdkb-ap | grep DISTRO_FEATURES
```

Clean custom package:

```bash
bitbake -c clean rdkb-ap-config
```

Build final image:

```bash
bitbake core-image-rdkb-ap
```

---

# BitBake in This Project

BitBake played a central role in the implementation of the RDK-B Raspberry Pi Access Point project. It was responsible for parsing the custom `core-image-rdkb-ap` image recipe, resolving dependencies for packages such as `hostapd`, `dnsmasq`, `iw`, `iptables`, and `rdkb-ap-config`, compiling the required components, generating the root filesystem, and producing the final bootable `.wic` image for the Raspberry Pi 4.

---

# Summary

BitBake is the core build engine of the Yocto Project. It automates the complete embedded Linux build process by interpreting recipes, managing dependencies, executing build tasks, and creating bootable images. Throughout this project, BitBake enabled the integration of custom recipes, networking packages, and system configurations into a single customized Linux image, making it the foundation of the RDK-B Access Point build process.