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

### 2. Instalasi Apache Hadoop

Apache Hadoop versi 3.5.0 ditempatkan pada direktori:

```bash
~/software/storage/hadoop
```

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

**c. Konfigurasi core-site.xml**

Mengatur alamat HDFS menggunakan `hdfs://localhost:9000`.

**d. Konfigurasi mapred-site.xml**

Mengatur framework MapReduce agar menggunakan YARN.

**e. Konfigurasi yarn-site.xml**

Mengatur layanan NodeManager agar dapat menjalankan proses MapReduce.

### 4. Memeriksa Konfigurasi HDFS

Pemeriksaan direktori NameNode:

```bash
hdfs getconf -confKey dfs.namenode.name.dir
```

Pemeriksaan direktori DataNode:

```bash
hdfs getconf -confKey dfs.datanode.data.dir
```

### 5. Menjalankan HDFS

HDFS dijalankan menggunakan perintah:

```bash
sbin/start-dfs.sh
```

Kemudian memeriksa proses Hadoop:

```bash
jps
```

Hasil pemeriksaan menunjukkan proses NameNode, DataNode, dan SecondaryNameNode telah berjalan.

### 6. Memeriksa File pada HDFS

Untuk melihat direktori utama HDFS:

```bash
hdfs dfs -ls /
```

Untuk melihat dataset yang tersimpan:

```bash
hdfs dfs -ls /user/aldo/datasets
```

Hasilnya menunjukkan terdapat tiga file dataset, yaitu `airline-passengers.csv`, `iris.csv`, dan `shampoo.csv`.

## D. Hasil Praktikum

Berdasarkan hasil praktikum, Apache Hadoop 3.5.0 berhasil dikonfigurasi dan layanan HDFS berhasil dijalankan. Hal tersebut dibuktikan dengan munculnya proses NameNode, DataNode, dan SecondaryNameNode ketika menjalankan perintah `jps`. Selain itu, direktori dan file dataset yang tersimpan di HDFS juga berhasil diperiksa menggunakan perintah `hdfs dfs -ls`.

## E. Kesimpulan

Dari praktikum Modul 06, dapat disimpulkan bahwa instalasi dan konfigurasi Apache Hadoop memerlukan pengaturan Java serta beberapa file konfigurasi Hadoop. Setelah konfigurasi dilakukan, layanan HDFS berhasil dijalankan dan file dataset dapat ditampilkan melalui terminal Ubuntu. Praktikum ini memberikan pemahaman mengenai dasar penggunaan Hadoop untuk pengelolaan data terdistribusi.

---

**Catatan:** Konfigurasi YARN sudah dilakukan, tetapi pengujian menjalankan job MapReduce belum dilakukan.
