# Workspace Setup

## Overview

After preparing the Ubuntu development environment and installing all required software packages, the next step is to create a dedicated workspace for the Yocto Project.

A well-organized workspace simplifies project management by separating the build environment, metadata layers, custom recipes, and generated output files. Throughout this project, all development activities were performed inside a single workspace named **rdkb-pi**.

---

# Workspace Structure

The workspace contains the Yocto Project source, additional layers, custom RDK-B layer, and the build directory.

```
Home Directory
│
└── rdkb-pi/
    │
    ├── poky/
    │   ├── bitbake/
    │   ├── meta/
    │   ├── meta-poky/
    │   ├── meta-yocto-bsp/
    │   ├── oe-init-build-env
    │   └── build/
    │
    ├── meta-openembedded/
    │
    ├── meta-raspberrypi/
    │
    └── meta-rdkb-ap/
```

---

# Why Create a Separate Workspace?

Creating a dedicated workspace provides several advantages:

- Keeps all project files organized.
- Separates source code from generated build artifacts.
- Simplifies backups and version control.
- Makes collaboration easier.
- Prevents accidental modification of system files.

---

# Create the Workspace Directory

Move to the home directory.

```bash
cd ~
```

Create the project workspace.

```bash
mkdir rdkb-pi
```

Enter the workspace.

```bash
cd rdkb-pi
```

Verify the current working directory.

```bash
pwd
```

Example Output

```text
/home/scl/rdkb-pi
```

---

# Clone the Poky Repository

Poky is the reference distribution of the Yocto Project. It contains the build system, BitBake, core metadata, and essential scripts.

Clone the Poky repository.

```bash
git clone -b kirkstone git://git.yoctoproject.org/poky
```

After cloning, verify the directory.

```bash
ls
```

Expected Output

```text
poky
```

---

# Clone meta-openembedded

The **meta-openembedded** layer provides a large collection of additional software packages that are not included in the core Yocto metadata.

Clone the repository.

```bash
git clone -b kirkstone https://github.com/openembedded/meta-openembedded.git
```

Verify the directory.

```bash
ls
```

Expected Output

```text
meta-openembedded
poky
```

---

# Clone meta-raspberrypi

The **meta-raspberrypi** layer provides Board Support Package (BSP) support for Raspberry Pi hardware.

Clone the repository.

```bash
git clone -b kirkstone https://github.com/agherzan/meta-raspberrypi.git
```

Verify the directory.

```bash
ls
```

Expected Output

```text
meta-openembedded
meta-raspberrypi
poky
```

---

# Custom Layer

During this project, a custom layer named **meta-rdkb-ap** was created to store all project-specific recipes and configuration files.

This layer was added later during the customization phase.

Final workspace structure:

```text
rdkb-pi/

├── poky
├── meta-openembedded
├── meta-raspberrypi
└── meta-rdkb-ap
```

---

# Initialize the Build Environment

Navigate to the Poky directory.

```bash
cd ~/rdkb-pi/poky
```

Initialize the Yocto build environment.

```bash
source oe-init-build-env
```

After executing this command:

- The build environment variables are configured.
- A **build** directory is created automatically.
- The shell environment is prepared for BitBake.

---

# Build Directory Structure

After initialization, the following directory is created.

```text
build/

├── conf
│   ├── bblayers.conf
│   └── local.conf
│
├── downloads
│
├── sstate-cache
│
└── tmp
```

---

# Important Configuration Files

## local.conf

This file stores build-specific configuration.

Examples include:

- Target machine
- Parallel build settings
- Package formats
- Init system
- Image features

Location:

```text
build/conf/local.conf
```

---

## bblayers.conf

This file defines the metadata layers used during the build.

Location:

```text
build/conf/bblayers.conf
```

Initially, only the default Poky layers are present. Additional layers such as **meta-openembedded**, **meta-raspberrypi**, and **meta-rdkb-ap** are added later.

---

# Verify the Build Directory

Check that the build directory exists.

```bash
ls
```

Expected Output

```text
build
bitbake
meta
meta-poky
meta-yocto-bsp
oe-init-build-env
```

Enter the build directory.

```bash
cd build
```

Verify its contents.

```bash
ls
```

Expected Output

```text
conf
downloads
sstate-cache
tmp
```

---

# Verify BitBake

Confirm that BitBake is available.

```bash
bitbake --version
```

Example Output

```text
BitBake Build Tool Core version 2.x
```

This verifies that the build environment has been initialized successfully.

---

# Verify Environment Variables

Check the build directory.

```bash
echo $BUILDDIR
```

Example Output

```text
/home/scl/rdkb-pi/poky/build
```

Check the source directory.

```bash
echo $OEROOT
```

Example Output

```text
/home/scl/rdkb-pi/poky
```

These environment variables are automatically configured by the `oe-init-build-env` script.

---

# Workspace Verification Checklist

Before proceeding to the build configuration stage, ensure that:

- Workspace directory created.
- Poky repository cloned.
- meta-openembedded cloned.
- meta-raspberrypi cloned.
- Build environment initialized.
- Build directory created.
- BitBake available.
- Configuration files generated successfully.

---

# Summary

At the end of this stage, the Yocto development workspace is fully prepared. The required repositories have been downloaded, the build environment has been initialized, and the necessary configuration files have been generated.

This workspace serves as the foundation for the remainder of the project. In the next section, the Yocto build system will be configured by adding metadata layers, selecting the Raspberry Pi machine configuration, and preparing the environment for generating the custom RDK-B Access Point image.