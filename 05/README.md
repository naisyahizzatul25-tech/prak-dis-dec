# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Laporan Praktikum Modul 5: Fault Tolerance & Load Balancing

### Identitas Mahasiswa

* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1 (Semester 3)
* **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)


-Repository ini berisi laporan praktikum mengenai implementasi dan pengujian **Load Balancing** serta **Fault Tolerance** menggunakan framework Python ASGI BlackSheep, Nginx sebagai Reverse Proxy / Load Balancer, dan Docker Compose untuk kontainerisasi.

---

## 1. Tujuan Praktikum

1. Memahami konsep Fault Tolerance dan High Availability pada sistem terdistribusi.
2. Memahami peran Load Balancer dan Reverse Proxy dalam mendistribusikan beban trafik aplikasi.
3. Mampu melakukan scaling (menduplikasi) instance aplikasi web/API menggunakan Docker Compose.
4. Mengonfigurasi Nginx untuk mendistribusikan request HTTP ke beberapa kontainer backend.
5. Menguji keandalan sistem (*failure detection & recovery*) saat salah satu instance aplikasi mengalami gangguan/mati.

---

## 2. Dasar Teori

### 2.1 Fault Tolerance & High Availability
- **Fault Tolerance** adalah kemampuan suatu sistem terdistribusi untuk tetap beroperasi secara normal meskipun terjadi kegagalan (kegagalan komponen hardware, jaringan, atau software) pada salah satu node/komponennya.
- **High Availability (HA)** merujuk pada sistem yang dirancang untuk beroperasi tanpa henti (*zero downtime*) dalam jangka waktu lama dengan cara meminimalkan *Single Point of Failure* (SPOF).

### 2.2 Load Balancing & Reverse Proxy
- **Reverse Proxy**: Server perantara yang menerima request dari klien/internet dan meneruskannya ke satu atau beberapa server backend.
- **Load Balancing**: Teknik pembagian beban kerja secara merata ke beberapa server backend agar tidak ada satu pun server yang mengalami *overload*. Algoritma standar yang umum digunakan adalah *Round-Robin*, di mana request diteruskan secara bergiliran ke setiap server.
- **Nginx**: Salah satu web server dan reverse proxy berkinerja tinggi yang populer digunakan sebagai load balancer untuk lalu lintas HTTP/HTTPS.

### 2.3 BlackSheep & ASGI Framework
- **ASGI (Asynchronous Server Gateway Interface)**: Standar antarmuka antara web server asinkron dengan aplikasi Python.
- **BlackSheep**: Framework web/API Python berbasis asyncio yang memiliki kinerja sangat cepat (*high-performance*) dan mendukung fitur modern seperti validasi Pydantic, injeksi dependensi, serta generasi otomatis dokumentasi OpenAPI (Swagger).

---

## 3. Langkah Praktikum & Load Balancing Aplikasi

### 3.1 Persiapan Lingkungan Pengembangan
Langkah awal dimulai dengan penyiapan dependensi dasar pada sistem operasi Linux (Ubuntu/WSL) seperti paket Python virtual environment (`python3-venv`) dan `pip`.

sudo apt update
sudo apt install python3.14-venv python3-pip -y

<img width="940" height="670" alt="image" src="https://github.com/user-attachments/assets/b45efdc4-a767-4937-bdc8-b767d9c76c75" />
*Gambar 1: Proses update repository dan instalasi paket python3.14-venv serta python3-pip pada terminal.*

**Penjelasan Gambar 1:**
Perintah `sudo apt update` menyegarkan daftar indeks paket repository Linux, sedangkan `sudo apt install python3.14-venv python3-pip -y` mengunduh dan menginstal alat manajemen lingkungan virtual Python serta *package manager* (`pip`).

Selanjutnya, dibuat direktori kerja `workspace-05`, pembuatan lingkungan virtual `.venv`, dan instalasi framework aplikasi web BlackSheep:

mkdir workspace-05
cd workspace-05/
python3 -m venv .venv
source .venv/bin/activate
pip install blacksheep blacksheep-cli

<img width="940" height="682" alt="image" src="https://github.com/user-attachments/assets/33381888-d952-4016-b0b7-750b48f21569" />
*Gambar 2: Aktivasi virtual environment dan proses pengunduhan dependensi BlackSheep.*

<img width="940" height="393" alt="image" src="https://github.com/user-attachments/assets/94586a07-56d4-4b87-b14b-204fc9b59a45" />
*Gambar 3: Konfirmasi sukses instalasi paket blacksheep dan blacksheep-cli.*

**Penjelasan Gambar 2 & 3:**
Setelah mengaktifkan lingkungan virtual via `source .venv/bin/activate` (ditandai dengan prompt `(.venv)`), perintah `pip install` memasang kerangka kerja BlackSheep beserta pustaka pendukung seperti `uvicorn`, `rodi`, `guardpost`, `h11`, dan `h2`.

---

### 3.2 Pembuatan Aplikasi Web (`blacksheep_lb`)
Aplikasi web diinisialisasi menggunakan CLI bawaan BlackSheep:

blacksheep create

**Opsi konfigurasi pada prompt CLI:**
- **Project name**: `blacksheep_lb`
- **Project template**: `api`
- **Use controllers?**: `Yes`
- **Use OpenAPI Documentation?**: `Yes`
- **Library to read settings**: `essentials-configuration`
- **App settings format**: `YAML`

<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/cabd6013-3b79-4d3f-97f0-d2b1d530ab71" />
*Gambar 4: Inisialisasi struktur proyek blacksheep_lb berbasis template API via CLI.*

**Penjelasan Gambar 4:**
Perintah `blacksheep create` memicu wizard interaktif. Pilihan template `api` mengonstruksi struktur dasar proyek REST API berbasis arsitektur Controller dan OpenAPI (Swagger).

Masuk ke direktori proyek dan install seluruh paket dependensi pendukung:

cd blacksheep_lb
pip install -r requirements.txt
python dev.py

<img width="940" height="745" alt="image" src="https://github.com/user-attachments/assets/6b8c1c6b-e3b9-4682-90c3-9c12a2a3216f" />
*Gambar 5: Proses unduh dan verifikasi paket dari file requirements.txt.*

<img width="940" height="762" alt="image" src="https://github.com/user-attachments/assets/c87ec91a-c464-442f-94d7-a9828e15e353" />
*Gambar 6: Berhasil menginstal modul pydantic, uvicorn, uvloop, dan cryptography.*

<img width="940" height="430" alt="image" src="https://github.com/user-attachments/assets/7b54e87f-4afa-4a19-bfc3-f56c081392c2" />
*Gambar 7: Menjalankan dev server Uvicorn pada port 44777 dengan StatReload.*

<img width="940" height="265" alt="image" src="https://github.com/user-attachments/assets/2ba4a8b9-c479-4547-b6ac-5a08a74a66c8" />
*Gambar 8: Tampilan halaman OpenAPI / Swagger UI (http://localhost:44777/docs/).*

<img width="940" height="144" alt="image" src="https://github.com/user-attachments/assets/821a5a7e-255f-4273-8c8e-89ea8519511a" />
*Gambar 9: Menghentikan proses server dev Uvicorn menggunakan kombinasi tombol Ctrl+C.*

**Penjelasan Gambar 5 - 9:**
Setelah memasang dependensi pendukung dari `requirements.txt`, aplikasi dijalankan secara lokal dengan `python dev.py`. Aplikasi berjalan pada port `44777` dan menampilkan endpoint `GET /api/examples` serta `POST /api/examples` di Swagger UI sebelum akhirnya dihentikan.

---

### 3.3 Pengaturan Nginx Reverse Proxy & Load Balancer
Untuk mendistribusikan beban trafik ke beberapa instance aplikasi, dibuat konfigurasi Nginx dan Docker Compose di direktori utama `workspace-05`.

**1. Konfigurasi Nginx (`nginx/Dockerfile` & `nginx/nginx.conf`):**

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

<img width="940" height="572" alt="image" src="https://github.com/user-attachments/assets/f13c788f-f811-4e63-bc9a-bdf9017883de" />
*Gambar 10: Pembuatan direktori nginx, Dockerfile, dan nginx.conf menggunakan heredoc.*

**2. File Orchestration (`docker-compose.yml`):**

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

<img width="940" height="339" alt="image" src="https://github.com/user-attachments/assets/f82d37d0-0dbe-4bc1-8b3e-588eeaa4527a" />
*Gambar 11: Pembuatan file docker-compose.yml untuk mendefinisikan kontainer bs_app dan nginx.*

**Penjelasan Gambar 10 & 11:**
Script heredoc (`cat << 'EOF'`) digunakan untuk membuat file Dockerfile Nginx, `nginx.conf`, dan `docker-compose.yml`. Blok `upstream backend` merujuk pada kontainer `bs_app:80` yang akan didistribusikan oleh Nginx pada port `80`.

---

### 3.4 Eksekusi dan Pengujian Scaling (Load Balancing)
Menjalankan Docker Compose dengan melakukan scaling layanan aplikasi menjadi **2 instance**:

docker compose up --build -d --scale bs_app=2

<img width="940" height="969" alt="image" src="https://github.com/user-attachments/assets/1b6631e0-dcba-4659-98a7-cee5e180855e" />
*Gambar 12: Log proses pembuatan image Docker untuk Nginx dan Python Server Builder.*

<img width="847" height="553" alt="image" src="https://github.com/user-attachments/assets/40a0b7ca-7d00-40cf-a122-2efc751237d9" />
*Gambar 13: Kontainer workspace-05-bs_app-1, workspace-05-bs_app-2, dan nginx berhasil dijalankan.*

<img width="651" height="310" alt="image" src="https://github.com/user-attachments/assets/4089a4ae-3179-42b3-910b-42cf548523c2" />
*Gambar 14: Akses browser ke http://localhost/ menghasilkan respons Resource not found.*

**Penjelasan Gambar 12 - 14:**
Docker Compose membangun image dan menjalankan dua kontainer aplikasi backend (`workspace-05-bs_app-1` dan `workspace-05-bs_app-2`) serta satu kontainer `nginx`. Pengujian akses ke `http://localhost/` (port 80) menghasilkan `Resource not found` karena rute utama `/` belum didefinisikan pada template API BlackSheep.

---

## 4. Failure Detection & High Availability (Toleransi Kegagalan)

### 4.1 Simulasi High Availability (Pengujian Stop/Start Instance)

**1. Re-deploy & Menghentikan Salah Satu Instance Aplikasi (`bs_app-1`):**

docker compose up --build -d --scale bs_app=2
docker stop workspace-05-bs_app-1

<img width="854" height="969" alt="image" src="https://github.com/user-attachments/assets/667a3946-ee87-428e-bf6f-3df245cda840" />
*Gambar 15: Menjalankan ulang docker compose up --scale bs_app=2.*

<img width="940" height="107" alt="image" src="https://github.com/user-attachments/assets/d9d2bbd0-a6b2-4600-97e4-b5d79e102973" />
*Gambar 16: Eksekusi perintah docker stop workspace-05-bs_app-1 untuk mematikan instance 1.*

**2. Pengujian Akses Browser (`http://localhost/docs/`):**
- **Hasil**: Halaman Dokumentasi API tetap dapat diakses dengan normal tanpa eror (*Zero Downtime*).
- **Analisis**: Nginx mendeteksi `bs_app-1` mati dan mengalihkan seluruh trafik ke `bs_app-2`.

<img width="940" height="265" alt="image" src="https://github.com/user-attachments/assets/7fe31d98-baed-4080-8aeb-96b7d6fcdaf3" />
*Gambar 17: Halaman Swagger UI OpenAPI yang tetap aktif melayani request via Nginx.*

**3. Pengujian Endpoint API (GET & POST):**
- Request `GET /api/examples` menghasilkan respon status `200 OK`.
- Request `POST /api/examples` berhasil menambahkan data dengan respon status `200 OK`.

<img width="940" height="462" alt="image" src="https://github.com/user-attachments/assets/87289906-497f-4661-a59f-54fdf89bb0a9" />
*Gambar 18: Eksekusi request GET /api/examples mengembalikan status 200 OK.*

<img width="940" height="493" alt="image" src="https://github.com/user-attachments/assets/e774e56f-7068-4af3-8397-b58baa74540e" />
*Gambar 19: Eksekusi request POST /api/examples mengembalikan payload JSON sukses.*

**4. Recovery Instance & Stopping Containers:**
Menyalakan kembali instance `bs_app-1`, mengecek status kontainer, dan menghentikan seluruh layanan setelah pengujian selesai:

docker start workspace-05-bs_app-1
docker ps --format '{{.Image}} {{.Names}} {{.Status}}'
docker compose down

<img width="940" height="175" alt="image" src="https://github.com/user-attachments/assets/9c00bc78-9426-49d5-8a41-aeca70137f4d" />
<img width="940" height="168" alt="image" src="https://github.com/user-attachments/assets/234c2458-7d47-4c73-b662-ae21e40fc978" />

*Gambar 20 dan 21: Menyalakan kembali bs_app-1 dan mematikan seluruh environment dengan docker compose down.*

---

### 4.2 Failure Detection (Verifikasi Pengujian Ulang)

Pengujian ulang dilakukan untuk memastikan bahwa deteksi kegagalan berjalan konsisten saat seluruh lingkungan kontainer diinisialisasi kembali.

<img width="866" height="1010" alt="image" src="https://github.com/user-attachments/assets/48d3f58f-978c-4a0f-b03d-750a6df2b9b8" />
*Gambar 21: Eksekusi ulang docker compose up untuk memverifikasi modul failure detection.*

<img width="940" height="271" alt="image" src="https://github.com/user-attachments/assets/fb6ddaef-bf04-416c-bf5b-59f1d70c1fa9" />
*Gambar 22: Verifikasi halaman Swagger UI API kembali aktif.*

<img width="940" height="463" alt="image" src="https://github.com/user-attachments/assets/1fbbb873-80e3-48bc-b043-739ed3e5daf1" />
*Gambar 23: Verifikasi ulang respons 200 OK pada request GET /api/examples.*

<img width="940" height="505" alt="image" src="https://github.com/user-attachments/assets/6b9b4956-9bc2-4264-b35e-56cfa05080d5" />
*Gambar 24: Verifikasi ulang respons 200 OK pada request POST /api/examples.*

<img width="940" height="234" alt="image" src="https://github.com/user-attachments/assets/d37ef7c2-a297-4e1e-b3d9-371446a4ac37" />
*Gambar 25: Menjalankan docker stop, docker start, dan teardown akhir dengan docker compose down.*

**Penjelasan Gambar 15 - 24:**
Rangkaian gambar ini membuktikan prinsip *Fault Tolerance* dan *High Availability*. Ketika salah satu backend (`bs_app-1`) dimatikan via `docker stop`, Nginx secara otomatis mengalihkan request pengguna ke `bs_app-2` yang aktif tanpa memicu *downtime*. Saat instance `bs_app-1` dihidupkan kembali via `docker start`, Nginx memasukkan kembali instance tersebut ke alokasi pembagian beban kerja secara dinamis.

---

## 5. Kesimpulan

1. **Load Balancing**: Nginx sebagai Reverse Proxy berhasil membagi beban request HTTP secara seimbang ke beberapa instance aplikasi backend (`bs_app-1` dan `bs_app-2`).
2. **High Availability & Fault Tolerance**: Arsitektur ini membuktikan bahwa sistem memiliki kemampuan toleransi kegagalan. Jika salah satu server/instance aplikasi mati, Nginx secara fleksibel dan otomatis mengalihkan request ke server lain yang aktif tanpa mengganggu pengguna akhir (*Zero Downtime*).
