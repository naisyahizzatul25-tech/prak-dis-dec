
Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

Modul 4: Replikasi pada Sistem Terdistribusi (PostgreSQL Replication)

Identitas Mahasiswa

Nama: Naisyah Izzatul Jannah K.

NIM: 255410040

Kelas: Informatika IF 1 (Semester 3)

Universitas: Universitas Teknologi Digital Indonesia (UTDI)

Tujuan Praktikum

Memahami konsep dasar dan arsitektur Replikasi Database (Streaming Replication dan Logical Replication) pada PostgreSQL.

Mampu mengonfigurasi Replikasi Fisik (Streaming Replication) menggunakan Docker Compose dan menguji sifat Read-Only pada node replica.

Mampu mengonfigurasi Replikasi Logis (Logical Replication) dengan skema Publisher-Subscriber dan menguji sifat Read-Write pada node subscriber.

Mampu menangani konflik data serta melakukan sinkronisasi Sequence ID pada node subscriber.

Part 4.1: Physical / Streaming Replication

4.1.1 Verifikasi Lingkungan Docker

Pemeriksaan awal dilakukan untuk memastikan kesiapan perangkat lunak Docker Engine dan Docker Compose pada lingkungan WSL (Windows Subsystem for Linux).

Command:

docker --version && docker compose version


Screenshot Output:

Pembahasan:
Sistem mengembalikan informasi Docker version 29.9.2 dan Docker Compose version v5.6.1. Hal ini menandakan engine containerization dan orkestrator compose sudah terpasang dengan benar dan siap mengelola service multi-container.

4.1.2 Pembuatan File Konfigurasi Streaming Replication

Langkah 1: Persiapan Direktori Kerja

Command:

mkdir -p streaming-replication && cd streaming-replication


Screenshot Output:

Pembahasan:
Perintah ini membuat direktori terisolasi bernama streaming-replication dan berpindah ke dalamnya. Isolasi direktori mencegah terjadinya bentrok file konfigurasi dengan modul praktikum lainnya.

Langkah 2: Inisialisasi Script Replikasi (00_init.sql)

Command:

cat << 'EOF' > 00_init.sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
EOF


Screenshot Output:

Pembahasan:
File SQL ini digunakan saat bootstrapping node primary:

CREATE USER replicator WITH REPLICATION: Membuat role khusus dengan atribut REPLICATION agar dapat membaca stream WAL (Write-Ahead Logging).

pg_create_physical_replication_slot: Membuat physical replication slot bernama 'replication_slot'. Slot ini memastikan node primary menahan log WAL sampai node replica mengonfirmasi penerimaan data, mencegah data loss jika jaringan tidak stabil.

Langkah 3: Konfigurasi Akses Host (01_hba.sh)

Command:

cat << 'EOF' > 01_hba.sh
#!/bin/bash
echo "host replication replicator all scram-sha-256" >> "$PGDATA/pg_hba.conf"
EOF
chmod +x 01_hba.sh


Screenshot Output:

Pembahasan:
Script shell ini menambahkan aturan otentikasi ke file pg_hba.conf pada node primary. Rule host replication replicator all scram-sha-256 mengizinkan user replicator melakukan koneksi replikasi dari IP mana pun (all) dengan metode otentikasi terenkripsi scram-sha-256. Hak akses eksekusi ditambahkan menggunakan chmod +x.

Langkah 4: Konfigurasi Docker Compose (docker-compose.yaml)

Command:

cat << 'EOF' > docker-compose.yaml
x-postgres-common: &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U zuser -d zdb"]
    interval: 10s
    timeout: 5s
    retries: 5

services:
  postgres_primary:
    <<: *postgres-common
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: zuser
      POSTGRES_DB: zdb
      POSTGRES_PASSWORD: zpass
      POSTGRES_HOST_AUTH_METHOD: scram-sha-256
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    command:
      - postgres
      - -c
      - wal_level=replica
      - -c
      - hot_standby=on
      - -c
      - max_wal_senders=10
      - -c
      - max_replication_slots=10
      - -c
      - hot_standby_feedback=on
    volumes:
      - ./00_init.sql:/docker-entrypoint-initdb.d/00_init.sql
      - ./01_hba.sh:/docker-entrypoint-initdb.d/01_hba.sh

  postgres_replica:
    <<: *postgres-common
    ports:
      - 5433:5432
    environment:
      PGUSER: replicator
      PGPASSWORD: replicator_password
      PGDATA: /var/lib/postgresql/18/docker
    command:
      - bash
      - -c
      - |
        until pg_basebackup --pgdata=/var/lib/postgresql/18/docker -R --slot=replication_slot --host=postgres_primary --port=5432 -U replicator -X stream; do
          echo 'pg_basebackup failed. Retrying in 5 seconds...'
          sleep 5
        done
        echo 'Backup done, starting replica...'
        chmod 0700 /var/lib/postgresql/18/docker
        postgres
    depends_on:
      - postgres_primary
EOF


Screenshot Output:

Pembahasan:

postgres_primary: Mengaktifkan parameter utama engine PostgreSQL untuk streaming replication, yaitu wal_level=replica, hot_standby=on, serta mengonfigurasi batas pengiriman log WAL via max_wal_senders.

postgres_replica: Sebelum service PostgreSQL berjalan, container mengeksekusi utility pg_basebackup secara looping untuk mengunduh salinan fisik biner data (PGDATA) secara utuh dari postgres_primary menggunakan koneksi user replicator. Parameter -R otomatis mengonfigurasi standby.signal pada replica.

Langkah 5: Pembuatan Utility Alias (env.sh)

Command:

cat << 'EOF' > env.sh
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dps="docker ps"
alias der="docker exec -it streaming-replication-postgres_replica-1 bash"
alias dep="docker exec -it streaming-replication-postgres_primary-1 bash"
alias dlr="docker logs streaming-replication-postgres_replica-1"
alias dlp="docker logs streaming-replication-postgres_primary-1"
EOF
source env.sh


Screenshot Output:

Pembahasan:
Script ini mendefinisikan shorthand command (alias) untuk mempercepat manajemen container melalui terminal tanpa perlu mengetikkan perintah docker exec atau docker compose secara berulang.

4.1.3 Pelaksanaan dan Verifikasi Status Container

Langkah 1: Pengaktifan Service

Command:

dcu


Screenshot Output:

Pembahasan:
Docker Compose membentuk jaringan virtual streaming-replication_default dan merilis/menjalankan container postgres_primary serta postgres_replica di latar belakang (detached mode).

Langkah 2: Cek Status Healthcheck

Command:

dps


Screenshot Output:

Pembahasan:
Perintah docker ps menampilkan status kedua container:

Container streaming-replication-postgres_primary-1 berstatus Up (healthy) pada port 5432.

Container streaming-replication-postgres_replica-1 berstatus Up (healthy) pada port 5433.
Ini menandakan pg_basebackup pada replica berhasil diselesaikan dan instance replica telah aktif penuh.

4.1.4 Pengujian Replikasi Data & Sifat Read-Only

Langkah 1 & 2: Pemeriksaan Slot Replikasi di Primary

Command:

dep
psql -U zuser -d zdb -c "SELECT client_addr, state, sync_state FROM pg_stat_replication;"


Screenshot Output:

Pembahasan:
Query pada view sistem pg_stat_replication di container primary menghasilkan record:

client_addr: 172.18.0.3 (IP internal container replica)

state: streaming

sync_state: async
Hasil ini mengonfirmasi bahwa koneksi stream WAL antara primary dan replica telah terhubung dan aktif bekerja dalam mode asinkron.

Langkah 3: Keluar Container Primary

Command:

exit


Screenshot Output:

Langkah 4: Pembuatan Data Sampel pada Primary

Command:

dep
psql -U zuser -d zdb -c "CREATE TABLE tes_replikasi (id SERIAL PRIMARY KEY, nama VARCHAR(50));"
psql -U zuser -d zdb -c "INSERT INTO tes_replikasi (nama) VALUES ('Data Pertama'), ('Data Kedua');"
exit


Screenshot Output:

Pembahasan:
Proses DDL (CREATE TABLE) dan DML (INSERT) dieksekusi di database primary. Perubahan fisik berkas data dicatat dalam log WAL dan disiarkan ke aliran streaming.

Langkah 5: Verifikasi Sinkronisasi Data di Replica

Command:

der
psql -U zuser -d zdb -c "SELECT * FROM tes_replikasi;"
exit


Screenshot Output:

Pembahasan:
Query SELECT pada node replica menampilkan baris data Data Pertama (ID 1) dan Data Kedua (ID 2). Ini membuktikan data dari primary berhasil direplikasi secara otomatis ke replica secara real-time.

Langkah 6: Pengujian Transaksi Penulisan (Read-Only Test) di Replica

Command:

der
psql -U zuser -d zdb -c "INSERT INTO tes_replikasi (nama) VALUES ('Data Tiga');"
exit


Screenshot Output:

Pembahasan:
Sistem menolak operasi DML penulisan dengan pesan kesalahan:
ERROR: cannot execute INSERT in a read-only transaction
Hal ini membuktikan bahwa node replica dalam Streaming Replication bersifat Standby Read-Only, sehingga aman dari manipulasi data lokal secara langsung.

Langkah 7: Cleanup Container Streaming Replication

Command:

dcd


Screenshot Output:

Pembahasan:
Perintah docker compose down menghentikan dan menghapus seluruh container serta jaringan yang berkaitan dengan Streaming Replication agar lingkungan bersih sebelum memulai praktikum berikutnya.

Part 4.2: Logical Replication

4.2.1 Konfigurasi & Inisialisasi Logical Replication

Persiapan File docker-compose.yaml

Command:

cat << 'EOF' > docker-compose.yaml
x-postgres-common: &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  environment:
    POSTGRES_USER: zuser
    POSTGRES_DB: zdb
    POSTGRES_PASSWORD: zpass
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U zuser -d zdb"]
    interval: 10s
    timeout: 5s
    retries: 5

services:
  postgres_pub:
    <<: *postgres-common
    ports:
      - 5432:5432
    command: ["postgres", "-c", "wal_level=logical"]

  postgres_sub:
    <<: *postgres-common
    ports:
      - 5433:5432
    command: ["postgres"]
EOF


Screenshot Output:

Pembahasan:

Arsitektur Logical Replication tidak membutuhkan pembacaan file biner data (PGDATA) secara utuh.

Container postgres_pub (Publisher) dikonfigurasi dengan parameter wal_level=logical agar log WAL menyertakan informasi pengkodean tingkat baris/tabel.

Container postgres_sub (Subscriber) berstatus sebagai server PostgreSQL mandiri tanpa perlu inisialisasi pg_basebackup.

Pembuatan Script Alias dan Peluncuran Service

Command:

cat << 'EOF' > env.sh
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dps="docker ps"
alias dep="docker exec -it logical-replication-postgres_pub-1 bash"
alias der="docker exec -it logical-replication-postgres_sub-1 bash"
alias dlp="docker logs logical-replication-postgres_pub-1"
alias dlr="docker logs logical-replication-postgres_sub-1"
EOF
source env.sh
dcu
dps


Screenshot Output:

Pembahasan:
Container postgres_pub dan postgres_sub dinyalakan. Verifikasi via dps mengonfirmasi kedua container berada pada kondisi Up (healthy).

4.2.2 Konfigurasi Publication dan Subscription

Konfigurasi di Node Publisher (postgres_pub)

Command:

dep
psql -U zuser -d zdb -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));"
psql -U zuser -d zdb -c "CREATE PUBLICATION my_pub FOR TABLE users;"
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Budi'), ('Siti');"
exit


Screenshot Output:

Pembahasan:

Membuat skema tabel users.

Perintah CREATE PUBLICATION my_pub FOR TABLE users; mendaftarkan tabel users sebagai objek logical replication yang akan dipublikasikan ke subscriber yang terhubung.

Mengisi data sampel (Budi dan Siti).

Konfigurasi di Node Subscriber (postgres_sub)

Command:

der
psql -U zuser -d zdb -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));"
psql -U zuser -d zdb -c "CREATE SUBSCRIPTION my_sub CONNECTION 'host=postgres_pub port=5432 dbname=zdb user=zuser password=zpass' PUBLICATION my_pub;"
exit


Screenshot Output:

Pembahasan:

Skema tabel users harus dibuat secara manual terlebih dahulu pada subscriber (karena logical replication hanya mengalirkan data baris, bukan skema/DDL).

Perintah CREATE SUBSCRIPTION menghubungkan subscriber ke instance publisher via string koneksi TCP.

Output terminal NOTICE: created replication slot "my_sub" on publisher menandakan bahwa slot replikasi logis pada publisher sukses dibentuk secara otomatis.

4.2.3 Verifikasi Data & Pengujian Fitur Read-Write

Verifikasi Data Awal Replikasi di Subscriber

Command:

der
psql -U zuser -d zdb -c "SELECT * FROM users;"
exit


Screenshot Output:

Pembahasan:
Output SELECT memperlihatkan data 1 | Budi dan 2 | Siti telah masuk ke subscriber. Ini mengonfirmasi bahwa sinkronisasi data antar tabel berhasil.

Pengujian Penulisan Lokal & Penanganan Konflik Sequence ID

Command:

der
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Cici');"
exit


Screenshot Output:

Pembahasan & Evaluasi Masalah:
Saat memasukkan data lokal (Cici), muncul pesan kesalahan:
ERROR: duplicate key value violates unique constraint "users_pkey"
DETAIL: Key (id)=(1) already exists.

Penyebab: Data Budi (id 1) dan Siti (id 2) masuk ke subscriber melalui jalur pengaliran Logical Replication, bukan transaksi INSERT lokal. Hal ini menyebabkan objek sequence generator (users_id_seq) di subscriber masih tertinggal pada nilai awal 1.

Solusi Sinkronisasi Sequence ID

Command:

der
psql -U zuser -d zdb -c "SELECT setval('users_id_seq', (SELECT MAX(id) FROM users));"
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Cici');"
psql -U zuser -d zdb -c "SELECT * FROM users;"
exit


Screenshot Output:

Pembahasan:

SELECT setval(...): Memperbarui posisi nilai terakhir sequence generator users_id_seq agar menyesuaikan dengan nilai MAX(id) yang ada pada tabel (yaitu 2).

Perintah INSERT INTO users (name) VALUES ('Cici') berikutnya berjalan sukses tanpa hambatan dan mendapatkan ID otomatis 3.

Hasil query akhir menampilkan:

1 | Budi (Hasil Logical Replication dari Publisher)

2 | Siti (Hasil Logical Replication dari Publisher)

3 | Cici (Hasil penulisan Read-Write lokal di Subscriber)

Kesimpulan

Streaming Replication (Physical Replication)

Beroperasi pada tingkat biner/berkas data (PGDATA / WAL physical).

Node Replica merupakan salinan 1:1 dari Primary dan bersifat Read-Only (menolak transaksi DML/DDL langsung).

Sangat cocok diterapkan untuk skenario High Availability (HA), Disaster Recovery (DR), dan Read-Scaling utuh.

Logical Replication

Beroperasi pada tingkat objek/baris data berdasarkan konsep Publisher-Subscriber.

Node Subscriber bersifat Read-Write, memungkinkan penambahan/perubahan data lokal secara independen di luar tabel yang disinkronkan.

Perlu perhatian khusus pada penanganan Sequence ID (SERIAL) pada subscriber saat melakukan operasi penulisan lokal untuk menghindari konflik Unique Primary Key Constraint.

Sangat cocok digunakan untuk konsolidasi data antar database, migrasi versi major PostgreSQL, atau replikasi selektif tabel tertentu.
