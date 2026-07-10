# Meta Layer Creation

## Introduction

Yocto uses a modular architecture where software components, configurations, recipes, and metadata are organized into **layers**. Each layer serves a specific purpose and can be added or removed independently without affecting the core build system.

For this project, instead of modifying the existing Yocto or Raspberry Pi layers, a new custom layer named **meta-rdkb-ap** was created. This layer contains all the custom recipes, configuration files, scripts, and services required to transform a standard Raspberry Pi Linux image into an RDK-B Wi-Fi Access Point.

Creating a separate layer keeps the project organized, reusable, and easier to maintain.

---

# Why Create a Custom Layer?

Directly modifying existing layers such as:

- meta
- meta-raspberrypi
- meta-openembedded

is considered poor practice because:

- Future updates may overwrite custom changes.
- Debugging becomes difficult.
- Reusability is reduced.
- Project maintenance becomes complicated.

Instead, Yocto recommends placing all customizations inside a dedicated layer.

---

# Benefits of a Custom Layer

Using a custom layer provides several advantages:

- Keeps custom code separate from official layers.
- Simplifies project maintenance.
- Makes future updates easier.
- Allows easy sharing of the project.
- Supports version control using Git.
- Follows Yocto Project best practices.

---

# Project Layer

For this project, the custom layer created was:

```text
meta-rdkb-ap
```

This layer contains:

- Custom image recipe
- Access Point configuration package
- Network configuration script
- Systemd service
- HostAPD configuration
- DNSMasq configuration
- Layer configuration files

---

# Layer Architecture

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
│   ├── hostapd/
│   │   └── hostapd_%.bbappend
│   │
│   └── rdkb-ap-config/
│       ├── rdkb-ap-config.bb
│       └── files/
│           ├── hostapd.conf
│           ├── dnsmasq.conf
│           ├── rdkb-network.sh
│           └── rdkb-network.service
```

Each directory has a specific responsibility.

---

# Layer Components

## conf/

Contains:

```text
layer.conf
```

Purpose:

Registers the layer with BitBake.

Defines:

- Layer name
- Search paths
- Recipe priorities
- Compatibility

Without this file, BitBake cannot recognize the layer.

---

## recipes-core/

Contains:

```text
core-image-rdkb-ap.bb
```

Purpose:

Defines the complete Linux image.

Specifies:

- Packages
- Dependencies
- Additional software
- Custom configurations

---

## recipes-connectivity/

Contains recipes related to networking.

Examples:

```text
hostapd

dnsmasq

rdkb-ap-config
```

---

## files/

Contains static configuration files.

Examples:

```text
hostapd.conf

dnsmasq.conf

rdkb-network.sh

rdkb-network.service
```

These files are copied into the root filesystem during image generation.

---

# Creating the Layer

Navigate to the project directory.

Example:

```bash
cd ~/rdkb-pi
```

Create the layer manually:

```bash
mkdir -p meta-rdkb-ap
```

Or use Yocto's helper tool:

```bash
bitbake-layers create-layer meta-rdkb-ap
```

---

# Generated Layer Structure

The command generates:

```text
meta-rdkb-ap/

conf/

recipes-example/

COPYING

README
```

The example recipe directory is not required for this project and can be removed.

---

# Creating Required Directories

Create the project directories:

```bash
mkdir -p meta-rdkb-ap/recipes-core/images

mkdir -p meta-rdkb-ap/recipes-connectivity/rdkb-ap-config/files

mkdir -p meta-rdkb-ap/recipes-connectivity/hostapd
```

Final structure:

```text
meta-rdkb-ap/

conf/

recipes-core/

recipes-connectivity/
```

---

# layer.conf

Every Yocto layer must contain:

```text
conf/layer.conf
```

Purpose:

This file informs BitBake about:

- Layer location
- Recipe paths
- Layer priority
- Compatible Yocto versions

Typical configuration:

```conf
BBPATH .= ":${LAYERDIR}"

BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
            ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "rdkb-ap"

BBFILE_PATTERN_rdkb-ap = "^${LAYERDIR}/"

BBFILE_PRIORITY_rdkb-ap = "6"

LAYERSERIES_COMPAT_rdkb-ap = "kirkstone"
```

---

# Registering the Layer

After creating the layer, it must be added to the Yocto build.

Navigate to the build directory:

```bash
cd poky/build
```

Add the layer:

```bash
bitbake-layers add-layer ../../meta-rdkb-ap
```

---

# Verifying the Layer

Display all active layers:

```bash
bitbake-layers show-layers
```

Expected output:

```text
meta

meta-poky

meta-yocto-bsp

meta-openembedded

meta-raspberrypi

meta-rdkb-ap
```

This confirms that BitBake has successfully loaded the custom layer.

---

# Layer Priority

Multiple layers may contain recipes with the same name.

BitBake uses layer priority to decide which recipe should be used.

Example:

```text
BBFILE_PRIORITY_rdkb-ap = "6"
```

Higher priority layers override lower priority layers.

This was useful when customizing HostAPD using a `.bbappend` file.

---

# How BitBake Uses the Layer

Build process:

```text
BitBake
    │
    ▼
Reads bblayers.conf
    │
    ▼
Loads meta-rdkb-ap
    │
    ▼
Reads layer.conf
    │
    ▼
Finds Recipes
    │
    ▼
Builds Packages
    │
    ▼
Generates Image
```

---

# Integration with the Build

The custom layer integrates into the Yocto build as follows:

```text
Yocto Build System

        │

        ▼

+---------------------------+

| meta                      |

+---------------------------+

        │

+---------------------------+

| meta-poky                 |

+---------------------------+

        │

+---------------------------+

| meta-openembedded         |

+---------------------------+

        │

+---------------------------+

| meta-raspberrypi          |

+---------------------------+

        │

+---------------------------+

| meta-rdkb-ap              |

|                           |

| Image Recipe              |

| AP Config                 |

| Network Script            |

| Systemd Service           |

| HostAPD Config            |

| DNSMasq Config            |

+---------------------------+

        │

        ▼

Final Raspberry Pi Image
```

---

# Why This Layer Was Required

Without a custom layer:

- No custom image recipe
- No Wi-Fi Access Point configuration
- No network initialization script
- No custom systemd service
- No automatic AP startup

The Raspberry Pi would boot as a standard Linux system rather than as an RDK-B Access Point.

---

# Best Practices Followed

During this project, the following Yocto best practices were followed:

- Created a separate custom layer.
- Avoided modifying official layers.
- Stored configuration files inside the layer.
- Used custom recipes for project-specific software.
- Used `.bbappend` only when extending existing packages.
- Maintained a clean directory hierarchy.
- Registered the layer through `bblayers.conf`.

---

# Summary

The `meta-rdkb-ap` layer is the foundation of the project's customization. It isolates all project-specific configurations, recipes, scripts, and services from the standard Yocto layers. By organizing the work into a dedicated layer, the project remains modular, maintainable, and fully aligned with Yocto development practices. This layer enabled the integration of the custom image recipe, Wi-Fi Access Point configuration, networking scripts, and systemd services that transformed a standard Raspberry Pi image into a functional RDK-B Access Point.