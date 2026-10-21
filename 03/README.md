## Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Modul 2: Manajemen Proses Linux (WSL) & Implementasi GraphQL Server

### Identitas Mahasiswa
- **Nama:** Naisyah Izzatul Jannah K.
- **NIM:** 255410040
- **Kelas:** Informatika IF 1 (Semester 3)
- **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)

Berikut adalah draf bagian **Tujuan** dan **Dasar Teori** yang disusun menggunakan format Markdown yang rapi dan terstruktur (lengkap dengan heading dan daftar poin), sehingga aman untuk disalin langsung ke file `README.md` GitHub Anda tanpa berantakan:

---

## Tujuan Praktikum

1. Memahami konsep dan mekanisme sinkronisasi waktu fisik pada sistem terdistribusi menggunakan NTP (*Network Time Protocol*) melalui utilitas **Chrony**.
2. Mempelajari cara kerja **Vector Clock** untuk mengurutkan kejadian (*event*) dan melacak hubungan kausalitas secara logis tanpa bergantung sepenuhnya pada jam fisik.
3. Menganalisis permasalahan konkurensi yang timbul akibat ketiadaan sinkronisasi—khususnya **Data Race** dan **Deadlock**—serta mengimplementasikan mekanisme pencegahannya menggunakan *Locking* dan *Timeout*.
4. Memahami prinsip dasar algoritma konsensus **Raft** dalam menangani pemilihan pemimpin (*leader election*) dan manajemen replikasi log pada sistem terdistribusi.

---

## Dasar Teori

### 3.1 Sinkronisasi Waktu (NTP & Chrony)

Dalam sistem terdistribusi, setiap komputer atau node memiliki jam fisik internal (*hardware/system clock*) yang rentan mengalami pergeseran waktu (*clock drift*) akibat perbedaan perangkat keras. Untuk menjaga keselarasan waktu antar-node, digunakan protokol jaringan seperti **NTP (Network Time Protocol)** atau **PTP (Precision Time Protocol)**. **Chrony** merupakan implementasi perangkat lunak sumber terbuka yang berfungsi untuk menyinkronkan jam sistem Linux dengan server NTP eksternal secara akurat dan stabil, bahkan pada kondisi jaringan yang berfluktuasi.

### 3.2 Vector Clock

Karena jam fisik tidak pernah sepenuhnya seragam di berbagai mesin yang terdistribusi, pengurutan kejadian secara waktu nyata menjadi tidak akurat. Untuk mengatasi hal ini, digunakan **Vector Clock**. Vector clock adalah struktur data berupa array nilai integer pada setiap proses yang bertambah secara monoton. Melalui operasi pengiriman dan penerimaan pesan, jam vektor memungkinkan sistem untuk mengevaluasi hubungan kausalitas (*happens-before relationship*) dan menentukan apakah dua kejadian terjadi secara berurutan atau konkuren.

### 3.3 Masalah Konkurensi Tanpa Sinkronisasi

Model pemrograman multithreaded atau asinkron sering kali bersifat non-deterministik. Tanpa manajemen sinkronisasi yang tepat, hal ini memicu masalah utama berikut:

* **Data Race / Race Conditions:** Terjadi ketika dua atau lebih *thread* mengakses variabel bersama secara konkuren di mana setidaknya satu akses berupa operasi tulis (*write*) tanpa adanya pengamanan eksklusif (*Mutual Exclusion*). Hal ini menyebabkan data akhir menjadi korup atau salah. Solusinya adalah menggunakan objek pengunci seperti `threading.Lock`.
* **Deadlock:** Kondisi di mana sekumpulan *thread* mengalami kemacetan total (*hang*) karena saling menunggu sumber daya (*lock*) yang sedang dipegang oleh *thread* lain dalam bentuk lingkaran (*circular wait*). Pencegahan dapat dilakukan dengan memberikan batas waktu (*timeout*) saat proses penguncian.

### 3.4 Algoritma Konsensus Raft

Konsensus adalah pilar utama dalam sistem terdistribusi agar sekumpulan node dapat mencapai kesepakatan mutlak terhadap suatu nilai atau status (misalnya siapa yang menjadi pengontrol utama). **Algoritma Raft** adalah protokol konsensus yang dirancang agar lebih mudah dipahami dibandingkan Paxos. Raft membagi manajemen konsensus ke dalam tiga peran utama node:

1. **Follower:** Node pasif yang hanya merespons permintaan dari *Leader* atau *Candidate*.
2. **Candidate:** Status sementara node ketika masa tunggu pemilihan (*election timeout*) habis, di mana node akan meminta suara (*vote*) dari node lain.
3. **Leader:** Node penguasa terpilih yang bertanggung jawab penuh mengelola penambahan log (*append entries*) dan mengirimkan sinyal detak jantung (*heartbeat*) secara berkala kepada para *Follower*.
---

## Praktik
## 3.1 Sinkronisasi Waktu

### Pengantar Protokol Waktu:
Terdapat dua protokol utama yang biasanya digunakan untuk sinkronisasi waktu dalam sistem terdistribusi, yaitu NTP (Network Time Protocol) dan PTP (Precision Time Protocol).

### Langkah Kerja (Linux / BSD / Mac):
* Menginstal Chrony (https://chrony-project.org/).
* Menggunakan perintah Chrony pada terminal untuk melakukan sinkronisasi waktu ke suatu server NTP.

### Pembahasan & Penjelasan Berdasarkan Gambar Terminal:
* Perintah `chronyc tracking` dijalankan untuk memantau status sinkronisasi waktu sistem dengan rujukan jam PHC0 pada tingkat Stratum 1.
* Nilai system time menunjukkan deviasi/offset yang sangat kecil terhadap waktu NTP.
* Perintah `chronyc sources -v` digunakan untuk menampilkan daftar server/sumber NTP yang terhubung dengan status terbaik serta tingkat keterjangkauan (reach) yang stabil.

*[Lampirkan Screenshot Terminal Chrony di sini]*

---

## 3.2 Vector Clock

### Konsep Dasar:
Vector clock digunakan untuk pengurutan event (kejadian) dalam suatu sistem terdistribusi tanpa harus bergantung sepenuhnya pada jam fisik.

### Langkah Kerja:
1. Membuat berkas program Vector Clock sesuai struktur class dan logika pada modul.
2. Menjalankan simulasi program untuk mengamati perubahan nilai vektor pada beberapa proses.

### Pembahasan & Penjelasan Berdasarkan Keluaran Terminal:
* Inisialisasi awal sistem dengan 3 proses ($P_0, P_1, P_2$) menghasilkan nilai vektor `[0, 0, 0]` untuk masing-masing proses.
* Ketika suatu proses melakukan increment lokal, pengiriman pesan (`send_message`), atau penerimaan pesan (`receive_message`), nilai jam vektor diperbarui menggunakan operasi maksimum antar-komponen vektor dan penambahan nilai lokal.
* Pengujian kausalitas menunjukkan hasil bahwa `vc0 happens before vc1` bernilai **True**, sedangkan hubungan konkurensi (`vc0 is concurrent with vc2`) dievaluasi bernilai **False** karena memiliki hubungan kausal langsung.

*[Lampirkan Screenshot Kode Program vclocks.py dan Hasil Terminalnya di sini]*

---

## 3.3 Problem Tanpa Sinkronisasi

### Identifikasi Masalah:
Pada sistem terdistribusi, ketiadaan sinkronisasi dapat memicu dua masalah utama, yaitu **data race / race conditions** dan **deadlock**. Pola pikir sekuensial tidak dapat diterapkan pada model asinkron atau multithreaded karena penyelesaian task bersifat tidak bisa diprediksi (non-deterministik).

### Langkah Kerja Program Multithreaded:
1. Menjalankan program contoh multithread yang melibatkan beberapa thread tugas (`t1` sampai `t5`) dengan operasi penundaan waktu (delay).
2. Mengamati keluaran terminal hingga mendapati variasi urutan penyelesaian task.

### Pembahasan & Penjelasan Gambar:
Hasil terminal menunjukkan urutan starting serentak, namun urutan selesainya dapat bervariasi bergantung pada penjadwalan CPU (*thread scheduler*) sistem operasi.

*[Lampirkan Screenshot Multithreaded Example di sini]*

---

### 3.3.1 Data Race / Race Conditions

#### Langkah Kerja:
1. Menjalankan program simulasi penarikan saldo bersamaan tanpa pengamanan (`race-conditions-01.py`).
2. Mengamati hasil keluaran saldo akhir yang menjadi negatif.
3. Menerapkan solusi pengamanan menggunakan `threading.Lock` (`race-conditions-02.py`).

#### Pembahasan & Penjelasan Berdasarkan Visualisasi dan Terminal:
* **Tanpa lock:** Dua thread membaca variabel global saldo secara bersamaan sebelum diperbarui, sehingga kedua thread menganggap saldo mencukupi dan melakukan pengurangan ganda yang berakibat saldo bernilai negatif (`-60`). Visualisasi alur menunjukkan terjadinya overlapping pembacaan data sebelum penyimpanan nilai baru.
* **Dengan lock:** Menggunakan objek pengunci (`Lock`), blok kode transaksi diatur secara eksklusif (*mutual exclusion*), memaksa thread kedua menunggu thread pertama selesai sehingga transaksi kedua ditolak karena saldo sudah tidak mencukupi.

*[Lampirkan Screenshot Kode dan Output Data Race serta Solusi Lock di sini]*

---

### 3.3.2 Deadlock

#### Langkah Kerja:
1. Menjalankan program kondisi deadlock (`deadlock-01.py`) yang melibatkan dua kunci (`lock_a` dan `lock_b`).
2. Mengamati berhentinya program akibat kondisi saling tunggu (*hang*).
3. Menjalankan program pencegahan deadlock menggunakan timeout (`deadlock-02.py`).

#### Pembahasan & Penjelasan Berdasarkan Visualisasi dan Terminal:
* **Kondisi deadlock** terjadi karena adanya *circular wait* (tunggu melingkar): Thread 1 memegang `Lock A` dan menunggu `Lock B`, sedangkan Thread 2 memegang `Lock B` dan menunggu `Lock A`.
* **Pencegahan** dilakukan dengan memberikan parameter batas waktu (*timeout*) pada fungsi penguncian. Jika dalam batas waktu tertentu kunci tidak didapat, thread akan membatalkan proses dan melepaskan kunci yang dipegangnya melalui blok pembersihan (`finally`), sehingga program terhindar dari kemacetan total.

*[Lampirkan Screenshot Kondisi Deadlock dan Solusi Timeout di sini]*

---

## 3.4 Algoritma Raft

### Pengantar Konsensus:
Algoritma Raft digunakan dalam sistem terdistribusi untuk mencapai konsensus, seperti pemilihan koordinator (*leader election*).

### Langkah Kerja:
1. Menjalankan program simulasi konsensus Raft berbasis Python (`raft.py`).
2. Mengamati perubahan status node dari setiap langkah (*step*).

### Pembahasan & Penjelasan Berdasarkan Terminal:
* Pada tahap awal, seluruh node berstatus sebagai pengikut (*follower*).
* Jika waktu tunggu pemilihan (*election timeout*) terlampaui tanpa adanya sinyal *heartbeat* dari pemimpin, node akan mengubah status menjadi kandidat (*candidate*), menaikkan nomor term, dan meminta dukungan suara.
* Ketika memperoleh suara mayoritas, node tersebut resmi terpilih menjadi pemimpin (*leader*) dan mulai mengirimkan sinyal *heartbeat* serta mengelola penambahan log (*append entries*).

*[Lampirkan Screenshot Output Simulasi Raft dari Step awal hingga akhir di sini]*

---

## Kesimpulan

1. **Sinkronisasi waktu fisik** menggunakan NTP (Chrony) memastikan keselarasan waktu antar-sistem dengan tingkat presisi yang tinggi.
2. **Penggunaan Vector Clock** memungkinkan pelacakan hubungan kausalitas dan urutan kejadian logis dalam lingkungan terdistribusi secara akurat.
3. **Ketiadaan sinkronisasi dan manajemen konkurensi** dapat menimbulkan masalah fatal seperti Data Race dan Deadlock, yang dapat diatasi secara efektif menggunakan mekanisme Locking (*Mutual Exclusion*) dan Timeout.
4. **Algoritma Raft** menyediakan pendekatan konsensus yang terstruktur dan handal untuk mengelola pemilihan pemimpin (*leader*) serta replikasi log pada sistem terdistribusi.
