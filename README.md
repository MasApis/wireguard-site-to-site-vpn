# Implementasi VPN Site-to-Site Menggunakan WireGuard

Repositori ini digunakan untuk mendokumentasikan proyek mata kuliah Jaringan Komputer, mulai dari tahap perencanaan, konfigurasi routing, hingga implementasi dan analisis keamanan koneksi VPN *Site-to-Site* menggunakan WireGuard.

---

## 👥 Anggota Kelompok
* **Muhammad Fadhil Ferdinan** (2401020047)
* **Abdul Hafidz** (2401020051)
* **Muhammad Rahman Afrialdi** (2401020058)
* **Vito Hendriansyah** (2401020084)
* **Adib Farhan Sihombing** (2401020090)

---

## 📌 Peta Jalan Proyek (Roadmap)
- [x] **Progres 1:** Perencanaan VPN dan Subnetting *(Current)*
- [ ] **Progres 2:** Routing Dasar
- [ ] **Progres 3:** Implementasi WireGuard
- [ ] **Final:** Analisis Keamanan VPN

---

## 🛠️ Progres 1: Perencanaan VPN & Subnetting

Pada tahap awal ini, kami merancang topologi interkoneksi jaringan antar dua lokasi (*Site A* dan *Site B*) menggunakan jalur terenkripsi WireGuard, lengkap dengan alokasi pengalamatan IP.

### A. Dokumen & Diagram Topologi
* **Proposal & Analisis Kebutuhan:** Dapat diakses pada file dokumen di dalam folder `docs/`
* **Diagram Topologi Jaringan:** Dapat dilihat pada folder `diagram/` (tersedia format `.png` dan mentahan file `.drawio` / `.vsdx`)

### B. Tabel IP dan Subnetting

### B. Tabel IP dan Subnetting

| Lokasi / Segmen | Perangkat / Interface | Alamat IP | Subnet Mask / Prefix | Keterangan / Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Link Publik (WAN A)**| Router-ISP (ether2) | `203.0.113.1` | `255.255.255.252` (/30)| Gerbang publik arah Kantor Pusat |
| | Mikrotik-R-A (ether1) | `203.0.113.2` | `255.255.255.252` (/30)| IP WAN Router Pusat (Gateway: `203.0.113.1`) |
| **Link Publik (WAN B)**| Router-ISP (ether3) | `198.51.100.1` | `255.255.255.252` (/30)| Gerbang publik arah Kantor Cabang |
| | Mikrotik-R-B (ether1) | `198.51.100.2` | `255.255.255.252` (/30)| IP WAN Router Cabang (Gateway: `198.51.100.1`) |
| **Site A (Pusat)** | Network ID LAN | `172.16.10.0` | `255.255.255.0` (/24) | Subnet lokal internal Kantor Pusat |
| | Mikrotik-R-A (ether2) | `172.16.10.1` | `255.255.255.0` (/24) | IP Gateway lokal untuk LAN Pusat |
| | Ubuntu-WG-A (ens3) | `172.16.10.2` | `255.255.255.0` (/24) | Server VPN Pusat (Gateway: `172.16.10.1`) |
| **Site B (Cabang)**| Network ID LAN | `172.16.20.0` | `255.255.255.0` (/24) | Subnet lokal internal Kantor Cabang |
| | Mikrotik-R-B (ether2) | `172.16.20.1` | `255.255.255.0` (/24) | IP Gateway lokal untuk LAN Cabang |
| | Ubuntu-WG-B (ens3) | `172.16.20.2` | `255.255.255.0` (/24) | Server VPN Cabang (Gateway: `172.16.20.1`) |
| **WireGuard Tunnel**| Network ID VPN | `10.254.254.0`| `255.255.255.252` (/30)| Segmen network virtual *inter-tunnel* |
| | Interface `wg0` (Pusat)| `10.254.254.1`| Point-to-Point (/30)  | IP Tunnel Endpoint Server Pusat |
| | Interface `wg0` (Cabang)| `10.254.254.2`| Point-to-Point (/30)  | IP Tunnel Endpoint Server Cabang |

---
*Pembaruan dokumentasi untuk Progres 2 (Routing Dasar) akan diperbarui pada milestone berikutnya.*