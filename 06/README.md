# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Laporan Praktikum Modul 6: Instalasi & Konfigurasi Apache Hadoop Single-Node Cluster (HDFS & YARN) di WSL

### Identitas Mahasiswa

* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1 (Semester 3)
* **Universitas:** Universitas Teknologi Digital Indonesia (UTDI
  
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

<img width="940" height="598" alt="image" src="https://github.com/user-attachments/assets/d3c53648-1110-48b9-abdb-74e5807df123" />
*Gambar 1: Pembaruan indeks paket repository dan upgrade sistem di terminal WSL.*

**Penjelasan Gambar 1:**
Perintah `sudo apt update && sudo apt upgrade -y` menyegarkan repositori Linux dan memperbarui dependensi yang terpasang agar lingkungan OS siap untuk instalasi Java dan Hadoop.

Menginstal OpenJDK 17 sebagai persyaratan runtime kompilasi Hadoop:

sudo apt install openjdk-17-jdk -y

<img width="940" height="157" alt="image" src="https://github.com/user-attachments/assets/ce6c8587-0c2a-4641-a943-0a3c40404512" />
*Gambar 2: Proses instalasi pustaka OpenJDK 17 JDK.*

**Penjelasan Gambar 2:**
Hadoop membutuhkan Java Development Kit (JDK) untuk menjalankan komponen Java virtual machine-nya. Verifikasi menunjukkan OpenJDK 17 telah terpasang di sistem.

Memeriksa versi Java yang aktif untuk memastikan instalasi berhasil:

java -version

<img width="940" height="85" alt="image" src="https://github.com/user-attachments/assets/b0b778cb-2f3d-467e-8c66-ba9f2b20de7a" />
*Gambar 3: Output verifikasi versi Java (OpenJDK 17.0.21).*

**Penjelasan Gambar 3:**
Perintah `java -version` mengonfirmasi bahwa OpenJDK versi 17.0.21 telah aktif dengan 64-Bit Server VM.

Menginstal dan mengaktifkan layanan OpenSSH Server:

sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh

<img width="940" height="237" alt="image" src="https://github.com/user-attachments/assets/8ab7d2fc-7a7c-4fa5-9a57-246523b2b6ff" />
*Gambar 4: Instalasi openssh-server serta pembuatkan symbolic link ssh.service.*

**Penjelasan Gambar 4:**
Apache Hadoop mengoperasikan skrip kontrol menggunakan SSH. Perintah `systemctl enable ssh` memastikan daemon SSH aktif secara permanen.

Memeriksa status operasional SSH server:

sudo systemctl status ssh

<img width="940" height="380" alt="image" src="https://github.com/user-attachments/assets/8ed1a7eb-4f37-4622-85d6-839d14eaf6d5" />
*Gambar 5: Verifikasi status aktif (running) pada layanan OpenSSH Server port 22.*

**Penjelasan Gambar 5:**
Status `active (running)` membuktikan bahwa layanan OpenBSD Secure Shell server telah mendengarkan koneksi pada port 22.

---

### Tahap 2: Pengaturan Environment Variable Java (`JAVA_HOME`)

Mencari lokasi direktori absolut tempat Java terinstal:

readlink -f $(which java)

<img width="882" height="94" alt="image" src="https://github.com/user-attachments/assets/bc7e643a-0b93-4931-9ab5-8eb5d8c548f0" />
*Gambar 6: Menelusuri lokasi absolut direktori Java (/usr/lib/jvm/java-17-openjdk-amd64/bin/java).*

**Penjelasan Gambar 6:**
Perintah `readlink -f $(which java)` mengembalikan path asli biner Java sehingga lokasi `JAVA_HOME` dapat ditentukan secara presisi.

Membuka file konfigurasi `.bashrc` menggunakan editor nano:

nano ~/.bashrc

<img width="729" height="69" alt="image" src="https://github.com/user-attachments/assets/39d9d4d3-755b-4283-bc3c-7f6e6bdb107c" />
*Gambar 7: Eksekusi perintah nano ~/.bashrc untuk menyunting variabel lingkungan pengguna.*

**Penjelasan Gambar 7:**
Penyuntingan `.bashrc` dilakukan agar variabel lingkungan `JAVA_HOME` dimuat otomatis setiap kali sesi terminal baru dibuka.

Menambahkan definisi `JAVA_HOME` dan perbaikan `PATH` pada bagian paling bawah file `.bashrc`:

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

<img width="940" height="484" alt="image" src="https://github.com/user-attachments/assets/a026f625-9385-468d-b0dd-0fbdd5c23981" />
*Gambar 8: Penambahan ekspor JAVA_HOME dan pembaruan variabel PATH pada bagian paling bawah file .bashrc.*

**Penjelasan Gambar 8:**
Variabel `JAVA_HOME` diatur menunjuk ke `/usr/lib/jvm/java-17-openjdk-amd64` agar aplikasi Hadoop dapat menemukan runtime Java.

Menerapkan perubahan file `.bashrc` tanpa harus restart terminal:

source ~/.bashrc

<img width="678" height="59" alt="image" src="https://github.com/user-attachments/assets/cdaefba7-4f12-4827-b240-bb33c94fda89" />
*Gambar 9: Eksekusi perintah source ~/.bashrc untuk merefresh konfigurasi shell.*

**Penjelasan Gambar 9:**
Perintah `source ~/.bashrc` memuat ulang seluruh ekspor variabel lingkungan terbaru ke dalam sesi terminal aktif.

---

### Tahap 3: Instalasi `pdsh` & Konfigurasi SSH Keyless

Menginstal utilitas `pdsh` (Parallel Distributed Shell) untuk komunikasi antar node:

sudo apt install pdsh -y

<img width="940" height="610" alt="image" src="https://github.com/user-attachments/assets/ea278eac-ef6a-4164-bf09-2756e00f2beb" />
*Gambar 10: Proses instalasi paket pdsh, genders, dan ssh-askpass.*

**Penjelasan Gambar 10:**
Utilitas `pdsh` digunakan oleh skrip manajemen Hadoop untuk mengeksekusi perintah secara paralel ke beberapa node/localhost.

Membuat pasangan kunci SSH publik dan privat tanpa passphrase:

ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa

<img width="940" height="397" alt="image" src="https://github.com/user-attachments/assets/fe3b6f8c-4ef6-41ad-bdf3-caa91654fa55" />
*Gambar 11: Proses pembuatan pasangan kunci RSA 3072-bit SSH.*

**Penjelasan Gambar 11:**
Perintah `ssh-keygen` menghasilkan berkas `id_rsa` (kunci privat) dan `id_rsa.pub` (kunci publik) untuk otentikasi aman tanpa kata sandi.

Mengonfirmasi pembuatan ulang kunci SSH (overwrite key):

<img width="940" height="512" alt="image" src="https://github.com/user-attachments/assets/ff1cbe4b-70ac-4a4b-a3e8-968c732fa0ed" />
*Gambar 12: Pembaruan fingerprint kunci SSH RSA di direktori ~/.ssh/.*

**Penjelasan Gambar 12:**
Mengonfirmasi pembuatan ulang kunci SSH untuk memastikan bahwa tidak ada konflik dengan sertifikat otentikasi lama.

Mendaftarkan kunci publik ke file `authorized_keys` dan mengubah hak aksesnya:

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

<img width="940" height="82" alt="image" src="https://github.com/user-attachments/assets/730f19c5-9fd1-45ff-bdb4-6bf0a3754507" />

*Gambar 13: Menambahkan id_rsa.pub ke authorized_keys serta mengamankan hak akses file (chmod 0600).*

**Penjelasan Gambar 13:**
Menambahkan kunci publik ke `authorized_keys` memungkinkan login SSH ke `localhost` tanpa memasukkan kata sandi. `chmod 0600` mengunci hak akses file demi keamanan.

Menguji koneksi SSH otomatis ke localhost:

ssh localhost

<img width="940" height="539" alt="image" src="https://github.com/user-attachments/assets/9185d2ca-995a-4030-af1f-c9834f2994ee" />
*Gambar 14: Keberhasilan login SSH keyless ke localhost Ubuntu WSL.*

**Penjelasan Gambar 14:**
Koneksi SSH ke `localhost` berhasil dilakukan tanpa prompt kata sandi. Hal ini menandakan bahwa otentikasi SSH *keyless* siap digunakan oleh daemon Hadoop.

---

### Tahap 4: Unduh dan Ekstrak Apache Hadoop 3.5.0

Membuat direktori kerja penyimpanan software dan masuk ke dalamnya:

mkdir -p ~/software/storage/
cd ~/software/storage/

<img width="803" height="97" alt="image" src="https://github.com/user-attachments/assets/873d9ac4-9c43-48a3-8bf0-d1000a62fc63" />
*Gambar 15: Membuat dan berpindah ke direktori ~/software/storage/.*

**Penjelasan Gambar 15:**
Direktori ini difungsikan sebagai lokasi pengunduhan dan penempatan berkas biner aplikasi Hadoop.

Mengunduh berkas biner resmi Apache Hadoop versi 3.5.0:

wget https://downloads.apache.org/hadoop/common/hadoop-3.5.0/hadoop-3.5.0.tar.gz

<img width="940" height="194" alt="image" src="https://github.com/user-attachments/assets/b0af9c78-ed19-4e7b-b277-725f0bf6a3dd" />
*Gambar 16: Pengunduhan berkas hadoop-3.5.0.tar.gz sebesar 554 MB via wget.*

**Penjelasan Gambar 16:**
Berkas arsip Hadoop 3.5.0 diunduh dari repositori resmi Apache.

Mengekstrak berkas tarball dan membuat symbolic link:

tar -xzvf hadoop-3.5.0.tar.gz
ln -s hadoop-3.5.0 hadoop

<img width="940" height="508" alt="image" src="https://github.com/user-attachments/assets/5343d4e7-6e35-49cf-a863-7a671cf27e2e" />
*Gambar 17: Ekstraksi direktori berkas hadoop-3.5.0 dan pembuatan symbolic link.*

**Penjelasan Gambar 17:**
Proses ekstraksi menguraikan direktori biner Hadoop, lalu `ln -s` membuat tautan direktori `hadoop` agar mempermudah navigasi path.

---

### Tahap 5: Daftarkan Environment Variable Hadoop

Membuat direktori penyimpanan skrip environment bash custom:

mkdir -p ~/env/bash/storage/

<img width="940" height="26" alt="image" src="https://github.com/user-attachments/assets/520f6b6c-832e-486d-8d82-9391c2480aa8" />
*Gambar 18: Pembuatan direktori ~/env/bash/storage/ untuk penampungan skrip lingkungan.*

**Penjelasan Gambar 18:**
Direktori ini digunakan untuk memisahkan skrip entri environment PATH agar mudah dikelola.

Membuat skrip `hadoop.sh` untuk mendaftarkan direktori biner Hadoop ke variabel `PATH`:

echo 'export PATH="$HOME/software/storage/hadoop/bin:$HOME/software/storage/hadoop/sbin:$PATH"' > ~/env/bash/storage/hadoop.sh

<img width="940" height="34" alt="image" src="https://github.com/user-attachments/assets/82c32851-3b86-4721-9bc5-159f9c2272ed" />
*Gambar 19: Pembuatan skrip hadoop.sh yang berisi perintah ekspor PATH Hadoop.*

**Penjelasan Gambar 19:**
Perintah ini memasukkan path `hadoop/bin` dan `hadoop/sbin` ke variabel `PATH` agar seluruh perintah Hadoop dapat dipanggil dari lokasi direktori mana pun.

Mengaktifkan skrip environment Hadoop:

source ~/env/bash/storage/hadoop.sh

<img width="940" height="50" alt="image" src="https://github.com/user-attachments/assets/0447589a-8b32-45dd-b9e0-46e997fd54f9" />
*Gambar 20: Memuat variabel PATH terbaru dari hadoop.sh ke sesi terminal.*

**Penjelasan Gambar 20:**
Perintah `source` memperbarui variabel lingkungan PATH dalam sesi aktif sehingga utilitas Hadoop langsung terdeteksi terminal.

Menguji pemanggilan CLI `hadoop` untuk mengonfirmasi pendaftaran environment:

hadoop

<img width="932" height="939" alt="image" src="https://github.com/user-attachments/assets/f59e6452-9a66-4940-9394-3ca4eed9717a" />
*Gambar 21: Tampilan panduan bantuan penggunaan CLI Hadoop (Client & Admin Commands).*

**Penjelasan Gambar 21:**
Output daftar perintah `hadoop` (seperti `fs`, `checknative`, `version`) membuktikan bahwa biner Hadoop telah sukses terdaftar pada sistem PATH.

---

### Tahap 6: Konfigurasi File XML Apache Hadoop

Berpindah ke direktori konfigurasi XML Hadoop:

cd ~/software/storage/hadoop/etc/hadoop/

<img width="940" height="56" alt="image" src="https://github.com/user-attachments/assets/458f7849-79b0-479c-9c6e-3b1465721215" />
*Gambar 22: Berpindah ke direktori ~/software/storage/hadoop/etc/hadoop/.*

**Penjelasan Gambar 22:**
Direktori ini menyimpan seluruh berkas konfigurasi XML utama untuk mengatur perilaku HDFS, YARN, dan MapReduce.

Satu per satu file XML disunting menggunakan nano editor:

1. **Konfigurasi `hdfs-site.xml`**:
   Membuka file: `nano hdfs-site.xml`
   Menambahkan properti jumlah replikasi blok HDFS menjadi 1 (karena mode *Single-Node*):

<img width="940" height="64" alt="image" src="https://github.com/user-attachments/assets/65f4e84d-c258-4124-b68d-c02c810221dc" />
<img width="940" height="687" alt="image" src="https://github.com/user-attachments/assets/ac587077-78d5-4cbe-9ab4-99436bfa9ce1" />
*Gambar 23: Pengaturan properti dfs.replication = 1 pada hdfs-site.xml.*

**Penjelasan Gambar 23:**
Parameter `dfs.replication` diisi dengan nilai `1` karena sistem berjalan pada satu node tunggal tanpa ada node duplikasi tambahan.

2. **Konfigurasi `core-site.xml`**:
   Membuka file: `nano core-site.xml`
   Menentukan URI default sistem berkas NameNode ke `hdfs://localhost:9000` dan izin proxy user:

<img width="940" height="79" alt="image" src="https://github.com/user-attachments/assets/104d51aa-3492-4599-a727-2ad2b2344ef4" />
<img width="940" height="686" alt="image" src="https://github.com/user-attachments/assets/c18e3c66-a62b-4e5d-9369-4ce05866a203" />
*Gambar 24: Pengaturan fs.defaultFS ke hdfs://localhost:9000 serta konfigurasi proxyuser pada core-site.xml.*

**Penjelasan Gambar 24:**
Nilai `fs.defaultFS` menentukan alamat Master NameNode pada `hdfs://localhost:9000`. Properti `hadoop.proxyuser` memberikan hak otorisasi akun pengguna.

3. **Konfigurasi `mapred-site.xml`**:
   Membuka file: `nano mapred-site.xml`
   Menentukan kerangka kerja MapReduce menggunakan YARN:

<img width="940" height="53" alt="image" src="https://github.com/user-attachments/assets/a318e346-4f84-44ea-8176-538dcfdd0c37" />
<img width="940" height="687" alt="image" src="https://github.com/user-attachments/assets/f631dcd4-0188-427a-89af-1c5f35112a39" />
*Gambar 25: Pengaturan mapreduce.framework.name = yarn dan classpath aplikasi pada mapred-site.xml.*

**Penjelasan Gambar 25:**
Parameter ini menginstruksikan eksekusi tugas MapReduce agar dikelola sepenuhnya oleh pengelola sumber daya YARN.

4. **Konfigurasi `yarn-site.xml`**:
   Membuka file: `nano yarn-site.xml`
   Mengaktifkan layanan *auxiliary shuffle* dan pengaturan alokasi memori NodeManager:

<img width="940" height="48" alt="image" src="https://github.com/user-attachments/assets/177a71cb-1762-453d-928c-4f6cf8709485" />
<img width="940" height="685" alt="image" src="https://github.com/user-attachments/assets/24b19321-c9fc-4415-964e-374954aecbcb" />
*Gambar 26: Pengaturan auxiliary service mapreduce_shuffle dan batas memori 4096 MB pada yarn-site.xml.*

**Penjelasan Gambar 26:**
File `yarn-site.xml` mengonfigurasi `mapreduce_shuffle`, batas alokasi memori NodeManager (`4096 MB`), serta rasio memori virtual ke fisik.

---

### Tahap 7: Inisialisasi NameNode, Memulai Daemon HDFS, dan Manajemen File Datasets

Melakukan format pada NameNode sebelum menjalankan daemon HDFS untuk pertama kali:

hdfs namenode -format

<img width="940" height="635" alt="image" src="https://github.com/user-attachments/assets/86d4d701-ff23-4c28-b82d-35622a82e8d9" />
*Gambar 27: Eksekusi perintah hdfs namenode -format untuk inisialisasi struktur direktori metadata NameNode.*

**Penjelasan Gambar 27:**
Perintah `hdfs namenode -format` menginisialisasi direktori metadata NameNode dan membuat Cluster ID baru untuk sistem berkas HDFS.

Memulai seluruh daemon HDFS (NameNode, DataNode, SecondaryNameNode):

start-dfs.sh

<img width="940" height="138" alt="image" src="https://github.com/user-attachments/assets/c0ff4a3d-8b19-4a53-a199-e8d76514f2e8" />
*Gambar 28: Menjalankan skrip start-dfs.sh untuk mengaktifkan daemon NameNode, DataNode, dan SecondaryNameNode.*

**Penjelasan Gambar 28:**
Skrip `start-dfs.sh` secara otomatis menyalakan proses NameNode, DataNode, dan SecondaryNameNode pada localhost melalui koneksi SSH.

Memeriksa proses Java Hadoop yang sedang aktif menggunakan `jps`:

jps

<img width="923" height="148" alt="image" src="https://github.com/user-attachments/assets/a925a0fd-ddea-4d36-9699-a929b475aa37" />
*Gambar 29: Tampilan output jps yang mengonfirmasi bahwa NameNode, DataNode, dan SecondaryNameNode aktif.*

**Penjelasan Gambar 29:**
Perintah `jps` (Java Virtual Machine Process Status Tool) memverifikasi bahwa ketiga daemon utama HDFS (`NameNode`, `DataNode`, dan `SecondaryNameNode`) telah berjalan normal di latar belakang.

---

### Tahap 8: Tugas Modul (Pengelolaan File Datasets di HDFS)

Membuat direktori penyimpanan dataset di lingkungan HDFS:

hdfs dfs -mkdir -p /user/acer/datasets

<img width="940" height="53" alt="image" src="https://github.com/user-attachments/assets/264a8a73-1d36-4cb5-b3b5-b588e80d7e59" />
*Gambar 30: Eksekusi perintah hdfs dfs -mkdir untuk membuat struktur folder /user/acer/datasets.*

**Penjelasan Gambar 30:**
Perintah `-mkdir -p` membuat direktori baru `/user/acer/datasets` pada sistem berkas terdistribusi HDFS.

Mengunduh file berkas CSV dataset dari repositori publik ke sistem lokal WSL:

wget https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv -O data2.csv
wget https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.csv -O data3.csv

<img width="940" height="426" alt="image" src="https://github.com/user-attachments/assets/eeac1c0f-889c-4b8a-9fd3-6a8d5e064a34" />
*Gambar 31: Proses pengunduhan berkas dataset data2.csv dan data3.csv via wget.*

**Penjelasan Gambar 31:**
File CSV lokal `data2.csv` (Titanic dataset) dan `data3.csv` (USGS Earthquake dataset) berhasil diunduh ke sistem berkas lokal.

Mengunggah file CSV dari sistem lokal ke direktori HDFS:

hdfs dfs -put data1.csv /user/acer/datasets/
hdfs dfs -put data2.csv /user/acer/datasets/
hdfs dfs -put data3.csv /user/acer/datasets/

<img width="940" height="74" alt="image" src="https://github.com/user-attachments/assets/8f1471bd-683d-4636-9424-94cb657333a8" />
*Gambar 32: Eksekusi perintah hdfs dfs -put untuk mentransfer berkas data1.csv, data2.csv, dan data3.csv ke HDFS.*

**Penjelasan Gambar 32:**
Perintah `hdfs dfs -put` menyalin berkas data dari sistem file lokal WSL ke dalam direktori terdistribusi HDFS `/user/acer/datasets/`.

Memeriksa dan menampilkan isi daftar file pada direktori HDFS:

hdfs dfs -ls /user/acer/datasets

<img width="940" height="121" alt="image" src="https://github.com/user-attachments/assets/f44d385e-e493-462b-a042-d19a2af37a5d" />
*Gambar 33: Tampilan daftar berkas terdaftar pada HDFS beserta atribut ukuran file dan izin akses.*

**Penjelasan Gambar 33:**
Perintah `hdfs dfs -ls` mengonfirmasi bahwa ketiga berkas (`data1.csv`, `data2.csv`, dan `data3.csv`) telah berhasil tersimpan di HDFS dengan faktor replikasi `1` dan pemilik `acer`.

---

## 4. Kesimpulan

1. **Instalasi & Environment**: Pengaturan OpenJDK 17, SSH Keyless, serta pendaftaran `JAVA_HOME` dan variabel `PATH` pada sistem operasi WSL berhasil diselesaikan sehingga seluruh perintah Hadoop dapat dipanggil dari terminal.
2. **Konfigurasi Cluster Single-Node**: File XML (`hdfs-site.xml`, `core-site.xml`, `mapred-site.xml`, dan `yarn-site.xml`) telah dikonfigurasi untuk menjalankan kluster *pseudo-distributed* dengan replikasi tunggal.
3. **Operasional HDFS**: Format NameNode dan eksekusi `start-dfs.sh` berjalan sukses, ditandai dengan aktifnya daemon `NameNode`, `DataNode`, dan `SecondaryNameNode` pada pengecekan `jps`.
4. **Manajemen Data**: Operasi pembuatan direktori (`-mkdir`), pengunggahan file (`-put`), serta daftar isi file (`-ls`) pada sistem berkas HDFS berjalan lancar tanpa mengalami kesalahan.
