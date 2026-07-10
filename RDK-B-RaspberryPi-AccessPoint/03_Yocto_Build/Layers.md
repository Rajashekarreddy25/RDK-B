# Layers in Yocto

## Introduction

Layers are one of the most important concepts in the Yocto Project. A layer is a collection of metadata that contains recipes, configuration files, patches, classes, machine configurations, and package definitions. Layers make the build system modular, reusable, and maintainable.

Instead of placing every recipe in a single directory, Yocto separates functionality into different layers. During the build process, BitBake reads these layers and combines them to generate the final Linux image.

In this project, multiple layers were used to build a customized RDK-B Raspberry Pi Access Point image.

---

# Why Layers are Required

Without layers:

- Every package would exist in one location.
- Managing updates becomes difficult.
- Multiple developers cannot work independently.
- Customizations overwrite the original files.

Using layers provides:

- Modular development
- Easy maintenance
- Better code organization
- Vendor customization
- Reusability

---

# Layer Structure

A typical Yocto layer contains the following structure:

```text
meta-layer-name/
│
├── conf/
│   └── layer.conf
│
├── recipes-core/
│
├── recipes-kernel/
│
├── recipes-connectivity/
│
├── recipes-networking/
│
├── classes/
│
└── files/
```

---

# Layers Used in This Project

The project used the following layers.

---

## 1. meta

Purpose:

Core metadata of the Yocto Project.

Contains:

- Basic classes
- Common recipes
- Build infrastructure

Example:

```text
poky/meta
```

---

## 2. meta-poky

Purpose:

Provides the Poky reference distribution.

Contains:

- Default distribution configuration
- Package groups
- Sample images

Example:

```text
poky/meta-poky
```

---

## 3. meta-yocto-bsp

Purpose:

Provides reference Board Support Packages.

Contains:

- BSP configurations
- Machine support

Example:

```text
poky/meta-yocto-bsp
```

---

## 4. meta-openembedded

This repository provides additional packages that are not available in the basic Yocto layers.

It contains several sub-layers.

### meta-oe

Provides general-purpose packages.

Examples:

- vim
- nano
- htop
- utilities

---

### meta-python

Provides Python packages.

Examples:

- python3
- pip
- python libraries

---

### meta-networking

Provides networking packages.

Examples:

- hostapd
- dnsmasq
- iptables
- iw
- wireless tools

Without this layer, networking packages required for the Access Point cannot be built.

---

## 5. meta-raspberrypi

Purpose:

Provides Raspberry Pi Board Support Package.

Contains:

- Raspberry Pi machine configuration
- Bootloader configuration
- GPU firmware
- Kernel configuration
- Device Tree

Repository:

```text
meta-raspberrypi
```

Machine used:

```text
raspberrypi4-64
```

---

## 6. meta-rdkb-ap (Custom Layer)

This is the custom layer created specifically for this project.

Purpose:

Customize the default Yocto image to build an RDK-B Wi-Fi Access Point.

The layer contains:

- Image recipe
- Custom package
- Configuration files
- Systemd service
- Startup scripts

Directory structure:

```text
meta-rdkb-ap/
│
├── conf/
│
├── recipes-core/
│   └── images/
│
├── recipes-connectivity/
│   └── rdkb-ap-config/
│
└── files/
```

---

# Creating the Custom Layer

The custom layer was created using:

```bash
bitbake-layers create-layer ../meta-rdkb-ap
```

Example:

```bash
cd ~/rdkb-pi

bitbake-layers create-layer meta-rdkb-ap
```

Output:

```text
meta-rdkb-ap/
```

---

# Adding the Layer

After creating the layer, it must be added to the Yocto build.

Command:

```bash
bitbake-layers add-layer ../meta-rdkb-ap
```

Verification:

```bash
bitbake-layers show-layers
```

Example output:

```text
layer                 path
==================================================
meta                  poky/meta
meta-poky            poky/meta-poky
meta-yocto-bsp       poky/meta-yocto-bsp
meta-oe              meta-openembedded/meta-oe
meta-python          meta-openembedded/meta-python
meta-networking      meta-openembedded/meta-networking
meta-raspberrypi     meta-raspberrypi
meta-rdkb-ap         meta-rdkb-ap
```

---

# layer.conf

Every layer contains a configuration file.

Location:

```text
meta-rdkb-ap/conf/layer.conf
```

Purpose:

- Registers the layer with BitBake
- Specifies recipe search paths
- Defines compatibility
- Sets layer priority

Example:

```conf
BBPATH .= ":${LAYERDIR}"

BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
            ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "meta-rdkb-ap"

BBFILE_PATTERN_meta-rdkb-ap = "^${LAYERDIR}/"

BBFILE_PRIORITY_meta-rdkb-ap = "6"

LAYERSERIES_COMPAT_meta-rdkb-ap = "kirkstone"
```

---

# Recipe Search Process

During the build, BitBake searches the layers in the order defined by their priorities.

Example:

```text
meta
      ↓
meta-poky
      ↓
meta-openembedded
      ↓
meta-raspberrypi
      ↓
meta-rdkb-ap
```

If two recipes have the same name:

- Higher priority layer wins.
- `.bbappend` files modify existing recipes without replacing them.

---

# Recipes Added in meta-rdkb-ap

The custom layer includes the following recipes:

| Recipe | Purpose |
|---------|----------|
| `core-image-rdkb-ap.bb` | Defines the custom RDK-B image |
| `rdkb-ap-config.bb` | Installs configuration files and scripts |

---

# Files Added Through the Custom Layer

The following configuration files were installed into the image:

```text
hostapd.conf
dnsmasq.conf
rdkb-network.sh
rdkb-network.service
```

These files configure:

- Wi-Fi Access Point (hostapd)
- DHCP Server (dnsmasq)
- Static IP assignment
- IP forwarding
- NAT using iptables
- Automatic startup with systemd

---

# Layer Dependency

The dependency relationship among the layers is shown below:

```text
meta
│
├── meta-poky
│
├── meta-yocto-bsp
│
├── meta-openembedded
│   ├── meta-oe
│   ├── meta-python
│   └── meta-networking
│
├── meta-raspberrypi
│
└── meta-rdkb-ap
```

---

# Layer Verification Commands

Display all layers:

```bash
bitbake-layers show-layers
```

Display all recipes:

```bash
bitbake-layers show-recipes
```

Show recipe append files:

```bash
bitbake-layers show-appends
```

Show available images:

```bash
bitbake-layers show-recipes | grep image
```

---

# Summary

The layered architecture of Yocto made it possible to keep the project modular and maintainable. Standard functionality was provided by the official Yocto and OpenEmbedded layers, Raspberry Pi hardware support came from the `meta-raspberrypi` layer, and all project-specific customizations were isolated within the `meta-rdkb-ap` layer. This separation ensured that the original metadata remained unchanged while allowing the project to define its own image recipe, configuration files, startup services, and networking scripts.