# Laporan Praktikum Modul 6: Instalasi & Konfigurasi Apache Hadoop Single-Node Cluster (HDFS & YARN) di WSL

Repository ini berisi laporan praktikum mengenai langkah-langkah instalasi, pengonfigurasian, serta pengujian **Apache Hadoop Single-Node Cluster** (HDFS dan YARN) di lingkungan Windows Subsystem for Linux (WSL), hingga tahap manipulasi data file (*datasets*) pada HDFS.

---

## 1. Tujuan Praktikum

1. Memahami arsitektur dasar Apache Hadoop, HDFS (Hadoop Distributed File System), dan YARN (Yet Another Resource Negotiator).
2. Mampu mengonfigurasi Java Development Kit (JDK 17) serta SSH Keyless Authentication pada WSL.
3. Mampu melakukan instalasi dan pemetaan environment variable Apache Hadoop 3.5.0.
4. Mampu mengonfigurasi file XML Hadoop (`hdfs-site.xml`, `core-site.xml`, `mapred-site.xml`, dan `yarn-site.xml`).
5. Mampu menjalankan daemon Hadoop (NameNode, DataNode, SecondaryNameNode) dan mengelola file pada HDFS.

---

## 2. Dasar Teori

### 2.1 Apache Hadoop & Single-Node Cluster
Apache Hadoop adalah kerangka kerja (*framework*) open-source yang digunakan untuk penyimpanan dan pemrosesan data berskala besar (*Big Data*) secara terdistribusi. Pada praktikum ini digunakan mode *Single-Node Cluster* (Pseudo-Distributed Mode), di mana seluruh daemon Hadoop dijalankan pada satu mesin virtual (WSL) untuk mensimulasikan lingkungan terdistribusi.

### 2.2 Komponen Utama Hadoop
- **HDFS (Hadoop Distributed File System)**: Sistem berkas terdistribusi yang menyediakan akses *high-throughput* ke data aplikasi. Komponen dasarnya meliputi:
  - **NameNode**: Master node yang mengelola direktori dan metadata sistem berkas.
  - **DataNode**: Worker node yang menyimpan blok-blok data aktual.
  - **SecondaryNameNode**: Node pembuat checkpoint berkala bagi NameNode.
- **YARN & MapReduce**:
  - **YARN**: Sistem manajemen sumber daya dan penjadwalan *job*.
  - **MapReduce**: Model pemrograman untuk pemrosesan data paralel skala besar.

---

## 3. Langkah Praktikum

### Tahap 1: Instalasi Java (JDK 17) & SSH Server

Langkah awal dimulai dengan menyegarkan daftar indeks paket dan memperbarui sistem pada WSL:

sudo apt update && sudo apt upgrade -y

![Update dan Upgrade Paket Sistem](path/to/gambar-1.png)
*Gambar 1: Pembaruan indeks paket repository dan upgrade sistem di terminal WSL.*

**Penjelasan Gambar 1:**
Perintah `sudo apt update && sudo apt upgrade -y` menyegarkan repositori Linux dan memperbarui dependensi yang terpasang agar lingkungan OS siap untuk instalasi Java dan Hadoop.

Menginstal OpenJDK 17 sebagai persyaratan runtime kompilasi Hadoop:

sudo apt install openjdk-17-jdk -y

![Instalasi OpenJDK 17](path/to/gambar-2.png)
*Gambar 2: Proses instalasi pustaka OpenJDK 17 JDK.*

**Penjelasan Gambar 2:**
Hadoop membutuhkan Java Development Kit (JDK) untuk menjalankan komponen Java virtual machine-nya. Verifikasi menunjukkan OpenJDK 17 telah terpasang di sistem.

Memeriksa versi Java yang aktif untuk memastikan instalasi berhasil:

java -version

![Verifikasi Versi Java](path/to/gambar-3.png)
*Gambar 3: Output verifikasi versi Java (OpenJDK 17.0.21).*

**Penjelasan Gambar 3:**
Perintah `java -version` mengonfirmasi bahwa OpenJDK versi 17.0.21 telah aktif dengan 64-Bit Server VM.

Menginstal dan mengaktifkan layanan OpenSSH Server:

sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh

![Instalasi dan Aktivasi OpenSSH Server](path/to/gambar-4.png)
*Gambar 4: Instalasi openssh-server serta pembuatkan symbolic link ssh.service.*

**Penjelasan Gambar 4:**
Apache Hadoop mengoperasikan skrip kontrol menggunakan SSH. Perintah `systemctl enable ssh` memastikan daemon SSH aktif secara permanen.

Memeriksa status operasional SSH server:

sudo systemctl status ssh

![Pemeriksaan Status SSH Service](path/to/gambar-5.png)
*Gambar 5: Verifikasi status aktif (running) pada layanan OpenSSH Server port 22.*

**Penjelasan Gambar 5:**
Status `active (running)` membuktikan bahwa layanan OpenBSD Secure Shell server telah mendengarkan koneksi pada port 22.

---

### Tahap 2: Pengaturan Environment Variable Java (`JAVA_HOME`)

Mencari lokasi direktori absolut tempat Java terinstal:

readlink -f $(which java)

![Mencari Path Instalasi Java](path/to/gambar-6.png)
*Gambar 6: Menelusuri lokasi absolut direktori Java (/usr/lib/jvm/java-17-openjdk-amd64/bin/java).*

**Penjelasan Gambar 6:**
Perintah `readlink -f $(which java)` mengembalikan path asli biner Java sehingga lokasi `JAVA_HOME` dapat ditentukan secara presisi.

Membuka file konfigurasi `.bashrc` menggunakan editor nano:

nano ~/.bashrc

![Membuka File .bashrc](path/to/gambar-7.png)
*Gambar 7: Eksekusi perintah nano ~/.bashrc untuk menyunting variabel lingkungan pengguna.*

**Penjelasan Gambar 7:**
Penyuntingan `.bashrc` dilakukan agar variabel lingkungan `JAVA_HOME` dimuat otomatis setiap kali sesi terminal baru dibuka.

Menambahkan definisi `JAVA_HOME` dan perbaikan `PATH` pada bagian paling bawah file `.bashrc`:

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

![Menambahkan Variabel JAVA_HOME di .bashrc](path/to/gambar-8.png)
*Gambar 8: Penambahan ekspor JAVA_HOME dan pembaruan variabel PATH pada bagian paling bawah file .bashrc.*

**Penjelasan Gambar 8:**
Variabel `JAVA_HOME` diatur menunjuk ke `/usr/lib/jvm/java-17-openjdk-amd64` agar aplikasi Hadoop dapat menemukan runtime Java.

Menerapkan perubahan file `.bashrc` tanpa harus restart terminal:

source ~/.bashrc

![Menerapkan Perubahan .bashrc](path/to/gambar-9.png)
*Gambar 9: Eksekusi perintah source ~/.bashrc untuk merefresh konfigurasi shell.*

**Penjelasan Gambar 9:**
Perintah `source ~/.bashrc` memuat ulang seluruh ekspor variabel lingkungan terbaru ke dalam sesi terminal aktif.

---

### Tahap 3: Instalasi `pdsh` & Konfigurasi SSH Keyless

Menginstal utilitas `pdsh` (Parallel Distributed Shell) untuk komunikasi antar node:

sudo apt install pdsh -y

![Instalasi Utilitas pdsh](path/to/gambar-10.png)
*Gambar 10: Proses instalasi paket pdsh, genders, dan ssh-askpass.*

**Penjelasan Gambar 10:**
Utilitas `pdsh` digunakan oleh skrip manajemen Hadoop untuk mengeksekusi perintah secara paralel ke beberapa node/localhost.

Membuat pasangan kunci SSH publik dan privat tanpa passphrase:

ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa

![Membuat SSH Key RSA](path/to/gambar-11.png)
*Gambar 11: Proses pembuatan pasangan kunci RSA 3072-bit SSH.*

**Penjelasan Gambar 11:**
Perintah `ssh-keygen` menghasilkan berkas `id_rsa` (kunci privat) dan `id_rsa.pub` (kunci publik) untuk otentikasi aman tanpa kata sandi.

Mengonfirmasi pembuatan ulang kunci SSH (overwrite key):

![Konfirmasi Overwrite SSH Key](path/to/gambar-12.png)
*Gambar 12: Pembaruan fingerprint kunci SSH RSA di direktori ~/.ssh/.*

**Penjelasan Gambar 12:**
Mengonfirmasi pembuatan ulang kunci SSH untuk memastikan bahwa tidak ada konflik dengan sertifikat otentikasi lama.

Mendaftarkan kunci publik ke file `authorized_keys` dan mengubah hak aksesnya:

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

![Mendaftarkan Authorized Keys](path/to/gambar-13.png)
*Gambar 13: Menambahkan id_rsa.pub ke authorized_keys serta mengamankan hak akses file (chmod 0600).*

**Penjelasan Gambar 13:**
Menambahkan kunci publik ke `authorized_keys` memungkinkan login SSH ke `localhost` tanpa memasukkan kata sandi. `chmod 0600` mengunci hak akses file demi keamanan.

Menguji koneksi SSH otomatis ke localhost:

ssh localhost

![Uji Coba SSH Localhost Keyless](path/to/gambar-14.png)
*Gambar 14: Keberhasilan login SSH keyless ke localhost Ubuntu WSL.*

**Penjelasan Gambar 14:**
Koneksi SSH ke `localhost` berhasil dilakukan tanpa prompt kata sandi. Hal ini menandakan bahwa otentikasi SSH *keyless* siap digunakan oleh daemon Hadoop.

---

### Tahap 4: Unduh dan Ekstrak Apache Hadoop 3.5.0

Membuat direktori kerja penyimpanan software dan masuk ke dalamnya:

mkdir -p ~/software/storage/
cd ~/software/storage/

![Membuat Direktori Software Storage](path/to/gambar-15.png)
*Gambar 15: Membuat dan berpindah ke direktori ~/software/storage/.*

**Penjelasan Gambar 15:**
Direktori ini difungsikan sebagai lokasi pengunduhan dan penempatan berkas biner aplikasi Hadoop.

Mengunduh berkas biner resmi Apache Hadoop versi 3.5.0:

wget https://downloads.apache.org/hadoop/common/hadoop-3.5.0/hadoop-3.5.0.tar.gz

![Unduh Archive Apache Hadoop 3.5.0](path/to/gambar-16.png)
*Gambar 16: Pengunduhan berkas hadoop-3.5.0.tar.gz sebesar 554 MB via wget.*

**Penjelasan Gambar 16:**
Berkas arsip Hadoop 3.5.0 diunduh dari repositori resmi Apache.

Mengekstrak berkas tarball dan membuat symbolic link:

tar -xzvf hadoop-3.5.0.tar.gz
ln -s hadoop-3.5.0 hadoop

![Ekstrak Tarball Hadoop](path/to/gambar-17.png)
*Gambar 17: Ekstraksi direktori berkas hadoop-3.5.0 dan pembuatan symbolic link.*

**Penjelasan Gambar 17:**
Proses ekstraksi menguraikan direktori biner Hadoop, lalu `ln -s` membuat tautan direktori `hadoop` agar mempermudah navigasi path.

---

### Tahap 5: Daftarkan Environment Variable Hadoop

Membuat direktori penyimpanan skrip environment bash custom:

mkdir -p ~/env/bash/storage/

![Membuat Direktori Env Custom](path/to/gambar-18.png)
*Gambar 18: Pembuatan direktori ~/env/bash/storage/ untuk penampungan skrip lingkungan.*

**Penjelasan Gambar 18:**
Direktori ini digunakan untuk memisahkan skrip entri environment PATH agar mudah dikelola.

Membuat skrip `hadoop.sh` untuk mendaftarkan direktori biner Hadoop ke variabel `PATH`:

echo 'export PATH="$HOME/software/storage/hadoop/bin:$HOME/software/storage/hadoop/sbin:$PATH"' > ~/env/bash/storage/hadoop.sh

![Membuat Skrip hadoop.sh](path/to/gambar-19.png)
*Gambar 19: Pembuatan skrip hadoop.sh yang berisi perintah ekspor PATH Hadoop.*

**Penjelasan Gambar 19:**
Perintah ini memasukkan path `hadoop/bin` dan `hadoop/sbin` ke variabel `PATH` agar seluruh perintah Hadoop dapat dipanggil dari lokasi direktori mana pun.

Mengaktifkan skrip environment Hadoop:

source ~/env/bash/storage/hadoop.sh

![Source Skrip hadoop.sh](path/to/gambar-20.png)
*Gambar 20: Memuat variabel PATH terbaru dari hadoop.sh ke sesi terminal.*

**Penjelasan Gambar 20:**
Perintah `source` memperbarui variabel lingkungan PATH dalam sesi aktif sehingga utilitas Hadoop langsung terdeteksi terminal.

Menguji pemanggilan CLI `hadoop` untuk mengonfirmasi pendaftaran environment:

hadoop

![Uji Coba Perintah CLI Hadoop](path/to/gambar-21.png)
*Gambar 21: Tampilan panduan bantuan penggunaan CLI Hadoop (Client & Admin Commands).*

**Penjelasan Gambar 21:**
Output daftar perintah `hadoop` (seperti `fs`, `checknative`, `version`) membuktikan bahwa biner Hadoop telah sukses terdaftar pada sistem PATH.

---

### Tahap 6: Konfigurasi File XML Apache Hadoop

Berpindah ke direktori konfigurasi XML Hadoop:

cd ~/software/storage/hadoop/etc/hadoop/

![Navigasi ke Direktori Konfigurasi Hadoop](path/to/gambar-22.png)
*Gambar 22: Berpindah ke direktori ~/software/storage/hadoop/etc/hadoop/.*

**Penjelasan Gambar 22:**
Direktori ini menyimpan seluruh berkas konfigurasi XML utama untuk mengatur perilaku HDFS, YARN, dan MapReduce.

Satu per satu file XML disunting menggunakan nano editor:

1. **Konfigurasi `hdfs-site.xml`**:
   Membuka file: `nano hdfs-site.xml`
   Menambahkan properti jumlah replikasi blok HDFS menjadi 1 (karena mode *Single-Node*):

![Konfigurasi hdfs-site.xml](path/to/gambar-23.png)
*Gambar 23: Pengaturan properti dfs.replication = 1 pada hdfs-site.xml.*

**Penjelasan Gambar 23:**
Parameter `dfs.replication` diisi dengan nilai `1` karena sistem berjalan pada satu node tunggal tanpa ada node duplikasi tambahan.

2. **Konfigurasi `core-site.xml`**:
   Membuka file: `nano core-site.xml`
   Menentukan URI default sistem berkas NameNode ke `hdfs://localhost:9000` dan izin proxy user:

![Konfigurasi core-site.xml](path/to/gambar-24.png)
*Gambar 24: Pengaturan fs.defaultFS ke hdfs://localhost:9000 serta konfigurasi proxyuser pada core-site.xml.*

**Penjelasan Gambar 24:**
Nilai `fs.defaultFS` menentukan alamat Master NameNode pada `hdfs://localhost:9000`. Properti `hadoop.proxyuser` memberikan hak otorisasi akun pengguna.

3. **Konfigurasi `mapred-site.xml`**:
   Membuka file: `nano mapred-site.xml`
   Menentukan kerangka kerja MapReduce menggunakan YARN:

![Konfigurasi mapred-site.xml](path/to/gambar-25.png)
*Gambar 25: Pengaturan mapreduce.framework.name = yarn dan classpath aplikasi pada mapred-site.xml.*

**Penjelasan Gambar 25:**
Parameter ini menginstruksikan eksekusi tugas MapReduce agar dikelola sepenuhnya oleh pengelola sumber daya YARN.

4. **Konfigurasi `yarn-site.xml`**:
   Membuka file: `nano yarn-site.xml`
   Mengaktifkan layanan *auxiliary shuffle* dan pengaturan alokasi memori NodeManager:

![Konfigurasi yarn-site.xml](path/to/gambar-26.png)
*Gambar 26: Pengaturan auxiliary service mapreduce_shuffle dan batas memori 4096 MB pada yarn-site.xml.*

**Penjelasan Gambar 26:**
File `yarn-site.xml` mengonfigurasi `mapreduce_shuffle`, batas alokasi memori NodeManager (`4096 MB`), serta rasio memori virtual ke fisik.

---

### Tahap 7: Inisialisasi NameNode, Memulai Daemon HDFS, dan Manajemen File Datasets

Melakukan format pada NameNode sebelum menjalankan daemon HDFS untuk pertama kali:

hdfs namenode -format

![Format NameNode HDFS](path/to/gambar-27.png)
*Gambar 27: Eksekusi perintah hdfs namenode -format untuk inisialisasi struktur direktori metadata NameNode.*

**Penjelasan Gambar 27:**
Perintah `hdfs namenode -format` menginisialisasi direktori metadata NameNode dan membuat Cluster ID baru untuk sistem berkas HDFS.

Memulai seluruh daemon HDFS (NameNode, DataNode, SecondaryNameNode):

start-dfs.sh

![Memulai Daemon HDFS](path/to/gambar-28.png)
*Gambar 28: Menjalankan skrip start-dfs.sh untuk mengaktifkan daemon NameNode, DataNode, dan SecondaryNameNode.*

**Penjelasan Gambar 28:**
Skrip `start-dfs.sh` secara otomatis menyalakan proses NameNode, DataNode, dan SecondaryNameNode pada localhost melalui koneksi SSH.

Memeriksa proses Java Hadoop yang sedang aktif menggunakan `jps`:

jps

![Verifikasi Proses Java jps](path/to/gambar-29.png)
*Gambar 29: Tampilan output jps yang mengonfirmasi bahwa NameNode, DataNode, dan SecondaryNameNode aktif.*

**Penjelasan Gambar 29:**
Perintah `jps` (Java Virtual Machine Process Status Tool) memverifikasi bahwa ketiga daemon utama HDFS (`NameNode`, `DataNode`, dan `SecondaryNameNode`) telah berjalan normal di latar belakang.

---

### Tahap 8: Tugas Modul (Pengelolaan File Datasets di HDFS)

Membuat direktori penyimpanan dataset di lingkungan HDFS:

hdfs dfs -mkdir -p /user/acer/datasets

![Membuat Direktori di HDFS](path/to/gambar-30.png)
*Gambar 30: Eksekusi perintah hdfs dfs -mkdir untuk membuat struktur folder /user/acer/datasets.*

**Penjelasan Gambar 30:**
Perintah `-mkdir -p` membuat direktori baru `/user/acer/datasets` pada sistem berkas terdistribusi HDFS.

Mengunduh file berkas CSV dataset dari repositori publik ke sistem lokal WSL:

wget https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv -O data2.csv
wget https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.csv -O data3.csv

![Mengunduh File Dataset CSV](path/to/gambar-31.png)
*Gambar 31: Proses pengunduhan berkas dataset data2.csv dan data3.csv via wget.*

**Penjelasan Gambar 31:**
File CSV lokal `data2.csv` (Titanic dataset) dan `data3.csv` (USGS Earthquake dataset) berhasil diunduh ke sistem berkas lokal.

Mengunggah file CSV dari sistem lokal ke direktori HDFS:

hdfs dfs -put data1.csv /user/acer/datasets/
hdfs dfs -put data2.csv /user/acer/datasets/
hdfs dfs -put data3.csv /user/acer/datasets/

![Mengunggah File ke HDFS](path/to/gambar-32.png)
*Gambar 32: Eksekusi perintah hdfs dfs -put untuk mentransfer berkas data1.csv, data2.csv, dan data3.csv ke HDFS.*

**Penjelasan Gambar 32:**
Perintah `hdfs dfs -put` menyalin berkas data dari sistem file lokal WSL ke dalam direktori terdistribusi HDFS `/user/acer/datasets/`.

Memeriksa dan menampilkan isi daftar file pada direktori HDFS:

hdfs dfs -ls /user/acer/datasets

![Menampilkan Daftar Berkas HDFS](path/to/gambar-33.png)
*Gambar 33: Tampilan daftar berkas terdaftar pada HDFS beserta atribut ukuran file dan izin akses.*

**Penjelasan Gambar 33:**
Perintah `hdfs dfs -ls` mengonfirmasi bahwa ketiga berkas (`data1.csv`, `data2.csv`, dan `data3.csv`) telah berhasil tersimpan di HDFS dengan faktor replikasi `1` dan pemilik `acer`.

---

## 4. Kesimpulan

1. **Instalasi & Environment**: Pengaturan OpenJDK 17, SSH Keyless, serta pendaftaran `JAVA_HOME` dan variabel `PATH` pada sistem operasi WSL berhasil diselesaikan sehingga seluruh perintah Hadoop dapat dipanggil dari terminal.
2. **Konfigurasi Cluster Single-Node**: File XML (`hdfs-site.xml`, `core-site.xml`, `mapred-site.xml`, dan `yarn-site.xml`) telah dikonfigurasi untuk menjalankan kluster *pseudo-distributed* dengan replikasi tunggal.
3. **Operasional HDFS**: Format NameNode dan eksekusi `start-dfs.sh` berjalan sukses, ditandai dengan aktifnya daemon `NameNode`, `DataNode`, dan `SecondaryNameNode` pada pengecekan `jps`.
4. **Manajemen Data**: Operasi pembuatan direktori (`-mkdir`), pengunggahan file (`-put`), serta daftar isi file (`-ls`) pada sistem berkas HDFS berjalan lancar tanpa mengalami kesalahan.
