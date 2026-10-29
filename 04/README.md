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

<img width="940" height="319" alt="image" src="https://github.com/user-attachments/assets/d36835cb-e554-433d-82cb-0596b196c2c6" />

* **Pembahasan:**
  Pemeriksaan versi Docker Engine dan Docker Compose dilakukan untuk memastikan seluruh utilitas kontainerisasi telah terinstal dan berjalan normal pada lingkungan WSL. Output menampilkan Docker versi `29.9.2` dan Docker Compose `v5.6.1` siap digunakan untuk mengorkestrasi node PostgreSQL.

---

### 4.1.2 Persiapan File Konfigurasi

#### Langkah 1: Membuat Direktori Kerja

<img width="940" height="99" alt="image" src="https://github.com/user-attachments/assets/931401b6-9048-476a-b84f-d0f22024741d" />

* **Pembahasan:**
  Proses pembuatan direktori kerja `streaming-replication` dilakukan agar seluruh skrip dan konfigurasi replikasi fisik tersimpan secara terisolasi dan rapi.

#### Langkah 2: Script Inisialisasi Primary (`00_init.sql`)

<img width="940" height="148" alt="image" src="https://github.com/user-attachments/assets/98bf4f7f-bfc6-49d0-b4b3-6a304a814c6d" />

* **Pembahasan:**
  Penyusunan skrip SQL inisialisasi pada node Primary bertugas membuat pengguna `replicator` dengan otorisasi `REPLICATION` serta mendaftarkan slot replikasi fisik bernama `replication_slot`. Slot ini berfungsi mencegah node Primary menghapus log WAL sebelum disinkronisasikan ke node Replica.

#### Langkah 3: Konfigurasi Otentikasi (`01_hba.sh`)

<img width="940" height="187" alt="image" src="https://github.com/user-attachments/assets/13cc189e-d3fa-4cd6-842b-8ee3fa39270e" />

* **Pembahasan:**
  Skrip shell dibuat untuk menambahkan aturan izin akses jaringan pada file `pg_hba.conf`. Konfigurasi ini mengizinkan otentikasi koneksi replikasi dari node luar menggunakan protokol enkripsi `scram-sha-256`.

#### Langkah 4: File Docker Compose (`docker-compose.yaml`)

<img width="1053" height="778" alt="image" src="https://github.com/user-attachments/assets/47e564c1-b0cd-4c4f-b413-629d3f6768f0" />

* **Pembahasan:**
  Pengaturan orkestrasi dua kontainer PostgreSQL disiapkan. Node `postgres_primary` berjalan pada port 5432 dengan mode `wal_level=replica`, sedangkan node `postgres_replica` pada port 5433 melakukan penyalinan data awal secara fisik (*physical base backup*) menggunakan perintah `pg_basebackup` dari Primary sebelum memulai layanannya.

#### Langkah 5: Membuat Alias Shell (`env.sh`)

<img width="935" height="245" alt="image" src="https://github.com/user-attachments/assets/5f59e16b-80fe-4760-9a6b-44d56c8bafec" />

* **Pembahasan:**
  File variabel lingkungan `env.sh` disusun untuk menyediakan perintah singkat (*alias*) guna mempermudah proses penyalaan, pematian, pembuatan shell interaktif, hingga pemantauan log kontainer.

---

### 4.1.3 Menjalankan dan Memeriksa Container

#### Langkah 1: Menyalakan Service

<img width="814" height="210" alt="image" src="https://github.com/user-attachments/assets/2d65441c-f8b1-46fb-bfc9-858d6e2ea427" />

* **Pembahasan:**
  Eksekusi perintah penyalaan kontainer berhasil mengaktifkan jaringan virtual `streaming-replication_default` beserta dua kontainer `postgres_primary` dan `postgres_replica` secara latar belakang (*detached mode*).

#### Langkah 2: Verifikasi Status Healthy

<img width="1034" height="130" alt="image" src="https://github.com/user-attachments/assets/0de74f0e-2a2e-4042-b495-0c095ef9f753" />

* **Pembahasan:**
  Pengecekan status kontainer menunjukkan kedua layanan dalam kondisi `Up (healthy)`. Hal ini menandakan tahapan `pg_basebackup` pada Replica telah selesai dengan sukses dan kedua basis data siap melayani kueri.

---

### 4.1.4 Verifikasi & Pengujian Streaming Replication

#### Langkah 1 & 2: Masuk ke Primary dan Cek Status Replikasi

<img width="939" height="144" alt="image" src="https://github.com/user-attachments/assets/20fac8e2-003c-4665-b19b-0ab4c486673a" />
<img width="940" height="142" alt="image" src="https://github.com/user-attachments/assets/106ca3be-7cfb-4db8-98f1-9ba713486a52" />


* **Pembahasan:**
  Pengujian status koneksi replikasi melalui kueri tabel sistem `pg_stat_replication` di Primary menampilkan IP client `172.18.0.3` dengan `state = streaming` dan `sync_state = async`. Ini mengonfirmasi bahwa alur replikasi fisik asinkron telah terhubung dengan sempurna.

#### Langkah 3 & 4: Masuk ke Primary dan Tambahkan Data Baru

<img width="940" height="98" alt="image" src="https://github.com/user-attachments/assets/8aaedf08-ed1c-4afe-ae99-def0c3629b91" />
<img width="867" height="98" alt="image" src="https://github.com/user-attachments/assets/f640f3c9-6f8d-4af7-ad85-2e9975e1ee6a" />
<img width="940" height="127" alt="image" src="https://github.com/user-attachments/assets/7fe4fe92-2835-4650-936d-02a6feacdbdb" />


* **Pembahasan:**
  Pembuatan tabel `tes_replikasi` serta pengisian data sampel (`'Data Pertama'` dan `'Data Kedua'`) berhasil dieksekusi pada node Primary.

#### Langkah 5: Masuk ke Replica dan Verifikasi Data Ter-replikasi

<img width="940" height="66" alt="image" src="https://github.com/user-attachments/assets/491d2e76-6d76-4afd-be25-34c6097dded3" />
<img width="940" height="247" alt="image" src="https://github.com/user-attachments/assets/f402e90d-2926-4e46-b694-a5672c66339f" />

* **Pembahasan:**
  Pemeriksaan data pada node Replica membuktikan bahwa seluruh struktur tabel dan isi baris data dari Primary terduplikasi secara otomatis dan akurat secara *real-time*.

#### Langkah 6: Uji Sifat Read-Only pada Node Replica

<img width="789" height="70" alt="image" src="https://github.com/user-attachments/assets/540f992c-161d-447a-bcd8-e640d6b3947c" />
<img width="940" height="120" alt="image" src="https://github.com/user-attachments/assets/3acb4a2c-6938-4c3f-925a-3bdbb76f44f4" />


* **Pembahasan:**
  Pengujian eksekusi perintah `INSERT` pada node Replica menghasilkan pesan kesalahan `ERROR: cannot execute INSERT in a read-only transaction`. Hasil ini menguji dan membuktikan karakteristik utama *Streaming Replication* di mana node Replica bersifat murni **Read-Only**.

#### Langkah 7: Membersihkan Environment Streaming Replication

<img width="940" height="106" alt="image" src="https://github.com/user-attachments/assets/cd5dde48-c59e-4e87-b6c5-7f3091ad56e7" />

* **Pembahasan:**
  Proses pembersihan environment berhasil mematikan serta menghapus seluruh kontainer dan jaringan virtual *Streaming Replication*.

---

## Part 4.2: Logical Replication

### 4.2.1 Konfigurasi & Pengujian Logical Replication

#### Langkah 1: Persiapan File `docker-compose.yaml`

<img width="940" height="565" alt="image" src="https://github.com/user-attachments/assets/b42a3107-8b69-495c-8c2d-83ea72e8eee5" />

* **Pembahasan:**
  Penyusunan arsitektur *Logical Replication* menggunakan dua kontainer independen (`postgres_pub` dan `postgres_sub`). Node Publisher dikonfigurasi khusus dengan opsi `wal_level=logical`.

#### Langkah 2: Membuat File Alias `env.sh`

<img width="940" height="272" alt="image" src="https://github.com/user-attachments/assets/f1edce0a-55e8-4e5f-a311-9b54a1bea2ec" />

* **Pembahasan:**
  File alias lingkungan disiapkan kembali untuk mempermudah navigasi perintah CLI pada direktori proyek *Logical Replication*.

#### Langkah 3: Menyalakan Service & Verifikasi Status

<img width="940" height="110" alt="image" src="https://github.com/user-attachments/assets/d0344e77-f438-4e76-b990-dab272945d33" />
<img width="940" height="128" alt="image" src="https://github.com/user-attachments/assets/07b20076-2eef-4606-88a4-963c6d0e33b5" />


* **Pembahasan:**
  Eksekusi pengaktifan layanan berhasil menyalakan kontainer Publisher dan Subscriber hingga keduanya mencapai status `healthy`.

#### Langkah 4: Setup Publisher (`postgres_pub`)

<img width="940" height="60" alt="image" src="https://github.com/user-attachments/assets/3b118184-778b-442b-8b99-60aa778db510" />
<img width="940" height="177" alt="image" src="https://github.com/user-attachments/assets/56dcdeca-b3a7-4d4b-aa5a-5945354d9180" />

* **Pembahasan:**
  Pada node Publisher, tabel `users` dibuat lalu didaftarkan ke dalam publikasi data menggunakan perintah `CREATE PUBLICATION my_pub FOR TABLE users;`. Setelah itu, dua data sampel (`'Budi'` dan `'Siti'`) dimasukkan.

#### Langkah 5: Setup Subscriber (`postgres_sub`)

<img width="860" height="73" alt="image" src="https://github.com/user-attachments/assets/a1f30c66-1a76-4102-9b22-56a6bf439cb7" />
<img width="940" height="155" alt="image" src="https://github.com/user-attachments/assets/51eacda8-a49e-4d07-adfe-78901a31f398" />

* **Pembahasan:**
  Skema tabel `users` dibuat secara mandiri pada node Subscriber, kemudian dihubungkan ke Publisher melalui perintah `CREATE SUBSCRIPTION`. Sistem menampilkan konfirmasi keberhasilan pembuatan *replication slot* pada Publisher.

#### Langkah 6: Verifikasi Replikasi Data pada Subscriber

<img width="791" height="57" alt="image" src="https://github.com/user-attachments/assets/b91cc1dc-b1c0-45dc-899a-a986aa872644" />
<img width="940" height="293" alt="image" src="https://github.com/user-attachments/assets/7f49a396-c14d-42b1-9185-dc30b2232846" />

* **Pembahasan:**
  Eksekusi kueri `SELECT` pada Subscriber menunjukkan data `'Budi'` dan `'Siti'` telah otomatis tereplikasi dari Publisher melalui mekanisme Replikasi Logis.

#### Langkah 7: Pengujian Read-Write & Kendala Conflict Sequence

<img width="772" height="66" alt="image" src="https://github.com/user-attachments/assets/e013cd59-701f-4eb8-a2f6-f76fe5ad41d1" />
<img width="940" height="264" alt="image" src="https://github.com/user-attachments/assets/fe379c93-b7cb-4529-8473-7bc313171ead" />

* **Pembahasan:**
  Upaya penyisipan data `'Cici'` secara lokal pada Subscriber menghasilkan kesalahan `ERROR: duplicate key value violates unique constraint "users_pkey"`. Hal ini terjadi karena *Logical Replication* hanya mentransfer data baris (*row-level*), sehingga pointer pencacah *sequence ID* lokal pada Subscriber belum tersinkronkan dengan data hasil replikasi.

#### Langkah 8: Sinkronisasi Sequence & Keberhasilan Read-Write

<img width="817" height="104" alt="image" src="https://github.com/user-attachments/assets/fc1407c4-7536-42a6-abb0-12f172bb0a0e" />
<img width="940" height="332" alt="image" src="https://github.com/user-attachments/assets/8d835879-8e1e-4d25-b0e0-360939ecdc3c" />

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
