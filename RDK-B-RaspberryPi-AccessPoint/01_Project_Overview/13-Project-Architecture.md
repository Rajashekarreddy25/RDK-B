# 13. Complete Project Architecture

```
                    +---------------------+
                    | Development Machine |
                    | Ubuntu 22.04        |
                    +----------+----------+
                               |
                               |
                        Yocto Build
                               |
                               ▼
                   Custom meta-rdkb-ap Layer
                               |
          ------------------------------------------
          |               |              |          |
          ▼               ▼              ▼          ▼
      hostapd        dnsmasq     rdkb-network    Image
          |               |             |          |
          ------------------------------------------
                               |
                               ▼
                 core-image-rdkb-ap.wic.bz2
                               |
                               ▼
                        Raspberry Pi 4
                               |
                    -------------------
                    |                 |
                  wlan0             eth0
                    |                 |
             WiFi Clients         Internet
                    |
          DHCP + DNS + NAT
```

---

## Boot Flow

```
Power ON

↓

Bootloader

↓

Linux Kernel

↓

systemd

↓

rdkb-network.service

↓

hostapd.service

↓

dnsmasq.service

↓

WiFi Ready
```