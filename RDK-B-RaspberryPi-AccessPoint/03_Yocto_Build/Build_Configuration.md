# Build Configuration

## Overview

After downloading Poky and all required Yocto layers, the next step is configuring the build environment.

Yocto stores all build configurations inside the **build/conf** directory.

```
build/
└── conf/
    ├── local.conf
    └── bblayers.conf
```

These two files determine how the image will be built.

---

# local.conf

`local.conf` contains machine configuration, compiler settings, image type, package manager, CPU settings and other build parameters.

Location:

```
poky/build/conf/local.conf
```

---

## Setting Raspberry Pi 4 Machine

The target hardware used in this project is Raspberry Pi 4 (64-bit).

Set

```conf
MACHINE = "raspberrypi4-64"
```

This tells BitBake to build an image specifically for Raspberry Pi 4.

---

## Download Directory

Instead of downloading packages every build, Yocto stores downloaded sources.

Example

```conf
DL_DIR ?= "${TOPDIR}/downloads"
```

Benefits

- Saves internet bandwidth
- Faster rebuilds
- Packages downloaded only once

---

## Shared State Cache (SSTATE)

Compiled packages are stored in SSTATE.

```conf
SSTATE_DIR ?= "${TOPDIR}/sstate-cache"
```

Advantages

- Faster rebuild
- Reuse previously compiled packages
- Saves build time

---

## Number of Threads

Compilation speed depends on CPU cores.

Example

```conf
BB_NUMBER_THREADS = "8"
PARALLEL_MAKE = "-j8"
```

Increase or decrease according to available CPU cores.

---

## Package Format

Yocto supports multiple package formats.

Examples

```
rpm
deb
ipk
```

In this project RPM package format is used.

---

## Image Output Format

Yocto can generate multiple image formats.

Examples

```
ext4
wic
tar
bmap
```

Our project generates

```
wic.bz2
wic.bmap
tar.bz2
ext3
```

The WIC image is flashed directly into the SD Card.

---

# Using systemd Instead of SysV Init

Initially, the generated image did not contain systemd.

As a result

- systemctl command missing
- service files ignored
- AP service could not start

To solve this problem the following configuration was added.

```conf
INIT_MANAGER = "systemd"

DISTRO_FEATURES:append = " systemd"

DISTRO_FEATURES_BACKFILL_CONSIDERED += "sysvinit"

VIRTUAL-RUNTIME_init_manager = "systemd"

VIRTUAL-RUNTIME_initscripts = ""

DISTRO_FEATURES:remove = "sysvinit"
```

---

## Explanation

### INIT_MANAGER

```conf
INIT_MANAGER = "systemd"
```

Selects systemd as the init system.

---

### DISTRO_FEATURES

```conf
DISTRO_FEATURES:append = " systemd"
```

Enables systemd feature throughout the distribution.

---

### Remove SysV

```conf
DISTRO_FEATURES:remove = "sysvinit"
```

Prevents SysV init scripts from being included.

---

### Runtime Init Manager

```conf
VIRTUAL-RUNTIME_init_manager = "systemd"
```

Makes systemd the runtime init process.

---

### Disable Old Init Scripts

```conf
VIRTUAL-RUNTIME_initscripts = ""
```

Removes unnecessary SysV initialization scripts.

---

# Verifying Configuration

Configuration can be verified using

```bash
bitbake -e core-image-rdkb-ap | grep "^INIT_MANAGER="
```

Expected

```
INIT_MANAGER="systemd"
```

---

Check runtime manager

```bash
bitbake -e core-image-rdkb-ap | grep "^VIRTUAL-RUNTIME_init_manager="
```

Expected

```
VIRTUAL-RUNTIME_init_manager="systemd"
```

---

Verify distribution features

```bash
bitbake -e core-image-rdkb-ap | grep "^DISTRO_FEATURES="
```

Output should contain

```
systemd
```

and should not contain

```
sysvinit
```

---

# Build Directory Structure

```
build/
│
├── conf/
│   ├── local.conf
│   └── bblayers.conf
│
├── downloads/
│
├── sstate-cache/
│
├── tmp/
│
└── cache/
```

---

## Purpose of Each Directory

| Directory | Purpose |
|------------|----------|
| conf | Build configuration files |
| downloads | Downloaded source packages |
| sstate-cache | Shared compiled objects |
| tmp | Temporary build output |
| cache | BitBake cache |

---

# Important Configuration Used in This Project

| Setting | Value |
|----------|-------|
| MACHINE | raspberrypi4-64 |
| INIT_MANAGER | systemd |
| Package Format | RPM |
| Image Format | wic.bz2 |
| Target Architecture | ARM64 |
| Build System | Poky Kirkstone |

---

# Common Configuration Mistakes

### Wrong MACHINE

Results in an image that cannot boot on Raspberry Pi.

---

### Forgetting systemd

Effects

- systemctl missing
- Services not enabled
- Boot issues

---

### Incorrect DISTRO_FEATURES

May result in

- Missing packages
- Wrong init system
- Build failures

---

### Excessive Thread Count

Using more threads than available CPU cores may reduce performance instead of improving it.

---

# Summary

The build configuration defines how Yocto generates the Linux image. During this project, systemd was explicitly enabled, SysV init was removed, and Raspberry Pi 4 was selected as the target machine. Proper configuration ensured that the final image included systemd, supported custom services, and successfully booted with all required networking components.