# Image Recipe

## Introduction

In Yocto Project, an **Image Recipe** defines the final operating system image that will be generated for the target device. It specifies:

- Base image to inherit
- Software packages to install
- Custom applications
- Configuration packages
- System services
- Additional utilities

For this project, a custom image recipe named **core-image-rdkb-ap.bb** was created. This recipe extends the standard Yocto base image by including all the packages required to transform the Raspberry Pi 4 into an RDK-B Wi-Fi Access Point.

---

# Why Create a Custom Image Recipe?

Yocto provides several predefined images such as:

- core-image-minimal
- core-image-base
- core-image-full-cmdline
- core-image-sato

These images provide generic Linux functionality but do not include the components required for a Wi-Fi Access Point.

Instead of modifying existing image recipes, a new custom image recipe was created.

Advantages include:

- Easy customization
- Better maintainability
- Reusable configuration
- Modular development
- Simplified debugging

---

# Image Recipe Location

The custom image recipe is located at:

```text
meta-rdkb-ap/
└── recipes-core/
    └── images/
        └── core-image-rdkb-ap.bb
```

This follows the standard Yocto directory structure.

---

# Purpose of the Image Recipe

The image recipe performs the following tasks:

- Inherits a base Linux image
- Adds networking packages
- Adds Wi-Fi utilities
- Installs custom configuration package
- Includes firewall utilities
- Generates the final bootable Raspberry Pi image

---

# Complete Image Recipe

The final image recipe used in this project is shown below.

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

---

# Understanding Each Line

## SUMMARY

```bitbake
SUMMARY = "RDK-B Raspberry Pi Access Point Image"
```

Provides a short description of the image.

This information appears during BitBake parsing and helps identify the purpose of the recipe.

---

## LICENSE

```bitbake
LICENSE = "MIT"
```

Specifies the license associated with the image recipe.

Since this recipe only combines existing packages, the MIT license is appropriate.

---

## Base Image

```bitbake
require recipes-core/images/core-image-base.bb
```

This line imports the standard **core-image-base** recipe.

The resulting image already includes:

- Linux kernel
- BusyBox
- Basic shell
- Networking support
- SSH support (when included)
- System utilities

Instead of building everything from scratch, this project extends the existing base image.

---

# Adding Packages

The following statement installs additional software into the image.

```bitbake
IMAGE_INSTALL:append = " \
    hostapd \
    dnsmasq \
    iw \
    iptables \
    rdkb-ap-config \
"
```

Each package has a specific role.

---

# hostapd

```text
hostapd
```

Purpose:

Creates the wireless Access Point.

Responsibilities:

- Broadcast SSID
- WPA2 authentication
- Wi-Fi encryption
- Client association

Without HostAPD:

- No wireless network
- No SSID broadcast
- No Wi-Fi connectivity

---

# dnsmasq

```text
dnsmasq
```

Purpose:

Provides DHCP and DNS services.

Responsibilities:

- Assign IP addresses
- Manage DHCP leases
- Provide DNS forwarding

Without dnsmasq:

- Clients connect to Wi-Fi
- But receive no IP address
- Internet becomes unavailable

---

# iw

```text
iw
```

Purpose:

Wireless interface configuration utility.

Used for:

- Display wireless interfaces
- Configure wireless devices
- Verify Wi-Fi status
- Debug wireless hardware

Example:

```bash
iw dev
```

---

# iptables

```text
iptables
```

Purpose:

Provides firewall and Network Address Translation (NAT).

Responsibilities:

- Enable packet forwarding
- Perform masquerading
- Share Ethernet internet with Wi-Fi clients

Without iptables:

- Clients receive IP addresses
- Clients connect to the AP
- Internet access does not work

---

# rdkb-ap-config

```text
rdkb-ap-config
```

This is the custom package developed specifically for this project.

It installs:

- hostapd.conf
- dnsmasq.conf
- rdkb-network.sh
- rdkb-network.service

This package automates the Access Point configuration during boot.

---

# Build Dependency Flow

The image recipe instructs BitBake to build and include the required packages.

```text
core-image-rdkb-ap.bb

        │

        ▼

Build hostapd

        │

        ▼

Build dnsmasq

        │

        ▼

Build iw

        │

        ▼

Build iptables

        │

        ▼

Build rdkb-ap-config

        │

        ▼

Create Root Filesystem

        │

        ▼

Generate Raspberry Pi Image
```

---

# Image Generation Process

```text
Image Recipe

        │

        ▼

BitBake

        │

        ▼

Collect Packages

        │

        ▼

Resolve Dependencies

        │

        ▼

Create Root Filesystem

        │

        ▼

Generate Kernel

        │

        ▼

Generate Boot Partition

        │

        ▼

Generate .wic Image
```

---

# Verifying Installed Packages

After building the image, the package list can be verified using:

```bash
bitbake -e core-image-rdkb-ap | grep "^IMAGE_INSTALL"
```

Example output:

```text
IMAGE_INSTALL="packagegroup-core-boot \
packagegroup-base-extended \
hostapd \
dnsmasq \
iw \
iptables \
rdkb-ap-config"
```

This confirms that all required packages are included in the final image.

---

# Output Image

Building this recipe generates several output files in the deployment directory.

Example:

```text
tmp/deploy/images/raspberrypi4-64/

core-image-rdkb-ap-raspberrypi4-64-<timestamp>.rootfs.wic.bz2

core-image-rdkb-ap.wic.bz2

core-image-rdkb-ap.wic.bmap

core-image-rdkb-ap.manifest

core-image-rdkb-ap.testdata.json
```

The `.wic.bz2` file is later decompressed and flashed to the Raspberry Pi SD card.

---

# Advantages of the Custom Image Recipe

Using a dedicated image recipe provides several benefits:

- Centralized package management
- Easy addition or removal of packages
- Reproducible builds
- Clear project organization
- Reusable for future enhancements
- Follows Yocto best practices

---

# Challenges Encountered

During development, several modifications were made to the image recipe.

Initially, only the following packages were included:

```text
hostapd
dnsmasq
iw
rdkb-ap-config
```

The Raspberry Pi successfully booted and clients could connect to the Wi-Fi Access Point. However, connected devices could not access the Internet because NAT functionality was missing.

The issue was resolved by adding:

```text
iptables
```

to the `IMAGE_INSTALL` list. After rebuilding the image, NAT rules configured by the network script functioned correctly, allowing multiple Wi-Fi clients to access the Internet through the Raspberry Pi's Ethernet connection.

---

# Summary

The `core-image-rdkb-ap.bb` image recipe is the central definition of the customized operating system built for this project. By extending `core-image-base` and including HostAPD, DNSMasq, iptables, wireless utilities, and the custom `rdkb-ap-config` package, it creates a complete Raspberry Pi image capable of operating as an RDK-B Wi-Fi Access Point. This recipe ensures that all required software is automatically included during every build, resulting in a consistent, reproducible, and maintainable embedded Linux image.