# LAPORAN PRAKTIKUM
## INSTALASI DAN KONFIGURASI APACHE HADOOP 3.5.0

**Nama:** Maria Widya Febrianti  
**NIM:** 255410011  
**Modul:** Modul 06  
**Dosen Penyusun:**   

---

## A. Tujuan Praktikum

Praktikum ini bertujuan untuk memahami proses instalasi dan konfigurasi Apache Hadoop, mengatur komponen Hadoop melalui file konfigurasi XML, serta menjalankan dan memeriksa layanan Hadoop Distributed File System (HDFS).

## B. Alat dan Bahan

1. Sistem operasi Ubuntu melalui WSL.
2. Java OpenJDK 17.
3. Apache Hadoop versi 3.5.0.
4. Terminal Ubuntu.

## C. Langkah-Langkah Praktikum

### 1. Konfigurasi Java

Java digunakan sebagai lingkungan untuk menjalankan Apache Hadoop. Versi Java yang digunakan adalah OpenJDK 17.

```bash
java -version
```
<img width="1006" height="107" alt="image" src="https://github.com/user-attachments/assets/7dac8359-7380-42f8-9e23-ec2bbd25777c" />

### 2. Instalasi Apache Hadoop

Apache Hadoop versi 3.5.0 ditempatkan pada direktori:

```bash
~/software/storage/hadoop
```
<img width="940" height="488" alt="image" src="https://github.com/user-attachments/assets/d1b1ff28-a489-4cf6-a3b2-1f69c5d47eca" />
<img width="1015" height="93" alt="image" src="https://github.com/user-attachments/assets/21ad9d04-ab4f-47b6-9c6b-f4e22183ef4a" />

Pemeriksaan versi Hadoop dilakukan menggunakan perintah:

```bash
hadoop version
```

### 3. Konfigurasi Hadoop

Konfigurasi dilakukan dengan mengatur beberapa file yang terdapat pada direktori `etc/hadoop`.

**a. Konfigurasi hadoop-env.sh**

Mengatur lokasi Java menggunakan variabel `JAVA_HOME`.

**b. Konfigurasi hdfs-site.xml**

Mengatur jumlah replikasi dan lokasi penyimpanan NameNode serta DataNode.
<img width="1896" height="976" alt="Screenshot 2026-11-10 002450" src="https://github.com/user-attachments/assets/0832318f-8b67-4ecc-9df6-3574d00c2c9f" />

**c. Konfigurasi core-site.xml**

Mengatur alamat HDFS menggunakan `hdfs://localhost:9000`.
<img width="1898" height="563" alt="Screenshot 2026-11-10 002243" src="https://github.com/user-attachments/assets/e85424f3-d6b3-422a-b98d-8e75a138bc5c" />

**d. Konfigurasi mapred-site.xml**

Mengatur framework MapReduce agar menggunakan YARN.
<img width="1031" height="760" alt="image" src="https://github.com/user-attachments/assets/66ee8772-de20-469f-ac0f-d86545a4e059" />

**e. Konfigurasi yarn-site.xml**

Mengatur layanan NodeManager agar dapat menjalankan proses MapReduce.
<img width="1041" height="749" alt="image" src="https://github.com/user-attachments/assets/17ff9a10-380a-435c-8c81-bed85eedd228" />

### 4. Memeriksa Konfigurasi HDFS

Pemeriksaan direktori NameNode:

```bash
hdfs getconf -confKey dfs.namenode.name.dir
```
<img width="1020" height="742" alt="image" src="https://github.com/user-attachments/assets/12fb41a7-8876-40c6-925d-c572e73f1193" />

Pemeriksaan direktori DataNode:

```bash
hdfs getconf -confKey dfs.datanode.data.dir
```
<img width="1046" height="718" alt="image" src="https://github.com/user-attachments/assets/5e347faa-3a35-4265-b812-584effaaf7bd" />

### 5. Menjalankan HDFS

HDFS dijalankan menggunakan perintah:

```bash
sbin/start-dfs.sh
```


Kemudian memeriksa proses Hadoop:

```bash
jps
```
<img width="1004" height="155" alt="image" src="https://github.com/user-attachments/assets/dae410ed-7122-47a2-8344-3385eba57082" />

Hasil pemeriksaan menunjukkan proses NameNode, DataNode, dan SecondaryNameNode telah berjalan.

### 6. Memeriksa File pada HDFS

Untuk melihat direktori utama HDFS:

```bash
hdfs dfs -ls /
```
<img width="1023" height="744" alt="image" src="https://github.com/user-attachments/assets/73bccf99-e21b-451e-b769-a1282c63cfe0" />

Untuk melihat dataset yang tersimpan:

```bash
hdfs dfs -ls /user/aldo/datasets
```
<img width="1024" height="762" alt="image" src="https://github.com/user-attachments/assets/493ca0b9-73f1-4a15-bbf8-bbcc241b1069" />

Hasilnya menunjukkan terdapat tiga file dataset, yaitu `airline-passengers.csv`, `iris.csv`, dan `shampoo.csv`.
<img width="1033" height="138" alt="image" src="https://github.com/user-attachments/assets/b5a4cc73-9187-4481-93a1-fa7504ce7b10" />

## D. Hasil Praktikum

Berdasarkan hasil praktikum, Apache Hadoop 3.5.0 berhasil dikonfigurasi dan layanan HDFS berhasil dijalankan. Hal tersebut dibuktikan dengan munculnya proses NameNode, DataNode, dan SecondaryNameNode ketika menjalankan perintah `jps`. Selain itu, direktori dan file dataset yang tersimpan di HDFS juga berhasil diperiksa menggunakan perintah `hdfs dfs -ls`.

## E. Kesimpulan

Dari praktikum Modul 06, dapat disimpulkan bahwa instalasi dan konfigurasi Apache Hadoop memerlukan pengaturan Java serta beberapa file konfigurasi Hadoop. Setelah konfigurasi dilakukan, layanan HDFS berhasil dijalankan dan file dataset dapat ditampilkan melalui terminal Ubuntu. Praktikum ini memberikan pemahaman mengenai dasar penggunaan Hadoop untuk pengelolaan data terdistribusi.

---

**Catatan:** Konfigurasi YARN sudah dilakukan, tetapi pengujian menjalankan job MapReduce belum dilakukan.
