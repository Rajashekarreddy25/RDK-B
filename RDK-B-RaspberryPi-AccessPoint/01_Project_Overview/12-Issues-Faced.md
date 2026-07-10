# 12. Issues Faced During Development

The project required several debugging iterations before reaching the final working solution.

---

# Issue 1

## Problem

systemctl command missing

### Cause

Image built using SysV init.

### Solution

Enabled systemd.

```
INIT_MANAGER = "systemd"

DISTRO_FEATURES:append = " systemd"

DISTRO_FEATURES:remove = "sysvinit"

VIRTUAL-RUNTIME_init_manager="systemd"
```

---

# Issue 2

## Problem

Generated image did not contain latest changes.

### Cause

Old WIC image still existed.

### Solution

Delete previous images.

```
rm *.wic

rm *.wic.bz2
```

Rebuild image.

---

# Issue 3

## Problem

losetup returned

```
unexpected arguments
```

### Cause

Incorrect command syntax.

### Solution

```
sudo losetup -Pf --show image.wic
```

---

# Issue 4

## Problem

hostapd configuration conflict

```
file /etc/hostapd.conf conflicts
```

### Cause

Both hostapd package and custom recipe installed same file.

### Solution

Removed duplicate installation.

---

# Issue 5

## Problem

No systemctl found in image.

### Cause

Old image verification.

### Solution

Mounted latest WIC image.

Verified

```
find /mnt/root -name systemctl
```

---

# Issue 6

## Problem

Phone connected

No Internet

### Cause

No NAT.

### Solution

Installed

```
iptables
```

Added

```
MASQUERADE

FORWARD

RELATED ESTABLISHED
```

rules.

---

# Issue 7

## Problem

Phone received WiFi but no IP.

### Cause

dnsmasq configuration incorrect.

### Solution

Updated

```
/etc/dnsmasq.d/rdkb-ap.conf
```

---

# Issue 8

## Problem

hostapd service failed.

### Cause

wlan0 not configured before service startup.

### Solution

Created

```
rdkb-network.service
```

Executed before

```
hostapd.service
```

---

# Issue 9

## Problem

Cannot verify image contents.

### Solution

Mounted image using

```
losetup

mount

find
```

Verified every binary before flashing.

---

# Final Outcome

Successfully implemented

- systemd
- hostapd
- dnsmasq
- DHCP
- NAT
- Internet Sharing
- Multi-device connectivity