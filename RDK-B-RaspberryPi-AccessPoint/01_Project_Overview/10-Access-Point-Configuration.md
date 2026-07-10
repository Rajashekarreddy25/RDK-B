# 10. Access Point Configuration

After booting Raspberry Pi with the generated image, the device must behave as a wireless Access Point.

---

## Objective

Configure Raspberry Pi to

- Broadcast SSID
- Authenticate clients
- Assign IP addresses
- Forward packets
- Enable Internet Sharing

---

## Components Used

| Component | Purpose |
|-----------|----------|
| hostapd | Wireless Access Point |
| dnsmasq | DHCP & DNS Server |
| systemd | Service Manager |
| iptables | NAT |

---

## hostapd Configuration

Location

```

/etc/hostapd.conf

```

Example

```
interface=wlan0
driver=nl80211

ssid=RDKB_AP

hw_mode=g
channel=6
country_code=IN

ieee80211n=1

auth_algs=1

ignore_broadcast_ssid=0

wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

---

## dnsmasq Configuration

Location

```

/etc/dnsmasq.d/rdkb-ap.conf

```

Contents

```
interface=wlan0
bind-interfaces

dhcp-range=192.168.10.10,192.168.10.100,255.255.255.0,12h

dhcp-option=3,192.168.10.1
dhcp-option=6,8.8.8.8
```

---

## Network Script

```

/usr/bin/rdkb-network.sh

```

Responsibilities

- Bring wlan0 UP
- Assign Static IP
- Enable IP Forwarding
- Configure NAT
- Configure Forwarding Rules

Example

```sh
ip link set wlan0 up

ip addr flush dev wlan0

ip addr add 192.168.10.1/24 dev wlan0

echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

iptables -A FORWARD -i eth0 -o wlan0 \
-m state --state RELATED,ESTABLISHED -j ACCEPT
```

---

## Systemd Service

```
rdkb-network.service
```

```
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

---

## Boot Sequence

```
System Boots
      │
      ▼

systemd
      │
      ▼

rdkb-network.service
      │
      ▼

Configure wlan0
      │
      ▼

Enable NAT
      │
      ▼

Start hostapd
      │
      ▼

Broadcast SSID
      │
      ▼

Start dnsmasq
      │
      ▼

Assign Client IP
```