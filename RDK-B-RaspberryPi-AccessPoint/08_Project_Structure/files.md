# files

## Introduction

The `files` directory is one of the most important parts of the `meta-rdkb-ap` layer. It contains all the custom configuration files, startup scripts, and service definitions required to transform a standard Yocto Linux image into a fully functional Wireless Access Point.

Unlike BitBake recipes, which describe *how* software is built and installed, the `files` directory stores the actual files that are copied into the target root filesystem during the image build process.

Every file inside this directory serves a specific purpose in configuring networking, enabling services, and automating system initialization.

---

# Purpose of the files Directory

The `files` directory is responsible for storing:

- Access Point configuration
- DHCP configuration
- Network initialization script
- systemd service definition

These files are automatically installed into the correct locations by the `rdkb-ap-config.bb` recipe during the Yocto build process.

---

# Directory Structure

The structure used in this project is shown below.

```text
files/

├── hostapd.conf

├── rdkb-ap.conf

├── rdkb-network.sh

└── rdkb-network.service
```

Each file contributes to a different stage of the system initialization process.

---

# Overall Workflow

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

          Raspberry Pi Boot

                  │

                  ▼

         Automatic AP Startup
```

---

# hostapd.conf

## Purpose

This file configures the Wireless Access Point.

It tells `hostapd` how the Wi-Fi interface should operate.

---

## Installed Location

```text
/etc/hostapd.conf
```

---

## Responsibilities

- Configure SSID
- Configure WPA2 security
- Configure wireless channel
- Configure country code
- Configure wireless interface
- Configure authentication

---

## Example Configuration

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

## Role in Project

```text
hostapd.conf

        │

        ▼

hostapd

        │

        ▼

Broadcast Wi-Fi

        │

        ▼

Client Connection
```

Without this file:

- No SSID
- No Wi-Fi
- No wireless clients

---

# rdkb-ap.conf

## Purpose

This file configures the DHCP server using `dnsmasq`.

It assigns IP addresses to wireless clients automatically.

---

## Installed Location

```text
/etc/dnsmasq.d/rdkb-ap.conf
```

---

## Responsibilities

- DHCP range
- Gateway
- DNS server
- Interface selection

---

## Example Configuration

```text
interface=wlan0

dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,24h

dhcp-option=3,192.168.10.1

dhcp-option=6,8.8.8.8
```

---

## Role in Project

```text
Client

     │

     ▼

Connects

     │

     ▼

dnsmasq

     │

     ▼

Assign IP Address
```

Without this file:

- Clients connect
- No IP address
- No communication

---

# rdkb-network.sh

## Purpose

This shell script performs all networking initialization required after boot.

It is automatically executed by `systemd`.

---

## Installed Location

```text
/usr/bin/rdkb-network.sh
```

---

## Responsibilities

- Bring up `wlan0`
- Configure static IP
- Enable IPv4 forwarding
- Configure NAT
- Configure forwarding rules

---

## Final Script Used

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

## Role in Project

```text
Boot

 │

 ▼

rdkb-network.sh

 │

 ├────────► Configure wlan0

 ├────────► Assign IP

 ├────────► Enable IP Forwarding

 └────────► Configure NAT

          │

          ▼

Internet Sharing
```

Without this script:

- No static IP
- No packet forwarding
- No Internet sharing

---

# rdkb-network.service

## Purpose

This is the custom `systemd` service responsible for executing the network initialization script automatically during boot.

---

## Installed Location

```text
/lib/systemd/system/rdkb-network.service
```

---

## Example Service File

```ini
[Unit]
Description=RDKB Network Initialization
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/rdkb-network.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

---

## Responsibilities

- Start automatically during boot
- Execute `rdkb-network.sh`
- Configure networking
- Ensure AP is ready before users connect

---

## Boot Sequence

```text
Power ON

     │

     ▼

systemd

     │

     ▼

rdkb-network.service

     │

     ▼

rdkb-network.sh

     │

     ▼

Network Ready

     │

     ▼

Clients Connect
```

---

# Installation Process

During the Yocto build, the `rdkb-ap-config.bb` recipe installs each file into the target filesystem.

```text
files/

     │

     ▼

BitBake Recipe

     │

     ├────────► hostapd.conf

     ├────────► rdkb-ap.conf

     ├────────► rdkb-network.sh

     └────────► rdkb-network.service

                │

                ▼

Root Filesystem

                │

                ▼

Generated Image
```

---

# Verification

Before flashing the SD card, all files were verified by mounting the generated image.

Verify hostapd configuration:

```bash
cat /mnt/verify-root/etc/hostapd.conf
```

Verify DHCP configuration:

```bash
cat /mnt/verify-root/etc/dnsmasq.d/rdkb-ap.conf
```

Verify network script:

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Verify service file:

```bash
cat /mnt/verify-root/lib/systemd/system/rdkb-network.service
```

These checks confirmed that all files had been correctly installed into the generated image.

---

# Actual Implementation in the Project

The `files` directory was used to store every custom file required for the Access Point implementation.

The files were automatically installed into the target root filesystem through the `rdkb-ap-config.bb` recipe. After each modification, the recipe was cleaned and rebuilt to ensure that the updated files were included in the generated image.

Before flashing, the image was mounted and inspected to verify that:

- `hostapd.conf` existed in `/etc`.
- `rdkb-ap.conf` existed in `/etc/dnsmasq.d`.
- `rdkb-network.sh` existed in `/usr/bin`.
- `rdkb-network.service` existed in `/lib/systemd/system`.

Only after successful verification was the image flashed to the Raspberry Pi.

---

# Advantages of Using the files Directory

Using a dedicated `files` directory provides several advantages:

- Keeps configuration files organized.
- Simplifies maintenance.
- Enables version control.
- Eliminates manual configuration after boot.
- Supports reproducible builds.
- Makes debugging easier.
- Follows Yocto Project best practices.

---

# Lessons Learned

Working with the `files` directory reinforced several important practices:

- Store all custom configuration files inside the layer.
- Avoid manual edits directly on the target device.
- Package every file through a BitBake recipe.
- Verify installed files before flashing.
- Clean the recipe after modifying any file to avoid stale build artifacts.
- Keep scripts modular and well documented.

---

# Conclusion

The `files` directory serves as the repository for all custom configuration files, startup scripts, and service definitions required by the RDK-B Raspberry Pi Access Point project. Together with the `rdkb-ap-config.bb` recipe, these files automate the complete networking setup during system boot, ensuring that every generated Yocto image is fully configured and ready to operate as a wireless Access Point without requiring any manual configuration after deployment.