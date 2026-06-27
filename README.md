# Implementasi VPN Site-to-Site Menggunakan WireGuard

Repositori ini digunakan untuk mendokumentasikan proyek mata kuliah **Jaringan Komputer dan Komunikasi Data**, yaitu implementasi **VPN Site-to-Site menggunakan WireGuard** pada GNS3.

Project ini mensimulasikan dua jaringan cabang yang saling terhubung melalui jaringan internet simulasi. VPN WireGuard digunakan untuk membentuk tunnel terenkripsi agar komunikasi antar cabang dapat berjalan secara aman.

---

## 👥 Anggota Kelompok

* **Muhammad Fadhil Ferdinan** (2401020047)
* **Abdul Hafidz** (2401020051)
* **Muhammad Rahman Afrialdi** (2401020058)
* **Vito Hendriansyah** (2401020084)
* **Adib Farhan Sihombing** (2401020090)

---

## 📌 Status Proyek

* [x] **Progres 1:** Perencanaan VPN dan Subnetting
* [x] **Progres 2:** Routing Dasar
* [x] **Progres 3:** Implementasi WireGuard
* [x] **Final:** Analisis Keamanan VPN

---

## 🛠️ Deskripsi Project

Project ini bertujuan untuk menghubungkan dua jaringan cabang, yaitu **Cabang A** dan **Cabang B**, menggunakan VPN Site-to-Site berbasis WireGuard.

Simulasi dilakukan menggunakan GNS3 dengan perangkat utama:

* MikroTik CHR sebagai router internet dan router cabang.
* Ubuntu Server 22.04 LTS sebagai endpoint WireGuard.
* Cloud-libvirt sebagai jalur internet dari host Fedora.
* WireGuard sebagai protokol VPN.

Pada project ini, **R-INTERNET** digunakan sebagai representasi jaringan internet. Sementara itu, **R-CABANG-A** dan **R-CABANG-B** berperan sebagai router pada masing-masing cabang.

WireGuard berjalan pada dua node Ubuntu, yaitu **UBUNTU-A** dan **UBUNTU-B**, dengan tunnel VPN menggunakan network `10.254.254.0/30`.

---

## 🎯 Tujuan Project

Tujuan dari project ini adalah:

1. Merancang topologi VPN Site-to-Site menggunakan GNS3.
2. Menentukan IP address dan subnetting untuk jaringan WAN, LAN, dan tunnel VPN.
3. Mengkonfigurasi routing dasar pada MikroTik.
4. Mengkonfigurasi NAT dan port forwarding untuk kebutuhan WireGuard.
5. Mengimplementasikan WireGuard pada Ubuntu Server.
6. Menguji koneksi tunnel VPN menggunakan `wg show`.
7. Menguji ping melalui VPN.
8. Menguji transfer file melalui VPN menggunakan SCP.
9. Menganalisis keamanan komunikasi antar cabang melalui tunnel terenkripsi.

---

## 🧩 Analisis Kebutuhan

### A. Perangkat Lunak

| Perangkat Lunak         | Fungsi                                      |
| ----------------------- | ------------------------------------------- |
| Fedora 44               | Sistem operasi host utama                   |
| GNS3                    | Platform simulasi jaringan                  |
| QEMU/KVM                | Virtualisasi node MikroTik dan Ubuntu       |
| MikroTik CHR / RouterOS | Router cabang dan router internet           |
| Ubuntu Server 22.04 LTS | Sistem operasi endpoint WireGuard           |
| WireGuard               | Teknologi VPN yang digunakan                |
| OpenSSH Server          | Akses SSH dan transfer file menggunakan SCP |
| Winbox / RoMON          | Manajemen perangkat MikroTik                |

### B. Perangkat Virtual

| Perangkat               | Jumlah | Keterangan                          |
| ----------------------- | -----: | ----------------------------------- |
| MikroTik CHR            |      3 | R-INTERNET, R-CABANG-A, R-CABANG-B  |
| Ubuntu Server 22.04 LTS |      2 | UBUNTU-A dan UBUNTU-B               |
| Cloud-libvirt           |      1 | Jalur internet dari host Fedora     |
| Host / Host1            |      2 | Jalur management tambahan untuk SSH |

---

## 🧱 Topologi Jaringan

Topologi yang digunakan pada project ini adalah sebagai berikut:

```text
                 Cloud-libvirt
                      |
                  R-INTERNET
                 /          \
        R-CABANG-A          R-CABANG-B
            |                    |
         UBUNTU-A             UBUNTU-B
            |                    |
          Host                Host1
```

### Keterangan Topologi

| Perangkat     | Fungsi                                                   |
| ------------- | -------------------------------------------------------- |
| Cloud-libvirt | Memberikan akses internet dari host Fedora ke R-INTERNET |
| R-INTERNET    | Router internet simulasi yang menghubungkan kedua cabang |
| R-CABANG-A    | Router pada sisi Cabang A                                |
| R-CABANG-B    | Router pada sisi Cabang B                                |
| UBUNTU-A      | Endpoint WireGuard pada sisi Cabang A                    |
| UBUNTU-B      | Endpoint WireGuard pada sisi Cabang B                    |
| Host / Host1  | Jalur management tambahan untuk akses SSH                |

Jalur fisik komunikasi antar site:

```text
UBUNTU-A → R-CABANG-A → R-INTERNET → R-CABANG-B → UBUNTU-B
```

Setelah WireGuard aktif, komunikasi antar site berjalan melalui interface virtual `wg0`.

---

## 🔌 Mapping Interface

| Device     | Interface   | Terhubung ke      | Fungsi                     |
| ---------- | ----------- | ----------------- | -------------------------- |
| R-INTERNET | ether1      | Cloud-libvirt     | Internet real / management |
| R-INTERNET | ether2      | R-CABANG-A ether1 | Link WAN ke Cabang A       |
| R-INTERNET | ether3      | R-CABANG-B ether1 | Link WAN ke Cabang B       |
| R-CABANG-A | ether1      | R-INTERNET ether2 | WAN Cabang A               |
| R-CABANG-A | ether2      | UBUNTU-A          | LAN Cabang A               |
| R-CABANG-B | ether1      | R-INTERNET ether3 | WAN Cabang B               |
| R-CABANG-B | ether2      | UBUNTU-B          | LAN Cabang B               |
| UBUNTU-A   | eth0 / ens3 | R-CABANG-A ether2 | Interface LAN Cabang A     |
| UBUNTU-B   | eth0 / ens3 | R-CABANG-B ether2 | Interface LAN Cabang B     |

---

## 🌐 Tabel IP Address dan Subnetting

### A. Alokasi IP Cloud dan Inter-Router

| Segmen           | Perangkat / Interface | IP Address              | Subnet / Prefix | Keterangan                     |
| ---------------- | --------------------- | ----------------------- | --------------- | ------------------------------ |
| Cloud / Internet | R-INTERNET ether1     | DHCP dari Cloud-libvirt | DHCP            | Internet real dari host Fedora |
| WAN Cabang A     | R-INTERNET ether2     | `10.0.0.1`              | `/30`           | Gateway WAN menuju R-CABANG-A  |
| WAN Cabang A     | R-CABANG-A ether1     | `10.0.0.2`              | `/30`           | IP WAN Router Cabang A         |
| WAN Cabang B     | R-INTERNET ether3     | `10.0.1.1`              | `/30`           | Gateway WAN menuju R-CABANG-B  |
| WAN Cabang B     | R-CABANG-B ether1     | `10.0.1.2`              | `/30`           | IP WAN Router Cabang B         |

### B. Alokasi IP LAN

| Lokasi / Segmen | Perangkat / Interface | IP Address    | Subnet / Prefix | Keterangan                  |
| --------------- | --------------------- | ------------- | --------------- | --------------------------- |
| LAN Cabang A    | Network ID            | `172.16.10.0` | `/24`           | Subnet lokal Cabang A       |
| LAN Cabang A    | R-CABANG-A ether2     | `172.16.10.1` | `/24`           | Gateway LAN Cabang A        |
| LAN Cabang A    | UBUNTU-A eth0 / ens3  | `172.16.10.2` | `/24`           | Endpoint WireGuard Cabang A |
| LAN Cabang B    | Network ID            | `172.16.20.0` | `/24`           | Subnet lokal Cabang B       |
| LAN Cabang B    | R-CABANG-B ether2     | `172.16.20.1` | `/24`           | Gateway LAN Cabang B        |
| LAN Cabang B    | UBUNTU-B eth0 / ens3  | `172.16.20.2` | `/24`           | Endpoint WireGuard Cabang B |

### C. Alokasi IP Tunnel WireGuard

| Segmen           | Interface    | IP Address     | Subnet / Prefix | Keterangan              |
| ---------------- | ------------ | -------------- | --------------- | ----------------------- |
| WireGuard Tunnel | Network ID   | `10.254.254.0` | `/30`           | Network tunnel VPN      |
| WireGuard Tunnel | UBUNTU-A wg0 | `10.254.254.1` | `/30`           | IP tunnel sisi Cabang A |
| WireGuard Tunnel | UBUNTU-B wg0 | `10.254.254.2` | `/30`           | IP tunnel sisi Cabang B |

---

## 📊 Ringkasan Subnetting

| Network           | Prefix | Jumlah Host Usable | Fungsi                        |
| ----------------- | -----: | -----------------: | ----------------------------- |
| `10.0.0.0/30`     |  `/30` |             2 host | Link R-INTERNET ke R-CABANG-A |
| `10.0.1.0/30`     |  `/30` |             2 host | Link R-INTERNET ke R-CABANG-B |
| `172.16.10.0/24`  |  `/24` |           254 host | LAN Cabang A                  |
| `172.16.20.0/24`  |  `/24` |           254 host | LAN Cabang B                  |
| `10.254.254.0/30` |  `/30` |             2 host | Tunnel WireGuard              |

---

## ⚙️ Progres 2: Routing Dasar

Pada tahap routing dasar, dilakukan konfigurasi IP address, default route, DNS, dan NAT pada MikroTik.

### A. R-INTERNET

```mikrotik
/ip dhcp-client add interface=ether1 disabled=no
/ip address add address=10.0.0.1/30 interface=ether2
/ip address add address=10.0.1.1/30 interface=ether3
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT internet via Cloud-libvirt"
```

### B. R-CABANG-A

```mikrotik
/ip address add address=10.0.0.2/30 interface=ether1
/ip address add address=172.16.10.1/24 interface=ether2
/ip route add dst-address=0.0.0.0/0 gateway=10.0.0.1
/ip dns set servers=10.0.0.1 allow-remote-requests=yes
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT internet Cabang A"
```

### C. R-CABANG-B

```mikrotik
/ip address add address=10.0.1.2/30 interface=ether1
/ip address add address=172.16.20.1/24 interface=ether2
/ip route add dst-address=0.0.0.0/0 gateway=10.0.1.1
/ip dns set servers=10.0.1.1 allow-remote-requests=yes
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT internet Cabang B"
```

### D. Pengujian Routing Dasar

Pengujian yang dilakukan:

```bash
ping 172.16.10.1
ping 172.16.20.1
ping 8.8.8.8
ping google.com
```

Dari sisi MikroTik:

```mikrotik
ping 10.0.0.1
ping 10.0.1.1
ping 10.0.0.2
ping 10.0.1.2
ping 8.8.8.8
```

Hasil pengujian routing dasar berhasil. UBUNTU-A dan UBUNTU-B dapat terhubung ke gateway masing-masing serta dapat mengakses internet.

---

## 🔐 Progres 3: Implementasi WireGuard

WireGuard diinstal pada kedua Ubuntu:

```bash
sudo apt update
sudo apt install wireguard wireguard-tools openssh-server net-tools traceroute -y
```

SSH diaktifkan untuk kebutuhan remote access dan transfer file:

```bash
sudo systemctl enable --now ssh
```

### A. Generate Key WireGuard

Pada masing-masing Ubuntu:

```bash
sudo su
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
cat privatekey
cat publickey
```

Private key digunakan pada perangkat lokal, sedangkan public key digunakan pada konfigurasi peer lawan.

### B. Port Forward WireGuard pada MikroTik

Karena WireGuard berjalan di Ubuntu yang berada di belakang router cabang, maka dibuat rule dst-nat.

Pada R-CABANG-A:

```mikrotik
/ip firewall nat add chain=dstnat protocol=udp dst-port=51820 in-interface=ether1 action=dst-nat to-addresses=172.16.10.2 to-ports=51820 comment="Port forward WireGuard ke UBUNTU-A"
```

Pada R-CABANG-B:

```mikrotik
/ip firewall nat add chain=dstnat protocol=udp dst-port=51820 in-interface=ether1 action=dst-nat to-addresses=172.16.20.2 to-ports=51820 comment="Port forward WireGuard ke UBUNTU-B"
```

### C. Bypass NAT Antar LAN VPN

Agar trafik antar LAN cabang tidak terkena masquerade, dibuat bypass NAT.

Pada R-CABANG-A:

```mikrotik
/ip firewall nat add chain=srcnat src-address=172.16.10.0/24 dst-address=172.16.20.0/24 action=accept comment="Bypass NAT ke LAN Cabang B via VPN"
```

Pada R-CABANG-B:

```mikrotik
/ip firewall nat add chain=srcnat src-address=172.16.20.0/24 dst-address=172.16.10.0/24 action=accept comment="Bypass NAT ke LAN Cabang A via VPN"
```

### D. Konfigurasi WireGuard UBUNTU-A

File konfigurasi:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Isi konfigurasi:

```ini
[Interface]
Address = 10.254.254.1/30
ListenPort = 51820
PrivateKey = PRIVATE_KEY_UBUNTU_A

[Peer]
PublicKey = PUBLIC_KEY_UBUNTU_B
AllowedIPs = 10.254.254.2/32, 172.16.20.0/24
Endpoint = 10.0.1.2:51820
PersistentKeepalive = 25
```

### E. Konfigurasi WireGuard UBUNTU-B

File konfigurasi:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Isi konfigurasi:

```ini
[Interface]
Address = 10.254.254.2/30
ListenPort = 51820
PrivateKey = PRIVATE_KEY_UBUNTU_B

[Peer]
PublicKey = PUBLIC_KEY_UBUNTU_A
AllowedIPs = 10.254.254.1/32, 172.16.10.0/24
Endpoint = 10.0.0.2:51820
PersistentKeepalive = 25
```

### F. Menjalankan WireGuard

Pada kedua Ubuntu:

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable --now wg-quick@wg0
```

Cek status WireGuard:

```bash
sudo wg show
```

---

## ✅ Hasil Pengujian

### A. Pengujian Tunnel VPN

Perintah:

```bash
sudo wg show
```

Hasil yang diperoleh:

```text
latest handshake
transfer received/sent
```

Status tersebut menunjukkan bahwa tunnel WireGuard berhasil terbentuk dan kedua peer sudah saling terhubung.

---

### B. Pengujian Ping Melalui VPN

Dari UBUNTU-A ke UBUNTU-B:

```bash
ping 10.254.254.2
```

Dari UBUNTU-B ke UBUNTU-A:

```bash
ping 10.254.254.1
```

Hasil pengujian: **berhasil reply**.

---

### C. Pengujian Ping Antar IP LAN Melalui VPN

Dari UBUNTU-A ke UBUNTU-B:

```bash
ping 172.16.20.2
```

Dari UBUNTU-B ke UBUNTU-A:

```bash
ping 172.16.10.2
```

Hasil pengujian: **berhasil reply**.

Hal ini menunjukkan bahwa jaringan Cabang A dan Cabang B dapat berkomunikasi melalui tunnel WireGuard.

---

### D. Pengujian Traceroute

Dari UBUNTU-A ke IP tunnel UBUNTU-B:

```bash
traceroute 10.254.254.2
```

Dari UBUNTU-A ke IP LAN UBUNTU-B:

```bash
traceroute 172.16.20.2
```

Hasil traceroute terlihat pendek karena sistem operasi melihat jalur melalui interface virtual `wg0`.

Secara fisik, paket tetap melewati:

```text
UBUNTU-A → R-CABANG-A → R-INTERNET → R-CABANG-B → UBUNTU-B
```

Namun secara virtual, komunikasi diarahkan melalui tunnel WireGuard.

---

### E. Pengujian Transfer File Melalui VPN

Pada UBUNTU-A dibuat file pengujian:

```bash
echo "Pengujian transfer file melalui VPN WireGuard berhasil" > test-vpn.txt
```

File dikirim ke UBUNTU-B melalui IP tunnel:

```bash
scp test-vpn.txt ubuntu@10.254.254.2:/home/ubuntu/
```

Pada UBUNTU-B dilakukan pengecekan:

```bash
ls
cat test-vpn.txt
```

Hasil pengujian: **file berhasil dikirim dan isi file dapat dibaca**.

---

## 🔎 Analisis Keamanan VPN

WireGuard membentuk tunnel terenkripsi antara UBUNTU-A dan UBUNTU-B. Walaupun paket tetap melewati router fisik seperti R-CABANG-A, R-INTERNET, dan R-CABANG-B, isi data tidak dapat dibaca oleh router di tengah karena sudah dienkripsi oleh WireGuard.

Router di tengah hanya melihat trafik UDP port `51820`, sedangkan isi komunikasi berada di dalam tunnel VPN.

Manfaat keamanan dari implementasi ini:

1. Data antar cabang dikirim melalui tunnel terenkripsi.
2. Setiap peer WireGuard menggunakan pasangan public key dan private key.
3. Komunikasi antar site tidak dikirim secara terbuka.
4. Transfer file melalui SCP dapat dilakukan lewat jalur VPN.
5. Cocok digunakan untuk menghubungkan jaringan kantor cabang secara aman.

---

## 📁 Struktur Repositori

Struktur repositori yang disarankan:

```text
.
├── README.md
├── docs/
│   ├── proposal.pdf
│   ├── laporan-progress-1.pdf
│   ├── laporan-progress-2.pdf
│   ├── laporan-progress-3.pdf
│   └── laporan-final.pdf
├── diagram/
│   ├── topologi.png
│   └── topologi.drawio
└── gns3/
    └── VPN-Site-to-Site-WireGuard.gns3project
```

Catatan:

* File konfigurasi WireGuard yang diunggah ke repository sebaiknya menggunakan format `.example`.
* Jangan mengunggah private key asli ke GitHub.
* Jika file GNS3 terlalu besar, gunakan export tanpa base images dan snapshots, atau gunakan Git LFS.

---

## 📁 Dokumen Project

Dokumen yang dikumpulkan:

| Dokumen                         | Status  |
| ------------------------------- | ------- |
| Proposal dan Analisis Kebutuhan | Selesai |
| Diagram Topologi                | Selesai |
| Tabel IP dan Subnetting         | Selesai |
| File GNS3                       | Selesai |
| Screenshot Konfigurasi          | Selesai |
| Video Demonstrasi               | Selesai |
| Hasil Pengujian                 | Selesai |
| Laporan Akhir                   | Selesai |
| Slide Presentasi                | Selesai |

---

## 📝 Kesimpulan

Project **Implementasi VPN Site-to-Site Menggunakan WireGuard** berhasil dilakukan pada GNS3.

Hasil akhir project menunjukkan bahwa:

1. Routing dasar antar router berhasil.
2. NAT dan port forwarding WireGuard berhasil dikonfigurasi.
3. Tunnel WireGuard berhasil terbentuk.
4. Ping melalui VPN berhasil.
5. Ping antar IP LAN melalui VPN berhasil.
6. Transfer file menggunakan SCP melalui VPN berhasil.
7. Komunikasi antar cabang dapat berjalan melalui tunnel terenkripsi.

Dengan demikian, WireGuard dapat digunakan sebagai solusi VPN Site-to-Site yang ringan, sederhana, dan aman untuk menghubungkan dua jaringan cabang melalui jaringan internet atau WAN simulasi.

