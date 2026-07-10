# hostapd_conflict

## Introduction

During the implementation of the RDK-B Raspberry Pi Access Point, one of the major challenges encountered was configuring the **hostapd** service correctly. Although the `hostapd` package was successfully included in the Yocto image, the Access Point did not start as expected during the initial boot.

The Raspberry Pi booted normally, but no wireless network (SSID) was visible to nearby devices. This indicated that the Access Point service was either not starting correctly or was missing the required configuration.

This document explains the problem, investigation process, root cause, and the final solution.

---

# Problem Description

After successfully building and flashing the Yocto image:

- Raspberry Pi booted successfully.
- Wireless interface (`wlan0`) was detected.
- `hostapd` package was installed.
- No Wi-Fi SSID was broadcast.

Users were unable to discover or connect to the Access Point.

---

# Symptoms

The following observations were made:

```text
✔ Raspberry Pi Booted

✔ wlan0 Interface Available

✔ hostapd Installed

✘ SSID Not Visible

✘ Clients Could Not Discover Wi-Fi

✘ Access Point Not Created
```

---

# Investigation

The first step was to verify whether the `hostapd` service was running.

```bash
systemctl status hostapd
```

Possible output observed during debugging:

```text
hostapd.service - Hostapd IEEE 802.11 Access Point

Active: failed
```

This confirmed that the service was installed but failed during startup.

---

# Verify Configuration File

The next step was to check whether the configuration file existed.

```bash
ls -l /etc/hostapd.conf
```

Then inspect the configuration.

```bash
cat /etc/hostapd.conf
```

Initially, either:

- the file was missing, or
- the configuration was incomplete.

Without a valid configuration, `hostapd` cannot initialize the wireless interface.

---

# Root Cause Analysis

The investigation revealed that although the `hostapd` package was included in the image, the required configuration file had not been properly installed.

`hostapd` depends on a valid configuration file containing:

- Wireless interface
- Driver
- SSID
- Authentication method
- WPA settings
- Channel
- Country code

Without this configuration, the service terminates immediately after startup.

---

# Solution

A custom configuration file named:

```text
hostapd.conf
```

was created inside the custom layer.

Directory:

```text
meta-rdkb-ap/

recipes-connectivity/

rdkb-ap-config/

files/
```

---

# Configuration Used

The final configuration used in this project was:

```text
interface=wlan0

driver=nl80211

ssid=RDKB_AP

hw_mode=g

channel=6

country_code=IN

ieee80211n=1

wpa=2

wpa_passphrase=12345678

wpa_key_mgmt=WPA-PSK

rsn_pairwise=CCMP
```

---

# Recipe Modification

The `rdkb-ap-config.bb` recipe was updated to install the configuration file.

Example:

```bitbake
install -m 0644 ${WORKDIR}/hostapd.conf \
${D}${sysconfdir}/hostapd.conf
```

This ensured that the configuration file became part of every generated image.

---

# Rebuild Process

After updating the recipe:

Clean the recipe:

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild the image:

```bash
bitbake core-image-rdkb-ap
```

Flash the updated image to the SD card.

---

# Verification Before Flashing

Before writing the image to the SD card, the generated image was verified.

Mount the image:

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Verify the configuration file:

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

The expected configuration was present.

---

# Verification After Boot

Check the service status.

```bash
systemctl status hostapd
```

Expected Output:

```text
Active: active (running)
```

Verify that the wireless network is being broadcast.

Using another device:

```text
SSID:

RDKB_AP
```

The Access Point was now visible.

---

# Project Workflow After Fix

```text
Power ON

      │

      ▼

Linux Boot

      │

      ▼

hostapd Starts

      │

      ▼

Reads hostapd.conf

      │

      ▼

Configure wlan0

      │

      ▼

Broadcast SSID

      │

      ▼

Client Discovers Wi-Fi
```

---

# Testing Performed

After the fix, the following tests were performed successfully:

- SSID was visible.
- Android phones detected the network.
- Laptops detected the network.
- Clients authenticated using WPA2.
- Multiple devices connected simultaneously.
- `hostapd` started automatically after every reboot.

---

# Lessons Learned

Working through this issue provided several important insights:

- Installing `hostapd` alone is not sufficient.
- A valid configuration file is mandatory.
- Every configuration file should be packaged inside the custom Yocto layer.
- Verifying the generated image before flashing saves significant debugging time.
- `systemctl` and `journalctl` are essential tools for diagnosing service startup failures.

---

# Best Practices

- Store `hostapd.conf` inside the `files` directory of the custom layer.
- Install the configuration using a BitBake recipe.
- Validate the configuration before rebuilding the image.
- Verify the installed file by mounting the generated image.
- Check the service status after every boot.
- Keep wireless parameters (SSID, channel, security) clearly documented.

---

# Final Result

After packaging the correct `hostapd.conf` into the Yocto image and rebuilding the project:

- The `hostapd` service started successfully during boot.
- The Raspberry Pi broadcast the configured SSID.
- Wireless clients could discover and connect to the Access Point.
- WPA2 authentication worked correctly.
- The Access Point operated reliably after every reboot without requiring any manual intervention.

This resolved the second major issue encountered during the implementation of the RDK-B Raspberry Pi Access Point project.