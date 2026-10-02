# 🐳 Panduan Docker Desktop — Aplikasi SIMONA

Panduan lengkap menjalankan aplikasi **SIMONA** secara lokal menggunakan Docker Desktop di Windows, mulai dari instalasi hingga aplikasi berjalan.

---

## 📋 Daftar Isi

- [Prasyarat Sistem](#-prasyarat-sistem)
- [1. Instalasi Docker Desktop](#1-instalasi-docker-desktop)
- [2. Verifikasi Instalasi](#2-verifikasi-instalasi)
- [3. Persiapan Project](#3-persiapan-project)
- [4. Konfigurasi Environment](#4-konfigurasi-environment)
- [5. Build & Jalankan Aplikasi](#5-build--jalankan-aplikasi)
- [6. Akses Aplikasi](#6-akses-aplikasi)
- [7. Perintah Docker Berguna](#7-perintah-docker-berguna)
- [8. Troubleshooting](#8-troubleshooting)

---

## 💻 Prasyarat Sistem

Pastikan komputer Anda memenuhi spesifikasi minimum berikut sebelum memulai:

| Komponen | Minimum |
|---|---|
| OS | Windows 10 64-bit (Build 19041+) atau Windows 11 |
| RAM | 4 GB (disarankan 8 GB) |
| Storage | 10 GB ruang kosong |
| CPU | Prosesor 64-bit dengan dukungan virtualisasi (Intel VT-x / AMD-V) |
| Fitur Windows | WSL 2 (Windows Subsystem for Linux 2) |

> **Catatan:** Virtualisasi harus diaktifkan di BIOS/UEFI. Jika belum, masuk ke BIOS dan aktifkan opsi *Intel Virtualization Technology* atau *AMD SVM Mode*.

---

## 1. Instalasi Docker Desktop

### Langkah 1.1 — Aktifkan WSL 2 (jika belum)

Buka **PowerShell sebagai Administrator** dan jalankan:

```powershell
# Aktifkan fitur WSL
wsl --install

# Set WSL 2 sebagai versi default
wsl --set-default-version 2
```

Restart komputer jika diminta.

---

### Langkah 1.2 — Unduh Docker Desktop

1. Buka browser dan kunjungi: **https://www.docker.com/products/docker-desktop/**
2. Klik tombol **"Download for Windows"**
3. File installer bernama `Docker Desktop Installer.exe` akan terunduh

---

### Langkah 1.3 — Instal Docker Desktop

1. Jalankan file `Docker Desktop Installer.exe` sebagai **Administrator** (klik kanan → *Run as administrator*)
2. Pada layar konfigurasi, pastikan opsi berikut **dicentang**:
   - ✅ **Use WSL 2 instead of Hyper-V** *(disarankan)*
   - ✅ **Add shortcut to desktop**
3. Klik **OK** dan tunggu proses instalasi selesai
4. Klik **Close and restart** untuk memulai ulang komputer

---

### Langkah 1.4 — Konfigurasi Awal Docker Desktop

Setelah restart:
1. Buka **Docker Desktop** dari Start Menu atau shortcut desktop
2. Setujui *Service Agreement* jika diminta
3. Pilih opsi **"Use recommended settings"** lalu klik **Finish**
4. Tunggu hingga Docker Engine benar-benar berjalan — ditandai dengan **ikon hijau** di taskbar

---

## 2. Verifikasi Instalasi

Buka **Command Prompt** atau **PowerShell** dan jalankan perintah berikut:

```bash
# Cek versi Docker
docker --version

# Cek versi Docker Compose
docker compose version

# Pastikan Docker Engine berjalan
docker info
```

✅ Jika berhasil, output akan menampilkan informasi versi. Contoh:
```
Docker version 27.x.x, build xxxxxxx
Docker Compose version v2.x.x
```

---

## 3. Persiapan Project

### Langkah 3.1 — Clone Repository

```bash
git clone <URL_REPOSITORY> simona
cd simona
```

### Langkah 3.2 — Pastikan Struktur File Docker Ada

Setelah clone, pastikan file-file berikut tersedia di direktori project:

```
simona/
├── Dockerfile                      # Konfigurasi build image (3-stage build)
├── docker-compose.yml              # Orkestrasi multi-service
├── .dockerignore                   # File yang diabaikan saat build
└── docker-compose/
    ├── entrypoint.sh               # Script startup container
    ├── nginx/
    │   └── app.conf                # Konfigurasi web server Nginx
    └── php/
        └── local.ini               # Konfigurasi PHP
```

---

## 4. Konfigurasi Environment

### Langkah 4.1 — Salin File Environment

```bash
# Salin file contoh environment
copy .env.example .env
```

### Langkah 4.2 — Edit File `.env`

Buka file `.env` dengan teks editor (Notepad, VS Code, dll.) dan sesuaikan nilai berikut:

```env
APP_NAME=SIMONA
APP_ENV=local
APP_KEY=                          # Akan digenerate otomatis (lihat langkah berikutnya)
APP_DEBUG=true
APP_URL=http://localhost:8000

# Tahun anggaran aktif
TAHUN_ANGGARAN=2026

# Database PostgreSQL — sesuaikan dengan kebutuhan
DB_CONNECTION=pgsql
DB_HOST=db                        # Jangan diubah (nama service di docker-compose)
DB_PORT=5432
DB_DATABASE=simona_db
DB_USERNAME=postgres
DB_PASSWORD=passwordrahasia123    # WAJIB diisi dengan password yang kuat

# Password default akun operator
OPERATOR_DEFAULT_PASSWORD=SimonaOpr2026!@#
```

> **⚠️ Penting:**
> - `DB_HOST=db` harus tetap `db` karena merujuk ke nama service PostgreSQL di dalam jaringan Docker
> - `DB_PASSWORD` **wajib diisi** — container database tidak akan berjalan jika kosong
> - Ubah `OPERATOR_DEFAULT_PASSWORD` sebelum digunakan di lingkungan produksi

### Langkah 4.3 — Generate APP_KEY

```bash
# Jalankan setelah container berhasil dibangun
docker compose run --rm app php artisan key:generate
```

---

## 5. Build & Jalankan Aplikasi

### Langkah 5.1 — Build Image Docker

```bash
# Build semua service (proses ini membutuhkan waktu beberapa menit pertama kali)
docker compose build
```

> **Info:** Proses build mencakup 3 tahap:
> - **Stage 1** — Build aset frontend (Node.js/Vite)
> - **Stage 2** — Install dependensi PHP (Composer)
> - **Stage 3** — Rakit runtime production (PHP-FPM + Nginx)

### Langkah 5.2 — Jalankan Semua Service

```bash
# Jalankan container di background
docker compose up -d
```

Docker akan menjalankan dua service secara bersamaan:

| Service | Container | Keterangan |
|---|---|---|
| `app` | `simona_app` | Aplikasi Laravel (Nginx + PHP-FPM) |
| `db` | `simona_db` | Database PostgreSQL 15 |

### Langkah 5.3 — Cek Status Container

```bash
docker compose ps
```

Output yang diharapkan:
```
NAME          IMAGE        COMMAND                  SERVICE   STATUS              PORTS
simona_app    simona-app   "/usr/local/bin/entr…"   app       Up (healthy)        0.0.0.0:8000->80/tcp
simona_db     postgres:15  "docker-entrypoint.s…"   db        Up (healthy)        5432/tcp
```

Pastikan kolom **STATUS** menunjukkan `Up (healthy)` pada kedua container.

### Langkah 5.4 — (Opsional) Jalankan Migrasi & Seeder Manual

Entrypoint otomatis menjalankan migrasi saat `APP_ENV=local`. Jika ingin menjalankan ulang secara manual:

```bash
# Jalankan migrasi database
docker compose exec app php artisan migrate

# Jalankan seeder role & permission
docker compose exec app php artisan db:seed --class=RoleAndPermissionSeeder
```

---

## 6. Akses Aplikasi

Setelah semua container berjalan, buka browser dan akses:

```
http://localhost:8000
```

> Akun default operator sudah dibuat oleh seeder. Gunakan password yang telah diset di `OPERATOR_DEFAULT_PASSWORD`.

---

## 7. Perintah Docker Berguna

### Melihat Log

```bash
# Lihat log semua service (real-time)
docker compose logs -f

# Lihat log hanya service app
docker compose logs -f app

# Lihat log hanya service database
docker compose logs -f db
```

### Masuk ke Dalam Container

```bash
# Masuk ke shell container app
docker compose exec app sh

# Jalankan perintah Artisan langsung
docker compose exec app php artisan <perintah>

# Contoh: lihat daftar route
docker compose exec app php artisan route:list
```

### Manajemen Container

```bash
# Hentikan semua container (data tetap tersimpan)
docker compose stop

# Hentikan dan hapus container (data volume tetap)
docker compose down

# Hentikan dan hapus container BESERTA data volume (⚠️ database akan terhapus!)
docker compose down -v

# Restart semua service
docker compose restart

# Restart hanya service app
docker compose restart app
```

### Build Ulang (setelah ada perubahan kode)

```bash
# Build ulang dan restart container
docker compose up -d --build
```

### Monitoring Resource

```bash
# Pantau penggunaan CPU, RAM container secara real-time
docker stats
```

---

## 8. Troubleshooting

Tabel berikut berisi masalah umum yang sering terjadi beserta solusinya:

| # | Masalah | Gejala | Kemungkinan Penyebab | Solusi |
|---|---|---|---|---|
| 1 | **Docker Desktop tidak bisa start** | Ikon Docker di taskbar terus loading / error | WSL 2 belum terinstall, virtualisasi BIOS nonaktif | Jalankan `wsl --install` di PowerShell Admin; aktifkan virtualisasi di BIOS |
| 2 | **Port 8000 sudah dipakai** | Error `bind: address already in use` saat `docker compose up` | Aplikasi lain sudah menggunakan port 8000 | Ganti port di `docker-compose.yml` ke `"8080:80"`, lalu akses `localhost:8080` |
| 3 | **Database tidak bisa terkoneksi** | Error `SQLSTATE[08006]` atau `Connection refused` di log | `DB_PASSWORD` kosong di `.env`, atau service `db` belum sehat | Pastikan `DB_PASSWORD` terisi di `.env`; tunggu `simona_db` status `healthy` |
| 4 | **Container app restart terus** | STATUS container app adalah `Restarting` | `APP_KEY` kosong di `.env`, atau error saat migrasi | Generate `APP_KEY`: `docker compose run --rm app php artisan key:generate` |
| 5 | **Halaman 500 / Error Laravel** | Browser menampilkan `500 Server Error` atau blank page | `APP_KEY` tidak di-set, permission storage salah | Cek log: `docker compose logs app`; pastikan `APP_KEY` terisi di `.env` |
| 6 | **Perubahan kode tidak terlihat** | Halaman lama masih tampil setelah edit file | Cache view/config masih aktif | Jalankan: `docker compose exec app php artisan view:clear && php artisan cache:clear` |
| 7 | **Build gagal / error npm** | Error saat `docker compose build` di tahap frontend | Cache npm korup atau dependensi tidak kompatibel | Jalankan `docker compose build --no-cache` untuk build ulang dari awal |
| 8 | **Volume database korup** | Error `database system was shut down` di log `db` | Container dihentikan paksa atau disk penuh | Hapus volume lama: `docker compose down -v` lalu `docker compose up -d` (⚠️ data terhapus) |
| 9 | **`docker compose` tidak dikenal** | Error `'docker-compose' is not recognized` | Menggunakan sintaks lama Docker Compose v1 | Gunakan `docker compose` (tanpa tanda `-`); update Docker Desktop ke versi terbaru |
| 10 | **Disk Docker penuh** | Error `no space left on device` saat build | Image/container lama menumpuk | Jalankan: `docker system prune -a` untuk membersihkan semua resource tidak terpakai |
| 11 | **Migrasi gagal: tabel sudah ada** | Error `table "xxx" already exists` | Migrasi sebagian sudah berjalan sebelumnya | Jalankan: `docker compose exec app php artisan migrate:fresh --seed` (⚠️ data terhapus) |
| 12 | **Permission error pada storage** | Error `Permission denied` di log PHP | File storage tidak bisa ditulis oleh container | Jalankan: `docker compose exec app chmod -R 775 storage bootstrap/cache` |

---

### Tips Umum

- **Selalu cek log terlebih dahulu** sebelum mencoba solusi lain:
  ```bash
  docker compose logs -f app
  ```

- **Urutan startup yang benar:** Database (`db`) harus `healthy` sebelum `app` dijalankan. Docker Compose menangani ini secara otomatis via `depends_on`.

- **Jangan jalankan** `docker compose down -v` kecuali Anda sudah yakin, karena perintah ini **menghapus semua data database**.

- Untuk melihat data di database menggunakan GUI, koneksikan tool seperti **DBeaver** atau **pgAdmin** ke:
  - Host: `localhost`
  - Port: `5432` *(pastikan expose port di `docker-compose.yml` sudah ada)*
  - Database: nilai `DB_DATABASE` di `.env`
  - Username & Password: sesuai `.env`

---

*Panduan ini dibuat untuk aplikasi SIMONA versi lokal. Untuk deployment ke lingkungan production, pastikan semua nilai sensitif di `.env` sudah diubah.*
