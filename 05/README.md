# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Modul 4: Replikasi pada Sistem Terdistribusi (PostgreSQL Replication)

### Identitas Mahasiswa

* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1 (Semester 3)
* **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)


Dokumentasi ini berisi penjelasan dan pembahasan detail dari alur pengujian Load Balancing dan Fault Tolerance menggunakan Nginx sebagai Reverse Proxy / Load Balancer dan BlackSheep (Python Framework) sebagai aplikasi backend di dalam lingkungan Docker.

4.1 Load Balancing Aplikasi

1. Persiapan Environment

Pada tahap awal, dilakukan penyiapan lingkungan kerja di sistem operasi Linux (WSL 2).

Pembaruan Sistem dan Instalasi Dependensi:

sudo apt update
sudo apt install python3.14-venv python3-pip -y


Perintah ini mengunduh indeks paket terbaru serta menginstal python3-venv untuk isolasi virtual environment dan pip sebagai pengelola paket Python.

Penjelasan Gambar 1:
Menampilkan proses eksekusi sudo apt update dan instalasi dependensi python3.14-venv serta python3-pip. Terminal mengonfirmasi daftar pustaka dasar (seperti gcc, make, python3-dev) yang ikut terpasang untuk mendukung kompilasi modul Python.

Inisialisasi Direktori dan Virtual Environment:

mkdir workspace-05
cd workspace-05/
python3 -m venv .venv
source .venv/bin/activate


Instalasi Framework BlackSheep:

pip install blacksheep blacksheep-cli


Menginstal framework web berkinerja tinggi blacksheep beserta antarmuka CLI-nya (blacksheep-cli).

Penjelasan Gambar 2:
Menampilkan proses pengunduhan dependensi paket blacksheep dan blacksheep-cli via pip di dalam virtual environment .venv.

Penjelasan Gambar 3:
Menampilkan pesan konfirmasi bahwa instalasi pustaka pendukung (seperti Jinja2, uvicorn, pydantic, httptools, dan blacksheep-cli) telah selesai dan berhasil terpasang (Successfully installed).

2. Pembuatan Aplikasi Web

Membuat project aplikasi backend baru menggunakan CLI BlackSheep.

Inisialisasi Project:

blacksheep create


Konfigurasi Interactive CLI:

Project name: blacksheep_lb

Project template: api

Use controllers?: Yes

Use OpenAPI Documentation?: Yes

Library to read settings: essentials-configuration

App settings format: YAML

Penjelasan Gambar 4:
Menampilkan proses pembuatan struktur direktori project blacksheep_lb berbasiskan template API RESTful dengan fitur dokumentasi Swagger/OpenAPI aktif.

Instalasi Dependensi Project:

cd blacksheep_lb
pip install -r requirements.txt


Penjelasan Gambar 5:
Menampilkan pemeriksaan dependensi dari file requirements.txt. Paket yang sudah ada pada virtual environment akan dilewati (Requirement already satisfied).

Penjelasan Gambar 6:
Menampilkan penyelesaian instalasi server ASGI (Asynchronous Server Gateway Interface) uvicorn dan uvloop untuk mengeksekusi aplikasi Python berkecepatan tinggi.

Pengujian Aplikasi Secara Lokal (Development Server):

python dev.py


Penjelasan Gambar 7:
Server pengembang Uvicorn aktif dan berjalan pada alamat http://localhost:44777. Terminal juga mencatat permintaan (log access) dengan respons HTTP status 200 OK saat halaman /docs/ diakses.

Verifikasi Dokumentasi API (Local):

Penjelasan Gambar 8:
Tampilan browser saat mengakses http://localhost:44777/docs/. Menampilkan antarmuka OpenAPI Swagger UI untuk endpoint aplikasi blacksheep_lb API versi 0.0.1 dengan route sampel (GET /api/examples dan POST /api/examples).

Menghentikan Server Dev:

Penjelasan Gambar 9:
Proses penghentian development server Uvicorn di terminal menggunakan kombinasi tombol Ctrl + C (Shutting down).

3. Pembuatan Konfigurasi Docker & Nginx

Menyusun file Dockerfile dan Nginx configuration untuk load balancing.

Konfigurasi Dockerfile & Nginx (nginx/nginx.conf):

mkdir -p nginx
cat << 'EOF' > nginx/Dockerfile
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf
EOF

cat << 'EOF' > nginx/nginx.conf
events {
    worker_connections 4096;
}
http {
    upstream backend {
        server bs_app:80;
    }
    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
}
EOF


Penjelasan Gambar 10:
Pembuatan skrip nginx/Dockerfile dan file konfigurasi nginx/nginx.conf menggunakan perintah cat << 'EOF'. Blok upstream backend dikonfigurasi mengarah ke bs_app:80 yang akan berfungsi sebagai penyeimbang beban (load balancer).

Konfigurasi Multi-Container (docker-compose.yml):

cat << 'EOF' > docker-compose.yml
services:
  bs_app:
    build: ./blacksheep_lb
  nginx:
    container_name: nginx
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      - bs_app
EOF


Penjelasan Gambar 11:
Menampilkan penulisan file docker-compose.yml untuk menghubungkan kontainer nginx (port 80) dengan layanan bs_app yang berjalan di jaringan Docker yang sama.

4. Menjalankan Docker Compose dengan Auto-Scaling

Mengompilasi image dan menjalankan beberapa instance kontainer aplikasi sekaligus.

Eksekusi Build Container:

docker compose up --build -d --scale bs_app=2


Penjelasan Gambar 12:
Proses kompilasi image kontainer workspace-05-nginx dan workspace-05-bs_app menggunakan Docker BuildKit.

Penjelasan Gambar 13:
Menampilkan tahap akhir pembuatan image dan pengaktifan kontainer. Opsi --scale bs_app=2 berhasil membuat dua instance kontainer backend: workspace-05-bs_app-1 dan workspace-05-bs_app-2, serta kontainer proxy nginx.

Pengujian Akses Melalui Nginx:

Penjelasan Gambar 14:
Mengakses http://localhost/ pada browser. Tampilan menghasilkan pesan Resource not found (HTTP Status 404). Hal ini membuktikan Nginx pada port 80 berhasil meneruskan trafik ke aplikasi BlackSheep di backend (karena endpoint akar / tidak didefinisikan di controller).

4.2 Failure Detection & Fault Tolerance

Tahap ini bertujuan untuk menguji keandalan sistem dalam menangani kegagalan (failover) jika salah satu instance backend mati.

1. Re-Build & Verifikasi Awal Status Kontainer

docker compose up --build -d --scale bs_app=2


Penjelasan Gambar 15:
Memastikan lingkungan load balancing aktif secara penuh dengan 2 kontainer bs_app (Running) dan 1 kontainer nginx (Running) sebelum pengujian simulasi kerusakan dimulai.

2. Simulasi Kegagalan Server (Stop Container)

Menghentikan kontainer pertama secara paksa untuk merepresentasikan insiden server crash atau down.

docker stop workspace-05-bs_app-1


Penjelasan Gambar 16:
Eksekusi perintah docker stop workspace-05-bs_app-1 pada terminal WSL. Output menampilkan nama workspace-05-bs_app-1, menandakan kontainer backend 1 telah mati.

3. Pengujian Ketersediaan Layanan (High Availability)

Membuka kembali browser di alamat http://localhost/docs/.

Penjelasan Gambar 17:
Halaman Swagger UI OpenAPI pada http://localhost/docs/ tetap dapat diakses dengan lancar tanpa error 502/504. Ini membuktikan mekanisme Fault Tolerance bekerja: Nginx secara otomatis mendeteksi matinya bs_app-1 dan mengalihkan seluruh trafik ke bs_app-2 yang masih aktif.

4. Pemulihan Sistem (Recovery)

Menyalakan kembali kontainer yang mati untuk mengembalikan kapasitas cluster ke kondisi normal.

docker start workspace-05-bs_app-1
docker ps --format '{{.Image}} {{.Names}} {{.Status}}'


Penjelasan Gambar 18:
Menampilkan proses pengaktifan kembali kontainer (docker start workspace-05-bs_app-1) dan verifikasi status dengan docker ps. Terlihat workspace-05-bs_app-1 kembali aktif (Up 58 seconds), mengindikasikan pemulihan sistem (recovery) berjalan sukses.

5. Pembersihan Lingkungan (Cleanup)

Menghentikan dan menghapus seluruh kontainer serta jaringan virtual setelah praktikum selesai.

docker compose down


Penjelasan Gambar 19:
Eksekusi perintah docker compose down. Seluruh kontainer (nginx, bs_app-1, bs_app-2) dan jaringan virtual (workspace-05_default) dihentikan dan dihapus dari sistem Docker (Removed).

Kesimpulan Praktikum

Load Balancing: Nginx berhasil dikonfigurasi sebagai Reverse Proxy dan Load Balancer yang mampu membagi beban permintaan ke multiple kontainer backend (bs_app-1 dan bs_app-2) yang di-scale menggunakan Docker Compose.

Fault Tolerance: Sistem terbukti memiliki tingkat ketahanan kesalahan yang baik (High Availability). Ketika salah satu node backend mengalami kegagalan/mati, Nginx secara otomatis mengarahkan trafik ke node yang sehat tanpa menyebabkan gangguan layanan (downtime) bagi pengguna.
