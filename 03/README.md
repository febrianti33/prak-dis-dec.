LAPORAN PRAKTIKUM
SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI

MODUL 3
SINKRONISASI PADA SISTEM TERDISTRIBUSI

Disusun untuk memenuhi tugas praktikum

Disusun oleh:
Maria Widya Febrianti
NIM: 255410011
Kelas: IF-1

PROGRAM STUDI INFORMATIKA
UNIVERSITAS TEKNOLOGI DIGITAL INDONESIA
2026
BAB I PENDAHULUAN
1.1 Latar Belakang
Sistem terdistribusi terdiri dari beberapa komputer atau proses yang berkomunikasi melalui jaringan. Setiap proses dapat berjalan pada waktu dan urutan yang berbeda sehingga dibutuhkan mekanisme sinkronisasi. Sinkronisasi membantu mengatur urutan kejadian, melindungi data bersama, mencegah proses saling menunggu, dan mengoordinasikan node.
1.2 Tujuan Praktikum
- Memahami sinkronisasi waktu menggunakan NTP.
- Memahami Vector Clock untuk mencatat urutan kejadian.
- Mengamati variasi keluaran program multithreading.
- Memahami race condition dan penggunaan lock.
- Memahami penyebab deadlock dan cara menghindarinya.
- Memahami pemilihan leader menggunakan algoritma Raft.
BAB II DASAR TEORI
2.1 Sinkronisasi Waktu
Sinkronisasi waktu adalah proses menyesuaikan waktu komputer dengan sumber waktu acuan. NTP (Network Time Protocol) digunakan untuk membantu menyelaraskan waktu perangkat. Modul juga menyebut PTP (Precision Time Protocol) sebagai protokol sinkronisasi waktu.
2.2 Vector Clock
Vector Clock mencatat kejadian dengan vektor bilangan. Kejadian lokal menaikkan komponen proses yang bersangkutan. Saat pesan diterima, proses membandingkan vektor lokal dan vektor pesan, mengambil nilai maksimum setiap komponen, lalu menaikkan komponen penerima.
2.3 Race Condition
Race condition terjadi ketika hasil program bergantung pada urutan beberapa thread yang mengakses data bersama. Tanpa perlindungan, pembaruan dari satu thread dapat tertimpa thread lain. Lock dapat melindungi bagian kritis agar hanya satu thread menjalankannya pada satu waktu.
2.4 Deadlock
Deadlock adalah kondisi ketika thread saling menunggu sumber daya yang dikunci thread lain sehingga tidak dapat melanjutkan pekerjaan. Pengambilan lock dengan urutan konsisten merupakan salah satu cara mengurangi risiko deadlock.
2.5 Algoritma Raft
Raft merupakan algoritma konsensus yang menggunakan peran follower, candidate, dan leader. Candidate meminta suara dari node lain dan dapat menjadi leader apabila memperoleh suara mayoritas pada term yang sesuai.
BAB III PELAKSANAAN DAN PEMBAHASAN
3.1 Sinkronisasi Waktu
Pada Windows, modul mengarahkan praktikan menggunakan aplikasi NetTime untuk menyinkronkan waktu dengan server NTP. Langkahnya adalah mengunduh aplikasi dari situs resminya, memasang dan menjalankan aplikasi, memeriksa server NTP, kemudian menjalankan pembaruan waktu. Setelah itu, status sinkronisasi diamati.

Pembahasan: NetTime membantu komputer menyesuaikan waktu lokal dengan waktu server NTP. Sinkronisasi diperlukan karena komputer dalam sistem terdistribusi dapat memiliki perbedaan waktu. Waktu yang lebih seragam membantu pencatatan aktivitas dan analisis log. Hasil sebenarnya harus dibuktikan dengan screenshot aplikasi setelah sinkronisasi.

Bukti praktikum: [Tempelkan screenshot NetTime setelah sinkronisasi.]
3.2 Vector Clock
Pada bagian ini dibuat modul Python sederhana untuk menaikkan jam lokal, mengirim vector clock, dan menerima pesan.
Kode vector_clock.py
class VectorClock:
    def __init__(self, jumlah_proses):
        self.clock = [0] * jumlah_proses

    def increment(self, proses):
        self.clock[proses] += 1

    def send_event(self, proses):
        self.increment(proses)
        return self.clock.copy()

    def receive_event(self, proses, pesan):
        for i in range(len(self.clock)):
            self.clock[i] = max(self.clock[i], pesan[i])
        self.increment(proses)

    def __str__(self):
        return str(self.clock)
Kode main.py
from vector_clock import VectorClock

p1 = VectorClock(3)
p2 = VectorClock(3)
p3 = VectorClock(3)

p1.increment(0)
print("P1 setelah event:", p1)
pesan = p1.send_event(0)
print("Pesan dari P1:", pesan)
p2.receive_event(1, pesan)
print("P2 setelah menerima pesan:", p2)
p3.increment(2)
print("P3 setelah event sendiri:", p3)
Contoh keluaran
P1 setelah event: [1, 0, 0]
Pesan dari P1: [2, 0, 0]
P2 setelah menerima pesan: [2, 1, 0]
P3 setelah event sendiri: [0, 0, 1]
Pembahasan: Ketika P1 menjalankan kejadian lokal, komponen pertama bertambah menjadi satu. Fungsi send_event menaikkan komponen P1 sekali lagi dan mengirim salinan vektor [2, 0, 0]. Ketika P2 menerima pesan, setiap komponen dibandingkan dan diambil nilai terbesarnya, lalu komponen P2 dinaikkan sehingga hasilnya [2, 1, 0]. P3 menjalankan kejadian lokal sendiri dan menghasilkan [0, 0, 1]. Perhitungan manual mengikuti langkah yang sama: menaikkan komponen proses, mengambil nilai maksimum saat menerima pesan, lalu menaikkan komponen penerima.

Bukti praktikum: [Tempelkan screenshot hasil program. Jalankan juga vclocks.py asli apabila dosen meminta keluaran file tersebut.]
3.3 Problem Tanpa Sinkronisasi
Thread berjalan secara konkuren sehingga urutan pesan mulai dan selesai dapat berbeda pada setiap eksekusi. Penjadwalan sistem operasi dan waktu yang dibutuhkan setiap thread memengaruhi urutan keluaran.
Contoh program multithreaded-example.py
import threading
import time
import random

def tugas(nama, jeda):
    print(f"{nama} mulai")
    time.sleep(jeda)
    print(f"{nama} selesai")

threads = []
for i in range(1, 4):
    t = threading.Thread(target=tugas,
        args=(f"Thread {i}", random.uniform(0.1, 1)))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
print("Semua tugas selesai.")
Contoh keluaran
Thread 1 mulai
Thread 2 mulai
Thread 3 mulai
Thread 2 selesai
Thread 3 selesai
Thread 1 selesai
Semua tugas selesai.
Pembahasan: Keluaran tersebut hanya salah satu kemungkinan. Karena waktu jeda setiap thread berbeda dan dipilih secara acak, thread yang mulai lebih dahulu belum tentu selesai lebih dahulu. Perintah join membuat program utama menunggu sampai semua thread selesai. Jalankan program beberapa kali dan lampirkan screenshot dari dua hasil yang berbeda.

Bukti praktikum: [Tempelkan screenshot dua percobaan.]
3.3.1 Data Race / Race Conditions
Program race-conditions-01.py
import threading
import time
saldo = 0

def tambah_saldo():
    global saldo
    for _ in range(100000):
        nilai = saldo
        time.sleep(0)
        saldo = nilai + 1

t1 = threading.Thread(target=tambah_saldo)
t2 = threading.Thread(target=tambah_saldo)
t1.start(); t2.start()
t1.join(); t2.join()
print("Saldo akhir:", saldo)
print("Saldo yang diharapkan:", 200000)
Pembahasan: Dua thread menambah variabel saldo yang sama. Proses membaca, menambah, dan menulis dilakukan dalam beberapa langkah. Jika kedua thread membaca nilai yang sama sebelum salah satunya menulis hasil, pembaruan dapat saling menimpa. Akibatnya, nilai akhir berpotensi lebih kecil dari 200000, bergantung pada penjadwalan dan lingkungan eksekusi.

Visualisasi manual: Gambarkan Thread 1 dan Thread 2 sama-sama membaca saldo 0, menghitung 0 + 1, lalu menulis saldo 1. Dua penambahan hanya menghasilkan saldo akhir 1, bukan 2.
Program race-conditions-02.py
import threading
import time
saldo = 0
kunci = threading.Lock()

def tambah_saldo():
    global saldo
    for _ in range(100000):
        with kunci:
            nilai = saldo
            time.sleep(0)
            saldo = nilai + 1

t1 = threading.Thread(target=tambah_saldo)
t2 = threading.Thread(target=tambah_saldo)
t1.start(); t2.start()
t1.join(); t2.join()
print("Saldo akhir:", saldo)
print("Saldo yang diharapkan:", 200000)
Pembahasan: Program perbaikan menggunakan threading.Lock untuk melindungi bagian kritis. Blok with kunci hanya dapat dijalankan satu thread pada satu waktu. Thread lain menunggu sampai kunci dilepaskan. Dengan demikian, proses pembaruan saldo tidak saling menimpa dan hasil yang diharapkan adalah 200000.

Bukti praktikum: [Tempelkan screenshot hasil sebelum dan sesudah penggunaan lock.]
3.3.2 Deadlock
Program deadlock-01.py
import threading
import time
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread_1():
    with lock_a:
        print("Thread 1 mengunci A")
        time.sleep(0.2)
        print("Thread 1 menunggu B")
        with lock_b:
            print("Thread 1 mendapatkan B")

def thread_2():
    with lock_b:
        print("Thread 2 mengunci B")
        time.sleep(0.2)
        print("Thread 2 menunggu A")
        with lock_a:
            print("Thread 2 mendapatkan A")

t1 = threading.Thread(target=thread_1)
t2 = threading.Thread(target=thread_2)
t1.start(); t2.start()
t1.join(); t2.join()
print("Program selesai")
Pembahasan: Thread 1 mengambil lock A lalu menunggu lock B, sedangkan Thread 2 mengambil lock B lalu menunggu lock A. Jika kedua thread telah memegang lock masing-masing, keduanya dapat saling menunggu dan tidak mencapai bagian yang melepaskan lock. Program dapat berhenti sebelum mencetak pesan akhir.

Visualisasi manual: Gambar Thread 1 memegang A dan menunggu B, sedangkan Thread 2 memegang B dan menunggu A. Gunakan panah dua arah untuk menunjukkan siklus saling menunggu.
Program deadlock-02.py
import threading
import time
lock_a = threading.Lock()
lock_b = threading.Lock()

def tugas(nama):
    with lock_a:
        print(nama, "mendapatkan Lock A")
        time.sleep(0.1)
        with lock_b:
            print(nama, "mendapatkan Lock B")
            print(nama, "menyelesaikan tugas")

t1 = threading.Thread(target=tugas, args=("Thread 1",))
t2 = threading.Thread(target=tugas, args=("Thread 2",))
t1.start(); t2.start()
t1.join(); t2.join()
print("Program selesai")
Pembahasan: Semua thread mengambil lock A terlebih dahulu, lalu lock B. Urutan pengambilan yang konsisten mencegah siklus saling menunggu seperti contoh sebelumnya. Thread yang belum memperoleh lock A menunggu sebelum memegang lock apa pun, sehingga thread yang sedang bekerja dapat menyelesaikan tugas dan melepaskan lock.

Bukti praktikum: [Tempelkan screenshot hasil deadlock-02.py.]
3.4 Algoritma Raft
Raft merupakan algoritma konsensus untuk membantu node mencapai kesepakatan. Pada pemilihan leader, sebuah node menjadi candidate dan meminta suara. Jika memperoleh mayoritas suara pada term yang sesuai, candidate dapat menjadi leader. Untuk tiga node, mayoritas berarti dua suara.
Kode raft_module.py (simulasi konsep)
class RaftNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.state = "FOLLOWER"
        self.term = 0

    def election_demo(self, total_nodes, votes_received):
        self.term += 1
        self.state = "CANDIDATE"
        majority = total_nodes // 2 + 1
        print(f"Node {self.node_id} memulai pemilihan term {self.term}")
        print(f"Suara diterima: {votes_received}")
        if votes_received >= majority:
            self.state = "LEADER"
            print(f"Node {self.node_id} terpilih sebagai LEADER")
        else:
            self.state = "FOLLOWER"
            print("Suara belum mencapai mayoritas")

    def get_state(self):
        return self.state
Kode main_raft.py
from raft_module import RaftNode

node1 = RaftNode(1)
node2 = RaftNode(2)
node3 = RaftNode(3)

node2.election_demo(total_nodes=3, votes_received=2)
print("\nStatus akhir node:")
print("Node 1:", node1.get_state())
print("Node 2:", node2.get_state())
print("Node 3:", node3.get_state())
Contoh keluaran
Node 2 memulai pemilihan term 1
Suara diterima: 2
Node 2 terpilih sebagai LEADER

Status akhir node:
Node 1: FOLLOWER
Node 2: LEADER
Node 3: FOLLOWER
Pembahasan: Node 2 memperoleh dua suara dari tiga node. Karena dua merupakan suara mayoritas, Node 2 ditetapkan sebagai leader dalam simulasi. Contoh ini hanya menunjukkan konsep mayoritas dan perubahan status; suara dimasukkan sebagai nilai, bukan dikirim melalui komunikasi antarnode. Untuk laporan sesuai modul, jalankan juga raft.py asli dan jelaskan mekanisme serta keluaran sebenarnya.

Visualisasi manual: Gambarkan tiga node. Node 2 sebagai candidate meminta suara dari Node 1 dan Node 3. Setelah memperoleh mayoritas, Node 2 menjadi leader, sedangkan Node 1 dan Node 3 menjadi follower.

Bukti praktikum: [Tempelkan screenshot raft.py asli dan simulasi modul yang dibuat.]
BAB IV PENUTUP
4.1 Kesimpulan
Berdasarkan praktikum Modul 3, saya memahami bahwa sinkronisasi penting dalam sistem terdistribusi. Sinkronisasi waktu membantu menyelaraskan jam perangkat, sedangkan Vector Clock mencatat hubungan urutan kejadian pada beberapa proses. Dalam multithreading, urutan keluaran dapat berbeda karena thread berjalan secara konkuren. Race condition terjadi saat beberapa thread mengubah data bersama tanpa perlindungan, sedangkan deadlock muncul ketika thread saling menunggu lock. Penggunaan lock dan urutan penguncian yang konsisten dapat membantu mengatasi masalah tersebut. Saya juga mempelajari prinsip pemilihan leader pada Raft, yaitu kandidat perlu memperoleh suara mayoritas.
4.2 Saran
Setiap program sebaiknya dijalankan langsung dan diamati lebih dari satu kali, terutama program multithreading. Screenshot hasil aktual perlu disimpan sebagai bukti praktikum. Program contoh untuk memahami konsep juga perlu dibedakan dari program asli pada modul agar laporan sesuai dengan tugas yang diberikan.
DAFTAR PUSTAKA
Purnomosidi, B. (t.t.). Modul 3: Sinkronisasi pada Sistem Terdistribusi. Praktikum Sistem Terdistribusi dan Terdesentralisasi, Universitas Teknologi Digital Indonesia.

Chrony Project. (t.t.). Chrony. https://chrony-project.org/

NetTime. (t.t.). NetTime. https://www.timesynctool.com/
