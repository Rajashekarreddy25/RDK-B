# Performance

## Introduction

Performance evaluation is an important phase in validating an embedded Linux system. After implementing the custom RDK-B Access Point image, the Raspberry Pi was tested to evaluate its stability, responsiveness, and ability to provide wireless connectivity under normal operating conditions.

The objective of these tests was not to perform benchmark measurements but to verify that the system could reliably operate as an Access Point while serving multiple wireless clients without service interruption.

---

# Objective

The objectives of the performance testing are:

- Measure system boot behavior.
- Verify CPU utilization during normal operation.
- Verify memory utilization.
- Evaluate wireless stability.
- Verify Access Point responsiveness.
- Verify DHCP performance.
- Verify Internet sharing performance.
- Test multiple simultaneous wireless clients.
- Observe overall system stability.

---

# Test Environment

| Component | Description |
|-----------|-------------|
| Board | Raspberry Pi 4 Model B |
| Processor | Broadcom BCM2711 Quad-Core Cortex-A72 |
| RAM | 4 GB |
| Operating System | Custom Yocto + RDK-B Image |
| Access Point | hostapd |
| DHCP Server | dnsmasq |
| NAT | iptables |
| Clients Connected | 6 Devices |

---

# Test Parameters

The following parameters were observed during testing:

- System boot
- CPU utilization
- Memory utilization
- Running services
- Wireless stability
- DHCP response
- Internet connectivity
- Multiple client support

---

# Boot Performance

The Raspberry Pi booted successfully into the custom Yocto image.

During boot:

- Linux Kernel initialized successfully.
- Root filesystem mounted.
- Systemd started.
- Network services initialized.
- hostapd started.
- dnsmasq started.
- rdkb-network service executed successfully.

The system reached a fully operational state without requiring manual intervention.

---

# CPU Utilization

CPU usage was monitored using:

```bash
top
```

or

```bash
htop
```

Observed behavior:

- Low CPU utilization during idle operation.
- Small increase when clients connected.
- Temporary spikes during Internet browsing.
- CPU returned to idle after network activity.

No abnormal CPU usage was observed.

---

# Memory Utilization

Memory usage was monitored using:

```bash
free -h
```

Observed behavior:

- Sufficient free memory available.
- hostapd consumed minimal memory.
- dnsmasq consumed very little memory.
- No memory exhaustion occurred.
- No swapping was observed.

The Raspberry Pi had adequate memory resources for the implemented functionality.

---

# Wireless Stability

Wireless connectivity was evaluated by maintaining client connections over an extended period.

Observed results:

- No unexpected disconnections.
- Stable Wi-Fi signal.
- Successful client reconnections.
- Continuous communication.
- Consistent Access Point availability.

The wireless interface remained stable throughout testing.

---

# DHCP Performance

The DHCP server was evaluated by connecting multiple wireless clients.

Observed behavior:

- IP addresses assigned immediately.
- No duplicate IP addresses.
- Successful lease generation.
- Fast DHCP negotiation.
- Reliable address allocation.

Each client received:

- IP Address
- Gateway
- DNS Server

automatically.

---

# NAT Performance

Internet sharing was tested by browsing websites from connected devices.

Observed behavior:

- Stable Internet connectivity.
- Successful packet forwarding.
- DNS resolution successful.
- No packet forwarding failures.
- Consistent Internet access.

All connected devices were able to access the Internet simultaneously.

---

# Multiple Client Performance

A total of six wireless devices were connected to the Access Point simultaneously.

Connected devices:

- Android Phone
- Laptop
- Tablet
- Smartphone
- Desktop (Wi-Fi Adapter)
- Additional Mobile Phone

Observed behavior:

- All devices connected successfully.
- DHCP assigned unique IP addresses.
- Internet available on every device.
- No client disconnects.
- Stable wireless communication.

---

# Service Stability

The following services were monitored during testing:

```bash
systemctl status hostapd
```

```bash
systemctl status dnsmasq
```

```bash
systemctl status rdkb-network
```

Observed status:

```text
Active: active (running)
```

No unexpected service failures occurred.

---

# Resource Monitoring Commands

CPU Usage:

```bash
top
```

or

```bash
htop
```

Memory Usage:

```bash
free -h
```

Disk Usage:

```bash
df -h
```

Running Services:

```bash
systemctl --type=service
```

Kernel Information:

```bash
uname -a
```

System Uptime:

```bash
uptime
```

---

# Performance Summary

| Parameter | Observation |
|-----------|-------------|
| System Boot | Successful |
| CPU Usage | Low under normal load |
| Memory Usage | Stable |
| Wi-Fi Stability | Stable |
| DHCP Response | Immediate |
| NAT Performance | Stable |
| Internet Connectivity | Successful |
| Multiple Clients | 6 Connected Successfully |
| Service Stability | No Failures |

---

# Observations

During performance evaluation:

- The Raspberry Pi remained responsive throughout testing.
- Access Point services operated continuously without interruption.
- DHCP and NAT worked efficiently.
- Multiple wireless clients were handled without noticeable degradation.
- Internet connectivity remained available for all connected devices.
- CPU and memory usage remained within acceptable limits for the implemented workload.

The custom Yocto image demonstrated reliable performance for an embedded wireless Access Point application.

---

# Limitations

The following tests were outside the scope of this project:

- Wireless throughput benchmarking.
- Long-duration stress testing (24–48 hours).
- Maximum client capacity testing.
- Packet loss analysis under heavy traffic.
- Performance comparison with commercial wireless routers.

These evaluations may be considered for future enhancements.

---

# Conclusion

The performance evaluation confirmed that the Raspberry Pi 4 running the custom Yocto-built RDK-B image provides a stable and reliable wireless Access Point solution. Throughout testing, the system successfully hosted six simultaneous wireless clients, assigned IP addresses using `dnsmasq`, forwarded Internet traffic through `iptables`, and maintained uninterrupted operation. CPU and memory usage remained within acceptable limits, and no unexpected service failures or client disconnections were observed. Overall, the implemented solution met the functional and performance objectives defined for the project.