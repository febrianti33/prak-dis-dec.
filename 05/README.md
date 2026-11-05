
---


| **Nama** | Maria Widya Febrianti |
| **NIM** | 255410011 |
| **Kelas** | IF-1 |
| **Program Studi** | Informatika (Undergraduate Student) |
| **Institusi** | Universitas Teknologi Digital Indonesia (UTDI) |


# 🚀 BlackSheep API with Docker & Nginx Load Balancer

Dokumentasi ini berisi panduan alur kerja lengkap dalam membangun, menguji, dan melakukan containerisasi aplikasi **BlackSheep API** (Python) menggunakan **uv**, **Docker**, serta **Nginx** sebagai *reverse proxy* dan *load balancer*.

---

## 📋 Penjelasan Singkat Alur Proyek

Proyek ini bertujuan untuk membangun arsitektur API yang tangguh, cepat, dan terdistribusi:
1. **Framework & Environment**: Menggunakan **BlackSheep** (Python API framework berkinerja tinggi) yang dikelola dengan **`uv`** dari Astral untuk manajemen paket yang sangat cepat.
2. **Reverse Proxy & Load Balancing**: Menggunakan **Nginx** sebagai pintu masuk (*gateway*) trafik port `80` yang mendistribusikan permintaan HTTP secara bergantian ke beberapa instans aplikasi.
3. **Containerization & Scaling**: Menggunakan **Docker Compose** untuk mengorkestrasi layanan dan melakukan pengadaan *multi-container* (`bs_app` diskalakan menjadi 2 kontainer).

---

## 📸 Dokumentasi & Hasil Tangkapan Layar

### 1. Persiapan Environment & Installasi Tooling
Pembaruan paket Ubuntu, verifikasi Python 3.14, instalasi `uv`, serta Docker.

![Persiapan Environment dan Instalasi Tooling](./docs/images/01-environment-setup.png)

---

### 2. Inisialisasi BlackSheep API
Pembuatan proyek `blacksheep_lb`, setup virtual environment via `uv`, dan pengujian Swagger UI lokal (`http://localhost:44777/docs/`).

![Swagger UI BlackSheep API](./docs/images/02-swagger-ui.png)

---

### 3. Konfigurasi & Scaling Container
Eksekusi Docker Compose untuk menjalankan Nginx dan melipatgandakan instans aplikasi (`--scale bs_app=2`).

![Docker Compose Scaling](./docs/images/03-docker-scaling.png)

---

### 4. Pengujian Load Balancer & Logs
Pengujian pengiriman 10 permintaan HTTP via `curl` dan verifikasi bahwa log trafik terbagi ke kontainer `bs_app-1` dan `bs_app-2`.

![Pengujian Load Balancer dan Logs](./docs/images/04-curl-test-and-logs.png)

---

