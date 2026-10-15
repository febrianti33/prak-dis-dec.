# LAPORAN PRAKTIKUM SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI
## Modul 2: Komunikasi Antar Proses pada Sistem Terdistribusi

- **Nama:** Maria Widya Febrianti
- **NIM:** 255410011
- **Kelas:** IF-1
- **Program Studi:** Informatika
- **Universitas:** Universitas Teknologi Digital Indonesia (UTDI) Yogyakarta

---

## I. PENDAHULUAN
Proses merupakan hasil dari eksekusi program atau aplikasi yang bersifat *executable*, dikelola oleh sistem operasi, serta terdiri atas *executable code*, *data*, *resources*, dan informasi mengenai *state* (seperti *stack* dan *heap*). Pada satu node tunggal, seluruh pengelolaan proses, alokasi sumber daya, hingga komunikasi antar proses sepenuhnya dikendalikan oleh sistem operasi secara transparan terhadap pengguna.   

Sebaliknya, komunikasi antar proses pada sistem terdistribusi melibatkan lebih dari satu node yang tidak berada pada *clock* yang sama serta tidak memungkinkan penggunaan *shared memory* demi alasan keamanan. Oleh karena itu, dibutuhkan mekanisme khusus seperti penggunaan arsitektur **GraphQL** untuk melayani *query* dari *client* ke *server*.

---

## II. PRAKTIKUM BAGIAN I: PROSES PADA SATU NODE
Pada bagian ini, praktikum dilakukan untuk mengamati dan mengelola proses pada sistem operasi:
1. **Menampilkan Proses:** Proses yang berjalan pada sistem operasi diamati menggunakan Task Manager (pada Windows) maupun utilitas `htop` (pada lingkungan Linux/WSL).   
2. **Memantau Aplikasi:** Aplikasi yang sedang aktif dipantau untuk melihat penggunaan sumber daya seperti CPU dan memori secara *real-time*.
3. **Pengelolaan Proses:** Dilakukan uji coba untuk mematikan proses dari aplikasi secara langsung melalui pengelola proses (*process manager*), bukan melalui tombol keluar normal dari aplikasi tersebut.
   
<img width="933" height="663" alt="foto1" src="https://github.com/user-attachments/assets/81ce9b82-db94-4528-ad21-9860d244d9d6" />

---

## III. PRAKTIKUM BAGIAN II: KOMUNIKASI ANTAR PROSES MENGGUNAKAN GRAPHQL
Bagian ini berfokus pada pembuatan *server* GraphQL menggunakan bahasa pemrograman Python dengan bantuan pustaka **Strawberry**.   

### 1. Persiapan Workspace dan Instalasi
- Membuat *workspace* baru dengan nama `workspace-01` menggunakan versi Python terbaru.
  <img width="502" height="48" alt="image" src="https://github.com/user-attachments/assets/64d327c1-4881-4b17-85e2-7c2c9aef4aac" />

- Membuat dan mengaktifkan *virtual environment* di dalam direktori proyek.
- Menginstal paket `strawberry-graphql` beserta *command-line tools* menggunakan manajer paket `uv`:
  ```bash
  uv pip install 'strawberry-graphql [cli]'


  2. Implementasi Schema Server (schema.py)
File schema dibuat dengan mendefinisikan tipe data menggunakan dekorator @strawberry.type serta kelas Query untuk menangani permintaan data buku:

Python
import strawberry
from typing import List

@strawberry.type
class Book:
    title: str
    author: str

@strawberry.type
class Query:
    @strawberry.field
    def books(self) -> List[Book]:
        return [
            Book(title="The Great Gatsby", author="F. Scott Fitzgerald"),
        ]

schema = strawberry.Schema(query=Query)
3. Menjalankan Server dan Pengujian via Browser
Server dijalankan melalui terminal dengan perintah:

Bash
strawberry dev schema
Server aktif dan dapat diakses pada alamat http://localhost:8000/graphql.

Antarmuka GraphQL Playground digunakan untuk mengirimkan query pengujian:

GraphQL
books {
  title
  author
}
Hasil respons data diterima oleh client dalam format JSON dengan sukses.

Server kemudian dimatikan dengan menekan kombinasi tombol <Ctrl-C> pada terminal.

IV. TUGAS MANDIRI: PEMBUATAN PROGRAM CLIENT
Sebagai bagian dari tugas mandiri untuk menghubungkan client dan server pada sistem terdistribusi, dibuat sebuah program client menggunakan bahasa Python dengan pustaka requests untuk melakukan permintaan data secara otomatis ke server GraphQL.

Kode Program Client (client.py)
Python
import requests

# URL endpoint GraphQL server
url = "http://localhost:8000/graphql"

# Query GraphQL untuk mengambil data buku
query = """
    query {
      books {
        title
        author
      }
    }
"""

# Mengirimkan HTTP POST request ke server GraphQL
response = requests.post(url, json={"query": query})

# Menampilkan hasil respons dari server
if response.status_code == 200:
    result = response.json()
    print("Data dari Server GraphQL:")
    print(result)
else:
    print(f"Gagal terhubung. Status code: {response.status_code}")

    
V. KESIMPULAN

Praktikum ini memberikan pemahaman mendalam mengenai perbedaan pengelolaan proses pada sistem node tunggal yang dikendalikan penuh oleh sistem operasi, serta komunikasi antar proses pada sistem terdistribusi yang memerlukan mekanisme khusus seperti GraphQL. Melalui penggunaan Python dan Strawberry, pembuatan server serta integrasinya dengan program client dapat dilakukan secara efisien, terstruktur, dan transparan.
