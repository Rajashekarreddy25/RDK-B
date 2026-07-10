# Network_Script

## Introduction

The `rdkb-network.sh` script is the core network configuration script used in this project. It is responsible for preparing the Raspberry Pi to operate as a wireless Access Point.

When the Raspberry Pi boots, the script is executed automatically by the custom `rdkb-network.service` systemd service. It performs all the necessary network initialization before the `hostapd` and `dnsmasq` services start.

Without this script, the Raspberry Pi would not assign a static IP address to the wireless interface, would not enable packet forwarding, and connected clients would not be able to access the Internet.

---

# Purpose of the Script

The script performs the following tasks:

- Bring up the wireless interface (`wlan0`)
- Remove any existing IP configuration
- Assign a static IP address
- Enable IPv4 forwarding
- Configure NAT
- Configure packet forwarding rules
- Prepare the network for HostAPD and DNSMasq

---

# Complete Script

The final version of the script used in this project is shown below.

```bash
#!/bin/sh

# Bring up Wi-Fi interface
ip link set wlan0 up

# Remove existing IP configuration
ip addr flush dev wlan0

# Assign static IP address
ip addr add 192.168.10.1/24 dev wlan0

# Enable IPv4 forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Configure NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Allow forwarding from Wi-Fi to Ethernet
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Allow return traffic
iptables -A FORWARD \
-i eth0 \
-o wlan0 \
-m state --state RELATED,ESTABLISHED \
-j ACCEPT

exit 0
```

---

# Script Execution Flow

```text
System Boot

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

Configure wlan0

      │

      ▼

Enable IP Forwarding

      │

      ▼

Configure NAT

      │

      ▼

HostAPD Starts

      │

      ▼

DNSMasq Starts

      │

      ▼

Access Point Ready
```

---

# Line-by-Line Explanation

## 1. Shebang

```bash
#!/bin/sh
```

This tells Linux to execute the script using the Bourne shell.

---

## 2. Bring Up the Wireless Interface

```bash
ip link set wlan0 up
```

Initially, the wireless interface is in the DOWN state.

This command activates it.

Verification:

```bash
ip link show wlan0
```

Expected output:

```text
state UP
```

---

## 3. Remove Existing IP Configuration

```bash
ip addr flush dev wlan0
```

If an IP address already exists on `wlan0`, it is removed before assigning the new static IP.

This prevents:

- Duplicate addresses
- Old DHCP configurations
- Multiple IP assignments

---

## 4. Assign Static IP Address

```bash
ip addr add 192.168.10.1/24 dev wlan0
```

Assigns the gateway IP address for the Access Point.

Configuration:

| Parameter | Value |
|----------|--------|
| Interface | wlan0 |
| IP Address | 192.168.10.1 |
| Subnet | /24 |
| Network | 192.168.10.0 |

Every wireless client uses this address as its default gateway.

Verification:

```bash
ip addr show wlan0
```

Expected:

```text
inet 192.168.10.1/24
```

---

## 5. Enable IPv4 Forwarding

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Linux disables packet forwarding by default.

This command enables routing between:

```text
wlan0

↓

eth0
```

Verification:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected:

```text
1
```

---

## 6. Configure NAT

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

This rule translates the private IP addresses of wireless clients into the Raspberry Pi's Ethernet IP address.

Example:

Before:

```text
Phone

192.168.10.20
```

After NAT:

```text
192.168.1.50
```

This allows Internet communication.

---

## 7. Allow Packet Forwarding

```bash
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Allows traffic to flow from:

```text
wlan0

↓

eth0
```

Without this rule:

```text
Packets

↓

Dropped
```

---

## 8. Allow Return Traffic

```bash
iptables -A FORWARD \
-i eth0 \
-o wlan0 \
-m state --state RELATED,ESTABLISHED \
-j ACCEPT
```

Allows reply packets coming from the Internet to return to the wireless client.

Example:

```text
Phone

↓

Request

↓

Internet

↓

Response

↓

Phone
```

Without this rule, only outgoing packets would work while incoming responses would be blocked.

---

## 9. Exit

```bash
exit 0
```

Returns a success status to systemd.

A return code of `0` indicates successful execution.

---

# Integration with systemd

The script is executed by the following service.

```text
/lib/systemd/system/rdkb-network.service
```

Service configuration:

```ini
[Unit]
Description=RDK-B Network Configuration
Before=hostapd.service dnsmasq.service

[Service]
Type=oneshot
ExecStart=/usr/bin/rdkb-network.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

This ensures that the script executes before HostAPD and DNSMasq start.

---

# Installation Through BitBake

The script is packaged by the custom recipe:

```text
rdkb-ap-config.bb
```

Installation command:

```bitbake
install -m 0755 ${WORKDIR}/rdkb-network.sh \
${D}${bindir}/rdkb-network.sh
```

After the image is built, the script is available at:

```text
/usr/bin/rdkb-network.sh
```

Verification:

```bash
find /mnt/verify-root -name rdkb-network.sh
```

Expected output:

```text
/usr/bin/rdkb-network.sh
```

---

# Evolution of the Script

The script evolved during the project as different issues were discovered.

### Initial Version

The first implementation only configured the wireless interface.

```bash
#!/bin/sh

ip link set wlan0 up

ip addr flush dev wlan0

ip addr add 192.168.10.1/24 dev wlan0

exit 0
```

Result:

- Access Point created
- DHCP working
- Clients connected
- No Internet access

---

### Intermediate Version

IPv4 forwarding was added.

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Result:

- Routing enabled
- Internet still unavailable

---

### Final Version

NAT and forwarding rules were added.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD \
-i eth0 \
-o wlan0 \
-m state --state RELATED,ESTABLISHED \
-j ACCEPT
```

Result:

- Internet access successful
- Multiple wireless clients connected simultaneously
- Stable operation after every reboot

---

# Verification Steps

Check interface:

```bash
ip addr show wlan0
```

---

Check forwarding:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

---

Check NAT:

```bash
iptables -t nat -L
```

---

Check forwarding rules:

```bash
iptables -L FORWARD
```

---

Check service status:

```bash
systemctl status rdkb-network.service
```

---

# Challenges Faced

During implementation, several issues were encountered.

### Interface Not Configured

Initially, `wlan0` remained down because the script did not explicitly enable it.

Solution:

```bash
ip link set wlan0 up
```

---

### Connected but No Internet

Wireless clients successfully connected to the SSID and received IP addresses, but Internet access failed.

Root cause:

- IPv4 forwarding disabled
- NAT rules missing
- Forwarding rules absent

Solution:

Added IP forwarding and `iptables` configuration.

---

### Service Execution Order

Initially, HostAPD started before the network interface was configured.

This caused HostAPD to fail.

Solution:

Configured the systemd service with:

```ini
Before=hostapd.service dnsmasq.service
```

ensuring the script executed first.

---

# Final Workflow

```text
Power ON

      │

      ▼

Linux Boot

      │

      ▼

systemd

      │

      ▼

rdkb-network.sh

      │

      ▼

Configure wlan0

      │

      ▼

Enable IP Forwarding

      │

      ▼

Configure NAT

      │

      ▼

Start HostAPD

      │

      ▼

Broadcast SSID

      │

      ▼

Start DNSMasq

      │

      ▼

Assign Client IP Address

      │

      ▼

Internet Access Available
```

---

# Summary

The `rdkb-network.sh` script is the central component responsible for network initialization in this project. It configures the wireless interface, assigns the Access Point's static IP address, enables IPv4 forwarding, and establishes NAT and forwarding rules using `iptables`. Throughout the project's development, the script evolved from a simple interface configuration utility into a complete network initialization solution, enabling automatic Internet sharing for multiple wireless clients after every system boot.