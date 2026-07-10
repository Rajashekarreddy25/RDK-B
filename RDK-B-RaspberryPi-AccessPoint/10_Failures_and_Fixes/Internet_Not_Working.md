# Internet_Not_Working

## Introduction

After resolving the DHCP issue, all wireless clients were successfully connecting to the Raspberry Pi Access Point and receiving valid IP addresses. The network configuration appeared to be correct, but clients were still unable to access the Internet.

This became one of the most challenging debugging phases of the project because all networking components seemed to be working individually:

- Wi-Fi Access Point was operational.
- DHCP server was assigning IP addresses.
- Clients received gateway and DNS information.
- Raspberry Pi itself had Internet access through Ethernet.

However, the connected wireless clients could not browse websites, ping external servers, or use any Internet-based applications.

This document explains the investigation process, root cause, debugging steps, and the final solution implemented.

---

# Problem Description

After booting the Raspberry Pi:

- Access Point was visible.
- Clients connected successfully.
- DHCP assigned IP addresses.
- Gateway was correctly configured.
- DNS server was assigned.

Despite this, clients displayed:

```text
Connected

No Internet
```

or

```text
Connected

Internet may not be available
```

---

# Symptoms

```text
✔ Raspberry Pi Booted

✔ Ethernet Connected

✔ Wi-Fi Access Point Running

✔ DHCP Working

✔ Clients Received IP Address

✔ Gateway Assigned

✔ DNS Assigned

✘ No Internet Access
```

---

# Network Status

At this stage, the network looked like:

```text
Internet

      │

      ▼

Ethernet (eth0)

      │

      ▼

Raspberry Pi

      │

      ▼

Wi-Fi (wlan0)

      │

      ▼

Connected Clients

      │

      ▼

No Internet
```

The Raspberry Pi could reach the Internet, but packets from wireless clients were not forwarded to the Ethernet interface.

---

# Initial Investigation

The first step was to verify Internet connectivity on the Raspberry Pi itself.

```bash
ping 8.8.8.8
```

Result:

```text
64 bytes from 8.8.8.8
```

Next, verify DNS.

```bash
ping google.com
```

Result:

```text
PING google.com...
```

The Raspberry Pi had full Internet connectivity.

Therefore, the issue was isolated to packet forwarding between `wlan0` and `eth0`.

---

# Verify Client Configuration

On a connected client:

Network Information:

```text
IP Address

192.168.10.20
```

Gateway

```text
192.168.10.1
```

DNS

```text
8.8.8.8
```

The DHCP configuration was correct.

---

# Verify IP Forwarding

Check the forwarding setting.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Output:

```text
0
```

This indicated that the Linux kernel was **not forwarding packets** between interfaces.

---

# Temporary Test

Enable packet forwarding manually.

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Verify:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Output:

```text
1
```

After enabling forwarding, clients still could not access the Internet.

This confirmed that another component was missing.

---

# Verify Firewall Rules

Display forwarding rules.

```bash
iptables -L
```

Output:

```text
No forwarding rules found.
```

Display NAT table.

```bash
iptables -t nat -L
```

Output:

```text
No MASQUERADE rule.
```

This revealed the primary cause of the issue.

---

# Root Cause Analysis

The Raspberry Pi was forwarding packets internally, but outgoing packets still carried the private client IP addresses.

Example:

```text
Client

192.168.10.15

↓

Internet
```

The Internet router does not recognize private addresses from the wireless subnet.

The packets needed to be translated to the Raspberry Pi's Ethernet IP address before leaving `eth0`.

This translation is performed using **Network Address Translation (NAT)**.

Since no NAT rule existed, packets were discarded by the upstream router.

---

# Solution

The network initialization script:

```text
rdkb-network.sh
```

was updated to perform three additional tasks:

1. Enable IP forwarding.
2. Configure NAT.
3. Allow packet forwarding between interfaces.

---

# Updated Script

The following commands were added:

Enable IP forwarding.

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Enable NAT.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Allow forwarding.

```bash
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Allow return traffic.

```bash
iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

---

# Final Script

The final `rdkb-network.sh` contained:

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

# Rebuild Process

After updating the script:

Clean recipe.

```bash
bitbake -c clean rdkb-ap-config
```

Rebuild image.

```bash
bitbake core-image-rdkb-ap
```

Flash updated image.

Boot Raspberry Pi.

---

# Verification

Check forwarding.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Output:

```text
1
```

Display NAT rule.

```bash
iptables -t nat -L
```

Expected:

```text
MASQUERADE
```

Display forwarding rules.

```bash
iptables -L
```

Expected:

```text
ACCEPT
```

---

# Client Verification

Connect an Android phone.

Network Information:

```text
IP Address

192.168.10.21
```

Gateway

```text
192.168.10.1
```

Internet Status

```text
Connected
```

Open browser.

Websites loaded successfully.

---

# Network Flow After Fix

```text
Internet

      │

      ▼

Ethernet (eth0)

      │

      ▼

Raspberry Pi

      │

      ▼

NAT (iptables)

      │

      ▼

Packet Forwarding

      │

      ▼

wlan0

      │

      ▼

Wireless Client

      │

      ▼

Internet Access
```

---

# Testing Performed

The following tests were completed successfully:

### Test 1

Android Phone

```text
PASS
```

Internet working.

---

### Test 2

Windows Laptop

```text
PASS
```

Internet working.

---

### Test 3

Ubuntu Laptop

```text
PASS
```

Internet working.

---

### Test 4

Multiple Devices

```text
6 Devices Connected Simultaneously

PASS
```

All devices accessed the Internet without interruption.

---

### Test 5

Reboot Raspberry Pi

```text
PASS
```

Internet sharing started automatically after boot.

---

# Commands Used During Debugging

Check Internet.

```bash
ping 8.8.8.8
```

Check DNS.

```bash
ping google.com
```

Verify IP forwarding.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Display NAT rules.

```bash
iptables -t nat -L
```

Display forwarding rules.

```bash
iptables -L
```

Display connected stations.

```bash
iw dev wlan0 station dump
```

---

# Lessons Learned

This issue provided several important networking insights:

- DHCP alone does not provide Internet connectivity.
- Linux must be configured to forward packets between interfaces.
- NAT is mandatory when sharing an Internet connection between different networks.
- IP forwarding and NAT work together to enable Internet sharing.
- Verifying image contents before flashing helps reduce debugging time.
- Testing each networking layer individually simplifies troubleshooting.

---

# Best Practices

- Always enable IP forwarding before configuring NAT.
- Verify `iptables` rules after every image build.
- Keep networking initialization inside a single startup script.
- Test both local connectivity and Internet connectivity separately.
- Validate automatic startup after every reboot.
- Confirm that firewall rules are applied correctly before connecting clients.

---

# Final Result

After enabling IPv4 packet forwarding and configuring NAT using `iptables`, the Raspberry Pi successfully routed traffic between the wireless and Ethernet interfaces.

The final system achieved the following:

- Wireless clients connected successfully.
- DHCP assigned valid IP addresses.
- Gateway and DNS information were distributed correctly.
- Internet traffic was translated using NAT.
- Up to **six devices** connected simultaneously and accessed the Internet without any issues.
- All networking components initialized automatically after every reboot.

This resolved the final networking issue encountered during the implementation of the RDK-B Raspberry Pi Access Point project and completed the functional Access Point setup.