Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

Modul 4: Replikasi pada Sistem Terdistribusi (PostgreSQL Replication)

Identitas Mahasiswa

Nama: Naisyah Izzatul Jannah K.

NIM: 255410040

Kelas: Informatika IF 1 (Semester 3)

Universitas: Universitas Teknologi Digital Indonesia (UTDI)

Tujuan Praktikum

Memahami konsep dasar dan arsitektur Replikasi Database (Streaming Replication dan Logical Replication) pada PostgreSQL.

Mampu mengonfigurasi Replikasi Fisik (Streaming Replication) menggunakan Docker Compose serta menguji sifat Read-Only pada node replica.

Mampu mengonfigurasi Replikasi Logis (Logical Replication) dengan mekanisme Publication dan Subscription, serta menguji fitur Read-Write dan penanganan konflik sequence pada node subscriber.

Menganalisis perbedaan karakteristik, kelebihan, dan penggunaan Streaming Replication dibanding Logical Replication dalam arsitektur sistem terdistribusi.

Pembahasan Praktikum

Part 4.1: Physical / Streaming Replication

4.1.1 Pemeriksaan Versi Docker

Langkah awal adalah memastikan bahwa Docker Engine dan plugin Docker Compose telah terinstal serta berjalan dengan baik di lingkungan WSL.

docker --version && docker compose version



Pembahasan:
Perintah ini mengeksekusi dua pengecekan versi sekaligus. Output menunjukkan Docker versi 29.9.2 dan Docker Compose versi v5.6.1. Hal ini mengonfirmasi bahwa seluruh kebutuhan utilitas containerization telah siap untuk mengorkestrasi node primary dan replica PostgreSQL.

4.1.2 Konfigurasi File Streaming Replication

Langkah 1: Membuat Direktori Kerja

mkdir -p streaming-replication && cd streaming-replication



Pembahasan:
Perintah ini membuat folder terisolasi bernama streaming-replication dan berpindah ke dalamnya agar seluruh file konfigurasi modul 4.1 tersimpan rapi.

Langkah 2: Membuat Script Inisialisasi User Replikasi (00_init.sql)

cat << 'EOF' > 00_init.sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
EOF



Pembahasan:
Script SQL ini dipasang pada node Primary saat booting. Script membuat role replicator dengan kewenangan REPLICATION dan password 'replicator_password', serta mendaftarkan slot replikasi fisik bernama 'replication_slot'. Slot ini menjamin bahwa log WAL (Write-Ahead Logging) di primary tidak akan dihapus sebelum berhasil dikirim dan disinkronkan ke replica.

Langkah 3: Membuat Script Konfigurasi Hak Akses HBA (01_hba.sh)

cat << 'EOF' > 01_hba.sh
#!/bin/bash
echo "host replication replicator all scram-sha-256" >> "$PGDATA/pg_hba.conf"
EOF
chmod +x 01_hba.sh



Pembahasan:
Script shell ini secara otomatis menyisipkan aturan koneksi pada file pg_hba.conf di primary. Aturan ini mengizinkan pengguna replicator terhubung dari host manapun untuk proses replikasi dengan metode autentikasi scram-sha-256. Perintah chmod +x memberikan izin eksekusi pada script.

Langkah 4: Menyusun Konfigurasi Container (docker-compose.yaml)

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



Pembahasan:
Dokumen ini mengonfigurasi dua service: postgres_primary (port 5432) dan postgres_replica (port 5433). Node primary mengaktifkan parameter wal_level=replica dan hot_standby=on. Node replica tidak menginisialisasi database kosong, melainkan mengeksekusi pg_basebackup untuk menyalin seluruh direktori data fisik dari primary secara otomatis sebelum menjalankan layanan PostgreSQL.

Langkah 5: Membuat File Alias Environment (env.sh)

cat << 'EOF' > env.sh
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dps="docker ps"
alias der="docker exec -it streaming-replication-postgres_replica-1 bash"
alias dep="docker exec -it streaming-replication-postgres_primary-1 bash"
alias dlr="docker logs streaming-replication-postgres_replica-1"
alias dlp="docker logs streaming-replication-postgres_primary-1"
EOF



Pembahasan:
Membuat shortcut terminal untuk mempermudah manajemen container Streaming Replication, seperti masuk ke shell primary (dep), replica (der), dan memantau status (dps).

4.1.3 Menjalankan & Memeriksa Status Container

Langkah 1: Menyalakan Container

source env.sh
dcu



Pembahasan:
Mengaktifkan alias lingkungan dan mengeksekusi docker compose up -d. Output menunjukkan jaringan virtual dan kedua container (postgres_primary dan postgres_replica) berhasil dibuat dan dijalankan di latar belakang.

Langkah 2: Verifikasi Healthcheck Container

dps



Pembahasan:
Perintah dps menampilkan status kedua container dengan kondisi Up (healthy). Ini membuktikan bahwa proses kloning fisik data via pg_basebackup di replica telah selesai dan kedua database beroperasi dengan normal.

4.1.4 Verifikasi & Pengujian Streaming Replication

Langkah 1 & 2: Memeriksa Status Replikasi pada Primary

dep
psql -U zuser -d zdb -c "SELECT client_addr, state, sync_state FROM pg_stat_replication;"



Pembahasan:
Setelah masuk ke container primary (dep), query ke tampilan sistem pg_stat_replication menunjukkan adanya 1 koneksi aktif dari IP replica (172.18.0.3) dengan state = streaming dan sync_state = async. Ini menegaskan bahwa saluran Streaming Replication terhubung sempurna.

Langkah 3 & 4: Penulisan Data Baru di Node Primary

exit
dep
psql -U zuser -d zdb -c "CREATE TABLE tes_replikasi (id SERIAL PRIMARY KEY, nama VARCHAR(50));"
psql -U zuser -d zdb -c "INSERT INTO tes_replikasi (nama) VALUES ('Data Pertama'), ('Data Kedua');"
exit



Pembahasan:
Pada node primary, dibuat tabel baru tes_replikasi dan diisi dua baris data ('Data Pertama', 'Data Kedua'). Operasi DDL dan DML ini ditulis ke log WAL primary untuk disiarkan ke replica.

Langkah 5: Memeriksa Sinkronisasi Data di Node Replica

der
psql -U zuser -d zdb -c "SELECT * FROM tes_replikasi;"
exit



Pembahasan:
Masuk ke container replica (der) dan mengeksekusi query SELECT. Data 'Data Pertama' dan 'Data Kedua' tampil secara identik, membuktikan bahwa replikasi fisik data secara real-time telah berhasil.

Langkah 6: Uji Sifat Read-Only pada Replica

der
psql -U zuser -d zdb -c "INSERT INTO tes_replikasi (nama) VALUES ('Data Tiga');"
exit



Pembahasan:
Saat mencoba memasukkan data baru langsung pada node replica, PostgreSQL menolak transaksi dengan pesan kesalahan ERROR: cannot execute INSERT in a read-only transaction. Hal ini mengonfirmasi bahwa node replica pada Streaming Replication bersifat Read-Only (Hot Standby).

Langkah 7: Membersihkan Container Streaming Replication

dcd



Pembahasan:
Perintah dcd (docker compose down) mematikan dan menghapus seluruh container serta network fisik replikasi agar lingkungan kembali bersih sebelum melangkah ke praktikum berikutnya.

Part 4.2: Logical Replication

4.2.1 Inisialisasi & Pengujian Logical Replication

Langkah 1: Membuat File docker-compose.yaml Logical Replication

mkdir -p ~/logical-replication && cd ~/logical-replication
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



Pembahasan:
Menyusun konfigurasi dua instance database independen: postgres_pub (Publisher) dan postgres_sub (Subscriber). Node publisher dikonfigurasi khusus dengan wal_level=logical agar dapat memparsing perubahan data level baris (row-level change stream).

Langkah 2: Membuat File Alias Environment (env.sh)

cat << 'EOF' > env.sh
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dps="docker ps"
alias dep="docker exec -it logical-replication-postgres_pub-1 bash"
alias der="docker exec -it logical-replication-postgres_sub-1 bash"
alias dlp="docker logs logical-replication-postgres_pub-1"
alias dlr="docker logs logical-replication-postgres_sub-1"
EOF



Pembahasan:
Menyiapkan alias perintah terminal untuk mengelola container postgres_pub (dep) dan postgres_sub (der).

Langkah 3: Menjalankan Container & Mengecek Status

source env.sh
dcu
dps



Pembahasan:
Container postgres_pub dan postgres_sub dinyalakan. Pengecekan dps memastikan kedua container berstatus Up (healthy).

Langkah 4: Membuat Tabel & Publication pada Publisher

dep
psql -U zuser -d zdb -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));"
psql -U zuser -d zdb -c "CREATE PUBLICATION my_pub FOR TABLE users;"
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Budi'), ('Siti');"
exit



Pembahasan:
Pada node publisher (dep), skema tabel users dibuat dan dipublikasikan menggunakan CREATE PUBLICATION my_pub FOR TABLE users;. Setelah itu dimasukkan dua baris data awal ('Budi', 'Siti').

Langkah 5: Membuat Tabel & Subscription pada Subscriber

der
psql -U zuser -d zdb -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));"
psql -U zuser -d zdb -c "CREATE SUBSCRIPTION my_sub CONNECTION 'host=postgres_pub port=5432 dbname=zdb user=zuser password=zpass' PUBLICATION my_pub;"
exit



Pembahasan:
Pada node subscriber (der), tabel users dibuat mandiri. Selanjutnya mengeksekusi CREATE SUBSCRIPTION my_sub .... Output NOTICE: created replication slot "my_sub" on publisher menandakan slot replikasi logis pada publisher berhasil terhubung ke subscriber.

Langkah 6: Verifikasi Replikasi Data pada Subscriber

der
psql -U zuser -d zdb -c "SELECT * FROM users;"
exit



Pembahasan:
Query SELECT pada subscriber menampilkan data 'Budi' dan 'Siti'. Ini membuktikan data awal dari publisher langsung tersinkronisasi secara otomatis saat subscription dibuat.

Langkah 7: Pengujian Fitur Read-Write & Conflict Sequence

der
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Cici');"
exit



Pembahasan:
Saat mencoba memasukkan data 'Cici' secara lokal di subscriber, muncul pesan ERROR: duplicate key value violates unique constraint "users_pkey" (Key id=1 already exists). Hal ini terjadi karena Logical Replication mendplikasi data tanpa memperbarui nilai internal sequence auto-increment pada subscriber, sehingga sequence lokal masih menunjuk ke nilai 1.

Langkah 8: Sinkronisasi Sequence & Verifikasi Penulisan Lokal

der
psql -U zuser -d zdb -c "SELECT setval('users_id_seq', (SELECT MAX(id) FROM users));"
psql -U zuser -d zdb -c "INSERT INTO users (name) VALUES ('Cici');"
psql -U zuser -d zdb -c "SELECT * FROM users;"
exit



Pembahasan:
Fungsi setval() dieksekusi untuk menyesuaikan nilai sequence users_id_seq ke angka 2 (nilai id maksimum saat ini). Setelah disinkronkan, operasi INSERT lokal untuk 'Cici' berhasil dilakukan. Tampilan akhir menunjukkan data 1 | Budi (replicated), 2 | Siti (replicated), dan 3 | Cici (inserted locally). Ini membuktikan bahwa node Subscriber pada Logical Replication tetap berstatus Read-Write.

Kesimpulan

Berdasarkan praktikum yang telah dilakukan, dapat disimpulkan bahwa:

Streaming Replication (Physical Replication) bekerja pada tingkat blok fisik / file storage (wal_level=replica) dengan bantuan utilitas pg_basebackup. Node replica bersifat Standby Read-Only, sehingga cocok digunakan untuk skenario High Availability (HA), Disaster Recovery, dan Read-Heavy Load Balancing.

Logical Replication bekerja pada tingkat logis / data tabel (wal_level=logical) berbasis mekanisme Publish-Subscribe. Node subscriber tetap berstatus Read-Write dan memungkinkan replikasi selektif per tabel antar-versi PostgreSQL yang berbeda.

Penanganan penulisan lokal pada Logical Replication memerlukan perhatian ekstra terhadap primary key sequence (SERIAL), di mana penyelarasan nilai setval() secara manual atau otomatis diperlukan untuk mencegah timbulnya unique constraint violation.
