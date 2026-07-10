# Yocto_Commands

## Introduction

Throughout this project, numerous Yocto commands were used to configure the build environment, manage layers, build the Linux image, clean cached artifacts, and verify the build configuration. This document serves as a complete reference for all Yocto-related commands used during the development of the RDK-B Raspberry Pi Access Point project.

---

# Yocto Workflow

```text
Create Workspace

        │

        ▼

Initialize Build Environment

        │

        ▼

Configure Build

        │

        ▼

Add Layers

        │

        ▼

Build Image

        │

        ▼

Generate Output Images

        │

        ▼

Flash Raspberry Pi
```

---

# 1. Clone Poky Repository

Clone the Yocto Project reference distribution.

```bash
git clone git://git.yoctoproject.org/poky
```

Purpose:

- Downloads the Yocto build system.
- Creates the build environment.

---

# 2. Checkout Required Branch

Switch to the required Yocto release.

Example:

```bash
git checkout kirkstone
```

Purpose:

- Uses a stable Yocto version.
- Matches compatible layers.

---

# 3. Source the Build Environment

Initialize the Yocto build environment.

```bash
source oe-init-build-env
```

Purpose:

- Creates the `build/` directory.
- Sets environment variables.
- Prepares BitBake.

Result:

```text
build/

conf/

tmp/
```

---

# 4. View Available Machines

Display supported hardware platforms.

```bash
ls meta-raspberrypi/conf/machine/
```

Purpose:

- Lists Raspberry Pi machine configurations.

Example:

```text
raspberrypi4.conf

raspberrypi4-64.conf
```

---

# 5. Configure Machine

Edit:

```bash
nano conf/local.conf
```

Set:

```text
MACHINE = "raspberrypi4-64"
```

Purpose:

- Select Raspberry Pi 4 (64-bit).

---

# 6. Add Custom Layer

Register the custom layer.

```bash
bitbake-layers add-layer ../meta-rdkb-ap
```

Purpose:

- Makes BitBake recognize the custom recipes.

---

# 7. List Installed Layers

Display all registered layers.

```bash
bitbake-layers show-layers
```

Example:

```text
meta

meta-poky

meta-yocto-bsp

meta-oe

meta-python

meta-networking

meta-raspberrypi

meta-rdkb-ap
```

Purpose:

- Verify layer configuration.

---

# 8. Show Available Recipes

Display all recipes.

```bash
bitbake-layers show-recipes
```

Purpose:

- Verify recipe availability.

---

# 9. Show Image Recipes

Display image recipes only.

```bash
bitbake-layers show-recipes | grep image
```

Purpose:

- Verify image recipe registration.

---

# 10. Verify Build Configuration

Display current build configuration.

```bash
bitbake -e core-image-rdkb-ap
```

Purpose:

- Display build variables.
- Debug image configuration.

---

# 11. Verify Installed Packages

Display installed image packages.

```bash
bitbake -e core-image-rdkb-ap | grep "^IMAGE_INSTALL"
```

Example:

```text
IMAGE_INSTALL="hostapd dnsmasq iw iptables rdkb-ap-config"
```

Purpose:

- Confirm required packages are included.

---

# 12. Build the Image

Generate the complete Linux image.

```bash
bitbake core-image-rdkb-ap
```

Purpose:

- Compile packages.
- Create root filesystem.
- Generate deployable images.

---

# 13. Clean a Recipe

Remove build artifacts for a specific recipe.

```bash
bitbake -c clean rdkb-ap-config
```

Purpose:

- Force recipe rebuild.
- Apply configuration changes.

---

# 14. Clean Complete Image

Remove image build output.

```bash
bitbake -c clean core-image-rdkb-ap
```

Purpose:

- Rebuild image from scratch.

---

# 15. Clean Shared State Cache

Remove cached build artifacts.

```bash
bitbake -c cleansstate rdkb-ap-config
```

Purpose:

- Eliminate cached packages.
- Force complete recompilation.

---

# 16. List Build Tasks

Display available BitBake tasks.

```bash
bitbake -c listtasks core-image-rdkb-ap
```

Purpose:

- View recipe tasks.

---

# 17. Execute a Specific Task

Run only one task.

Example:

```bash
bitbake -c compile hostapd
```

Purpose:

- Debug build process.

---

# 18. Show Build Dependencies

Display recipe dependencies.

```bash
bitbake -g core-image-rdkb-ap
```

Purpose:

- Generate dependency graphs.

Output:

```text
pn-buildlist

task-depends.dot
```

---

# 19. Verify Layer Priority

Display priorities.

```bash
bitbake-layers show-layers
```

Purpose:

- Resolve recipe conflicts.

---

# 20. Display Package Environment

Display package variables.

Example:

```bash
bitbake -e hostapd
```

Purpose:

- Debug recipe variables.

---

# 21. Locate Build Output

Navigate to generated images.

```bash
cd tmp/deploy/images/raspberrypi4-64
```

Purpose:

- Access generated image files.

---

# 22. View Build Logs

Navigate to build logs.

```bash
cd tmp/work
```

Purpose:

- Debug build failures.

---

# 23. Verify Generated Images

List deployment files.

```bash
ls tmp/deploy/images/raspberrypi4-64/
```

Example:

```text
core-image-rdkb-ap.rootfs.wic.bz2

Image

bcm2711-rpi-4-b.dtb
```

---

# 24. Search for Recipes

Search a recipe.

```bash
bitbake-layers show-recipes | grep hostapd
```

Purpose:

- Confirm recipe availability.

---

# 25. Display Layer Information

Display layer details.

```bash
bitbake-layers show-layers
```

Purpose:

- Verify active layers.

---

# Commands Used During This Project

The following commands were frequently used:

```bash
source oe-init-build-env
```

```bash
bitbake-layers add-layer ../meta-rdkb-ap
```

```bash
bitbake-layers show-layers
```

```bash
bitbake core-image-rdkb-ap
```

```bash
bitbake -c clean rdkb-ap-config
```

```bash
bitbake -e core-image-rdkb-ap
```

```bash
bitbake -e core-image-rdkb-ap | grep "^IMAGE_INSTALL"
```

```bash
ls tmp/deploy/images/raspberrypi4-64/
```

---

# Best Practices

- Always source the build environment before running BitBake.
- Verify layers after adding a new layer.
- Clean recipes after modifying configuration files.
- Verify `IMAGE_INSTALL` before rebuilding.
- Inspect generated images before flashing.
- Keep the build environment organized.
- Use `cleansstate` only when necessary, as it significantly increases build time.

---

# Conclusion

The Yocto commands documented here were essential throughout the development of the RDK-B Raspberry Pi Access Point project. They enabled workspace initialization, layer management, build configuration, image generation, recipe maintenance, and debugging. Mastering these commands made it possible to efficiently build, customize, and verify the embedded Linux image while following Yocto Project best practices.