# meta-rdkb-ap

## Introduction

The `meta-rdkb-ap` layer is the custom Yocto layer developed specifically for this project. It contains all the files required to transform a standard Yocto Linux image into a fully functional Wireless Access Point running on the Raspberry Pi 4.

Instead of modifying existing Yocto layers, a separate custom layer was created to keep the project modular, maintainable, and reusable. This approach follows Yocto Project best practices by isolating project-specific customizations from upstream layers.

The `meta-rdkb-ap` layer includes:

- Custom image recipe
- Configuration files
- Network scripts
- Systemd service files
- Recipes for installing project-specific components

---

# Purpose of the Layer

The primary objectives of the `meta-rdkb-ap` layer are:

- Create a custom Linux image.
- Install networking packages.
- Configure Wi-Fi Access Point.
- Configure DHCP server.
- Configure NAT.
- Install startup scripts.
- Enable automatic service startup.

---

# Layer Position in Yocto

```text
Yocto Project

│

├── meta

├── meta-poky

├── meta-yocto-bsp

├── meta-openembedded

│      ├── meta-oe

│      ├── meta-python

│      └── meta-networking

│

└── meta-rdkb-ap
       │
       ├── Image Recipe
       ├── Connectivity Recipes
       ├── Configuration Files
       ├── Scripts
       └── Services
```

The `meta-rdkb-ap` layer extends the functionality provided by the standard Yocto layers without modifying them directly.

---

# Directory Structure

The layer is organized as follows:

```text
meta-rdkb-ap/

├── conf/
│   └── layer.conf
│
├── recipes-core/
│   └── images/
│       └── core-image-rdkb-ap.bb
│
├── recipes-connectivity/
│   └── rdkb-ap-config/
│       ├── rdkb-ap-config.bb
│       └── files/
│            ├── hostapd.conf
│            ├── rdkb-ap.conf
│            ├── rdkb-network.sh
│            └── rdkb-network.service
│
└── README
```

Each directory has a specific purpose.

---

# conf/

The `conf` directory contains layer configuration files.

Example:

```text
conf/

└── layer.conf
```

The `layer.conf` file informs BitBake about:

- Layer location
- Recipe search paths
- Layer priority
- Compatibility

Typical responsibilities include:

- Registering the layer
- Setting `BBPATH`
- Defining `BBFILES`
- Declaring compatible Yocto versions

Without this file, BitBake cannot recognize the custom layer.

---

# recipes-core/

This directory contains custom image recipes.

Example:

```text
recipes-core/

└── images/

    └── core-image-rdkb-ap.bb
```

The image recipe defines:

- Packages to install
- Base image configuration
- Final root filesystem contents

Example packages included:

```text
hostapd

dnsmasq

iw

iptables

rdkb-ap-config
```

The image recipe is the entry point for building the final system.

---

# recipes-connectivity/

This directory contains recipes related to networking and connectivity.

Example:

```text
recipes-connectivity/

└── rdkb-ap-config/
```

This recipe installs all project-specific networking files required for the Access Point.

---

# rdkb-ap-config.bb

The custom BitBake recipe performs the following tasks:

- Installs configuration files.
- Installs startup scripts.
- Installs systemd service files.
- Enables automatic service startup.
- Declares runtime dependencies.

The recipe integrates all networking components into the final image.

---

# files/

The `files` directory stores the actual files copied into the target root filesystem during the build.

Structure:

```text
files/

├── hostapd.conf

├── rdkb-ap.conf

├── rdkb-network.sh

└── rdkb-network.service
```

These files are installed into their appropriate locations by the BitBake recipe.

---

# hostapd.conf

Purpose:

Configure the wireless Access Point.

Installed to:

```text
/etc/hostapd.conf
```

Contains:

- SSID
- WPA2 password
- Channel
- Country code
- Driver
- Interface

---

# rdkb-ap.conf

Purpose:

Configure the DHCP server.

Installed to:

```text
/etc/dnsmasq.d/
```

Contains:

- DHCP range
- Gateway
- DNS server
- Interface

---

# rdkb-network.sh

Purpose:

Configure networking during boot.

Installed to:

```text
/usr/bin/
```

Responsibilities:

- Bring up `wlan0`
- Assign static IP
- Enable IP forwarding
- Configure NAT
- Configure forwarding rules

This script is executed automatically by `systemd`.

---

# rdkb-network.service

Purpose:

Start the network configuration script during system boot.

Installed to:

```text
/ lib/systemd/system/
```

Responsibilities:

- Execute `rdkb-network.sh`
- Ensure networking is configured automatically
- Eliminate manual configuration after every reboot

---

# Integration with Build Process

The interaction between the layer components is illustrated below.

```text
meta-rdkb-ap

      │

      ▼

Image Recipe

      │

      ▼

Includes rdkb-ap-config

      │

      ▼

Installs Files

      │

      ├────────► hostapd.conf

      ├────────► rdkb-ap.conf

      ├────────► rdkb-network.sh

      └────────► rdkb-network.service

      │

      ▼

Generated Root Filesystem

      │

      ▼

Flashed to Raspberry Pi

      │

      ▼

Automatic AP Configuration
```

---

# Advantages of Using a Separate Layer

Creating a dedicated layer provides several advantages:

- Keeps project organized.
- Avoids modifying upstream Yocto layers.
- Simplifies maintenance.
- Enables easy reuse in future projects.
- Supports version control.
- Makes debugging easier.
- Follows Yocto Project best practices.

---

# Verification

After building the image, the layer contents were verified by mounting the generated `.wic` image and checking the installed files.

Examples:

Verify script:

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Verify configuration:

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Verify DHCP configuration:

```bash
cat /mnt/verify-root/etc/dnsmasq.d/rdkb-ap.conf
```

Verify service:

```bash
ls /mnt/verify-root/lib/systemd/system
```

These checks confirmed that the files provided by the `meta-rdkb-ap` layer were correctly installed into the final image.

---

# Lessons Learned

Developing the `meta-rdkb-ap` layer provided several important insights:

- Custom functionality should always reside in a dedicated layer.
- Recipes should install only project-specific files.
- Configuration files should be version-controlled.
- Modular layer design simplifies debugging and future enhancements.
- Separating image recipes from package recipes improves maintainability.
- Verifying image contents before flashing reduces deployment errors.

---

# Conclusion

The `meta-rdkb-ap` layer is the core customization layer of the RDK-B Raspberry Pi Access Point project. It encapsulates all project-specific recipes, configuration files, scripts, and service definitions required to transform a standard Yocto Linux image into a fully functional wireless Access Point. By following Yocto's layered architecture, the project remains modular, maintainable, and easily extensible, providing a solid foundation for future networking and RDK-B enhancements.