# SSH_Test

## Introduction

Secure Shell (SSH) is a network protocol that enables secure remote access to a Linux system over a network. In this project, SSH was used to remotely access the Raspberry Pi after it booted successfully. This eliminated the need for a monitor, keyboard, or mouse, allowing the Raspberry Pi to be managed entirely from another computer on the same network.

Testing SSH functionality was important because it verified:

- The Raspberry Pi was connected to the network.
- The SSH server was running.
- Remote login was successful.
- Commands could be executed remotely.
- File transfers and administration could be performed without physical access.

---

# Objective

The objectives of this test are:

- Verify SSH server availability.
- Verify remote login.
- Verify command execution.
- Verify network connectivity.
- Ensure stable remote administration.

---

# Test Environment

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| Operating System | Custom Yocto + RDK-B Image |
| SSH Server | OpenSSH |
| Client System | Ubuntu 22.04 |
| Network | Ethernet/Wi-Fi |

---

# SSH Architecture

```text
            Ubuntu PC

                │

                │ SSH

                ▼

        Raspberry Pi 4

        Custom RDK-B Image

                │

        Linux Terminal
```

---

# Prerequisites

Before testing SSH, ensure:

- Raspberry Pi has booted successfully.
- Raspberry Pi is connected to the network.
- SSH server is installed.
- SSH service is running.
- Ubuntu PC and Raspberry Pi are on the same network.

---

# Step 1 – Determine Raspberry Pi IP Address

Login to the Raspberry Pi locally or through the serial console.

Check the Ethernet interface.

```bash
ip addr show eth0
```

Example output:

```text
inet 192.168.1.105/24
```

The Raspberry Pi IP address is:

```text
192.168.1.105
```

---

# Step 2 – Verify Network Connectivity

From the Ubuntu host:

```bash
ping 192.168.1.105
```

Expected output:

```text
64 bytes from 192.168.1.105

icmp_seq=1 ttl=64
```

This confirms that the Raspberry Pi is reachable.

---

# Step 3 – Verify SSH Server

On Raspberry Pi:

```bash
systemctl status ssh
```

or

```bash
systemctl status sshd
```

Expected output:

```text
Active: active (running)
```

---

# Step 4 – Connect Using SSH

From the Ubuntu host:

```bash
ssh root@192.168.1.105
```

If another user exists:

```bash
ssh <username>@192.168.1.105
```

Example:

```bash
ssh root@192.168.1.105
```

---

# Step 5 – Accept Host Key

During the first connection:

```text
The authenticity of host cannot be established.

Are you sure you want to continue?

yes/no
```

Type:

```text
yes
```

The SSH key is stored for future connections.

---

# Step 6 – Login

Enter the password when prompted.

Example:

```text
Password:
```

After successful authentication:

```text
root@raspberrypi:~#
```

The remote terminal is now available.

---

# Step 7 – Execute Commands

Run a few Linux commands.

Check hostname:

```bash
hostname
```

Check kernel version:

```bash
uname -a
```

Check uptime:

```bash
uptime
```

Check memory:

```bash
free -h
```

Check disk usage:

```bash
df -h
```

All commands should execute successfully.

---

# Step 8 – Verify Network Services

Verify Access Point service.

```bash
systemctl status hostapd
```

Verify DHCP server.

```bash
systemctl status dnsmasq
```

Verify custom service.

```bash
systemctl status rdkb-network
```

Expected:

```text
Active: active (running)
```

---

# Step 9 – Verify Internet

Check Internet connectivity from Raspberry Pi.

```bash
ping -c 4 google.com
```

Expected:

```text
4 packets transmitted

4 received
```

---

# Step 10 – Logout

Terminate the SSH session.

```bash
exit
```

or

```bash
logout
```

---

# Commands Used

Determine IP:

```bash
ip addr show eth0
```

Ping Raspberry Pi:

```bash
ping 192.168.1.105
```

SSH Login:

```bash
ssh root@192.168.1.105
```

Check kernel:

```bash
uname -a
```

Check memory:

```bash
free -h
```

Check services:

```bash
systemctl status hostapd
```

Logout:

```bash
exit
```

---

# Expected Results

| Test | Expected Result |
|-------|-----------------|
| Raspberry Pi Reachable | Pass |
| SSH Server Running | Pass |
| Login Successful | Pass |
| Commands Execute | Pass |
| Network Stable | Pass |

---

# Actual Results

| Test | Result |
|-------|---------|
| Ping Successful | Pass |
| SSH Connected | Pass |
| Remote Login Successful | Pass |
| Commands Executed | Pass |
| Remote Administration Successful | Pass |

---

# Problems Encountered

## SSH Connection Refused

### Cause

The SSH server was not running.

### Solution

Check the service.

```bash
systemctl status ssh
```

Start it if necessary.

```bash
systemctl start ssh
```

Enable it on boot.

```bash
systemctl enable ssh
```

---

## Host Unreachable

### Cause

Incorrect IP address or network connectivity issue.

### Solution

Verify the Raspberry Pi IP address.

```bash
ip addr show eth0
```

Test connectivity.

```bash
ping <raspberry_pi_ip>
```

---

## Authentication Failure

### Cause

Incorrect username or password.

### Solution

Verify the correct login credentials and retry.

---

## Firewall Blocking SSH

### Cause

Port 22 blocked.

### Solution

Ensure SSH traffic is allowed through the firewall.

---

# Observations

During testing:

- SSH login was successful without noticeable delay.
- Command execution was responsive.
- No session disconnections occurred.
- All system services could be monitored remotely.
- Remote administration significantly simplified development and debugging.

---

# Advantages of SSH

Using SSH provided several benefits:

- Headless system management.
- Secure encrypted communication.
- Remote troubleshooting.
- Service monitoring.
- File management.
- Easy execution of Linux commands.

For embedded Linux development, SSH is one of the most important tools for remote system administration.

---

# Conclusion

The SSH functionality was successfully validated. The Raspberry Pi was accessible over the network, allowing secure remote login from the Ubuntu host system. System services, network configuration, and application status could all be monitored and managed remotely. This capability greatly improved the development workflow by eliminating the need for direct physical access to the Raspberry Pi during testing and debugging.