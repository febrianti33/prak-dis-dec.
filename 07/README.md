## Praktikum Sistem Terdistribusi dan Terdesentralisasi

**Nama:** Maria Widya Febrianti  
**NIM:** 255410011  
**Program Studi:** Informatika  
**Modul:** 7 – Cloud Computing  
**Dosen Penyusun:** Dr. Bambang Purnomosidi D. P.

---

# Laporan Praktikum: Kontainerisasi Aplikasi Web Flask (Flaskr) dengan Docker & Podman

Repository ini berisi laporan praktikum mengenai pengujian aplikasi web Python berbasis framework **Flask** (`flaskr`), pembuatan *container image* menggunakan **Dockerfile**, serta menjalankan dan memverifikasi kontainer aplikasi menggunakan **Docker** dan **Podman**[cite: 10].

---

## 1. Tujuan Praktikum

1. Memahami struktur proyek aplikasi web Python berbasis framework Flask (`flaskr`) dan penggunaan *virtual environment* (`.venv`).
2. Mampu melakukan instalasi paket lokal dalam mode pengembangan (`editable mode`) serta inisialisasi database aplikasi.
3. Mampu menyusun `Dockerfile` untuk membungkus aplikasi web Python beserta dependensinya ke dalam *container image*.
4. Mampu membangun (*build*) dan menjalankan (*run*) kontainer aplikasi menggunakan Docker maupun Podman dengan pemetaan port (*port forwarding*).
5. Mampu melakukan pengecekan status, proses internal, jaringan, dan pengujian HTTP pada kontainer yang berjalan.

---

## 2. Dasar Teori

### 2.1 Flask Web Framework
**Flask** adalah *micro web framework* berbahasa Python yang ringan dan fleksibel. Pada praktikum ini digunakan proyek tutorial resmi **Flaskr**, yaitu aplikasi blog sederhana yang memiliki fitur autentikasi pengguna (registrasi, login, logout) dan manajemen postingan menggunakan database SQLite.

### 2.2 Kontainerisasi (Docker & Podman)
- **Container Image**: Paket perangkat lunak mandiri yang ringan dan dapat dieksekusi, mencakup kode aplikasi, *runtime* (Python 3.14), pustaka sistem, dan pengaturan yang dibutuhkan untuk menjalankan aplikasi.
- **Docker**: Platform kontainerisasi populer yang menggunakan arsitektur *client-server* (Docker daemon) untuk membangun dan menjalankan kontainer.
- **Podman**: *Container engine* alternatif yang kompatibel dengan standar OCI (Open Container Initiative) dan perintah CLI Docker, namun berjalan tanpa arsitektur *daemon* terpusat (*daemonless*).

---

## 3. Langkah Praktikum

### Tahap 1: Persiapan Lingkungan & Uji Coba Aplikasi Flask Secara Lokal

Langkah awal dimulai dengan memeriksa versi Python, melihat struktur direktori proyek `flask-app`, dan mengaktifkan lingkungan virtual (`.venv`):

python --version
ls -la
source .venv/bin/activate
python --version

<img width="940" height="411" alt="image" src="https://github.com/user-attachments/assets/294e3102-ea13-4b0f-ab0f-99161c692969" />

*Gambar 1: Pemeriksaan versi Python 3.14.4, struktur proyek flask-app, dan aktivasi virtual environment (.venv).*

**Penjelasan Gambar 1:**
Direktori `~/src/flask-app` berisi kode sumber paket `flaskr`, direktori `instance`, `tests`, serta konfigurasi proyek `pyproject.toml`. Lingkungan virtual `.venv` diaktifkan dengan Python versi 3.14.4.

Selanjutnya, menginstal proyek `flaskr` ke dalam *virtual environment*, melakukan inisialisasi database SQLite, dan menjalankan server pengembangan Flask secara lokal:

uv pip install -e .
flask --app flaskr init-db
flask --app flaskr run

<img width="940" height="317" alt="image" src="https://github.com/user-attachments/assets/56140c74-e2f3-4de6-aac3-d4849d0f2d27" />

*Gambar 2: Instalasi paket flaskr dengan uv, inisialisasi database, dan menjalankan server Flask pada port 5000.*

**Penjelasan Gambar 2:**
Perintah `uv pip install -e .` memasang paket `flaskr` beserta dependensinya (`flask`, `jinja2`, `werkzeug`, `click`). Perintah `init-db` membuat tabel database awal, dan `flask run` menjalankan aplikasi di alamat lokal `http://127.0.0.1:5000`.

Menguji tampilan halaman utama aplikasi Flaskr melalui web browser:
<img width="940" height="167" alt="image" src="https://github.com/user-attachments/assets/6592432b-b106-4961-87b2-fed76a78925e" />

*Gambar 3: Tampilan halaman utama Flaskr (Posts) pada browser di alamat 127.0.0.1:5000.*

**Penjelasan Gambar 3:**
Halaman indeks aplikasi berhasil dimuat di browser, menampilkan judul **Flaskr**, daftar **Posts**, serta menu navigasi **Register** dan **Log In** di pojok kanan atas.

Menguji fitur autentikasi pada halaman `/auth/login` dengan memasukkan kredensial yang salah dan yang benar:

<img width="940" height="324" alt="image" src="https://github.com/user-attachments/assets/1014c543-822f-48f8-8638-af3b9caf1198" />

*Gambar 4: Pesan validasi "Incorrect password." saat percobaan login dengan kata sandi yang salah.*

<img width="940" height="239" alt="image" src="https://github.com/user-attachments/assets/2f3984ba-5549-4675-82ae-f03cbc42d0d7" />

*Gambar 5: Tampilan halaman utama setelah berhasil login sebagai pengguna "widi" dengan opsi membuat postingan (New).*

**Penjelasan Gambar 4 & 5:**
Aplikasi memvalidasi input pengguna dengan baik—menampilkan peringatan *Incorrect password* saat sandi keliru (Gambar 4), dan mengarahkan pengguna `widi` ke halaman utama dengan menu **New** dan **Log Out** saat login berhasil (Gambar 5).

---

### Tahap 2: Pembuatan Dockerfile & Deployment dengan Docker

Membuat berkas `Dockerfile` di dalam direktori proyek `~/src/flask-app` untuk mendefinisikan langkah pembuatan *container image*:

FROM python:3.14

RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -e .
RUN flask --app flaskr init-db

EXPOSE 5000
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]

<img width="940" height="228" alt="image" src="https://github.com/user-attachments/assets/36ee917e-0291-46fd-b00a-0dce72f192bb" />

Gambar 6: Isi berkas Dockerfile berbasis image python:3.14 untuk membungkus aplikasi Flaskr.

*Penjelasan Gambar 6:*
Dockerfile menggunakan base image python:3.14, menyalin seluruh kode ke folder /app, menginstal dependensi, menginisialisasi database di dalam image, membuka port 5000, dan mengatur perintah utama (CMD) agar Flask mendengarkan koneksi dari semua antarmuka jaringan (--host=0.0.0.0).

Membangun Docker image dengan tag flaskr:1.0.0 dan memeriksa daftar image yang tersedia:

docker build -t flaskr:1.0.0 .
docker images

<img width="940" height="311" alt="image" src="https://github.com/user-attachments/assets/35cae93b-67be-4f6d-8bc4-a364008c6969" />

Gambar 7: Log tahapan akhir proses build Dockerfile hingga berhasil membuat image flaskr:1.0.0.

<img width="940" height="121" alt="image" src="https://github.com/user-attachments/assets/add27cdd-0d5b-420f-b0e1-0654334c3dec" />

Gambar 8: Daftar Docker image lokal yang menampilkan flaskr:1.0.0 beserta ukuran disk dan kontennya.

*Penjelasan Gambar 7 & 8:*
Seluruh 8 langkah pada Dockerfile berhasil dieksekusi hingga menghasilkan image berlabel flaskr:1.0.0 dengan ID 7fd598056c02 (ukuran konten 424 MB) yang tercatat di daftar docker images.

Menjalankan kontainer dari image flaskr:1.0.0 dengan memetakan port 5001 di host ke port 5000 di dalam kontainer, lalu mengaksesnya melalui browser:

docker run -p 5001:5000 flaskr:1.0.0

<img width="940" height="316" alt="image" src="https://github.com/user-attachments/assets/34efa74a-5c3c-48c0-adb4-6d48563b5001" />

Gambar 9: Eksekusi docker run -p 5001:5000 flaskr:1.0.0 dan log request HTTP 200 yang masuk ke kontainer.

<img width="940" height="153" alt="image" src="https://github.com/user-attachments/assets/a4f6770a-ec77-4812-9db3-fdb8c73fa98d" />

Gambar 10: Tampilan aplikasi web Flaskr yang berjalan di dalam kontainer Docker pada alamat localhost:5001.

*Penjelasan Gambar 9 & 10:*
Parameter -p 5001:5000 meneruskan trafik dari localhost:5001 ke port internal 5000 milik kontainer. Browser berhasil memuat aplikasi Flaskr secara normal dan terminal kontainer mencatat kode status HTTP 200.

---

### Tahap 3: Pengujian & Inspeksi Kontainer Menggunakan Podman

Menguji kontainer flaskr:1.0.0 yang berjalan pada engine *Podman*, memeriksa pemetaan port, dan melakukan pengujian koneksi menggunakan curl:

podman ps
podman port -a
curl -v http://localhost:5001
podman logs --tail 100 d6e81ccce913

<img width="940" height="487" alt="image" src="https://github.com/user-attachments/assets/13118580-5c20-42ce-b3b5-544f10ee8ad6" />

Gambar 11: Pengecekan kontainer aktif dengan podman ps, pemetaan port, pengujian curl IPv6, dan pemeriksaan log kontainer.

*Penjelasan Gambar 11:*
Kontainer dengan ID d6e81ccce913 terpantau aktif (Up 3 minutes) dengan pemetaan port 0.0.0.0:5001->5000/tcp. Saat diakses menggunakan curl http://localhost:5001, sistem mencoba resolving ke alamat IPv6 ([::1]:5001) yang menghasilkan Connection reset by peer, sementara log kontainer mengonfirmasi server Flask mendengarkan pada alamat IPv4 (127.0.0.1 dan 172.24.112.139).

Memeriksa proses internal yang berjalan di dalam kontainer menggunakan podman top dan melakukan pengujian ulang menggunakan koneksi IPv4 secara eksplisit (curl -4):

podman top d6e81ccce913
curl -4 -v http://127.0.0.1:5001

<img width="940" height="455" alt="image" src="https://github.com/user-attachments/assets/3986f34c-d3a0-4fe0-9a69-01eea7be70f0" />

Gambar 12: Output podman top menampilkan proses Python Flask (PID 1) dan respons HTTP/1.1 200 OK saat diakses melalui IPv4 127.0.0.1:5001.

*Penjelasan Gambar 12:*
podman top memverifikasi bahwa proses utama di dalam kontainer (PID 1) adalah server Python Flask. Pengujian dengan curl -4 -v http://127.0.0.1:5001 berhasil terhubung melalui IPv4 dan mengembalikan respons lengkap *HTTP/1.1 200 OK* beserta struktur kode HTML halaman utama Flaskr.

Memeriksa konfigurasi jaringan (alamat IP internal) kontainer menggunakan podman inspect dan memverifikasi kembali respons HTML dengan curl -4:

podman inspect d6e81ccce913 --format '{{.NetworkSettings.IPAddress}}'
curl -4 http://127.0.0.1:5001

<img width="940" height="464" alt="image" src="https://github.com/user-attachments/assets/c55cdd34-7be7-425f-9c29-c58422cf5496" />

Gambar 13: Pemeriksaan IPAddress kontainer dengan podman inspect dan verifikasi output HTML dari 127.0.0.1:5001.

<img width="940" height="260" alt="image" src="https://github.com/user-attachments/assets/8a9c701f-cdb9-46a8-8987-1cbee2c5efd5" />

Gambar 14: Kelanjutan struktur HTML halaman Posts Flaskr yang diterima melalui curl IPv4.

*Penjelasan Gambar 13 & 14:*
Perintah podman inspect memeriksa pengaturan jaringan kontainer (kosong karena menggunakan mode jaringan rootless/slirp4netns default), dan curl -4 menampilkan keseluruhan elemen HTML dari bagian <head>, navigasi <nav>, hingga bagian <section class="content">.

Memastikan kembali aplikasi kontainer Podman dapat diakses secara visual melalui web browser:


Gambar 15: Tampilan halaman web Flaskr pada browser di alamat 127.0.0.1:5001 yang dilayani oleh kontainer Podman.

*Penjelasan Gambar 15:*
Pengujian akhir melalui browser pada http://127.0.0.1:5001 berjalan lancar dan menampilkan antarmuka web Flaskr secara utuh.

---

## 4. Kesimpulan

1. *Pengembangan & Pengujian Lokal: Aplikasi web Python berbasis Flask (flaskr) berhasil dikonfigurasi di dalam *virtual environment (.venv), meliputi instalasi paket lokal, inisialisasi database SQLite, serta pengujian fitur autentikasi pengguna.
2. *Kontainerisasi dengan Docker: Melalui pembuatan Dockerfile berbasis python:3.14, aplikasi berhasil dibungkus menjadi *container image mandiri (flaskr:1.0.0) dan dijalankan melalui pemetaan port (5001:5000).
3. *Portabilitas & Inspeksi dengan Podman: *Image aplikasi yang sama terbukti portabel dan dapat dijalankan di atas engine Podman. Pengujian konektivitas membuktikan bahwa pengalamatan IPv4 (127.0.0.1:5001) berhasil merutekan trafik dengan baik ke server Flask di dalam kontainer dengan status respons 200 OK.
