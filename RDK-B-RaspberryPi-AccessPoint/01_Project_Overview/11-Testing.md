# 11. Testing

Multiple tests were performed after booting Raspberry Pi.

---

## Test 1

Check systemd

```
systemctl status
```

Expected

```
running
```

---

## Test 2

Check hostapd

```
systemctl status hostapd
```

Expected

```
active (running)
```

---

## Test 3

Check dnsmasq

```
systemctl status dnsmasq
```

Expected

```
active (running)
```

---

## Test 4

Check Interface

```
ip addr
```

Expected

```
wlan0

192.168.10.1
```

---

## Test 5

Check DHCP

Connect phone

Verify

```
Phone receives

192.168.10.x
```

---

## Test 6

Ping Gateway

```
ping 192.168.10.1
```

---

## Test 7

Ping Internet

```
ping 8.8.8.8
```

---

## Test 8

Connected Clients

```
iw dev wlan0 station dump
```

Shows

- Connected MAC
- Signal
- TX Rate
- RX Rate

---

## Test 9

Connected Devices

```
cat /var/lib/misc/dnsmasq.leases
```

Displays

- MAC Address
- IP Address
- Lease Time

---

## Test 10

Verify NAT

```
iptables -t nat -L -n
```

Expected

```
MASQUERADE
```

---

## Test 11

Forwarding

```
cat /proc/sys/net/ipv4/ip_forward
```

Expected

```
1
```