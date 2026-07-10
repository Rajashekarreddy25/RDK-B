# Systemd_Debugging

## Introduction

`systemd` is the init system used in the custom Yocto image to manage system startup, background services, and dependencies. In this project, `systemd` was responsible for starting the following services automatically during boot:

- hostapd
- dnsmasq
- rdkb-network
- Other system services

Since the Access Point functionality depends entirely on these services starting in the correct order, several debugging activities were performed to verify service status, startup sequence, logs, and configuration.

This document describes the complete debugging process for `systemd` services encountered during the implementation of the RDK-B Access Point project.

---

# Role of systemd in the Project

The boot sequence of the Raspberry Pi is shown below.

```text
Power ON

    │

    ▼

Bootloader

    │

    ▼

Linux Kernel

    │

    ▼

Root Filesystem

    │

    ▼

systemd (PID 1)

    │

    ├──────────────► hostapd.service

    ├──────────────► dnsmasq.service

    ├──────────────► rdkb-network.service

    └──────────────► Other Linux Services
```

Without `systemd`, the Access Point services would not start automatically after boot.

---

# Services Used

| Service | Purpose |
|----------|----------|
| hostapd.service | Starts Wi-Fi Access Point |
| dnsmasq.service | DHCP and DNS server |
| rdkb-network.service | Configures IP forwarding and NAT |
| systemd-networkd | Network interface management |
| systemd-resolved | DNS resolution |

---

# Debugging Workflow

```text
Boot Raspberry Pi

        │

        ▼

Check Service Status

        │

        ▼

Is Service Running?

        │

   ┌────┴─────┐

   │          │

 YES         NO

   │          │

   ▼          ▼

Verify      Check Logs

Function      │

              ▼

       Identify Failure

              │

              ▼

       Correct Configuration

              │

              ▼

      Restart Service

              │

              ▼

       Verify Again
```

---

# Step 1 – Verify Service Status

The first step in debugging was checking whether the required services were running.

Example:

```bash
systemctl status hostapd
```

```bash
systemctl status dnsmasq
```

```bash
systemctl status rdkb-network
```

Expected output:

```text
Active: active (running)
```

---

# Step 2 – Verify Enabled Services

To confirm that services start automatically after boot:

```bash
systemctl is-enabled hostapd
```

```bash
systemctl is-enabled dnsmasq
```

```bash
systemctl is-enabled rdkb-network
```

Expected:

```text
enabled
```

---

# Step 3 – View Service Logs

If a service failed to start, logs were examined using `journalctl`.

Example:

```bash
journalctl -u hostapd
```

```bash
journalctl -u dnsmasq
```

```bash
journalctl -u rdkb-network
```

The logs provided detailed information about:

- Startup failures
- Configuration errors
- Missing files
- Invalid arguments
- Dependency failures

---

# Step 4 – Restart Services

After correcting configuration issues, services were restarted without rebooting the system.

```bash
systemctl restart hostapd
```

```bash
systemctl restart dnsmasq
```

```bash
systemctl restart rdkb-network
```

Service status was verified again.

---

# Step 5 – Verify Service Files

Each service file was checked to ensure it existed in the correct directory.

```bash
ls /lib/systemd/system
```

Expected:

```text
hostapd.service

dnsmasq.service

rdkb-network.service
```

---

# Step 6 – Verify Service Links

Systemd enables services by creating symbolic links.

Verify:

```bash
ls /etc/systemd/system/multi-user.target.wants
```

Expected:

```text
hostapd.service

dnsmasq.service

rdkb-network.service
```

This confirms that the services are enabled during boot.

---

# Issue 1 – Service Not Starting

## Symptoms

```text
Active: failed
```

---

## Debugging

View logs.

```bash
journalctl -u <service_name>
```

Example:

```bash
journalctl -u hostapd
```

---

## Cause

Possible causes included:

- Missing configuration file
- Invalid configuration
- Missing executable
- Incorrect permissions

---

## Solution

Correct the configuration and restart the service.

```bash
systemctl restart hostapd
```

---

# Issue 2 – Service Not Enabled

## Symptoms

Service worked after manual start but did not start automatically after reboot.

---

## Debugging

Check:

```bash
systemctl is-enabled hostapd
```

Output:

```text
disabled
```

---

## Solution

Enable service.

```bash
systemctl enable hostapd
```

Repeat for:

```bash
systemctl enable dnsmasq

systemctl enable rdkb-network
```

---

# Issue 3 – Missing Service File

## Symptoms

```bash
systemctl status rdkb-network
```

Output:

```text
Unit not found
```

---

## Cause

The recipe did not install the service file into:

```text
/lib/systemd/system
```

---

## Solution

Modify `rdkb-ap-config.bb`.

Example:

```bitbake
install -Dm644 rdkb-network.service \
${D}${systemd_system_unitdir}/rdkb-network.service
```

Rebuild:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

---

# Issue 4 – Modified Service Not Updated

## Symptoms

Changes made to the service file were not reflected after rebuilding.

---

## Cause

BitBake reused cached build artifacts.

---

## Solution

Clean the recipe.

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild the image.

```bash
bitbake core-image-rdkb-ap
```

Verify the updated service file inside the generated image before flashing.

---

# Issue 5 – Service Executed but Script Failed

## Symptoms

```text
rdkb-network.service

Active: failed
```

---

## Cause

The script referenced by the service contained errors or lacked executable permissions.

---

## Debugging

Check service definition.

```bash
cat /lib/systemd/system/rdkb-network.service
```

Verify script.

```bash
cat /usr/bin/rdkb-network.sh
```

Check permissions.

```bash
ls -l /usr/bin/rdkb-network.sh
```

---

## Solution

Ensure the script:

- Exists.
- Is executable.
- Contains valid commands.

Example:

```bash
chmod +x /usr/bin/rdkb-network.sh
```

---

# Useful systemd Commands

Check service:

```bash
systemctl status hostapd
```

Restart service:

```bash
systemctl restart hostapd
```

Stop service:

```bash
systemctl stop hostapd
```

Start service:

```bash
systemctl start hostapd
```

Enable service:

```bash
systemctl enable hostapd
```

Disable service:

```bash
systemctl disable hostapd
```

View logs:

```bash
journalctl -u hostapd
```

Reload systemd:

```bash
systemctl daemon-reload
```

List failed services:

```bash
systemctl --failed
```

---

# Actual Debugging Performed in the Project

During the implementation of this project, the following validations were performed:

- Verified that `hostapd.service` was installed.
- Verified that `dnsmasq.service` was installed.
- Verified that `rdkb-network.service` was installed.
- Checked symbolic links under `multi-user.target.wants`.
- Mounted the generated Yocto image and confirmed the presence of service files.
- Rebuilt the image after modifying `rdkb-network.sh`.
- Used `systemctl status` to verify successful service startup after boot.

These steps ensured that all required services started automatically without manual intervention.

---

# Lessons Learned

During debugging of `systemd`, the following lessons were learned:

- Always verify that service files are installed in the correct location.
- Enable services so they start automatically during boot.
- Use `journalctl` as the primary tool for diagnosing startup failures.
- Verify service dependencies before debugging application logic.
- Clean and rebuild recipes whenever service definitions are modified.
- Confirm generated image contents before flashing the SD card.

---

# Conclusion

`systemd` played a central role in the successful operation of the custom RDK-B Access Point. By managing the startup of `hostapd`, `dnsmasq`, and the custom `rdkb-network` service, it ensured that all networking components were initialized automatically during boot. Through systematic verification of service status, logs, configuration files, and startup behavior, all service-related issues were successfully resolved. Proper understanding of `systemd` significantly simplified debugging and improved the reliability of the final embedded Linux system.