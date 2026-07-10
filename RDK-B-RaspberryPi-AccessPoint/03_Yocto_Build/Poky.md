# Poky

## Overview

Poky is the reference build system provided by the Yocto Project. It combines the BitBake build engine, OpenEmbedded Core metadata, build scripts, and reference configuration into a single repository.

In this project, Poky served as the foundation for building a custom Linux image for the Raspberry Pi 4. Every component of the operating system—including the Linux kernel, root filesystem, networking packages, and custom RDK-B configuration—was built using Poky.

---

# What is Poky?

Poky is not an operating system.

It is a complete build environment that provides:

- BitBake build engine
- OpenEmbedded Core metadata
- Build scripts
- Default configuration
- Reference distribution

It simplifies the process of building custom Linux distributions for embedded devices.

---

# Why Poky?

Instead of manually collecting build tools and metadata, Poky provides everything required to start developing immediately.

Advantages include:

- Official Yocto reference distribution
- Stable build environment
- Integrated BitBake
- OpenEmbedded Core metadata
- Easy layer integration
- Supports multiple hardware platforms
- Regularly maintained by the Yocto Project

---

# Poky Repository

The repository was cloned using:

```bash
cd ~/rdkb-pi

git clone -b kirkstone git://git.yoctoproject.org/poky
```

Project structure after cloning:

```
rdkb-pi
│
├── poky
├── meta-openembedded
├── meta-raspberrypi
└── meta-rdkb-ap
```

---

# Poky Directory Structure

Inside the Poky repository:

```
poky/
│
├── bitbake/
├── documentation/
├── meta/
├── meta-poky/
├── meta-selftest/
├── meta-skeleton/
├── meta-yocto-bsp/
├── oe-init-build-env
├── scripts/
├── LICENSE
└── README
```

Each directory has a specific purpose.

---

# bitbake/

```
poky/
└── bitbake/
```

This directory contains the BitBake build engine.

Responsibilities:

- Parses recipes
- Resolves dependencies
- Executes build tasks
- Generates packages
- Creates filesystem images

BitBake is the "brain" of the Yocto build system.

---

# meta/

```
poky/
└── meta/
```

This is the OpenEmbedded-Core layer.

It contains:

- Base recipes
- Toolchain recipes
- Core libraries
- BusyBox
- systemd
- Kernel recipes
- Image recipes

Without this layer, Yocto cannot build a Linux system.

---

# meta-poky/

```
poky/
└── meta-poky/
```

This layer provides the default Poky distribution configuration.

Responsibilities:

- Distribution policies
- Default package selections
- Build configuration
- Distribution settings

---

# meta-yocto-bsp/

```
poky/
└── meta-yocto-bsp/
```

Provides example Board Support Packages (BSPs).

Although our project uses **meta-raspberrypi** for Raspberry Pi support, this layer is included with Poky as part of the reference distribution.

---

# meta-selftest/

Contains automated tests used by the Yocto Project developers.

Not required for normal image development.

---

# meta-skeleton/

Contains example recipes demonstrating how to create custom layers and recipes.

Useful for learning and reference.

---

# documentation/

Contains official Yocto documentation.

Includes:

- Manuals
- Reference guides
- Development documentation

---

# scripts/

Contains helper scripts used throughout the build process.

Examples include:

- Layer management
- Image generation
- Development utilities

---

# oe-init-build-env

One of the most important files in Poky.

Location:

```
poky/
└── oe-init-build-env
```

This script initializes the Yocto build environment.

---

# Why is oe-init-build-env Important?

Before using BitBake, the build environment must be configured.

The script performs several tasks:

- Creates the build directory
- Sets environment variables
- Configures paths
- Prepares BitBake
- Generates default configuration files

Without executing this script, BitBake cannot function correctly.

---

# Initializing the Build Environment

Navigate to the Poky directory.

```bash
cd ~/rdkb-pi/poky
```

Run:

```bash
source oe-init-build-env
```

After execution:

```
poky/

├── build
│
├── bitbake
├── meta
├── meta-poky
└── meta-yocto-bsp
```

A new **build/** directory is created automatically.

---

# Build Directory

The build directory contains all generated files.

```
build/
│
├── conf
├── downloads
├── sstate-cache
└── tmp
```

This directory should not be confused with the Poky source itself.

---

# conf/

Contains build configuration.

Files:

```
conf/

├── bblayers.conf
└── local.conf
```

These files define:

- Machine
- Layers
- Package format
- Build options
- Distribution settings

---

# downloads/

Stores downloaded source packages.

Advantages:

- Sources downloaded only once
- Faster rebuilds
- Reduced network usage

---

# sstate-cache/

Stores shared build artifacts.

Purpose:

Instead of recompiling unchanged packages, Yocto reuses previously built components.

Benefits:

- Faster rebuilds
- Reduced compilation time

---

# tmp/

Contains temporary build output.

Includes:

```
tmp/

├── deploy
├── work
├── sysroots
└── log
```

This directory grows significantly during compilation.

---

# Environment Variables

Running

```bash
source oe-init-build-env
```

creates several important environment variables.

Check the build directory:

```bash
echo $BUILDDIR
```

Example:

```
/home/scl/rdkb-pi/poky/build
```

Check Poky directory:

```bash
echo $OEROOT
```

Example:

```
/home/scl/rdkb-pi/poky
```

---

# Poky Build Workflow

```
Ubuntu Host
      │
      ▼
Poky
      │
      ▼
oe-init-build-env
      │
      ▼
Build Directory
      │
      ▼
Configuration Files
      │
      ▼
BitBake
      │
      ▼
Linux Image
```

---

# Poky in This Project

Poky was used to:

- Initialize the build environment
- Manage BitBake
- Build Linux packages
- Generate the kernel
- Generate the root filesystem
- Create bootable Raspberry Pi images

Additional layers were integrated with Poky:

```
Poky
 │
 ├── meta
 ├── meta-poky
 ├── meta-yocto-bsp
 │
 ├── meta-openembedded
 ├── meta-raspberrypi
 └── meta-rdkb-ap
```

This combination formed the complete build environment for the project.

---

# Role of Poky During Build

The overall process can be summarized as follows:

```
Developer
     │
     ▼
BitBake Command
     │
     ▼
Poky
     │
     ▼
Read Configuration
     │
     ▼
Load Metadata Layers
     │
     ▼
Parse Recipes
     │
     ▼
Compile Packages
     │
     ▼
Generate Root Filesystem
     │
     ▼
Generate WIC Image
     │
     ▼
Deploy Image
```

---

# Advantages of Using Poky

During this project, Poky provided several benefits:

- Stable development environment
- Official Yocto support
- Integrated BitBake
- Easy layer management
- Automatic dependency resolution
- Reproducible builds
- Support for custom recipes
- Easy integration of Raspberry Pi BSP

These features significantly simplified the development of the custom RDK-B Access Point image.

---

# Summary

Poky is the core of the Yocto build environment. It provides the build engine, metadata, configuration, and tools required to generate custom Linux distributions. In this project, Poky was responsible for initializing the build environment, coordinating BitBake, managing metadata layers, and producing the final bootable Raspberry Pi image.

With Poky successfully configured, the next step is to customize the build environment by configuring the target machine, enabling required features, selecting the init system, and preparing the build configuration for the Raspberry Pi platform.