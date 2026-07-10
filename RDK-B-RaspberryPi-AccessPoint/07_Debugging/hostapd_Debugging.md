# hostapd_Debugging

## Introduction

`hostapd` (Host Access Point Daemon) is the core software responsible for transforming the Raspberry Pi wireless interface into a Wi-Fi Access Point. Without `hostapd`, the Raspberry Pi cannot broadcast an SSID, authenticate wireless clients, or establish wireless communication.

During the implementation of this project, several issues related to `hostapd` were encountered and resolved. These included SSID visibility problems, client authentication failures, configuration errors, and service startup issues.

This document explains the complete debugging process followed to identify and resolve `hostapd`-related problems.

---

# Role of hostapd in the Project

```text
             Internet
                 │
                 │
              Ethernet
               (eth0)
                 │
                 ▼
        Raspberry Pi 4
      Custom Yocto Image
                 │
         hostapd Service
                 │
                 ▼
           Wireless AP
         SSID : RDKB_AP
                 │
      ┌──────────┼──────────┐
      │          │          │
      ▼          ▼          ▼
   Phone      Laptop      Tablet
```

The primary responsibilities of `hostapd` are:

- Broadcast the SSID.
- Authenticate wireless clients.
- Configure wireless security (WPA2).
- Manage wireless associations.
- Communicate with the Linux wireless stack using `nl80211`.

---

# hostapd Configuration File

The main configuration file used in the project:

```text
/etc/hostapd.conf
```

Contents:

```text
interface=wlan0
driver=nl80211

ssid=RDKB_AP

hw_mode=g
channel=6
country_code=IN

ieee80211n=1

auth_algs=1
ignore_broadcast_ssid=0

wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

---

# Debugging Workflow

```text
Boot Raspberry Pi

        │

        ▼

Start hostapd

        │

        ▼

Is SSID Visible?

        │

 ┌──────┴──────┐

 │             │

YES            NO

 │             │

 ▼             ▼

Connect     Check Service

 │             │

 ▼             ▼

Authenticate  View Logs

 │             │

 ▼             ▼

Internet?   Fix Configuration

 │             │

 └──────┬──────┘

        ▼

Retest
```

---

# Step 1 – Verify Service Status

Check whether `hostapd` is running.

```bash
systemctl status hostapd
```

Expected:

```text
Active: active (running)
```

---

# Step 2 – Verify Configuration File

Display configuration.

```bash
cat /etc/hostapd.conf
```

Verify:

- Interface
- Driver
- SSID
- WPA Password
- Country Code
- Channel

---

# Step 3 – Verify Wireless Interface

Display wireless interface.

```bash
ip link show wlan0
```

Expected:

```text
wlan0
```

If the interface is missing, `hostapd` cannot start.

---

# Step 4 – View Logs

Display logs.

```bash
journalctl -u hostapd
```

Logs reveal:

- Configuration errors
- Interface issues
- Authentication failures
- Driver problems

---

# Step 5 – Restart Service

After modifying the configuration:

```bash
systemctl restart hostapd
```

Verify again.

```bash
systemctl status hostapd
```

---

# Issue 1 – SSID Not Visible

## Symptoms

- Mobile devices could not detect the Wi-Fi network.
- Laptop Wi-Fi scan returned no SSID.

---

## Cause

`hostapd` failed to start.

---

## Debugging

```bash
systemctl status hostapd
```

```bash
journalctl -u hostapd
```

---

## Solution

Verify:

- `hostapd.conf`
- `wlan0`
- Driver (`nl80211`)

Restart service.

```bash
systemctl restart hostapd
```

---

## Result

SSID appeared successfully.

---

# Issue 2 – Authentication Failure

## Symptoms

Client attempted to connect but received:

```text
Authentication Failed
```

---

## Cause

Incorrect WPA configuration.

Examples:

- Invalid passphrase
- Incorrect WPA mode
- Typographical errors

---

## Debugging

Display configuration.

```bash
cat /etc/hostapd.conf
```

Verify:

```text
wpa=2

wpa_passphrase=12345678
```

---

## Solution

Correct configuration.

Restart service.

---

## Result

Clients connected successfully.

---

# Issue 3 – Service Failed to Start

## Symptoms

```text
Active: failed
```

---

## Cause

Possible reasons:

- Missing configuration
- Invalid interface
- Driver mismatch
- Incorrect country code

---

## Debugging

```bash
journalctl -u hostapd
```

---

## Solution

Correct configuration and restart.

```bash
systemctl restart hostapd
```

---

# Issue 4 – Incorrect Wireless Interface

## Symptoms

`hostapd` failed immediately.

---

## Cause

Configuration referenced an incorrect interface.

Example:

```text
interface=wlan1
```

while the actual interface was:

```text
wlan0
```

---

## Debugging

Check interfaces.

```bash
ip addr
```

---

## Solution

Update configuration.

```text
interface=wlan0
```

Restart service.

---

# Issue 5 – Configuration Changes Not Applied

## Symptoms

The modified `hostapd.conf` was not reflected after rebuilding.

---

## Cause

BitBake cache reused previous artifacts.

---

## Solution

Clean the recipe.

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild.

```bash
bitbake core-image-rdkb-ap
```

Verify image before flashing.

---

# Issue 6 – Verify hostapd in Image

Before flashing, verify that `hostapd` exists inside the generated image.

Mount image.

```bash
find /mnt/verify-root -name hostapd
```

Expected:

```text
/usr/sbin/hostapd
```

Verify configuration.

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Verify service.

```bash
ls /mnt/verify-root/lib/systemd/system
```

Expected:

```text
hostapd.service
```

---

# Monitoring Connected Clients

Display associated wireless stations.

```bash
iw dev wlan0 station dump
```

Example:

```text
Station xx:xx:xx:xx:xx:01

Station xx:xx:xx:xx:xx:02

Station xx:xx:xx:xx:xx:03
```

Detailed station information.

```bash
hostapd_cli all_sta
```

This command displays:

- MAC Address
- Signal Strength
- RX/TX Statistics
- Connection Time

---

# Useful hostapd Commands

Check service.

```bash
systemctl status hostapd
```

Restart service.

```bash
systemctl restart hostapd
```

Enable service.

```bash
systemctl enable hostapd
```

View logs.

```bash
journalctl -u hostapd
```

Display connected stations.

```bash
iw dev wlan0 station dump
```

Display all station details.

```bash
hostapd_cli all_sta
```

---

# Actual Debugging Performed

During the project, the following validations were carried out:

- Verified `hostapd` was installed inside the Yocto image.
- Confirmed `/etc/hostapd.conf` was copied correctly.
- Verified `hostapd.service` existed.
- Ensured the service was enabled at boot.
- Checked SSID visibility after every build.
- Tested authentication using multiple devices.
- Verified six simultaneous client connections.
- Used `iw` and `hostapd_cli` to monitor connected stations.

---

# Lessons Learned

The following lessons were learned while debugging `hostapd`:

- Always verify the wireless interface name before configuring `hostapd`.
- Ensure the configuration file is copied into the image during the build process.
- Use `journalctl` to diagnose startup failures.
- Verify image contents before flashing the SD card.
- Test wireless authentication after every configuration change.
- Confirm that `hostapd` starts automatically through `systemd`.

---

# Conclusion

`hostapd` is the foundation of the wireless Access Point implemented in this project. Through systematic debugging of configuration files, service status, wireless interfaces, and runtime logs, all wireless communication issues were successfully resolved. The final configuration enabled the Raspberry Pi to broadcast the `RDKB_AP` SSID, authenticate clients using WPA2 security, and maintain stable wireless connectivity for multiple simultaneous users. Proper understanding of `hostapd` was essential to achieving a reliable and fully functional embedded wireless Access Point.