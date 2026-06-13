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

| Lokasi / Segmen | Perangkat / Interface | Alamat IP | Subnet Mask / Prefix | Keterangan / Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Site A (Pusat)** | Network ID LAN | `192.168.10.0` | `255.255.255.0` (/24) | Subnet untuk client di Kantor Pusat |
| | Gateway A (LAN) | `192.168.10.1` | `255.255.255.0` (/24) | IP lokal internal Router Pusat |
| | Gateway A (WAN) | `180.250.10.2` | `255.255.255.252` (/30)| IP Publik interface luar (ke Internet) |
| | Client Pusat (PC-01)| `192.168.10.10`| `255.255.255.0` (/24) | Gateway: `192.168.10.1` |
| **Site B (Cabang)**| Network ID LAN | `192.168.20.0` | `255.255.255.0` (/24) | Subnet untuk client di Kantor Cabang |
| | Gateway B (LAN) | `192.168.20.1` | `255.255.255.0` (/24) | IP lokal internal Router Cabang |
| | Gateway B (WAN) | `180.250.20.2` | `255.255.255.252` (/30)| IP Publik interface luar (ke Internet) |
| | Client Cabang (PC-02)| `192.168.20.10`| `255.255.255.0` (/24) | Gateway: `192.168.20.1` |
| **WireGuard Tunnel**| Network ID VPN | `10.0.0.0` | `255.255.255.0` (/24) | Segmen IP khusus di dalam *tunnel* |
| | Interface `wg0` (Pusat)| `10.0.0.1` | Point-to-Point (/24) | Peer IP untuk Router Pusat |
| | Interface `wg0` (Cabang)| `10.0.0.2` | Point-to-Point (/24) | Peer IP untuk Router Cabang |

---
*Pembaruan dokumentasi untuk Progres 2 (Routing Dasar) akan diperbarui pada milestone berikutnya.*