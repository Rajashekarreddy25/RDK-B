# iptables_Debugging

## Introduction

`iptables` is the Linux firewall utility used to configure packet filtering, Network Address Translation (NAT), and packet forwarding. In this project, `iptables` was responsible for enabling Internet sharing between the Ethernet interface (`eth0`) and the wireless Access Point (`wlan0`).

Initially, the Raspberry Pi successfully created the Access Point, wireless clients could connect, and DHCP assigned valid IP addresses. However, none of the connected devices could access the Internet. After investigation, it was found that the `iptables` package was not included in the custom Yocto image and the NAT rules were missing.

This document explains the complete debugging process, the issues encountered, and the final solution that enabled Internet sharing for all connected devices.

---

# Role of iptables in the Project

```text
                  Internet
                      │
                      │
                 Ethernet (eth0)
                      │
                      ▼
              Raspberry Pi 4
          Custom Yocto Image
                      │
             iptables (NAT)
                      │
                      ▼
                 hostapd AP
                      │
             Wi-Fi Clients
```

`iptables` performs the following tasks:

- Enables Network Address Translation (NAT)
- Forwards packets between interfaces
- Allows return traffic
- Shares Internet connectivity
- Manages packet forwarding rules

---

# Network Flow

```text
Phone

    │

    ▼

wlan0

    │

    ▼

iptables

    │

    ▼

eth0

    │

    ▼

Internet
```

---

# Problem Encountered

After booting the Raspberry Pi:

- SSID was visible.
- Devices connected successfully.
- DHCP assigned valid IP addresses.
- Gateway was reachable.

However:

- Browser displayed **"No Internet"**.
- Applications could not access online services.
- Internet connectivity failed for all connected devices.

---

# Initial Observations

The following components were functioning correctly:

| Component | Status |
|-----------|--------|
| Raspberry Pi Boot | Successful |
| hostapd | Running |
| SSID Broadcast | Working |
| Wi-Fi Connection | Successful |
| dnsmasq | Running |
| DHCP | Working |
| IP Address Assignment | Working |
| Internet Access | Failed |

This indicated that the problem was occurring after DHCP assignment, during packet forwarding.

---

# Debugging Workflow

```text
Client Connects

        │

        ▼

Gets IP Address?

        │

       YES

        │

        ▼

Internet Available?

        │

 ┌──────┴───────┐

 │              │

YES             NO

 │              │

 ▼              ▼

Success     Check NAT

                │

                ▼

       Verify iptables

                │

                ▼

      Check IP Forwarding

                │

                ▼

     Update Network Script

                │

                ▼

        Rebuild Image

                │

                ▼

        Flash SD Card

                │

                ▼

      Internet Working
```

---

# Step 1 – Verify iptables Installation

The first step was checking whether `iptables` existed in the system.

```bash
which iptables
```

Output:

```text
command not found
```

Another check:

```bash
iptables --version
```

Output:

```text
iptables: command not found
```

This confirmed that the package was missing.

---

# Root Cause

The custom image recipe did not include the `iptables` package.

Without `iptables`, Linux could not:

- Perform NAT
- Forward packets
- Share Internet

---

# Solution 1 – Add iptables to Image

Edit:

```text
meta-rdkb-ap/

recipes-core/

images/

core-image-rdkb-ap.bb
```

Add:

```bitbake
IMAGE_INSTALL += "iptables"
```

Rebuild:

```bash
bitbake -c clean rdkb-ap-config

bitbake core-image-rdkb-ap
```

---

# Step 2 – Verify Package Inside Image

Before flashing, the generated image was mounted.

Search:

```bash
find /mnt/verify-root -name iptables
```

Expected:

```text
/usr/sbin/iptables
```

The package was now successfully included in the image.

---

# Step 3 – Verify IP Forwarding

Check:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Initial Output:

```text
0
```

Meaning:

IPv4 forwarding was disabled.

---

# Solution 2 – Enable IP Forwarding

Inside `rdkb-network.sh`:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Verify:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected:

```text
1
```

---

# Step 4 – Configure NAT

The following rules were added to `rdkb-network.sh`.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

These rules perform:

- Source NAT (MASQUERADE)
- Forward outgoing traffic
- Allow return traffic

---

# Final Network Script

```bash
#!/bin/sh

ip link set wlan0 up

ip addr flush dev wlan0

ip addr add 192.168.10.1/24 dev wlan0

echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT

exit 0
```

---

# Step 5 – Rebuild the Image

Since `rdkb-network.sh` was modified, the recipe cache had to be cleared.

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild:

```bash
bitbake core-image-rdkb-ap
```

---

# Step 6 – Verify Updated Script

Before flashing, verify the updated script.

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Confirm that the NAT commands were present.

---

# Step 7 – Flash Updated Image

Flash the latest image onto the SD card.

Boot Raspberry Pi.

---

# Verification

Connect a mobile phone.

Verify:

- Connected successfully
- IP assigned
- Gateway assigned
- DNS assigned

Open browser.

Result:

Internet working successfully.

---

# Commands Used During Debugging

Check iptables:

```bash
which iptables
```

Check version:

```bash
iptables --version
```

Enable forwarding:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Verify package:

```bash
find /mnt/verify-root -name iptables
```

Display network script:

```bash
cat /usr/bin/rdkb-network.sh
```

Rebuild image:

```bash
bitbake core-image-rdkb-ap
```

---

# Actual Debugging Performed

During this project, the following debugging steps were carried out:

- Verified that `iptables` was missing from the initial image.
- Updated `core-image-rdkb-ap.bb` to include the package.
- Rebuilt the Yocto image.
- Mounted the generated image and confirmed `/usr/sbin/iptables` was present.
- Modified `rdkb-network.sh` to enable IP forwarding and configure NAT.
- Cleaned the recipe to invalidate the BitBake cache.
- Rebuilt and reflashed the image.
- Verified Internet connectivity from multiple client devices.

---

# Lessons Learned

Several important lessons were learned during the debugging process:

- Successful Wi-Fi connection does not guarantee Internet access.
- DHCP and NAT are independent components; both must be configured correctly.
- Always verify that required packages are included in the image.
- Image verification before flashing helps detect missing files early.
- Whenever scripts are modified, clean the corresponding recipe to avoid using cached artifacts.
- IP forwarding must be enabled before NAT rules can function correctly.
- `iptables` is a critical component for Internet sharing in embedded Linux systems.

---

# Final Result

After implementing the required changes:

| Test | Result |
|------|--------|
| SSID Broadcast | Passed |
| Client Connection | Passed |
| DHCP Assignment | Passed |
| IP Forwarding | Passed |
| NAT Configuration | Passed |
| Internet Sharing | Passed |
| Multiple Clients | 6 Devices Connected Successfully |

---

# Conclusion

The absence of the `iptables` package was the primary reason Internet sharing failed in the initial implementation, despite successful Wi-Fi connectivity and DHCP operation. By adding `iptables` to the Yocto image, enabling IPv4 forwarding, and configuring NAT rules within the `rdkb-network.sh` script, the Raspberry Pi was successfully transformed into a fully functional wireless Access Point capable of sharing Internet connectivity with multiple wireless clients. This debugging exercise highlighted the importance of understanding the complete Linux networking stack and demonstrated how individual networking components must work together to provide end-to-end connectivity.