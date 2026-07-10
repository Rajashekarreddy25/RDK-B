# recipes-connectivity

## Introduction

The `recipes-connectivity` directory contains BitBake recipes related to networking and communication services. In this project, it was used to package all the custom configuration files, startup scripts, and systemd service files required to convert the Raspberry Pi into a fully functional Wireless Access Point.

Instead of manually copying files into the target filesystem, Yocto recipes were used to install everything automatically during the image build process. This approach ensures that every generated image is consistent, reproducible, and easy to maintain.

The primary recipe in this directory is:

```text
rdkb-ap-config.bb
```

This recipe packages all project-specific networking files into a single installable package.

---

# Purpose of recipes-connectivity

The `recipes-connectivity` directory is responsible for:

- Installing Access Point configuration files.
- Installing DHCP configuration.
- Installing network initialization scripts.
- Installing systemd service files.
- Enabling automatic network configuration during boot.
- Managing project-specific networking components.

---

# Directory Structure

The directory structure used in this project is shown below.

```text
recipes-connectivity/

└── rdkb-ap-config/

    ├── files/

    │   ├── hostapd.conf

    │   ├── rdkb-ap.conf

    │   ├── rdkb-network.sh

    │   └── rdkb-network.service

    │

    └── rdkb-ap-config.bb
```

The directory consists of:

- One BitBake recipe.
- One `files` directory containing all configuration files.

---

# rdkb-ap-config.bb

The `rdkb-ap-config.bb` recipe is responsible for installing all networking-related files into the target root filesystem.

Main responsibilities include:

- Installing configuration files.
- Installing executable scripts.
- Installing systemd service files.
- Setting file permissions.
- Enabling automatic startup.
- Declaring runtime dependencies.

---

# Build Flow

```text
BitBake

   │

   ▼

rdkb-ap-config.bb

   │

   ▼

Copies Files

   │

   ├────────► hostapd.conf

   ├────────► rdkb-ap.conf

   ├────────► rdkb-network.sh

   └────────► rdkb-network.service

   │

   ▼

Target Root Filesystem

   │

   ▼

Bootable Image
```

---

# Files Installed

The recipe installs the following files into the target system.

| File | Installation Location |
|------|------------------------|
| hostapd.conf | `/etc/hostapd.conf` |
| rdkb-ap.conf | `/etc/dnsmasq.d/rdkb-ap.conf` |
| rdkb-network.sh | `/usr/bin/rdkb-network.sh` |
| rdkb-network.service | `/lib/systemd/system/rdkb-network.service` |

---

# Installing hostapd Configuration

The recipe copies:

```text
hostapd.conf
```

Destination:

```text
/etc/hostapd.conf
```

Purpose:

- Configure SSID.
- Configure WPA2 security.
- Configure wireless interface.
- Configure Wi-Fi channel.
- Configure country code.

---

# Installing dnsmasq Configuration

The recipe installs:

```text
rdkb-ap.conf
```

Destination:

```text
/etc/dnsmasq.d/
```

Purpose:

- Configure DHCP server.
- Define DHCP range.
- Configure gateway.
- Configure DNS server.

---

# Installing Network Script

The recipe installs:

```text
rdkb-network.sh
```

Destination:

```text
/usr/bin/
```

Permissions:

```text
Executable
```

Responsibilities:

- Bring up `wlan0`.
- Assign static IP address.
- Enable IPv4 forwarding.
- Configure NAT using `iptables`.
- Configure packet forwarding.

---

# Installing systemd Service

The recipe installs:

```text
rdkb-network.service
```

Destination:

```text
/lib/systemd/system/
```

Purpose:

- Automatically execute `rdkb-network.sh` during boot.
- Eliminate manual network configuration.
- Ensure networking is initialized before users connect.

---

# Runtime Dependencies

The recipe specifies runtime dependencies to ensure that all required networking packages are available.

Examples include:

```text
hostapd

dnsmasq

iptables
```

These packages are installed through the image recipe and are required for the proper functioning of the custom configuration package.

---

# File Installation Process

The installation process during the build is illustrated below.

```text
files/

    │

    ├────────► hostapd.conf

    ├────────► rdkb-ap.conf

    ├────────► rdkb-network.sh

    └────────► rdkb-network.service

               │

               ▼

        rdkb-ap-config.bb

               │

               ▼

         Root Filesystem

               │

               ▼

        Raspberry Pi Image
```

---

# Image Verification

After building the image, the installed files were verified before flashing.

Verify the network script:

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Verify the Access Point configuration:

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Verify the DHCP configuration:

```bash
cat /mnt/verify-root/etc/dnsmasq.d/rdkb-ap.conf
```

Verify the systemd service:

```bash
ls /mnt/verify-root/lib/systemd/system
```

Expected output included:

```text
rdkb-network.service

hostapd.service

dnsmasq.service
```

---

# Rebuilding After Changes

Whenever any file inside the `files/` directory was modified, the package recipe needed to be rebuilt.

Clean the recipe:

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild the image:

```bash
bitbake core-image-rdkb-ap
```

This ensured that the latest configuration files were included in the generated image.

---

# Actual Implementation in the Project

During the development of this project, the `recipes-connectivity` directory was used to package all custom networking components.

The recipe successfully installed:

- `hostapd.conf`
- `rdkb-ap.conf`
- `rdkb-network.sh`
- `rdkb-network.service`

These files were automatically copied into the appropriate locations within the root filesystem during the Yocto build process. The generated image was then mounted and inspected to verify that every file had been installed correctly before flashing it onto the Raspberry Pi.

---

# Advantages of Using recipes-connectivity

Using a dedicated networking recipe provides several benefits:

- Keeps networking files organized.
- Avoids manual file copying.
- Simplifies future modifications.
- Supports reproducible builds.
- Improves project modularity.
- Makes debugging easier.
- Follows Yocto Project best practices.

---

# Lessons Learned

Working with the `recipes-connectivity` directory provided several valuable insights:

- All custom networking files should be packaged through BitBake recipes.
- Configuration files should never be copied manually after boot.
- Grouping related files into a single package improves maintainability.
- Cleaning the recipe after modifying files prevents stale build artifacts.
- Verifying installed files before flashing reduces debugging time.
- Using systemd services ensures automatic configuration during every boot.

---

# Conclusion

The `recipes-connectivity` directory plays a central role in integrating all networking-related customizations into the Yocto image. Through the `rdkb-ap-config.bb` recipe, configuration files, startup scripts, and systemd service definitions are packaged and installed automatically, ensuring that every generated image is fully configured as a wireless Access Point. This modular design improves maintainability, simplifies debugging, and aligns with Yocto Project best practices for embedded Linux development.