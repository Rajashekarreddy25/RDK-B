# RaspberryPi_Commands

## Introduction

The Raspberry Pi served as the target hardware platform for this project. After building the custom Yocto image, several Raspberry Pi-specific commands were used for booting, system verification, networking, storage management, and remote access.

This document summarizes all Raspberry Pi related commands that were used during the implementation, testing, and debugging of the RDK-B Raspberry Pi Access Point project.

---

# Raspberry Pi Workflow

```text
Build Image

      │

      ▼

Flash SD Card

      │

      ▼

Insert into Raspberry Pi

      │

      ▼

Power ON

      │

      ▼

Boot Linux

      │

      ▼

Verify Services

      │

      ▼

Connect Clients

      │

      ▼

Test Internet
```

---

# 1. Login to Raspberry Pi

Local Login

```text
login: root
```

Purpose:

- Access Raspberry Pi terminal.
- Perform configuration and testing.

---

# 2. Display System Information

```bash
uname -a
```

Example Output

```text
Linux raspberrypi 5.x.x
```

Purpose

- Verify Linux kernel.
- Confirm successful boot.

---

# 3. Display Hostname

```bash
hostname
```

Example Output

```text
raspberrypi
```

Purpose

- Verify device identity.

---

# 4. Display Current User

```bash
whoami
```

Expected Output

```text
root
```

Purpose

- Confirm active user.

---

# 5. Display Raspberry Pi CPU Information

```bash
cat /proc/cpuinfo
```

Purpose

- Verify processor information.
- Confirm hardware detection.

---

# 6. Display Memory Information

```bash
free -h
```

Purpose

- Check available RAM.

Example Output

```text
Mem:
```

---

# 7. Display Storage Usage

```bash
df -h
```

Purpose

- Verify filesystem usage.
- Check available storage.

---

# 8. List Block Devices

```bash
lsblk
```

Example Output

```text
mmcblk0

mmcblk0p1

mmcblk0p2
```

Purpose

- Verify SD card partitions.

---

# 9. Display Mounted Filesystems

```bash
mount
```

Purpose

- Verify mounted partitions.

---

# 10. Reboot Raspberry Pi

```bash
reboot
```

Purpose

- Restart system.
- Verify automatic startup.

---

# 11. Shutdown Raspberry Pi

```bash
poweroff
```

or

```bash
shutdown now
```

Purpose

- Safely power down device.

---

# 12. Verify Network Interfaces

```bash
ip addr
```

Purpose

- Verify Ethernet.
- Verify Wi-Fi.
- Check IP addresses.

---

# 13. Verify Wireless Interface

```bash
iw dev
```

Purpose

- Confirm wireless device detection.

---

# 14. Display Connected Wi-Fi Clients

```bash
iw dev wlan0 station dump
```

Purpose

- View connected stations.
- Monitor client activity.

Example Output

```text
Station AA:BB:CC:DD:EE:FF
signal: -45 dBm
```

---

# 15. Display Routing Table

```bash
ip route
```

Purpose

- Verify default gateway.

---

# 16. Verify Internet Connectivity

```bash
ping 8.8.8.8
```

Purpose

- Test Internet access.

---

# 17. Verify DNS Resolution

```bash
ping google.com
```

Purpose

- Verify DNS functionality.

---

# 18. Check hostapd Service

```bash
systemctl status hostapd
```

Purpose

- Verify Access Point operation.

---

# 19. Check dnsmasq Service

```bash
systemctl status dnsmasq
```

Purpose

- Verify DHCP server.

---

# 20. Check Custom Service

```bash
systemctl status rdkb-network.service
```

Purpose

- Verify network initialization.

---

# 21. Restart Networking Service

```bash
systemctl restart rdkb-network.service
```

Purpose

- Apply networking changes.

---

# 22. View System Logs

```bash
journalctl
```

Purpose

- Display system logs.

---

# 23. View hostapd Logs

```bash
journalctl -u hostapd
```

Purpose

- Debug Access Point.

---

# 24. View dnsmasq Logs

```bash
journalctl -u dnsmasq
```

Purpose

- Debug DHCP.

---

# 25. View Custom Service Logs

```bash
journalctl -u rdkb-network.service
```

Purpose

- Verify startup script execution.

---

# 26. Display Firewall Rules

```bash
iptables -L
```

Purpose

- Verify forwarding rules.

---

# 27. Display NAT Rules

```bash
iptables -t nat -L
```

Purpose

- Verify MASQUERADE rule.

---

# 28. Enable Packet Forwarding (Temporary)

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Purpose

- Enable routing.

---

# 29. Verify Packet Forwarding

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected Output

```text
1
```

---

# 30. Display Listening Ports

```bash
ss -tulpn
```

Purpose

- Verify running services.

---

# 31. Verify SSH Service

```bash
systemctl status sshd
```

Purpose

- Confirm remote login availability.

---

# 32. Connect to Raspberry Pi via SSH

From Host Machine

```bash
ssh root@192.168.10.1
```

Purpose

- Remote administration.
- File editing.
- System monitoring.

---

# 33. Copy Files Using SCP

```bash
scp file.txt root@192.168.10.1:/tmp
```

Purpose

- Transfer files.

---

# Commands Frequently Used During This Project

The following Raspberry Pi commands were used repeatedly during development.

Check interfaces

```bash
ip addr
```

Check connected clients

```bash
iw dev wlan0 station dump
```

Verify AP

```bash
systemctl status hostapd
```

Verify DHCP

```bash
systemctl status dnsmasq
```

Verify custom network service

```bash
systemctl status rdkb-network.service
```

View logs

```bash
journalctl -u hostapd
```

```bash
journalctl -u dnsmasq
```

```bash
journalctl -u rdkb-network.service
```

Display NAT rules

```bash
iptables -t nat -L
```

Test Internet

```bash
ping 8.8.8.8
```

Test DNS

```bash
ping google.com
```

Remote login

```bash
ssh root@192.168.10.1
```

---

# Typical Verification Sequence

The following sequence was followed after every successful image boot.

```text
Boot Raspberry Pi

        │

        ▼

Verify Interfaces

        │

        ▼

Verify hostapd

        │

        ▼

Verify dnsmasq

        │

        ▼

Verify rdkb-network.service

        │

        ▼

Connect Wi-Fi Client

        │

        ▼

Verify DHCP

        │

        ▼

Verify Internet

        │

        ▼

Verify Multiple Clients
```

---

# Best Practices

- Verify all services immediately after boot.
- Check system logs before modifying configuration files.
- Confirm NAT rules before troubleshooting Internet access.
- Use SSH for remote administration whenever possible.
- Test connectivity using both IP addresses and domain names.
- Monitor connected clients during stress testing.
- Reboot after significant configuration changes to ensure services start automatically.

---

# Lessons Learned

Working directly with the Raspberry Pi during this project provided several valuable insights:

- The Raspberry Pi is a reliable platform for embedded Linux networking projects.
- Verifying services after every boot helps identify issues early.
- SSH simplifies remote management and debugging.
- Regularly checking logs reduces troubleshooting time.
- Structured verification after flashing ensures consistent and reproducible results.

---

# Conclusion

The Raspberry Pi commands documented here were used throughout the implementation, testing, and validation of the RDK-B Raspberry Pi Access Point project. They enabled system monitoring, service management, networking verification, storage inspection, and remote administration. Mastering these commands ensured efficient debugging, reliable deployment, and successful operation of the custom Yocto-based Access Point.