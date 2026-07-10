# iptables

## Introduction

`iptables` is the Linux firewall utility responsible for filtering, forwarding, and modifying network packets. It provides the rules required for routing traffic between different network interfaces and is commonly used to implement Network Address Translation (NAT).

In this project, `iptables` was used to allow devices connected to the Raspberry Pi Access Point to access the Internet through the Ethernet interface (`eth0`).

Without `iptables`, wireless clients could successfully connect to the Access Point and obtain an IP address from the DHCP server, but they would not be able to access the Internet.

---

# Why iptables was Required

The Raspberry Pi has two network interfaces.

- **eth0** → Connected to the Internet
- **wlan0** → Works as the Wi-Fi Access Point

When a mobile phone connects to the Access Point, all packets first arrive at `wlan0`.

Those packets must then be forwarded to `eth0`.

Similarly, response packets arriving on `eth0` must be forwarded back to `wlan0`.

Linux does not perform this forwarding automatically.

To enable packet forwarding and Internet sharing, `iptables` rules were added.

---

# Network Topology

```text
                Internet
                    │
                    │
              Ethernet Cable
                    │
                    ▼
              Raspberry Pi
          +-------------------+
          |                   |
          |   eth0            |
          |                   |
          |-------------------|
          |                   |
          |   Linux Kernel    |
          |                   |
          |-------------------|
          |                   |
          |   wlan0 (AP)      |
          +-------------------+
                    │
          Wi-Fi Clients
      ┌────────┬────────┬────────┐
      │        │        │        │
      ▼        ▼        ▼        ▼
   Phone 1  Phone 2  Laptop  Tablet
```

---

# What is NAT?

NAT stands for

**Network Address Translation**

It converts private IP addresses into public IP addresses.

Example:

```text
Phone IP

192.168.10.20

↓

Raspberry Pi

↓

Translated to

192.168.1.50 (eth0)

↓

Internet
```

The Internet never sees the private IP address.

Instead, it sees the Raspberry Pi Ethernet address.

---

# Why NAT is Needed

The Access Point network uses private IP addresses.

```text
192.168.10.0/24
```

Private IP addresses cannot be routed over the public Internet.

Therefore, Linux must translate these private addresses into the Raspberry Pi's Ethernet IP.

This translation is performed using NAT.

---

# Packet Flow

The complete packet flow is shown below.

```text
Phone

192.168.10.20

      │

      ▼

wlan0

      │

      ▼

Linux Kernel

      │

      ▼

iptables NAT

      │

      ▼

eth0

192.168.1.50

      │

      ▼

Router

      │

      ▼

Internet
```

---

# IP Forwarding

Before Linux forwards packets between interfaces, IP forwarding must be enabled.

The following command enables packet forwarding.

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Equivalent command:

```bash
sysctl -w net.ipv4.ip_forward=1
```

Without enabling IP forwarding:

```text
Phone

↓

Raspberry Pi

↓

Packet Dropped
```

The packet never reaches the Ethernet interface.

---

# NAT Rule

The following rule was added.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

# Understanding the Rule

## -t nat

Selects the NAT table.

---

## POSTROUTING

Packets are modified just before leaving the Raspberry Pi.

---

## -o eth0

Applies the rule only to packets leaving through Ethernet.

---

## MASQUERADE

Replaces the source IP with the Ethernet IP.

Example:

Before:

```text
Source

192.168.10.25
```

After:

```text
Source

192.168.1.50
```

---

# Forward Rule

The next rule allows traffic from Wi-Fi to Ethernet.

```bash
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Meaning:

```text
Incoming Interface

wlan0

↓

Outgoing Interface

eth0

↓

Allow Packet
```

---

# Return Traffic Rule

The Internet sends response packets back.

Those packets must be allowed back into the Wi-Fi network.

```bash
iptables -A FORWARD \
-i eth0 \
-o wlan0 \
-m state \
--state RELATED,ESTABLISHED \
-j ACCEPT
```

---

# Understanding the Rule

```text
eth0

↓

Already Established Connection

↓

Allow Back

↓

wlan0
```

Without this rule:

```text
Request

Allowed

Response

Blocked
```

The client never receives data.

---

# Final Network Script

The complete `rdkb-network.sh` script used in the project is shown below.

```bash
#!/bin/sh

# Bring up Wi-Fi interface
ip link set wlan0 up

# Configure AP IP Address
ip addr flush dev wlan0
ip addr add 192.168.10.1/24 dev wlan0

# Enable IPv4 Forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Configure NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Allow Forwarding
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Allow Return Traffic
iptables -A FORWARD \
-i eth0 \
-o wlan0 \
-m state \
--state RELATED,ESTABLISHED \
-j ACCEPT

exit 0
```

---

# Installing iptables

Initially, `iptables` was not included in the image.

It was added to the image recipe.

```bitbake
IMAGE_INSTALL += "iptables"
```

After rebuilding the image, the utility became available.

Verification:

```bash
find /mnt/verify-root -name iptables
```

Output:

```text
/usr/sbin/iptables
```

---

# Verifying NAT Rules

List all NAT rules.

```bash
iptables -t nat -L
```

Expected output:

```text
Chain POSTROUTING

MASQUERADE
```

---

# Verify Forward Rules

```bash
iptables -L FORWARD
```

Expected:

```text
ACCEPT wlan0 → eth0

ACCEPT eth0 → wlan0
```

---

# Verifying IP Forwarding

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected output:

```text
1
```

A value of `1` confirms that IPv4 forwarding is enabled.

---

# Major Issue Encountered

One of the most significant issues during this project occurred after the Access Point was successfully created.

Observed behavior:

- SSID was visible.
- Mobile devices connected successfully.
- DHCP assigned valid IP addresses.
- Devices displayed **"Connected"**.

However:

- No websites could be opened.
- Internet access was unavailable.
- Applications reported no network connectivity.

Initially, HostAPD and DNSMasq were suspected to be misconfigured. After verifying their configurations, it became clear that both services were functioning correctly.

Further investigation revealed that the Raspberry Pi was assigning IP addresses but was not forwarding packets to the Ethernet interface. The missing components were:

- IPv4 forwarding
- NAT configuration
- Forwarding rules

These were implemented using `iptables`, after which Internet connectivity worked correctly for all connected devices.

---

# Final Data Flow

```text
Phone

↓

Connects to SSID

↓

Receives DHCP Address

↓

Sends Packet

↓

wlan0

↓

iptables

↓

eth0

↓

Internet

↓

Response

↓

eth0

↓

iptables

↓

wlan0

↓

Phone
```

---

# Advantages of Using iptables

Using `iptables` provided several benefits:

- Internet sharing
- NAT support
- Secure packet forwarding
- Stateful firewall functionality
- Controlled routing between interfaces
- Seamless integration with Linux networking

---

# Summary

`iptables` was one of the most critical components of this project. While HostAPD created the wireless network and DNSMasq assigned IP addresses, neither service alone could provide Internet access. By enabling IPv4 forwarding and configuring NAT and forwarding rules with `iptables`, the Raspberry Pi successfully routed traffic between the wireless clients and the Ethernet network. This completed the transformation of the Raspberry Pi into a fully functional wireless router capable of serving multiple devices simultaneously.