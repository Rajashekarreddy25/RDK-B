# Network_Commands

## Introduction

Networking commands were extensively used throughout the RDK-B Raspberry Pi Access Point project for configuring network interfaces, verifying connectivity, monitoring connected clients, troubleshooting Internet access, and debugging networking services.

These commands played a crucial role during both development and testing phases. They helped validate every stage of the networking stack, from wireless interface initialization to DHCP assignment and NAT-based Internet sharing.

This document serves as a comprehensive reference for all networking-related commands used during the project.

---

# Networking Workflow

```text
Ethernet Connected

        │

        ▼

Configure wlan0

        │

        ▼

Start hostapd

        │

        ▼

Start dnsmasq

        │

        ▼

Assign Client IP

        │

        ▼

Enable IP Forwarding

        │

        ▼

Configure NAT

        │

        ▼

Internet Sharing
```

---

# 1. ip

The `ip` command is the modern Linux networking utility used to configure and monitor network interfaces.

Display all interfaces:

```bash
ip addr
```

Example Output:

```text
1: lo

2: eth0

3: wlan0
```

Purpose:

- Display IP addresses
- Verify interface status
- Debug networking

---

# 2. Display Interface Information

Show detailed information about a specific interface.

```bash
ip addr show wlan0
```

Purpose:

- Verify static IP
- Confirm interface configuration

Expected Output:

```text
inet 192.168.10.1/24
```

---

# 3. Bring Interface Up

Enable an interface.

```bash
ip link set wlan0 up
```

Purpose:

- Activate Wi-Fi interface

Used inside:

```text
rdkb-network.sh
```

---

# 4. Bring Interface Down

Disable interface.

```bash
ip link set wlan0 down
```

Purpose:

- Reset interface
- Troubleshoot Wi-Fi

---

# 5. Assign Static IP

Assign IP address.

```bash
ip addr add 192.168.10.1/24 dev wlan0
```

Purpose:

- Configure AP gateway

---

# 6. Remove Existing IP

Clear interface configuration.

```bash
ip addr flush dev wlan0
```

Purpose:

- Prevent duplicate addresses

---

# 7. Display Routing Table

Show routing information.

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev eth0
```

Purpose:

- Verify default gateway
- Debug routing

---

# 8. ping

Test network connectivity.

Ping local interface:

```bash
ping 192.168.10.1
```

Ping Internet:

```bash
ping 8.8.8.8
```

Ping domain:

```bash
ping google.com
```

Purpose:

- Verify connectivity
- Test DNS
- Confirm Internet access

---

# 9. iw

Wireless management utility.

Display wireless devices.

```bash
iw dev
```

Purpose:

- Verify Wi-Fi interface

---

# 10. Display Connected Clients

Show connected Wi-Fi stations.

```bash
iw dev wlan0 station dump
```

Example Output:

```text
Station AA:BB:CC:DD:EE:FF

signal: -42 dBm

tx bitrate: 72.2 MBit/s

rx bitrate: 65.0 MBit/s
```

Purpose:

- Display connected clients
- Monitor signal strength
- View transmission rates

This command was frequently used during project testing.

---

# 11. Display Wireless Information

```bash
iw wlan0 info
```

Purpose:

- Verify Wi-Fi mode
- Confirm channel
- Check interface type

---

# 12. hostapd

Start Access Point manually.

```bash
hostapd /etc/hostapd.conf
```

Purpose:

- Debug AP configuration

Normally started by:

```text
systemd
```

---

# 13. Verify hostapd Status

```bash
systemctl status hostapd
```

Purpose:

- Confirm AP service is running

---

# 14. dnsmasq

Start DHCP server manually.

```bash
dnsmasq
```

Purpose:

- Debug DHCP configuration

Normally started by:

```text
systemd
```

---

# 15. Verify dnsmasq Status

```bash
systemctl status dnsmasq
```

Purpose:

- Confirm DHCP server operation

---

# 16. journalctl

Display networking logs.

hostapd logs:

```bash
journalctl -u hostapd
```

dnsmasq logs:

```bash
journalctl -u dnsmasq
```

Custom service logs:

```bash
journalctl -u rdkb-network.service
```

Purpose:

- Debug startup failures
- View runtime messages

---

# 17. iptables

Display firewall rules.

```bash
iptables -L
```

Purpose:

- Verify forwarding rules

---

# 18. Display NAT Rules

```bash
iptables -t nat -L
```

Purpose:

- Verify MASQUERADE rule

Expected:

```text
MASQUERADE
```

---

# 19. Add NAT Rule

Project command:

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Purpose:

- Share Ethernet Internet
- Enable NAT

---

# 20. Enable Packet Forwarding

Temporary:

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

Purpose:

- Allow routing

---

# 21. ss

Display network sockets.

```bash
ss -tulpn
```

Purpose:

- Verify listening services

Example:

```text
dnsmasq

hostapd

sshd
```

---

# 22. netstat

Display network connections.

```bash
netstat -tulpn
```

Purpose:

- Alternative to `ss`

---

# 23. arp

Display ARP table.

```bash
arp -a
```

Purpose:

- Show connected devices
- Verify client communication

---

# 24. hostnamectl

Display system hostname.

```bash
hostnamectl
```

Purpose:

- Verify system information

---

# 25. systemctl

Display service status.

```bash
systemctl status rdkb-network.service
```

Start service:

```bash
systemctl start rdkb-network.service
```

Restart service:

```bash
systemctl restart rdkb-network.service
```

Enable service:

```bash
systemctl enable rdkb-network.service
```

Purpose:

- Manage networking services

---

# Commands Used During Project Testing

During implementation and testing, the following networking commands were used frequently.

Display interfaces:

```bash
ip addr
```

Display routes:

```bash
ip route
```

Display connected stations:

```bash
iw dev wlan0 station dump
```

Verify AP:

```bash
systemctl status hostapd
```

Verify DHCP:

```bash
systemctl status dnsmasq
```

Verify custom service:

```bash
systemctl status rdkb-network.service
```

Check logs:

```bash
journalctl -u hostapd
```

```bash
journalctl -u dnsmasq
```

```bash
journalctl -u rdkb-network.service
```

Display firewall rules:

```bash
iptables -L
```

Display NAT rules:

```bash
iptables -t nat -L
```

Test Internet:

```bash
ping 8.8.8.8
```

Display wireless stations:

```bash
iw dev wlan0 station dump
```

Display listening services:

```bash
ss -tulpn
```

---

# Practical Debugging Sequence Used

During troubleshooting, the following sequence was followed.

```text
Check Interface

        │

        ▼

ip addr

        │

        ▼

Check AP

        │

        ▼

systemctl status hostapd

        │

        ▼

Check DHCP

        │

        ▼

systemctl status dnsmasq

        │

        ▼

Check Clients

        │

        ▼

iw dev wlan0 station dump

        │

        ▼

Check NAT

        │

        ▼

iptables -t nat -L

        │

        ▼

Test Internet

        │

        ▼

ping 8.8.8.8
```

---

# Best Practices

- Verify interface status before troubleshooting services.
- Confirm DHCP is working before debugging Internet access.
- Always check `iptables` rules when NAT is not functioning.
- Use `journalctl` instead of guessing service failures.
- Monitor connected stations using `iw`.
- Validate routing tables after enabling packet forwarding.
- Test both IP connectivity (`8.8.8.8`) and DNS resolution (`google.com`).

---

# Lessons Learned

Using Linux networking tools throughout the project provided several valuable insights:

- `ip` has replaced legacy tools such as `ifconfig` and provides more comprehensive functionality.
- `iw` is the preferred utility for monitoring wireless interfaces and connected stations.
- DHCP assignment does not guarantee Internet access; NAT and IP forwarding must also be configured.
- `iptables` is critical for Internet sharing between interfaces.
- System logs available through `journalctl` significantly simplify networking troubleshooting.
- A structured debugging approach—checking interfaces, services, clients, NAT, and finally Internet connectivity—reduces troubleshooting time and improves reliability.

---

# Conclusion

Networking commands were fundamental to the successful implementation and validation of the RDK-B Raspberry Pi Access Point project. They enabled configuration, monitoring, testing, and troubleshooting of every networking component, including wireless interfaces, DHCP, routing, NAT, and Internet connectivity. Mastering these commands greatly enhanced the ability to diagnose issues efficiently and ensured a stable, fully functional Access Point deployment.