# Jarkom-Modul-4-2026-K-40

Laporan Praktikum Modul 4 Komunikasi Data dan Jaringan Komputer 2026, *The Wired Expansion*.

Bagian laporan ini membahas **Subnetting dengan metode VLSM pada GNS3**.

| Nama | NRP |
|---|---|
| Chilmi Muhammad Ulin Nuha | 003 |
| Wahyu Yoga Wicaksono | 103 |

## Daftar Isi

- [Pendahuluan](#pendahuluan)
- [Topologi](#topologi)
- [Routing Table](#routing-table)
- [Subnetting GNS3: Metode VLSM](#subnetting-gns3-metode-vlsm)
  - [Perhitungan VLSM](#perhitungan-vlsm)
  - [Pembagian IP](#pembagian-ip)
  - [Konfigurasi Network](#konfigurasi-network)
    - [Router](#router)
    - [Host / Server](#host--server)
  - [Konfigurasi Routing](#konfigurasi-routing)
  - [Testing](#testing)

## Pendahuluan

Topologi terdiri dari 13 router dan 25 node host/server yang dihubungkan oleh switch. Seluruh node harus saling terhubung (ping) dengan pembagian IP yang efisien. Pada GNS3 digunakan metode **VLSM** (*Variable Length Subnet Mask*) dengan prefix `192.231.0.0`, sedangkan metode CIDR dikerjakan pada Cisco Packet Tracer.

Seluruh node pada GNS3 berupa Docker container. Konfigurasi IP dimasukkan melalui **Configure > Network configuration** pada tiap node, lalu skrip konfigurasi disimpan di `/root/config.sh`.

## Topologi

#### GNS3 (VLSM)

![Topologi GNS3 VLSM](images/topologi-gns3-vlsm.png)

Tiap subnet ditandai dengan lingkaran hijau dan label `A1` sampai `A25`. Penomoran mengikuti urutan penelusuran topologi dari Cyberia.

## Routing Table

| Nama Subnet | Rute | Jumlah IP | Netmask |
|---|---|---|---|
| A1 | Cyberia > Switch1 > JJ, MyuMyu | 126 | /25 |
| A2 | Cyberia > Wired | 2 | /30 |
| A3 | Wired > Switch2 > Psyche, Copland, Schumann | 4 | /29 |
| A4 | Wired > Navi | 2 | /30 |
| A5 | Wired > Tachibana | 2 | /30 |
| A6 | Psyche > Switch3 > Kari, Lin | 881 | /22 |
| A7 | Copland > Switch4 > Masayuki | 301 | /23 |
| A8 | Copland > Juri | 32 | /26 |
| A9 | Schumann > Akashic | 4 | /29 |
| A10 | Schumann > Accela | 2 | /30 |
| A11 | Accela > Switch5 > Omnipresence, Denpa | 514 | /22 |
| A12 | Navi > Switch6 > Alice, Reika | 254 | /24 |
| A13 | Navi > Switch7 > Taro | 58 | /26 |
| A14 | Navi > Switch8 > Yasuo, Miho, Kuma | 143 | /24 |
| A15 | Tachibana > Switch9 > Collective, Ghost | 267 | /23 |
| A16 | Tachibana > Deus | 2 | /30 |
| A17 | Deus > Switch10 > Lilith, Specter | 99 | /25 |
| A18 | Deus > Protocol7 | 2 | /30 |
| A19 | Protocol7 > Switch11 > Chisa, Hodgeson | 33 | /26 |
| A20 | Protocol7 > Knights | 2 | /30 |
| A21 | Knights > Eiri | 3 | /29 |
| A22 | Knights > Phantoma | 2 | /30 |
| A23 | Phantoma > Switch12 > Semi | 15 | /27 |
| A24 | Phantoma > Shibuya | 2 | /30 |
| A25 | Shibuya > Switch13 > Lain, Mika | 62 | /26 |
| **Total** | | **2814** | **/19** |

Jumlah IP = jumlah host + jumlah interface router pada subnet tersebut.

## Subnetting GNS3: Metode VLSM

### Perhitungan VLSM

1. Hitung kebutuhan IP tiap subnet (host + interface router).
2. Pilih blok terkecil yang cukup: `2^n - 2 >= kebutuhan`.
3. Urutkan dari blok terbesar (`/22`) ke terkecil (`/30`).
4. Alokasikan berurutan mulai dari `192.231.0.0`. Karena diurutkan dari yang terbesar, setiap network ID otomatis sejajar (aligned) dan tidak ada alamat yang terbuang di antara subnet.
5. Gabungkan seluruh blok menjadi satu agregat: `192.231.0.0/19`.

**Info**

- Total kebutuhan IP: **2814**
- Total alamat yang dialokasikan (jumlah ukuran blok): **4188**
- Kebutuhan terhadap alamat yang dialokasikan: 2814 dari 4188 (67.2%)
- Sisa di dalam blok (tidak terpakai tiap subnet): 1374 (32.8%)
- Agregat `192.231.0.0/19` berisi 8192 alamat, 4188 terpakai (51.1%). Rentang `192.231.16.92` sampai `192.231.31.255` masih kosong.

### Pembagian IP

Diurutkan dari netmask terbesar sesuai aturan VLSM.

| Subnet | Network ID | Netmask | Broadcast | Range IP |
|---|---|---|---|---|
| A6 | 192.231.0.0 | 255.255.252.0 | 192.231.3.255 | 192.231.0.1 - 192.231.3.254 |
| A11 | 192.231.4.0 | 255.255.252.0 | 192.231.7.255 | 192.231.4.1 - 192.231.7.254 |
| A7 | 192.231.8.0 | 255.255.254.0 | 192.231.9.255 | 192.231.8.1 - 192.231.9.254 |
| A15 | 192.231.10.0 | 255.255.254.0 | 192.231.11.255 | 192.231.10.1 - 192.231.11.254 |
| A12 | 192.231.12.0 | 255.255.255.0 | 192.231.12.255 | 192.231.12.1 - 192.231.12.254 |
| A14 | 192.231.13.0 | 255.255.255.0 | 192.231.13.255 | 192.231.13.1 - 192.231.13.254 |
| A1 | 192.231.14.0 | 255.255.255.128 | 192.231.14.127 | 192.231.14.1 - 192.231.14.126 |
| A17 | 192.231.14.128 | 255.255.255.128 | 192.231.14.255 | 192.231.14.129 - 192.231.14.254 |
| A25 | 192.231.15.0 | 255.255.255.192 | 192.231.15.63 | 192.231.15.1 - 192.231.15.62 |
| A13 | 192.231.15.64 | 255.255.255.192 | 192.231.15.127 | 192.231.15.65 - 192.231.15.126 |
| A19 | 192.231.15.128 | 255.255.255.192 | 192.231.15.191 | 192.231.15.129 - 192.231.15.190 |
| A8 | 192.231.15.192 | 255.255.255.192 | 192.231.15.255 | 192.231.15.193 - 192.231.15.254 |
| A23 | 192.231.16.0 | 255.255.255.224 | 192.231.16.31 | 192.231.16.1 - 192.231.16.30 |
| A3 | 192.231.16.32 | 255.255.255.248 | 192.231.16.39 | 192.231.16.33 - 192.231.16.38 |
| A9 | 192.231.16.40 | 255.255.255.248 | 192.231.16.47 | 192.231.16.41 - 192.231.16.46 |
| A21 | 192.231.16.48 | 255.255.255.248 | 192.231.16.55 | 192.231.16.49 - 192.231.16.54 |
| A2 | 192.231.16.56 | 255.255.255.252 | 192.231.16.59 | 192.231.16.57 - 192.231.16.58 |
| A4 | 192.231.16.60 | 255.255.255.252 | 192.231.16.63 | 192.231.16.61 - 192.231.16.62 |
| A5 | 192.231.16.64 | 255.255.255.252 | 192.231.16.67 | 192.231.16.65 - 192.231.16.66 |
| A10 | 192.231.16.68 | 255.255.255.252 | 192.231.16.71 | 192.231.16.69 - 192.231.16.70 |
| A16 | 192.231.16.72 | 255.255.255.252 | 192.231.16.75 | 192.231.16.73 - 192.231.16.74 |
| A18 | 192.231.16.76 | 255.255.255.252 | 192.231.16.79 | 192.231.16.77 - 192.231.16.78 |
| A20 | 192.231.16.80 | 255.255.255.252 | 192.231.16.83 | 192.231.16.81 - 192.231.16.82 |
| A22 | 192.231.16.84 | 255.255.255.252 | 192.231.16.87 | 192.231.16.85 - 192.231.16.86 |
| A24 | 192.231.16.88 | 255.255.255.252 | 192.231.16.91 | 192.231.16.89 - 192.231.16.90 |

### Konfigurasi Network

Cara memasukkan: klik kanan node > **Configure** > **Network configuration**, tempel isinya, **Save**, lalu **Stop** dan **Start** node. Skrip `config.sh` dibuat lewat **Console** node (klik kanan > Console), lalu dijalankan dengan `bash /root/config.sh`. Skrip memakai `ip ... replace` sehingga aman dijalankan berulang.

#### Router

**Cyberia**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.57/30 dev eth1
ip addr replace 192.231.14.1/25 dev eth2
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -C POSTROUTING -o eth0 -j MASQUERADE 2>/dev/null || iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
ip route replace 192.231.0.0/19 via 192.231.16.58 dev eth1
EOF
```

**Wired**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.58/30 dev eth0
ip addr replace 192.231.16.65/30 dev eth1
ip addr replace 192.231.16.61/30 dev eth2
ip addr replace 192.231.16.33/29 dev eth3
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.57
ip route replace 192.231.10.0/23 via 192.231.16.66
ip route replace 192.231.14.128/25 via 192.231.16.66
ip route replace 192.231.15.0/26 via 192.231.16.66
ip route replace 192.231.15.128/26 via 192.231.16.66
ip route replace 192.231.16.0/27 via 192.231.16.66
ip route replace 192.231.16.48/29 via 192.231.16.66
ip route replace 192.231.16.72/29 via 192.231.16.66
ip route replace 192.231.16.80/28 via 192.231.16.66
ip route replace 192.231.12.0/23 via 192.231.16.62
ip route replace 192.231.15.64/26 via 192.231.16.62
ip route replace 192.231.0.0/22 via 192.231.16.34
ip route replace 192.231.8.0/23 via 192.231.16.35
ip route replace 192.231.15.192/26 via 192.231.16.35
ip route replace 192.231.16.40/29 via 192.231.16.36
ip route replace 192.231.16.68/30 via 192.231.16.36
ip route replace 192.231.4.0/22 via 192.231.16.36
EOF
```

**Psyche**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.34/29 dev eth0
ip addr replace 192.231.0.1/22 dev eth1
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.33
EOF
```

**Copland**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.35/29 dev eth0
ip addr replace 192.231.8.1/23 dev eth1
ip addr replace 192.231.15.193/26 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.33
EOF
```

**Schumann**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.36/29 dev eth0
ip addr replace 192.231.16.69/30 dev eth1
ip addr replace 192.231.16.41/29 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.33
ip route replace 192.231.4.0/22 via 192.231.16.70
EOF
```

**Accela**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.70/30 dev eth0
ip addr replace 192.231.4.1/22 dev eth1
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.69
EOF
```

**Navi**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.62/30 dev eth0
ip addr replace 192.231.12.1/24 dev eth1
ip addr replace 192.231.15.65/26 dev eth2
ip addr replace 192.231.13.1/24 dev eth3
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.61
EOF
```

**Tachibana**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.66/30 dev eth0
ip addr replace 192.231.16.73/30 dev eth1
ip addr replace 192.231.10.1/23 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.65
ip route replace 192.231.14.128/25 via 192.231.16.74
ip route replace 192.231.15.0/26 via 192.231.16.74
ip route replace 192.231.15.128/26 via 192.231.16.74
ip route replace 192.231.16.0/27 via 192.231.16.74
ip route replace 192.231.16.48/29 via 192.231.16.74
ip route replace 192.231.16.76/30 via 192.231.16.74
ip route replace 192.231.16.80/28 via 192.231.16.74
EOF
```

**Deus**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.74/30 dev eth0
ip addr replace 192.231.16.77/30 dev eth1
ip addr replace 192.231.14.129/25 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.73
ip route replace 192.231.15.0/26 via 192.231.16.78
ip route replace 192.231.15.128/26 via 192.231.16.78
ip route replace 192.231.16.0/27 via 192.231.16.78
ip route replace 192.231.16.48/29 via 192.231.16.78
ip route replace 192.231.16.80/28 via 192.231.16.78
EOF
```

**protocol7**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.78/30 dev eth0
ip addr replace 192.231.16.81/30 dev eth1
ip addr replace 192.231.15.129/26 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.77
ip route replace 192.231.15.0/26 via 192.231.16.82
ip route replace 192.231.16.0/27 via 192.231.16.82
ip route replace 192.231.16.48/29 via 192.231.16.82
ip route replace 192.231.16.84/30 via 192.231.16.82
ip route replace 192.231.16.88/30 via 192.231.16.82
EOF
```

**knights**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.82/30 dev eth0
ip addr replace 192.231.16.85/30 dev eth1
ip addr replace 192.231.16.49/29 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.81
ip route replace 192.231.15.0/26 via 192.231.16.86
ip route replace 192.231.16.0/27 via 192.231.16.86
ip route replace 192.231.16.88/30 via 192.231.16.86
EOF
```

**Phantoma**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.86/30 dev eth0
ip addr replace 192.231.16.89/30 dev eth1
ip addr replace 192.231.16.1/27 dev eth2
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.85
ip route replace 192.231.15.0/26 via 192.231.16.90
EOF
```

**Shibuya**

```bash
cat > /root/config.sh <<'EOF'
#!/bin/bash
ip addr replace 192.231.16.90/30 dev eth0
ip addr replace 192.231.15.1/26 dev eth1
sysctl -w net.ipv4.ip_forward=1
ip route replace default via 192.231.16.89
EOF
```

Pada Cyberia, `eth0` mengambil IP lewat DHCP dari NAT (192.168.122.x) sehingga tidak diatur manual. Baris `iptables ... MASQUERADE` memberi akses keluar untuk seluruh subnet, dan rute `192.231.0.0/19 via 192.231.16.58` mengarahkan semua jaringan internal ke Wired.

#### Host / Server

**Kari**

```
auto eth0
iface eth0 inet static
    address 192.231.0.2
    netmask 255.255.252.0
    gateway 192.231.0.1
```

Skrip: `ip addr replace 192.231.0.2/22 dev eth0` dan `ip route replace default via 192.231.0.1`

**Lin**

```
auto eth0
iface eth0 inet static
    address 192.231.0.3
    netmask 255.255.252.0
    gateway 192.231.0.1
```

Skrip: `ip addr replace 192.231.0.3/22 dev eth0` dan `ip route replace default via 192.231.0.1`

**Masayuki**

```
auto eth0
iface eth0 inet static
    address 192.231.8.2
    netmask 255.255.254.0
    gateway 192.231.8.1
```

Skrip: `ip addr replace 192.231.8.2/23 dev eth0` dan `ip route replace default via 192.231.8.1`

**Juri**

```
auto eth0
iface eth0 inet static
    address 192.231.15.194
    netmask 255.255.255.192
    gateway 192.231.15.193
```

Skrip: `ip addr replace 192.231.15.194/26 dev eth0` dan `ip route replace default via 192.231.15.193`

**Akashic**

```
auto eth0
iface eth0 inet static
    address 192.231.16.42
    netmask 255.255.255.248
    gateway 192.231.16.41
```

Skrip: `ip addr replace 192.231.16.42/29 dev eth0` dan `ip route replace default via 192.231.16.41`

**Omnipresence**

```
auto eth0
iface eth0 inet static
    address 192.231.4.2
    netmask 255.255.252.0
    gateway 192.231.4.1
```

Skrip: `ip addr replace 192.231.4.2/22 dev eth0` dan `ip route replace default via 192.231.4.1`

**Denpa**

```
auto eth0
iface eth0 inet static
    address 192.231.4.3
    netmask 255.255.252.0
    gateway 192.231.4.1
```

Skrip: `ip addr replace 192.231.4.3/22 dev eth0` dan `ip route replace default via 192.231.4.1`

**Alice**

```
auto eth0
iface eth0 inet static
    address 192.231.12.2
    netmask 255.255.255.0
    gateway 192.231.12.1
```

Skrip: `ip addr replace 192.231.12.2/24 dev eth0` dan `ip route replace default via 192.231.12.1`

**Reika**

```
auto eth0
iface eth0 inet static
    address 192.231.12.3
    netmask 255.255.255.0
    gateway 192.231.12.1
```

Skrip: `ip addr replace 192.231.12.3/24 dev eth0` dan `ip route replace default via 192.231.12.1`

**Taro**

```
auto eth0
iface eth0 inet static
    address 192.231.15.66
    netmask 255.255.255.192
    gateway 192.231.15.65
```

Skrip: `ip addr replace 192.231.15.66/26 dev eth0` dan `ip route replace default via 192.231.15.65`

**Yasuo**

```
auto eth0
iface eth0 inet static
    address 192.231.13.2
    netmask 255.255.255.0
    gateway 192.231.13.1
```

Skrip: `ip addr replace 192.231.13.2/24 dev eth0` dan `ip route replace default via 192.231.13.1`

**Miho**

```
auto eth0
iface eth0 inet static
    address 192.231.13.3
    netmask 255.255.255.0
    gateway 192.231.13.1
```

Skrip: `ip addr replace 192.231.13.3/24 dev eth0` dan `ip route replace default via 192.231.13.1`

**Kuma**

```
auto eth0
iface eth0 inet static
    address 192.231.13.4
    netmask 255.255.255.0
    gateway 192.231.13.1
```

Skrip: `ip addr replace 192.231.13.4/24 dev eth0` dan `ip route replace default via 192.231.13.1`

**Collective**

```
auto eth0
iface eth0 inet static
    address 192.231.10.2
    netmask 255.255.254.0
    gateway 192.231.10.1
```

Skrip: `ip addr replace 192.231.10.2/23 dev eth0` dan `ip route replace default via 192.231.10.1`

**Ghost**

```
auto eth0
iface eth0 inet static
    address 192.231.10.3
    netmask 255.255.254.0
    gateway 192.231.10.1
```

Skrip: `ip addr replace 192.231.10.3/23 dev eth0` dan `ip route replace default via 192.231.10.1`

**Lilith**

```
auto eth0
iface eth0 inet static
    address 192.231.14.130
    netmask 255.255.255.128
    gateway 192.231.14.129
```

Skrip: `ip addr replace 192.231.14.130/25 dev eth0` dan `ip route replace default via 192.231.14.129`

**Specter**

```
auto eth0
iface eth0 inet static
    address 192.231.14.131
    netmask 255.255.255.128
    gateway 192.231.14.129
```

Skrip: `ip addr replace 192.231.14.131/25 dev eth0` dan `ip route replace default via 192.231.14.129`

**Chisa**

```
auto eth0
iface eth0 inet static
    address 192.231.15.130
    netmask 255.255.255.192
    gateway 192.231.15.129
```

Skrip: `ip addr replace 192.231.15.130/26 dev eth0` dan `ip route replace default via 192.231.15.129`

**Hodgeson**

```
auto eth0
iface eth0 inet static
    address 192.231.15.131
    netmask 255.255.255.192
    gateway 192.231.15.129
```

Skrip: `ip addr replace 192.231.15.131/26 dev eth0` dan `ip route replace default via 192.231.15.129`

**Eiri**

```
auto eth0
iface eth0 inet static
    address 192.231.16.50
    netmask 255.255.255.248
    gateway 192.231.16.49
```

Skrip: `ip addr replace 192.231.16.50/29 dev eth0` dan `ip route replace default via 192.231.16.49`

**Semi**

```
auto eth0
iface eth0 inet static
    address 192.231.16.2
    netmask 255.255.255.224
    gateway 192.231.16.1
```

Skrip: `ip addr replace 192.231.16.2/27 dev eth0` dan `ip route replace default via 192.231.16.1`

**Lain**

```
auto eth0
iface eth0 inet static
    address 192.231.15.2
    netmask 255.255.255.192
    gateway 192.231.15.1
```

Skrip: `ip addr replace 192.231.15.2/26 dev eth0` dan `ip route replace default via 192.231.15.1`

**Mika**

```
auto eth0
iface eth0 inet static
    address 192.231.15.3
    netmask 255.255.255.192
    gateway 192.231.15.1
```

Skrip: `ip addr replace 192.231.15.3/26 dev eth0` dan `ip route replace default via 192.231.15.1`

**JJ**

```
auto eth0
iface eth0 inet static
    address 192.231.14.2
    netmask 255.255.255.128
    gateway 192.231.14.1
```

Skrip: `ip addr replace 192.231.14.2/25 dev eth0` dan `ip route replace default via 192.231.14.1`

**MyuMyu**

```
auto eth0
iface eth0 inet static
    address 192.231.14.3
    netmask 255.255.255.128
    gateway 192.231.14.1
```

Skrip: `ip addr replace 192.231.14.3/25 dev eth0` dan `ip route replace default via 192.231.14.1`


### Testing

