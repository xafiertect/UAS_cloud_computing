# 🌐 Secure WordPress Deployment with Docker Compose
> **Milestone 1 & 2 - Ujian Akhir Semester (UAS) Cloud Computing**  
> *Teknik Informatika - Semester 4*

<p align="center">
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/WordPress-21759B?style=for-the-badge&logo=wordpress&logoColor=white" alt="WordPress" />
  <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL" />
  <img src="https://img.shields.io/badge/phpMyAdmin-F89820?style=for-the-badge&logo=phpmyadmin&logoColor=white" alt="phpMyAdmin" />
  <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis" />
  <img src="https://img.shields.io/badge/MinIO-C72E49?style=for-the-badge&logo=minio&logoColor=white" alt="MinIO" />
  <img src="https://img.shields.io/badge/GNU_Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="Bash" />
  <img src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux" />
</p>

---

## 👥 Tim Pengembang (Contributors)
* **Rizqi Maulidiyah** - *Lead DevOps & Cloud Engineer* 
* **Alfin Ardiansyah** - *QA Engineer & Technical Writer* 

---

## 📌 Deskripsi Proyek
Proyek ini mengimplementasikan infrastruktur berbasis kontainer untuk mendeploy **WordPress** dengan arsitektur jaringan terisolasi menggunakan **Docker Compose**. Proyek ini menekankan pada konsep **keamanan jaringan (Zero-Trust isolation)**, manajemen dependensi berbasis **healthcheck**, dan pengamanan kredensial menggunakan berkas lingkungan variabel `.env` terenkripsi lokal.

---

## 🏗️ Arsitektur Sistem & Isolasi Jaringan
Kami merancang arsitektur sistem dengan membagi infrastruktur ke dalam dua jaringan kustom (`bridge`):
1. **`frontend-net`**: Mengizinkan akses HTTP luar menuju kontainer aplikasi WordPress.
2. **`backend-net`**: Jaringan internal terisolasi yang menghubungkan database MySQL dengan WordPress dan phpMyAdmin. MySQL sengaja dibuat **tidak mengekspos port ke mesin host** untuk mencegah ancaman eksploitasi eksternal langsung.

### Diagram Arsitektur Jaringan (Network Design)
```mermaid
graph TD
    subgraph Host ["Mesin Host (Port Akses)"]
        P8080["Port 8080 (WordPress Web)"]
        P8081["Port 8081 (phpMyAdmin GUI)"]
    end

    subgraph FrontendNet ["frontend-net (Jaringan Frontend)"]
        WP["WordPress Container (wordpress:6.5-apache)"]
    end

    subgraph BackendNet ["backend-net (Jaringan Backend Terisolasi)"]
        WP
        DB["MySQL Database Container (mysql:8.0 - Port 3306)"]
        PMA["phpMyAdmin Container (phpmyadmin:5.2)"]
    end

    P8080 -->|Akses HTTP| WP
    P8081 -->|Akses GUI| PMA
    WP -->|Koneksi DB Internal| DB
    PMA -->|Manajemen DB Internal| DB
    
    style DB fill:#ffcccc,stroke:#ff0000,stroke-width:2px,color:#000
    style WP fill:#cce5ff,stroke:#004085,stroke-width:2px,color:#000
    style PMA fill:#d4edda,stroke:#155724,stroke-width:2px,color:#000
```

---

## 🛠️ Stack Teknologi (Tech Stack)

### Milestone 1
* **Containerization**: Docker & Docker Compose
* **Application**: WordPress v6.5 (Apache base image)
* **Database**: MySQL v8.0 (Persistent Volume)
* **Database Tool**: phpMyAdmin v5.2
* **Scripting**: Bash (Automated project bootstrap)

### Milestone 2
* **Object Cache**: Redis v7-alpine (Redis Object Cache plugin)
* **Object Storage**: MinIO Community Edition (S3-compatible)
* **Media Plugin**: WP Offload Media Lite v3.3.1

---

## 📁 Struktur Direktori
```text
wordpress-docker/
├── mysql/                    # Tempat penyimpanan persistent data kontainer MySQL
├── wordpress/                # Tempat berkas WordPress lokal (jika diperlukan mount)
├── .env                      # File konfigurasi sensitif (tidak dicommit ke Git)
├── .env.example              # File template konfigurasi untuk distribusi publik
├── .gitignore                # Mengabaikan file sensitif agar tidak terunggah ke Git
├── docker-compose.yml        # Berkas orkestrasi Docker Compose utama
├── README.md                 # Dokumentasi portofolio proyek
└── kontribusi-anggota.md     # Tabel pembagian tugas internal kelompok
```

---

## 🚀 Panduan Memulai & Instalasi

### Prasyarat
Pastikan mesin Anda memiliki dependensi berikut terpasang:
* **Docker Engine** v20.10+
* **Docker Compose** v2.0+

### Langkah-Langkah Deployment
1. **Clone Repositori**:
   ```bash
   git clone https://github.com/xafiertect/UAS_cloud_computing.git
   cd UAS_cloud_computing/wordpress-docker
   ```
2. **Setup Kredensial Lingkungan (.env)**:
   Salin template `.env.example` ke `.env` dan ganti kata sandi default dengan kata sandi yang kuat:
   ```bash
   cp .env.example .env
   ```
3. **Jalankan Orkestrasi Kontainer**:
   ```bash
   docker compose up -d
   ```
4. **Verifikasi Kontainer**:
   Pastikan seluruh layanan aktif dan berjalan:
   ```bash
   docker compose ps
   ```

---

## 🔍 Skenario Pengujian & Pengujian Kualitas (QA)
Guna menjamin keandalan sistem sebelum pengumpulan, tim QA melakukan serangkaian pengujian terstruktur:

| Kriteria Uji | Metode & Perintah Verifikasi | Status Diharapkan |
|--------------|------------------------------|--------------------|
| **Kesehatan Kontainer** | `docker compose ps` | Semua kontainer berstatus `running` & `mysql_db` berstatus `(healthy)` |
| **MySQL Healthcheck** | `docker inspect --format='{{json .State.Health.Status}}' mysql_db` | Menampilkan nilai `"healthy"` |
| **Isolasi Database** | `docker compose port db 3306` | Menghasilkan error/kosong (tidak boleh terakses dari host) |
| **Layanan Web HTTP** | `curl -I http://localhost:8080` | Respon HTTP/1.1 `302 Found` (redirect ke install wizard) |
| **Akses phpMyAdmin** | `curl -I http://localhost:8081` | Respon HTTP/1.1 `200 OK` |

---

## 🔒 Fitur Keamanan & Best Practice
* **Non-Root Database Port**: Port standard database MySQL `3306` tidak diekspos ke publik atau host eksternal untuk menghindari serangan brute-force.
* **Strict Dependencies with Healthchecks**: WordPress tidak akan dijalankan sebelum basis data MySQL dan Redis melaporkan status siap (`healthy`).
* **Kredensial Terisolasi**: Penggunaan environment variable (`.env`) dinamis mencegah bocornya data sensitif (password root) ke repositori publik.
* **Redis Auth**: Redis dikonfigurasi dengan password wajib (`requirepass`) sehingga tidak bisa diakses tanpa autentikasi.
* **MinIO Access Key**: WordPress menggunakan service account key tersendiri, bukan root credentials MinIO.

---

## 📸 Dokumentasi Milestone 2

### WordPress (localhost:8080)
> WordPress berhasil berjalan dan dapat diakses melalui browser.

![WordPress Homepage](wordpress-docker/doc%20milestone%202/localhost:8080page.png)

---

### phpMyAdmin (localhost:8081)
> phpMyAdmin terhubung ke MySQL dan menampilkan database `wordpress_db`.

![phpMyAdmin Dashboard](wordpress-docker/doc%20milestone%202/localhost:8081.png)

---

### MinIO Object Store (localhost:9003)
> MinIO berhasil berjalan dengan bucket `wordpress-media` sudah dibuat dan berstatus **PUBLIC**.

![MinIO Dashboard](wordpress-docker/doc%20milestone%202/localhost:9003.png)

---

## 🗂️ Arsitektur Milestone 2 — Service Lengkap

| Service | Container | Image | Port Host | Jaringan |
|---------|-----------|-------|-----------|----------|
| MySQL | `mysql_db` | `mysql:8.0` | — (internal) | `backend-net` |
| WordPress | `wordpress_app` | `wordpress:6.5-apache` | `8080` | `frontend-net`, `backend-net` |
| phpMyAdmin | `phpmyadmin_gui` | `phpmyadmin:5.2` | `8081` | `backend-net` |
| Redis | `wp_redis` | `redis:7-alpine` | — (internal) | `backend-net` |
| MinIO | `wp_minio` | `minio/minio:latest` | `9002` (API), `9003` (Dashboard) | `frontend-net`, `backend-net` |
