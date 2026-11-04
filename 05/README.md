# Laporan Praktikum Modul 5: Fault Tolerance & Load Balancing

Repository ini berisi laporan praktikum mengenai implementasi dan pengujian **Load Balancing** serta **Fault Tolerance** menggunakan framework Python ASGI BlackSheep, Nginx sebagai Reverse Proxy / Load Balancer, dan Docker Compose untuk kontainerisasi.

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

![Proses Instalasi Dependensi](path/to/gambar-3.1.png)
*Gambar 3.1: Proses instalasi dependensi dasar di terminal.*

Selanjutnya, dibuat direktori kerja `workspace-05`, pembuatan lingkungan virtual `.venv`, dan instalasi framework aplikasi web BlackSheep:

mkdir workspace-05
cd workspace-05/
python3 -m venv .venv
source .venv/bin/activate
pip install blacksheep blacksheep-cli

![Virtual Environment dan BlackSheep](path/to/gambar-3.2.png)
*Gambar 3.2: Aktivasi virtual environment dan instalasi paket BlackSheep.*

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

![Inisialisasi Proyek BlackSheep](path/to/gambar-3.3.png)
*Gambar 3.3: Inisialisasi proyek BlackSheep API via CLI.*

Masuk ke direktori proyek dan install seluruh paket dependensi pendukung:

cd blacksheep_lb
pip install -r requirements.txt
python dev.py

![Menjalankan Dev Server](path/to/gambar-3.4.png)
*Gambar 3.4: Menjalankan server pengembang aplikasi BlackSheep pada port 44777.*

**Pengujian Aplikasi Secara Lokal:**
- Aplikasi berjalan lokal pada port `44777` (`http://localhost:44777/docs/`).
- Saat diakses via browser pada endpoint `/docs/`, Swagger UI menampilkan dokumentasi endpoint API yaitu `GET /api/examples` dan `POST /api/examples`.

![OpenAPI Swagger UI](path/to/gambar-3.5.png)
*Gambar 3.5: Halaman OpenAPI / Swagger UI aplikasi BlackSheep lokal.*

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

![Konfigurasi Nginx dan Docker Compose](path/to/gambar-3.6.png)
*Gambar 3.6: Pembuatan file konfigurasi Nginx dan docker-compose.yml.*

---

### 3.4 Eksekusi dan Pengujian Scaling (Load Balancing)
Menjalankan Docker Compose dengan melakukan scaling layanan aplikasi menjadi **2 instance**:

docker compose up --build -d --scale bs_app=2

![Build dan Scaling Instance](path/to/gambar-3.7.png)
*Gambar 3.7: Build image dan eksekusi scaling 2 instance bs_app.*

**Hasil Status Container:**
- Layanan aplikasi memicu jalannya kontainer `workspace-05-bs_app-1` dan `workspace-05-bs_app-2`.
- Kontainer `nginx` bertindak sebagai pintu masuk utama pada port 80.

---

## 4. Failure Detection & High Availability (Toleransi Kegagalan)

### 4.1 Simulasi High Availability (Pengujian Stop/Start Instance)

**1. Menghentikan Salah Satu Instance Aplikasi (`bs_app-1`):**

docker stop workspace-05-bs_app-1

![Menghentikan Instance Aplikasi](path/to/gambar-4.1.png)
*Gambar 4.1: Menghentikan salah satu instance kontainer aplikasi.*

**2. Pengujian Akses Browser (`http://localhost/docs/`):**
- **Hasil**: Halaman Dokumentasi API tetap dapat diakses dengan normal tanpa adanya eror (*Zero Downtime*).
- **Analisis**: Nginx mendeteksi kegagalan pada `bs_app-1` dan langsung mengalihkan seluruh trafik ke `bs_app-2`.

![Aplikasi Tetap Aktif Via Nginx](path/to/gambar-4.2.png)
*Gambar 4.2: Aplikasi tetap aktif melayani request via Nginx.*

**3. Pengujian Endpoint API (GET & POST):**
- Request `GET /api/examples` menghasilkan respon status `200 OK`.
- Request `POST /api/examples` berhasil menambahkan data dengan respon status `200 OK`.

![Uji Coba GET API](path/to/gambar-4.3.png)
*Gambar 4.3: Uji coba request GET /api/examples pada Swagger UI.*

![Uji Coba POST API](path/to/gambar-4.4.png)
*Gambar 4.4: Uji coba request POST /api/examples pada Swagger UI.*

**4. Recovery Instance & Stopping Containers:**
Menyalakan kembali instance `bs_app-1` dan menghentikan seluruh layanan setelah pengujian selesai:

** Menyalakan kembali instance bs_app-1
docker start workspace-05-bs_app-1

** Memeriksa status kontainer
docker ps --format '{{.Image}} {{.Names}} {{.Status}}'

## Mematikan seluruh environment
docker compose down

![Recovery dan Teardown Environment](path/to/gambar-4.5.png)
*Gambar 4.5: Mengaktifkan kembali instance aplikasi lalu melakukan teardown environment dengan docker compose down.*

---

## 5. Kesimpulan

1. **Load Balancing**: Nginx sebagai Reverse Proxy berhasil membagi beban request HTTP secara seimbang ke beberapa instance aplikasi backend (`bs_app-1` dan `bs_app-2`).
2. **High Availability & Fault Tolerance**: Arsitektur ini membuktikan bahwa sistem memiliki kemampuan toleransi kegagalan. Jika salah satu server/instance aplikasi mati, Nginx secara fleksibel dan otomatis mengalihkan request ke server lain yang aktif tanpa mengganggu pengguna akhir (*Zero Downtime*).
