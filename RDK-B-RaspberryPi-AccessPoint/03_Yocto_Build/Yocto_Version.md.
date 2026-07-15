# Yocto Project Version Used

## Introduction

The RDK-B Raspberry Pi Access Point project was developed using the **Yocto Project Kirkstone (4.0 LTS)** release. Yocto was selected because it provides a flexible framework for building custom embedded Linux distributions from source code.

The **Kirkstone** release is a **Long Term Support (LTS)** version, making it suitable for production-quality embedded systems due to its stability, extended maintenance period, and broad community support.

This document describes the Yocto version used in the project, the supporting components, and the reasons for selecting this release.

---

# Yocto Release Information

| Component | Version |
|-----------|----------|
| Yocto Project | Kirkstone |
| Yocto Version | 4.0 LTS |
| Poky | Kirkstone Branch |
| BitBake | 2.0 |
| Linux Kernel | 5.15.x (Raspberry Pi BSP) |
| Target Machine | raspberrypi4-64 |
| Host Operating System | Ubuntu 22.04 LTS |
| Architecture | ARM64 (AArch64) |

---

# Why Yocto Kirkstone?

Several Yocto releases are available, such as Dunfell, Honister, Langdale, Mickledore, Scarthgap, and Kirkstone. For this project, **Kirkstone (4.0 LTS)** was selected because it provides a balance between stability, compatibility, and long-term support.

The advantages of using Kirkstone include:

- Long Term Support (LTS)
- Stable build environment
- Reliable Raspberry Pi 4 support
- Mature community layers
- Broad package availability
- Continuous bug fixes
- Security updates
- Production-ready environment

---

# Why Long Term Support (LTS)?

An LTS release receives maintenance updates for a longer period compared to standard releases.

Benefits include:

- Stable APIs
- Fewer breaking changes
- Better documentation
- Security patches
- Reliable package compatibility
- Preferred for commercial embedded products

Because this project focuses on embedded Linux development and networking, an LTS release provides a more dependable development platform.

---

# Components Used

The project was built using several components provided by the Yocto Project.

## Poky

Poky is the reference distribution of the Yocto Project.

It includes:

- BitBake
- OpenEmbedded-Core
- Default build configuration
- Development tools

Poky served as the base framework for building the custom Linux image.

---

## BitBake

BitBake is the build engine of the Yocto Project.

Responsibilities include:

- Parsing recipes
- Resolving dependencies
- Downloading source code
- Compiling software
- Packaging binaries
- Generating filesystem images

Every software component included in the project was built through BitBake.

---

## OpenEmbedded-Core

OpenEmbedded-Core provides the standard recipes required for building an embedded Linux system.

Examples include:

- BusyBox
- systemd
- Bash
- Core utilities
- Networking tools
- Libraries

These packages formed the foundation of the custom image.

---

## meta-openembedded

The project also included the **meta-openembedded** layer.

Purpose:

- Additional software packages
- Networking utilities
- Development libraries
- Extra applications

---

## meta-raspberrypi

The **meta-raspberrypi** layer provides Board Support Package (BSP) support for Raspberry Pi devices.

It includes:

- Raspberry Pi kernel
- Bootloader configuration
- Firmware
- Device tree files
- Hardware drivers

Without this layer, Yocto would not be able to generate an image compatible with the Raspberry Pi 4.

---

# Target Hardware

The final image was built specifically for:

| Parameter | Value |
|-----------|-------|
| Board | Raspberry Pi 4 Model B |
| Architecture | ARM64 |
| Machine | raspberrypi4-64 |
| Boot Medium | Micro SD Card |
| Wireless Interface | wlan0 |
| Ethernet Interface | eth0 |

---

# Development Host

The development environment consisted of the following software.

| Component | Version |
|-----------|----------|
| Ubuntu | 22.04 LTS |
| Git | Latest Stable |
| Python | 3.x |
| GCC | Ubuntu Default |
| BitBake | 2.0 |
| Yocto | Kirkstone |

---

# Repository Branches

The following repositories were cloned using the **kirkstone** branch.

## Poky

```bash
git clone -b kirkstone git://git.yoctoproject.org/poky
```

---

## meta-openembedded

```bash
git clone -b kirkstone https://github.com/openembedded/meta-openembedded.git
```

---

## meta-raspberrypi

```bash
git clone -b kirkstone https://github.com/agherzan/meta-raspberrypi.git
```

---

# Build Configuration

The machine configuration used during the build was:

```text
MACHINE = "raspberrypi4-64"
```

The build generated a 64-bit Linux image for Raspberry Pi 4.

---

# Image Generated

The build process generated a bootable image containing:

- Linux Kernel
- Root Filesystem
- Bootloader
- Firmware
- Networking utilities
- hostapd
- dnsmasq
- iptables
- systemd
- Custom RDK-B configuration

---

# Verifying the Yocto Version

The version can be verified after sourcing the build environment.

Initialize the environment:

```bash
source oe-init-build-env
```

Check BitBake version:

```bash
bitbake --version
```

Example Output:

```text
BitBake Build Tool Core version 2.0
```

Check Yocto version:

```bash
grep DISTRO_VERSION poky/meta-poky/conf/distro/poky.conf
```

Example Output:

```text
DISTRO_VERSION = "4.0.x"
```

Check current Git branch:

```bash
cd poky
git branch --show-current
```

Expected Output:

```text
kirkstone
```

---

# Why This Version Was Suitable for the Project

The project required:

- Stable networking stack
- Raspberry Pi 4 compatibility
- Long-term support
- Reliable package management
- Easy customization
- Community documentation

The Kirkstone release fulfilled all these requirements and allowed successful development of the custom Raspberry Pi Access Point image.

---

# Advantages Observed During Development

Using Yocto Kirkstone provided several practical benefits during implementation.

- Stable build process
- Mature documentation
- Reliable Raspberry Pi support
- Easy package integration
- Simple layer management
- Consistent BitBake behavior
- Well-tested networking packages
- Efficient image customization

These advantages significantly reduced build failures and simplified debugging.

---

# Lessons Learned

During the implementation of this project, several important concepts related to Yocto were learned.

- Understanding the Yocto Project architecture.
- Working with the Poky reference distribution.
- Managing layers using BitBake.
- Creating custom image recipes.
- Integrating third-party packages.
- Building complete Linux images from source.
- Deploying custom images to embedded hardware.

---

# Conclusion

The **Yocto Project Kirkstone (4.0 LTS)** served as the foundation for developing the RDK-B Raspberry Pi Access Point. Its long-term support, stability, and extensive Raspberry Pi compatibility made it an ideal choice for this embedded Linux project.

By leveraging Poky, BitBake, OpenEmbedded-Core, and additional community layers, a fully customized Linux image was successfully built and deployed. The resulting system provided automatic Wi-Fi Access Point functionality, DHCP services, NAT-based Internet sharing, and reliable startup behavior, demonstrating the flexibility and power of the Yocto Project for embedded system development.
