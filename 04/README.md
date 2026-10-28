# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Modul 4: Replikasi pada Sistem Terdistribusi (PostgreSQL Replication)

### Identitas Mahasiswa

* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1 (Semester 3)
* **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)

---

### Tujuan Praktikum

1. Memahami konsep dasar dan arsitektur Replikasi Database (*Streaming Replication* dan *Logical Replication*) pada PostgreSQL.
2. Mengkonfigurasi Replikasi Fisik (*Streaming Replication*) menggunakan Docker Compose dan menguji sifat *Read-Only* pada node replica.
3. Mengkonfigurasi Replikasi Logis (*Logical Replication*) dengan skema *Publisher-Subscriber* serta menguji sifat *Read-Write* dan penanganan konflik *sequence* pada node subscriber.
4. Menganalisis perbedaan karakteristik, kelebihan, serta skenario penggunaan antara *Streaming Replication* dan *Logical Replication*.

---

## Part 4.1: Physical / Streaming Replication

### 4.1.1 Verifikasi Instalasi Docker

<img src="img/4.1.1.png" width="800" />

* **Pembahasan:**
  Pemeriksaan versi Docker Engine dan Docker Compose dilakukan untuk memastikan seluruh utilitas kontainerisasi telah terinstal dan berjalan normal pada lingkungan WSL. Output menampilkan Docker versi `29.9.2` dan Docker Compose `v5.6.1` siap digunakan untuk mengorkestrasi node PostgreSQL.

---

### 4.1.2 Persiapan File Konfigurasi

#### Langkah 1: Membuat Direktori Kerja

<img src="img/4.1.2-langkah1.png" width="800" />

* **Pembahasan:**
  Proses pembuatan direktori kerja `streaming-replication` dilakukan agar seluruh skrip dan konfigurasi replikasi fisik tersimpan secara terisolasi dan rapi.

#### Langkah 2: Script Inisialisasi Primary (`00_init.sql`)

<img src="img/4.1.2-langkah2.png" width="800" />

* **Pembahasan:**
  Penyusunan skrip SQL inisialisasi pada node Primary bertugas membuat pengguna `replicator` dengan otorisasi `REPLICATION` serta mendaftarkan slot replikasi fisik bernama `replication_slot`. Slot ini berfungsi mencegah node Primary menghapus log WAL sebelum disinkronisasikan ke node Replica.

#### Langkah 3: Konfigurasi Otentikasi (`01_hba.sh`)

<img src="img/4.1.2-langkah3.png" width="800" />

* **Pembahasan:**
  Skrip shell dibuat untuk menambahkan aturan izin akses jaringan pada file `pg_hba.conf`. Konfigurasi ini mengizinkan otentikasi koneksi replikasi dari node luar menggunakan protokol enkripsi `scram-sha-256`.

#### Langkah 4: File Docker Compose (`docker-compose.yaml`)

<img src="img/4.1.2-langkah4.png" width="800" />

* **Pembahasan:**
  Pengaturan orkestrasi dua kontainer PostgreSQL disiapkan. Node `postgres_primary` berjalan pada port 5432 dengan mode `wal_level=replica`, sedangkan node `postgres_replica` pada port 5433 melakukan penyalinan data awal secara fisik (*physical base backup*) menggunakan perintah `pg_basebackup` dari Primary sebelum memulai layanannya.

#### Langkah 5: Membuat Alias Shell (`env.sh`)

<img src="img/4.1.2-langkah5.png" width="800" />

* **Pembahasan:**
  File variabel lingkungan `env.sh` disusun untuk menyediakan perintah singkat (*alias*) guna mempermudah proses penyalaan, pematian, pembuatan shell interaktif, hingga pemantauan log kontainer.

---

### 4.1.3 Menjalankan dan Memeriksa Container

#### Langkah 1: Menyalakan Service

<img src="img/4.1.3-langkah1.png" width="800" />

* **Pembahasan:**
  Eksekusi perintah penyalaan kontainer berhasil mengaktifkan jaringan virtual `streaming-replication_default` beserta dua kontainer `postgres_primary` dan `postgres_replica` secara latar belakang (*detached mode*).

#### Langkah 2: Verifikasi Status Healthy

<img src="img/4.1.3-langkah2.png" width="800" />

* **Pembahasan:**
  Pengecekan status kontainer menunjukkan kedua layanan dalam kondisi `Up (healthy)`. Hal ini menandakan tahapan `pg_basebackup` pada Replica telah selesai dengan sukses dan kedua basis data siap melayani kueri.

---

### 4.1.4 Verifikasi & Pengujian Streaming Replication

#### Langkah 1 & 2: Masuk ke Primary dan Cek Status Replikasi

<img src="img/4.1.4-langkah2.png" width="800" />

* **Pembahasan:**
  Pengujian status koneksi replikasi melalui kueri tabel sistem `pg_stat_replication` di Primary menampilkan IP client `172.18.0.3` dengan `state = streaming` dan `sync_state = async`. Ini mengonfirmasi bahwa alur replikasi fisik asinkron telah terhubung dengan sempurna.

#### Langkah 3 & 4: Masuk ke Primary dan Tambahkan Data Baru

<img src="img/4.1.4-langkah4.png" width="800" />

* **Pembahasan:**
  Pembuatan tabel `tes_replikasi` serta pengisian data sampel (`'Data Pertama'` dan `'Data Kedua'`) berhasil dieksekusi pada node Primary.

#### Langkah 5: Masuk ke Replica dan Verifikasi Data Ter-replikasi

<img src="img/4.1.4-langkah5.png" width="800" />

* **Pembahasan:**
  Pemeriksaan data pada node Replica membuktikan bahwa seluruh struktur tabel dan isi baris data dari Primary terduplikasi secara otomatis dan akurat secara *real-time*.

#### Langkah 6: Uji Sifat Read-Only pada Node Replica

<img src="img/4.1.4-langkah6.png" width="800" />

* **Pembahasan:**
  Pengujian eksekusi perintah `INSERT` pada node Replica menghasilkan pesan kesalahan `ERROR: cannot execute INSERT in a read-only transaction`. Hasil ini menguji dan membuktikan karakteristik utama *Streaming Replication* di mana node Replica bersifat murni **Read-Only**.

#### Langkah 7: Membersihkan Environment Streaming Replication

<img src="img/4.1.4-langkah7.png" width="800" />

* **Pembahasan:**
  Proses pembersihan environment berhasil mematikan serta menghapus seluruh kontainer dan jaringan virtual *Streaming Replication*.

---

## Part 4.2: Logical Replication

### 4.2.1 Konfigurasi & Pengujian Logical Replication

#### Langkah 1: Persiapan File `docker-compose.yaml`

<img src="img/4.2.1-docker-compose.png" width="800" />

* **Pembahasan:**
  Penyusunan arsitektur *Logical Replication* menggunakan dua kontainer independen (`postgres_pub` dan `postgres_sub`). Node Publisher dikonfigurasi khusus dengan opsi `wal_level=logical`.

#### Langkah 2: Membuat File Alias `env.sh`

<img src="img/4.2.1-env.png" width="800" />

* **Pembahasan:**
  File alias lingkungan disiapkan kembali untuk mempermudah navigasi perintah CLI pada direktori proyek *Logical Replication*.

#### Langkah 3: Menyalakan Service & Verifikasi Status

<img src="img/4.2.1-dcu-dps.png" width="800" />

* **Pembahasan:**
  Eksekusi pengaktifan layanan berhasil menyalakan kontainer Publisher dan Subscriber hingga keduanya mencapai status `healthy`.

#### Langkah 4: Setup Publisher (`postgres_pub`)

<img src="img/4.2.1-publisher-setup.png" width="800" />

* **Pembahasan:**
  Pada node Publisher, tabel `users` dibuat lalu didaftarkan ke dalam publikasi data menggunakan perintah `CREATE PUBLICATION my_pub FOR TABLE users;`. Setelah itu, dua data sampel (`'Budi'` dan `'Siti'`) dimasukkan.

#### Langkah 5: Setup Subscriber (`postgres_sub`)

<img src="img/4.2.1-subscriber-setup.png" width="800" />

* **Pembahasan:**
  Skema tabel `users` dibuat secara mandiri pada node Subscriber, kemudian dihubungkan ke Publisher melalui perintah `CREATE SUBSCRIPTION`. Sistem menampilkan konfirmasi keberhasilan pembuatan *replication slot* pada Publisher.

#### Langkah 6: Verifikasi Replikasi Data pada Subscriber

<img src="img/4.2.1-select-subscriber.png" width="800" />

* **Pembahasan:**
  Eksekusi kueri `SELECT` pada Subscriber menunjukkan data `'Budi'` dan `'Siti'` telah otomatis tereplikasi dari Publisher melalui mekanisme Replikasi Logis.

#### Langkah 7: Pengujian Read-Write & Kendala Conflict Sequence

<img src="img/4.2.1-insert-error.png" width="800" />

* **Pembahasan:**
  Upaya penyisipan data `'Cici'` secara lokal pada Subscriber menghasilkan kesalahan `ERROR: duplicate key value violates unique constraint "users_pkey"`. Hal ini terjadi karena *Logical Replication* hanya mentransfer data baris (*row-level*), sehingga pointer pencacah *sequence ID* lokal pada Subscriber belum tersinkronkan dengan data hasil replikasi.

#### Langkah 8: Sinkronisasi Sequence & Keberhasilan Read-Write

<img src="img/4.2.1-sequence-success.png" width="800" />

* **Pembahasan:**
  Setelah nilai pencacah *sequence* diperbarui mengikuti nilai ID maksimum (`setval`), operasi penambahan data `'Cici'` sukses dijalankan. Pengujian ini membuktikan bahwa node Subscriber pada *Logical Replication* bersifat **Read-Write** dan dapat menerima penulisan data lokal secara penuh.

---

## Kesimpulan

1. **Streaming Replication (Physical Replication):**
   * Bekerja pada tingkat fisik (*byte-by-byte*) dengan menduplikasi seluruh direktori basis data serta log WAL dari node Primary ke Replica.
   * Node Replica berfungsi sebagai *Hot Standby* yang bersifat murni **Read-Only** (tidak dapat menerima penulisan data).
   * Sangat handal digunakan untuk kebutuhan *High Availability* (HA), *Disaster Recovery*, dan penyeimbangan beban kueri pembacaan (*Read Scaling*).

2. **Logical Replication:**
   * Bekerja pada tingkat objek/tabel (*row-level*) menggunakan arsitektur *Publisher-Subscriber*.
   * Memungkinkan replikasi data secara selektif (hanya tabel tertentu yang didaftarkan pada publikasi).
   * Node Subscriber bersifat **Read-Write**, sehingga tetap dapat menerima operasi manipulasi data lokal.
   * Membutuhkan penyesuaian manual pada nilai *sequence* kolom auto-increment (`SERIAL`) jika dilakukan penulisan data lokal pada node Subscriber untuk menghindari konflik *Primary Key*.
   * Sangat ideal untuk skenario migrasi versi basis data, integrasi data antar sistem berbeda, serta konsolidasi data dari berbagai sumber.
