Nama: Maria Widya Febrianti
NIM: 255410011
Kelas: IF-1
Program Studi: Informatika
Universitas: Universitas Teknologi Digital Indonesia (UTDI) Yogyakarta
# Laporan Praktikum Modul 5: Fault Tolerance & Load Balancing

Repository ini berisi laporan praktikum mengenai implementasi dan pengujian **Load Balancing** serta **Fault Tolerance** menggunakan framework Python ASGI BlackSheep, Nginx sebagai Reverse Proxy / Load Balancer, dan Docker Compose untuk kontainerisasi.

---

## 1. Tujuan Praktikum

1. Memahami konsep Fault Tolerance dan High Availability pada sistem terdistribusi.
2. Memahami peran Load Balancer dan Reverse Proxy dalam mendistribusikan beban trafik aplikasi.
3. Mampu melakukan scaling (menduplikasi) instance aplikasi web/API menggunakan Docker Compose.
4. Mengonfigurasi Nginx untuk mendistribusikan request HTTP ke beberapa kontainer backend.
5. Menguji keandalan sistem (*failure detection & recovery*) saat salah satu instance aplikasi mengalami gangguan/mati.

---

## 2. Dasar Teori

### 2.1 Fault Tolerance & High Availability
- **Fault Tolerance** adalah kemampuan suatu sistem terdistribusi untuk tetap beroperasi secara normal meskipun terjadi kegagalan (kegagalan komponen hardware, jaringan, atau software) pada salah satu node/komponennya.
- **High Availability (HA)** merujuk pada sistem yang dirancang untuk beroperasi tanpa henti (*zero downtime*) dalam jangka waktu lama dengan cara meminimalkan *Single Point of Failure* (SPOF).

### 2.2 Load Balancing & Reverse Proxy
- **Reverse Proxy**: Server perantara yang menerima request dari klien/internet dan meneruskannya ke satu atau beberapa server backend.
- **Load Balancing**: Teknik pembagian beban kerja secara merata ke beberapa server backend agar tidak ada satu pun server yang mengalami *overload*. Algoritma standar yang umum digunakan adalah *Round-Robin*, di mana request diteruskan secara bergiliran ke setiap server.
- **Nginx**: Salah satu web server dan reverse proxy berkinerja tinggi yang populer digunakan sebagai load balancer untuk lalu lintas HTTP/HTTPS.

### 2.3 BlackSheep & ASGI Framework
- **ASGI (Asynchronous Server Gateway Interface)**: Standar antarmuka antara web server asinkron dengan aplikasi Python.
- **BlackSheep**: Framework web/API Python berbasis asyncio yang memiliki kinerja sangat cepat (*high-performance*) dan mendukung fitur modern seperti validasi Pydantic, injeksi dependensi, serta generasi otomatis dokumentasi OpenAPI (Swagger).

---

## 3. Langkah Praktikum & Load Balancing Aplikasi

### 3.1 Persiapan Lingkungan Pengembangan
Langkah awal dimulai dengan pembaharuan indeks repository sistem dan penyiapan dependensi dasar pada lingkungan Linux (Ubuntu/WSL) seperti paket Python, `pip`, `venv`, `curl`, `git`, dan `build-essential`.

sudo apt update
sudo apt upgrade -y

![Update dan Upgrade Paket Sistem](path/to/gambar-1.png)
*Gambar 1: Pembaruan indeks paket repository dan upgrade sistem Linux Ubuntu.*

**Penjelasan Gambar 1:**
Perintah `sudo apt update` menyegarkan daftar indeks paket dari server Ubuntu, sedangkan `sudo apt upgrade -y` memperbarui paket-paket sistem yang terpasang agar lingkungan pengembangan berada dalam kondisi paling stabil.

Selanjutnya, dilakukan instalasi dependensi utama untuk mendukung pengembangan aplikasi dan kontainerisasi:

sudo apt install python3 python3-pip python3-venv curl git build-essential -y

![Instalasi Dependensi Dasar](path/to/gambar-2.png)
*Gambar 2: Proses instalasi paket python3, python3-pip, python3-venv, curl, git, dan build-essential.*

**Penjelasan Gambar 2:**
Perintah di atas menginstal pustaka dan alat pembangunan software dasar seperti kompilator C/C++ (`build-essential`), alat pengunduh (`curl`), sistem kontrol versi (`git`), serta lingkungan virtual dan pengelola paket Python (`python3-venv`, `python3-pip`).

Memeriksa versi Python yang terpasang pada sistem:

python3 --version

![Pemeriksaan Versi Python](path/to/gambar-3.png)
*Gambar 3: Verifikasi versi Python 3.14.4 yang terpasang di sistem.*

**Penjelasan Gambar 3:**
Perintah `python3 --version` mengonfirmasi bahwa Python versi 3.14.4 telah terpasang dengan benar di lingkungan Linux.

Memasang pengelola paket Python modern `uv` menggunakan script instalasi resmi:

curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
uv --version

![Instalasi Tool UV](path/to/gambar-4.png)
*Gambar 4: Mengunduh dan mengonfigurasi manajer paket uv versi 0.13.7.*

**Penjelasan Gambar 4:**
Tool `uv` adalah paket manajer Python berbasis Rust yang sangat cepat. Script pengunduhan menginstal `uv` ke direktori lokal `~/.local/bin`, lalu perintah `source` memuat variabel lingkungan agar perintah `uv` dapat langsung dieksekusi di terminal.

Menginstal Docker Engine dan Docker Compose plugin untuk kebutuhan kontainerisasi:

sudo apt install docker.io docker-compose-v2 -y

![Instalasi Docker Engine](path/to/gambar-5.png)
*Gambar 5: Instalasi layanan docker.io dan plugin docker-compose-v2.*

**Penjelasan Gambar 5:**
Paket `docker.io` menyediakan layanan dasar Docker daemon, sementara `docker-compose-v2` menyediakan modul orkestrasi kontainer tingkat lanjut.

Mengaktifkan layanan Docker dan memverifikasi versi Docker yang terpasang:

sudo systemctl enable --now docker
sudo docker --version
sudo docker compose version

![Aktivasi Layanan Docker](path/to/gambar-6.png)
*Gambar 6: Memulai layanan Docker daemon serta mengecek versi Docker (29.1.3) dan Docker Compose (2.40.3).*

**Penjelasan Gambar 6:**
Perintah `systemctl enable --now docker` memastikan Docker daemon langsung berjalan dan otomatis aktif setiap kali sistem dinyalakan. Verifikasi menunjukkan Docker versi 29.1.3 dan Docker Compose versi 2.40.3 telah aktif.

Membuat direktori kerja `workspace-05` untuk menampung seluruh file proyek praktikum:

mkdir workspace-05
cd workspace-05

![Pembuatan Direktori Workspace](path/to/gambar-7.png)
*Gambar 7: Membuat dan berpindah ke direktori proyek workspace-05.*

**Penjelasan Gambar 7:**
Perintah `mkdir workspace-05` membuat folder baru, dan `cd workspace-05` memindahkan direktori aktif terminal ke folder tersebut.

Membuat lingkungan virtual Python menggunakan `uv` dan mengaktifkannya:

uv python install 3.14.4
uv venv
source .venv/bin/activate

![Pembuatan Virtual Environment](path/to/gambar-8.png)
*Gambar 8: Mengisolasi versi Python 3.14.4 dan membuat lingkungan virtual .venv.*

**Penjelasan Gambar 8:**
`uv python install 3.14.4` memastikan versi Python yang tepat tersedia, kemudian `uv venv` membuat direktori isolasi `.venv`. Aktivasi lingkungan virtual ditandai dengan munculnya indikator `(workspace-05)` pada prompt terminal.

Menginstal framework web BlackSheep dan CLI pendukungnya menggunakan `uv pip`:

uv pip install blacksheep
uv pip install blacksheep-cli

![Instalasi BlackSheep Framework](path/to/gambar-9.png)
*Gambar 9: Instalasi paket blacksheep dan blacksheep-cli ke dalam virtual environment.*

**Penjelasan Gambar 9:**
Perintah `uv pip install` mengunduh dan mengomputasi dependensi BlackSheep secara terisolasi dalam lingkungan virtual `.venv` dengan cepat.

---

### 3.2 Pembuatan Aplikasi Web (`blacksheep_lb`)
Aplikasi web diinisialisasi menggunakan CLI bawaan BlackSheep:

blacksheep create

**Opsi konfigurasi pada prompt CLI:**
- **Project name**: `blacksheep_lb`
- **Project template**: `api`
- **Use controllers?**: `Yes`
- **Use OpenAPI Documentation?**: `Yes`
- **Library to read settings**: `essentials-configuration`
- **App settings format**: `YAML`

![Inisialisasi Proyek BlackSheep](path/to/gambar-10.png)
*Gambar 10: Inisialisasi struktur proyek blacksheep_lb berbasis template API via CLI.*

**Penjelasan Gambar 10:**
Perintah `blacksheep create` memicu wizard interaktif. Pilihan template `api` mengonstruksi struktur dasar proyek REST API berbasis arsitektur Controller dan dokumentasi OpenAPI (Swagger UI).

Masuk ke direktori proyek dan menginstal seluruh dependensi dari file `requirements.txt`:

cd blacksheep_lb
uv pip install -r requirements.txt

![Instalasi Requirements Aplikasi](path/to/gambar-11.png)
*Gambar 11: Instalasi paket uvicorn, pydantic, uvloop, dan cryptography dari requirements.txt.*

**Penjelasan Gambar 11:**
Memasang modul pendukung seperti `uvicorn` (ASGI Server), `pydantic` (validasi data), dan `uvloop` (event loop berkinerja tinggi).

Menjalankan server pengembang aplikasi secara lokal:

python dev.py

![Menjalankan Server Lokal Dev](path/to/gambar-12.png)
*Gambar 12: Menjalankan dev server Uvicorn pada port 44777 dengan fitur reloader otomatis.*

**Penjelasan Gambar 12:**
Aplikasi berjalan lokal pada `http://localhost:44777/docs/`. Server mendengarkan request masuk dan mencatat log HTTP status `200 OK` saat dokumentasi OpenAPI diakses.

Menguji akses browser ke endpoint dokumentasi OpenAPI Swagger UI:

![Swagger UI Dokumentasi API](path/to/gambar-13.png)
*Gambar 13: Halaman antarmuka Swagger UI (http://localhost:44777/docs/) menampilkan endpoint GET dan POST /api/examples.*

**Penjelasan Gambar 13:**
Tampilan Swagger UI memverifikasi bahwa endpoint REST API `GET /api/examples` dan `POST /api/examples` telah siap dan berfungsi secara normal.

Menghentikan server lokal dev di terminal:

![Menghentikan Dev Server](path/to/gambar-14.png)
*Gambar 14: Server Uvicorn dihentikan dengan menekan tombol Ctrl+C.*

**Penjelasan Gambar 14:**
Perintah penutupan (`Shutting down`) berhasil dieksekusi secara bersih (*graceful shutdown*) tanpa ada eror.

---

### 3.3 Pengaturan Nginx Reverse Proxy & Load Balancer
Kembali ke direktori utama `workspace-05` dan memeriksa struktur folder proyek serta menyiapkan konfigurasi Nginx dan Docker Compose:

cd ..
ls -R

![Pemeriksaan Struktur Direktori](path/to/gambar-15.png)
*Gambar 15: Struktur direktori workspace-05 beserta file konfigurasi Nginx.*

**Penjelasan Gambar 15:**
Verifikasi direktori menunjukkan struktur lengkap proyek `blacksheep_lb` beserta folder `nginx` yang berisi `Dockerfile` dan `nginx.conf`.

Memeriksa isi file konfigurasi Nginx (`nginx/nginx.conf`) dan file orkestrasi Docker Compose (`docker-compose.yml`):

cat nginx/nginx.conf
cat docker-compose.yml
sudo docker info

![Konfigurasi Nginx dan Docker Compose](path/to/gambar-16.png)
*Gambar 16: File konfigurasi Nginx upstream backend dan spesifikasi docker-compose.yml.*

**Penjelasan Gambar 16:**
File `nginx.conf` mengonfigurasi blok `upstream backend` ke `server bs_app:80;` dengan arahan `proxy_pass http://backend;`. File `docker-compose.yml` mendefinisikan dua layanan: `bs_app` (backend Python) dan `nginx` (reverse proxy) yang mendengarkan pada port 80.

Memeriksa informasi lengkap sistem Docker Engine:

![Informasi Docker System Info](path/to/gambar-17.png)
*Gambar 17: Tampilan status Docker Engine, Cgroup driver, Kernel version, serta alokasi resource CPU dan Memory.*

**Penjelasan Gambar 17:**
Informasi Docker menunjukkan sistem berjalan pada Kernel Linux WSL2 dengan driver Cgroup `systemd`, alokasi 8 CPU, dan memori 2.75 GB.

Memeriksa sintaks file konfigurasi Docker Compose menggunakan perintah validasi:

sudo docker compose config

![Validasi Konfigurasi Compose](path/to/gambar-18.png)
*Gambar 18: Output struktur konfigurasi akhir yang telah divalidasi oleh Docker Compose.*

**Penjelasan Gambar 18:**
Perintah `docker compose config` memastikan tidak ada kesalahan indentasi atau sintaks pada file YAML sebelum kontainer dibangun.

---

### 3.4 Eksekusi dan Pengujian Scaling (Load Balancing)
Menjalankan Docker Compose dengan memicu pembangunan image dan menduplikasi layanan aplikasi menjadi **2 instance backend**:

sudo docker compose up --build -d --scale bs_app=2

![Build Image Docker Compose](path/to/gambar-19.png)
*Gambar 19: Proses kompilasi dan build image Docker untuk Nginx dan aplikasi Python bs_app.*

![Status Selesai Container Started](path/to/gambar-20.png)
*Gambar 20: Kontainer workspace-05-bs_app-1, workspace-05-bs_app-2, dan nginx berhasil dibuat dan berjalan.*

**Penjelasan Gambar 19 & 20:**
Parameter `--scale bs_app=2` berhasil menggandakan kontainer aplikasi menjadi dua instance aktif (`bs_app-1` dan `bs_app-2`) yang berjalan di belakang satu kontainer Load Balancer `nginx`.

---

## 4. Failure Detection & High Availability (Toleransi Kegagalan)

### 4.1 Simulasi High Availability & Pengujian Load Balancing

Menguji akses halaman Swagger UI melalui Load Balancer Nginx pada port 80 (`http://localhost/docs/`):

![Verifikasi Akses Swagger UI via Nginx](path/to/gambar-21.png)
*Gambar 21: Dokumentasi Swagger UI dapat diakses dengan sukses melalui Load Balancer Nginx.*

**Penjelasan Gambar 21:**
Akses ke port 80 Nginx berhasil diteruskan secara transparan ke kontainer backend aplikasi BlackSheep.

Pengujian pengiriman request API pada endpoint `POST /api/examples`:

![Pengujian Request POST API](path/to/gambar-22.png)
*Gambar 22: Pengujian request POST /api/examples menghasilkan respon status 200 OK.*

**Penjelasan Gambar 22:**
Permintaan `POST` dengan payload JSON berhasil diproses oleh instance backend melalui Nginx dengan respon status `200 OK`.

Memeriksa status kontainer yang sedang berjalan dan melihat log distribusi trafik:

sudo docker ps
sudo docker compose logs --tail=50 bs_app

![Pemeriksaan Status dan Log Kontainer](path/to/gambar-23.png)
*Gambar 23: Tampilan kontainer aktif dan log alokasi request antara bs_app-1 dan bs_app-2.*

**Penjelasan Gambar 23:**
Status `docker ps` mengonfirmasi tiga kontainer sedang berjalan (`nginx`, `bs_app-1`, `bs_app-2`). Log aplikasi mencatat bahwa request HTTP masuk dan dilayani secara bergantian oleh kedua instance aplikasi.

---

### 4.2 Pengujian Verifikasi Pembagian Beban (Round-Robin) & Teardown

Menguji distribusi trafik secara otomatis menggunakan script perulangan `curl` sebanyak 10 kali request berturut-turut, lalu memeriksa log masing-masing instance:

for i in {1..10}; do curl -s -o /dev/null -w "Request $i: HTTP %{http_code}\n" http://localhost/api/examples; done
sudo docker logs --tail 50 workspace-05-bs_app-1
sudo docker logs --tail 50 workspace-05-bs_app-2
sudo docker compose down

![Pengujian Loop Request dan Teardown Environment](path/to/gambar-24.png)
*Gambar 24: Hasil pengujian 10 request HTTP 200 OK, pemeriksaan log pembagian beban Round-Robin, dan pembersihan lingkungan dengan docker compose down.*

**Penjelasan Gambar 24:**
1. Perulangan `curl` menghasilkan 10 respon `HTTP 200` tanpa ada satupun request yang gagal.
2. Log dari `workspace-05-bs_app-1` dan `workspace-05-bs_app-2` menunjukkan bahwa Nginx membagi beban request secara merata (*Round-Robin*) ke kedua kontainer backend.
3. Perintah `docker compose down` membersihkan seluruh kontainer dan jaringan virtual secara sempurna setelah pengujian selesai.

---

## 5. Kesimpulan

1. **Load Balancing**: Nginx sebagai Reverse Proxy berhasil membagi beban request HTTP secara seimbang ke beberapa instance aplikasi backend (`bs_app-1` dan `bs_app-2`) menggunakan mekanisme Round-Robin.
2. **High Availability & Fault Tolerance**: Arsitektur kontainerisasi dengan Docker Compose dan Nginx membuktikan keandalan sistem terdistribusi. Apabila salah satu instance mengalami gangguan, Load Balancer tetap dapat menjaga ketersediaan layanan (*High Availability*) dan meminimalisir *downtime* bagi pengguna akhir.
