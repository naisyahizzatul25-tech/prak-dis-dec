#### Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Modul 2: Manajemen Proses Linux (WSL) & Implementasi GraphQL Server

### Identitas Mahasiswa
- **Nama:** Naisyah Izzatul Jannah K.
- **NIM:** 255410040
- **Kelas:** Informatika IF 1 (Semester 3)
- **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)

---

Laporan Praktikum Sistem Terdistribusi

3.1 Sinkronisasi Waktu

Pengantar Protokol Waktu

Terdapat dua protokol utama yang biasanya digunakan untuk sinkronisasi waktu dalam sistem terdistribusi, yaitu NTP (Network Time Protocol) dan PTP (Precision Time Protocol).

Langkah Kerja (Linux / BSD / Mac)

Menginstal Chrony (https://chrony-project.org/).

Menggunakan perintah Chrony pada terminal untuk melakukan sinkronisasi waktu ke suatu server NTP.

Pembahasan & Penjelasan Berdasarkan Gambar Terminal

Perintah chronyc tracking dijalankan untuk memantau status sinkronisasi waktu sistem dengan rujukan jam PHC0 pada tingkat Stratum 1.

Nilai system time menunjukkan deviasi/offset yang sangat kecil terhadap waktu NTP.

Perintah chronyc sources -v digunakan untuk menampilkan daftar server/sumber NTP yang terhubung dengan status terbaik serta tingkat keterjangkauan (reach) yang stabil.

[Lampirkan Screenshot Terminal Chrony di sini]

3.2 Vector Clock

Konsep Dasar

Vector clock digunakan untuk pengurutan event (kejadian) dalam suatu sistem terdistribusi tanpa harus bergantung sepenuhnya pada jam fisik.

Langkah Kerja

Membuat berkas program Vector Clock sesuai struktur class dan logika pada modul.

Menjalankan simulasi program untuk mengamati perubahan nilai vektor pada beberapa proses.

Pembahasan & Penjelasan Berdasarkan Keluaran Terminal

Inisialisasi awal sistem dengan 3 proses ($P_0, P_1, P_2$) menghasilkan nilai vektor [0, 0, 0] untuk masing-masing proses.

Ketika suatu proses melakukan increment lokal, pengiriman pesan (send_message), atau penerimaan pesan (receive_message), nilai jam vektor diperbarui menggunakan operasi maksimum antar-komponen vektor dan penambahan nilai lokal.

Pengujian kausalitas menunjukkan hasil bahwa vc0 happens before vc1 bernilai True, sedangkan hubungan konkurensi (vc0 is concurrent with vc2) dievaluasi bernilai False karena memiliki hubungan kausal langsung.

[Lampirkan Screenshot Kode Program vclocks.py dan Hasil Terminalnya di sini]

3.3 Problem Tanpa Sinkronisasi

Identifikasi Masalah

Pada sistem terdistribusi, ketiadaan sinkronisasi dapat memicu dua masalah utama, yaitu data race / race conditions dan deadlock. Pola pikir sekuensial tidak dapat diterapkan pada model asinkron atau multithreaded karena penyelesaian task bersifat tidak bisa diprediksi (non-deterministik).

Langkah Kerja Program Multithreaded

Menjalankan program contoh multithread yang melibatkan beberapa thread tugas (t1 sampai t5) dengan operasi penundaan waktu (delay).

Mengamati keluaran terminal hingga mendapati variasi urutan penyelesaian task.

Pembahasan & Penjelasan Gambar

Hasil terminal menunjukkan urutan starting serentak, namun urutan selesainya dapat bervariasi bergantung pada penjadwalan CPU (thread scheduler) sistem operasi.

[Lampirkan Screenshot Multithreaded Example di sini]

3.3.1 Data Race / Race Conditions

Langkah Kerja

Menjalankan program simulasi penarikan saldo bersamaan tanpa pengamanan (race-conditions-01.py).

Mengamati hasil keluaran saldo akhir yang menjadi negatif.

Menerapkan solusi pengamanan menggunakan threading.Lock (race-conditions-02.py).

Pembahasan & Penjelasan Berdasarkan Visualisasi dan Terminal

Tanpa Lock: Dua thread membaca variabel global saldo secara bersamaan sebelum diperbarui, sehingga kedua thread menganggap saldo mencukupi dan melakukan pengurangan ganda yang berakibat saldo bernilai negatif (-60). Visualisasi alur menunjukkan terjadinya overlapping pembacaan data sebelum penyimpanan nilai baru.

Dengan Lock: Dengan menambahkan objek pengunci (Lock), blok kode transaksi diatur secara eksklusif (mutual exclusion), memaksa thread kedua menunggu thread pertama selesai sehingga transaksi kedua ditolak karena saldo sudah tidak mencukupi.

[Lampirkan Screenshot Kode dan Output Data Race serta Solusi Lock di sini]

3.3.2 Deadlock

Langkah Kerja

Menjalankan program kondisi deadlock (deadlock-01.py) yang melibatkan dua kunci (lock_a dan lock_b).

Mengamati berhentinya program akibat kondisi saling tunggu (hang).

Menjalankan program pencegahan deadlock menggunakan timeout (deadlock-02.py).

Pembahasan & Penjelasan Berdasarkan Visualisasi dan Terminal

Kondisi Deadlock: Terjadi karena adanya circular wait (tunggu melingkar): Thread 1 memegang Lock A dan menunggu Lock B, sedangkan Thread 2 memegang Lock B dan menunggu Lock A.

Solusi Timeout: Pencegahan dilakukan dengan memberikan parameter batas waktu (timeout) pada fungsi penguncian. Jika dalam batas waktu tertentu kunci tidak didapat, thread akan membatalkan proses dan melepaskan kunci yang dipegangnya melalui blok pembersihan (finally), sehingga program terhindar dari kemacetan total.

[Lampirkan Screenshot Kondisi Deadlock dan Solusi Timeout di sini]

3.4 Algoritma Raft

Pengantar Konsensus

Algoritma Raft digunakan dalam sistem terdistribusi untuk mencapai konsensus, seperti pemilihan koordinator (leader election).

Langkah Kerja

Menjalankan program simulasi konsensus Raft berbasis Python (raft.py).

Mengamati perubahan status node dari setiap langkah (step).

Pembahasan & Penjelasan Berdasarkan Terminal

Pada tahap awal, seluruh node berstatus sebagai pengikut (follower).

Jika waktu tunggu pemilihan (election timeout) terlampaui tanpa adanya sinyal heartbeat dari pemimpin, node akan mengubah status menjadi kandidat (candidate), menaikkan nomor term, dan meminta dukungan suara.

Ketika memperoleh suara mayoritas, node tersebut resmi terpilih menjadi pemimpin (leader) dan mulai mengirimkan sinyal heartbeat serta mengelola penambahan log (append entries).

[Lampirkan Screenshot Output Simulasi Raft dari Step awal hingga akhir di sini]

Kesimpulan

Sinkronisasi Waktu Fisik: Penggunaan NTP (Chrony) memastikan keselarasan waktu antar-sistem dengan tingkat presisi yang tinggi.

Vector Clock: Memungkinkan pelacakan hubungan kausalitas dan urutan kejadian logis dalam lingkungan terdistribusi secara akurat.

Manajemen Konkurensi: Ketiadaan sinkronisasi dapat menimbulkan masalah fatal seperti Data Race dan Deadlock, yang dapat diatasi secara efektif menggunakan mekanisme Locking (Mutual Exclusion) dan Timeout.

Algoritma Konsensus: Algoritma Raft menyediakan pendekatan konsensus yang terstruktur dan handal untuk mengelola pemilihan pemimpin (leader) serta replikasi log pada sistem terdistribusi.

