# Ubuntu Installation

## Overview

The development of this project was carried out on **Ubuntu 22.04 LTS (64-bit)**. Ubuntu is one of the officially supported Linux distributions for the Yocto Project and provides all the necessary development tools required for building custom embedded Linux images.

A stable development environment is essential because Yocto performs extensive source compilation and package generation. Using a supported operating system minimizes compatibility issues during the build process.

---

# Why Ubuntu?

Ubuntu was selected for the following reasons:

- Officially supported by the Yocto Project
- Stable and reliable Linux distribution
- Excellent package management using APT
- Large community support
- Easy installation of development tools
- Compatible with Raspberry Pi development

---

# System Requirements

The following hardware specifications were recommended for building the project efficiently.

| Component | Recommended Specification |
|------------|---------------------------|
| Operating System | Ubuntu 22.04 LTS (64-bit) |
| Processor | Intel Core i5 / AMD Ryzen 5 or higher |
| RAM | Minimum 16 GB (32 GB Recommended) |
| Storage | At least 100 GB Free SSD Space |
| Internet | Stable Broadband Connection |

---

# Why High System Requirements?

Building a Yocto image requires compiling thousands of software packages.

During compilation the system performs:

- Source code extraction
- Cross compilation
- Package generation
- Root filesystem creation
- Image generation

The build process consumes a significant amount of:

- CPU
- RAM
- Disk Space

Having higher system resources reduces build time considerably.

---

# Installing Ubuntu

Ubuntu 22.04 LTS can be installed by downloading the ISO image from the official Ubuntu website.

Installation Steps:

1. Download Ubuntu ISO.
2. Create a bootable USB drive.
3. Boot the system using the USB drive.
4. Follow the Ubuntu installation wizard.
5. Create a user account.
6. Complete the installation.
7. Reboot the system.

---

# Update the Operating System

After installing Ubuntu, update all installed packages.

```bash
sudo apt update
sudo apt upgrade -y
```

These commands ensure that the operating system is updated with the latest security patches and package versions.

---

# Verify Ubuntu Version

Check the installed Ubuntu version.

```bash
lsb_release -a
```

Example Output

```text
Distributor ID: Ubuntu
Description: Ubuntu 22.04.5 LTS
Release: 22.04
Codename: jammy
```

Alternatively,

```bash
cat /etc/os-release
```

---

# Verify Kernel Version

Check the Linux kernel version.

```bash
uname -r
```

Example

```text
5.15.x-generic
```

---

# Verify System Architecture

Confirm that the operating system is 64-bit.

```bash
uname -m
```

Expected Output

```text
x86_64
```

This confirms that the host system is using a 64-bit architecture, which is recommended for Yocto builds.

---

# Verify Available Memory

Check installed RAM.

```bash
free -h
```

Example

```text
Mem: 16Gi
```

---

# Verify Available Disk Space

Yocto requires significant disk space during builds.

Check available storage.

```bash
df -h
```

Ensure that at least **100 GB** of free space is available before starting the build process.

---

# Internet Connectivity

Verify internet connectivity.

```bash
ping -c 4 google.com
```

Successful replies confirm that the system has internet access.

Internet connectivity is required for:

- Downloading source packages
- Cloning Git repositories
- Installing Ubuntu packages
- Synchronizing Yocto layers

---

# Development User

A normal (non-root) user account should be used for all development activities.

Example

```text
Username: scl
```

Avoid performing Yocto builds as the root user because it may lead to permission-related issues.

---

# Final Verification Checklist

Before proceeding to the next stage, verify the following:

- Ubuntu 22.04 LTS installed
- System updated successfully
- 64-bit architecture confirmed
- Sufficient RAM available
- Adequate free disk space
- Stable internet connection
- Development user created

---

# Summary

At the end of this stage, the Ubuntu development machine is fully prepared for setting up the Yocto Project build environment. The host system now satisfies the software and hardware requirements necessary for building a custom RDK-B Raspberry Pi image.