# Implementasi VPN Site-to-Site Menggunakan WireGuard

Repositori ini digunakan untuk mendokumentasikan **Progres 1** proyek mata kuliah Jaringan Komputer dan Komunikasi Data, yaitu tahap **perencanaan VPN dan subnetting** untuk implementasi VPN *Site-to-Site* menggunakan WireGuard pada GNS3.

---

## 👥 Anggota Kelompok

* **Muhammad Fadhil Ferdinan** (2401020047)
* **Abdul Hafidz** (2401020051)
* **Muhammad Rahman Afrialdi** (2401020058)
* **Vito Hendriansyah** (2401020084)
* **Adib Farhan Sihombing** (2401020090)

---

## 📌 Status Proyek

- [x] **Progres 1:** Perencanaan VPN dan Subnetting
- [ ] **Progres 2:** Routing Dasar
- [ ] **Progres 3:** Implementasi WireGuard
- [ ] **Final:** Analisis Keamanan VPN

---

## 🛠️ Progres 1: Perencanaan VPN dan Subnetting

Pada Progres 1, dilakukan perencanaan awal jaringan VPN *Site-to-Site* menggunakan WireGuard. Perencanaan ini meliputi analisis kebutuhan, rancangan topologi, serta tabel IP address dan subnetting.

Project ini dirancang untuk menghubungkan dua jaringan cabang, yaitu **Cabang A** dan **Cabang B**, melalui jaringan internet simulasi pada GNS3. Perangkat **R-INTERNET** digunakan sebagai representasi jaringan internet, sedangkan **R-CABANG-A** dan **R-CABANG-B** digunakan sebagai router pada masing-masing cabang.

WireGuard direncanakan berjalan pada dua node Ubuntu, yaitu **UBUNTU-A** dan **UBUNTU-B**, yang nantinya akan membentuk tunnel VPN dengan network `10.254.254.0/30`.

---

## 🎯 Tujuan Progres 1

Tujuan dari Progres 1 adalah:

1. Menentukan kebutuhan perangkat lunak dan perangkat virtual.
2. Merancang topologi jaringan VPN *Site-to-Site* di GNS3.
3. Menentukan alokasi IP address untuk jaringan WAN, LAN, dan tunnel VPN.
4. Menyusun tabel subnetting sebagai acuan konfigurasi pada progres berikutnya.
5. Menyiapkan dokumentasi awal berupa proposal, topologi, dan IP plan.

---

## 🧩 Analisis Kebutuhan

### A. Perangkat Lunak

| Perangkat Lunak | Fungsi |
|---|---|
| Fedora 44 | Sistem operasi host utama |
| GNS3 | Platform simulasi jaringan |
| QEMU/KVM | Virtualisasi node MikroTik dan Ubuntu |
| MikroTik CHR / RouterOS | Router cabang dan router internet |
| Ubuntu Server 22.04 LTS | Sistem operasi endpoint WireGuard |
| WireGuard | Teknologi VPN yang akan digunakan |
| Winbox / RoMON | Manajemen perangkat MikroTik |

### B. Perangkat Virtual

| Perangkat | Jumlah | Keterangan |
|---|---:|---|
| MikroTik CHR | 3 | R-INTERNET, R-CABANG-A, R-CABANG-B |
| Ubuntu Server 22.04 LTS | 2 | UBUNTU-A dan UBUNTU-B |
| Cloud-libvirt | 1 | Jalur internet dari host Fedora |
| Host / Host1 | 2 | Jalur management tambahan untuk SSH |

---

## 🧱 Topologi Jaringan

Topologi yang digunakan pada rancangan Progres 1 adalah sebagai berikut:

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

| Perangkat | Fungsi |
|---|---|
| Cloud-libvirt | Memberikan akses internet dari host Fedora ke R-INTERNET |
| R-INTERNET | Router internet simulasi yang menghubungkan kedua cabang |
| R-CABANG-A | Router pada sisi Cabang A |
| R-CABANG-B | Router pada sisi Cabang B |
| UBUNTU-A | Endpoint WireGuard pada sisi Cabang A |
| UBUNTU-B | Endpoint WireGuard pada sisi Cabang B |
| Host / Host1 | Jalur management tambahan untuk akses SSH |

Jalur utama komunikasi VPN yang direncanakan:

```text
UBUNTU-A → R-CABANG-A → R-INTERNET → R-CABANG-B → UBUNTU-B
```

---

## 🔌 Mapping Interface

| Device | Interface | Terhubung ke | Fungsi |
|---|---|---|---|
| R-INTERNET | ether1 | Cloud-libvirt | Internet real / management |
| R-INTERNET | ether2 | R-CABANG-A ether1 | Link WAN ke Cabang A |
| R-INTERNET | ether3 | R-CABANG-B ether1 | Link WAN ke Cabang B |
| R-CABANG-A | ether1 | R-INTERNET ether2 | WAN Cabang A |
| R-CABANG-A | ether2 | UBUNTU-A | LAN Cabang A |
| R-CABANG-B | ether1 | R-INTERNET ether3 | WAN Cabang B |
| R-CABANG-B | ether2 | UBUNTU-B | LAN Cabang B |
| UBUNTU-A | eth0 / ens3 | R-CABANG-A ether2 | Interface LAN Cabang A |
| UBUNTU-B | eth0 / ens3 | R-CABANG-B ether2 | Interface LAN Cabang B |

---

## 🌐 Tabel IP Address dan Subnetting

### A. Alokasi IP Cloud dan Inter-Router

| Segmen | Perangkat / Interface | IP Address | Subnet / Prefix | Keterangan |
|---|---|---|---|---|
| Cloud / Internet | R-INTERNET ether1 | DHCP dari Cloud-libvirt | DHCP | Internet real dari host Fedora |
| WAN Cabang A | R-INTERNET ether2 | `10.0.0.1` | `/30` | Gateway WAN menuju R-CABANG-A |
| WAN Cabang A | R-CABANG-A ether1 | `10.0.0.2` | `/30` | IP WAN Router Cabang A |
| WAN Cabang B | R-INTERNET ether3 | `10.0.1.1` | `/30` | Gateway WAN menuju R-CABANG-B |
| WAN Cabang B | R-CABANG-B ether1 | `10.0.1.2` | `/30` | IP WAN Router Cabang B |

### B. Alokasi IP LAN

| Lokasi / Segmen | Perangkat / Interface | IP Address | Subnet / Prefix | Keterangan |
|---|---|---|---|---|
| LAN Cabang A | Network ID | `172.16.10.0` | `/24` | Subnet lokal Cabang A |
| LAN Cabang A | R-CABANG-A ether2 | `172.16.10.1` | `/24` | Gateway LAN Cabang A |
| LAN Cabang A | UBUNTU-A eth0 / ens3 | `172.16.10.2` | `/24` | Endpoint WireGuard Cabang A |
| LAN Cabang B | Network ID | `172.16.20.0` | `/24` | Subnet lokal Cabang B |
| LAN Cabang B | R-CABANG-B ether2 | `172.16.20.1` | `/24` | Gateway LAN Cabang B |
| LAN Cabang B | UBUNTU-B eth0 / ens3 | `172.16.20.2` | `/24` | Endpoint WireGuard Cabang B |

### C. Alokasi IP Tunnel WireGuard

| Segmen | Interface | IP Address | Subnet / Prefix | Keterangan |
|---|---|---|---|---|
| WireGuard Tunnel | Network ID | `10.254.254.0` | `/30` | Network tunnel VPN |
| WireGuard Tunnel | UBUNTU-A wg0 | `10.254.254.1` | `/30` | IP tunnel sisi Cabang A |
| WireGuard Tunnel | UBUNTU-B wg0 | `10.254.254.2` | `/30` | IP tunnel sisi Cabang B |

---

## 📊 Ringkasan Subnetting

| Network | Prefix | Jumlah Host Usable | Fungsi |
|---|---:|---:|---|
| `10.0.0.0/30` | `/30` | 2 host | Link R-INTERNET ke R-CABANG-A |
| `10.0.1.0/30` | `/30` | 2 host | Link R-INTERNET ke R-CABANG-B |
| `172.16.10.0/24` | `/24` | 254 host | LAN Cabang A |
| `172.16.20.0/24` | `/24` | 254 host | LAN Cabang B |
| `10.254.254.0/30` | `/30` | 2 host | Tunnel WireGuard |

---

## 📁 Dokumen Progres 1

Dokumen yang dikumpulkan pada Progres 1:

| Dokumen | Status |
|---|---|
| Proposal dan Analisis Kebutuhan | Selesai |
| Diagram Topologi | Selesai |
| Tabel IP dan Subnetting | Selesai |

---

## 📝 Catatan

README ini hanya berisi dokumentasi **Progres 1**, yaitu tahap perencanaan VPN dan subnetting.  
Konfigurasi routing dasar, implementasi WireGuard, hasil pengujian tunnel, ping, dan transfer file akan didokumentasikan pada progres berikutnya.