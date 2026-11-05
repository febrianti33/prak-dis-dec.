
---

## 👤 Identitas Pengembang

* **Nama**: Maria Widya Febrianti
* **NIM**: 255410011
* **Kelas**: IF-1
* **Program Studi**: Informatika
* **Institusi**: Universitas Teknologi Digital Indonesia (UTDI)

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

## 📁 Struktur Proyek

```text
workspace-05/
├── blacksheep_lb/
│   ├── app/
│   │   ├── controllers/
│   │   │   └── examples.py
│   │   ├── docs/
│   │   ├── domain/
│   │   ├── main.py
│   │   └── services.py
│   ├── Dockerfile
│   ├── dev.py
│   ├── pyproject.toml
│   ├── requirements.txt
│   └── settings.yaml
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
└── docker-compose.yml
```

---

## 💻 Prasyarat & Lingkungan Pengembangan

* **Sistem Operasi**: Linux Ubuntu (WSL2)
* **Python**: 3.14.4
* **Package Manager**: `uv` (Astral)
* **Containerization**: Docker Engine & Docker Compose v2
* **Reverse Proxy**: Nginx

---

## 🛠️ Langkah Instalasi & Setup Detail

### 1. Update Sistem & Install Dependensi Dasar
Perbarui repositori `apt` dan install perkakas dasar pengembangan:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv curl git build-essential -y
```

Verifikasi versi Python:
```bash
python3 --version
# Output: Python 3.14.4
```

---

### 2. Install Package Manager `uv`
Gunakan `uv` dari Astral untuk manajemen *environment* dan paket Python berkecepatan tinggi:
```bash
curl -LsSf [https://astral.sh/uv/install.sh](https://astral.sh/uv/install.sh) | sh
source $HOME/.local/bin/env
```

Verifikasi versi `uv`:
```bash
uv --version
# Output: uv 0.13.7 (x86_64-unknown-linux-gnu)
```

---

### 3. Install Docker & Docker Compose
Instal Engine Docker dan plugin Docker Compose v2:
```bash
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl enable --now docker
```

Verifikasi instalasi Docker:
```bash
sudo docker --version
sudo docker compose version
sudo docker info
```

---

### 4. Setup Virtual Environment & Install BlackSheep
Buat direktori proyek `workspace-05`, lalu aktifkan virtual environment Python 3.14.4:
```bash
mkdir workspace-05
cd workspace-05

# Menginstal Python 3.14.4 via uv dan membuat virtual environment
uv python install 3.14.4
uv venv
source .venv/bin/activate

# Menginstal BlackSheep dan CLI-nya
uv pip install blacksheep
uv pip install blacksheep-cli
```

---

### 5. Inisialisasi Proyek BlackSheep
Jalankan perintah pembuatan proyek via CLI:
```bash
blacksheep create
```

**Opsi Interaktif:**
* **Project name**: `blacksheep_lb`
* **Project template**: `api`
* **Use controllers?**: `Yes`
* **Use OpenAPI Documentation?**: `Yes`
* **Library to read settings**: `essentials-configuration`
* **App settings format**: `YAML`

Jalankan dan uji coba aplikasi di server lokal:
```bash
cd blacksheep_lb
uv pip install -r requirements.txt
python dev.py
```
> **Browser Test**: Akses `http://localhost:44777/docs/` untuk membuka Swagger UI. Tekan `CTRL+C` di terminal untuk menghentikan server.

---

## 🐳 Konfigurasi Docker & Nginx

Kembali ke direktori utama (`workspace-05`) untuk menyiapkan konfigurasi Nginx dan Docker Compose.

### 1. Konfigurasi Nginx (`nginx/nginx.conf`)
File ini mengonfigurasi Nginx sebagai *reverse proxy* dan membagi beban trafik (*load balancer*) ke nama layanan `bs_app`:

```nginx
events {
    worker_connections 4096;
}

http {
    upstream backend {
        server bs_app:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

### 2. Konfigurasi Docker Compose (`docker-compose.yml`)
File orkestrasi untuk mengintegrasikan kontainer aplikasi BlackSheep dan Nginx:

```yaml
services:
  bs_app:
    build: ./blacksheep_lb
    networks:
      - default

  nginx:
    container_name: nginx
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      - bs_app
    networks:
      - default

networks:
  default:
    name: workspace-05_default
```

---

## 🧪 Eksekusi dan Pengujian Load Balancer

### 1. Build & Scale Kontainer
Jalankan Docker Compose dengan opsi `--scale` untuk membuat 2 instans dari aplikasi `bs_app`:

```bash
sudo docker compose up --build -d --scale bs_app=2
```

Cek kontainer yang sedang berjalan:
```bash
sudo docker ps
```

---

### 2. Uji Load Balancing dengan cURL
Kirimkan 10 permintaan HTTP secara berurutan ke port 80 (Nginx):

```bash
for i in {1..10}; do curl -s -o /dev/null -w "Request $i: HTTP %{http_code}\n" http://localhost/api/examples; done
```

---

### 3. Verifikasi Log Kontainer
Periksa log masing-masing instans kontainer untuk memverifikasi bahwa Nginx mendistribusikan beban secara bergantian:

```bash
# Log instans 1
sudo docker logs --tail=50 workspace-05-bs_app-1

# Log instans 2
sudo docker logs --tail=50 workspace-05-bs_app-2
```

---

### 4. Menghentikan Layanan
Untuk menghentikan dan membersihkan seluruh kontainer serta jaringan Docker:

```bash
sudo docker compose down
```

---

## 📌 Kesimpulan

Berdasarkan seluruh tahapan implementasi di atas, proyek ini berhasil membuktikan bahwa:
* Penggunaan **`uv`** dan framework **BlackSheep** mampu mempercepat proses inisialisasi pengembangan REST API berbasis Python dengan performa tinggi.
* Integrasi **Nginx** sebagai *reverse proxy* dan *load balancer* bersama **Docker Compose** berhasil mendistribusikan trafik secara merata ke beberapa instans kontainer aplikasi (`bs_app-1` dan `bs_app-2`), sehingga sistem menjadi lebih skalabel, andal, dan siap untuk deployment tingkat produksi.
