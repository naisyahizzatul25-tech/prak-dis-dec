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
*Gambar 3.1: Proses update repository dan instalasi paket python3.14-venv serta python3-pip pada terminal WSL/Linux.*

**Penjelasan Gambar 3.1:**
Perintah `sudo apt update` digunakan untuk menyegarkan daftar indeks paket repository Linux, sedangkan `sudo apt install python3.14-venv python3-pip -y` mengunduh dan menginstal alat manajemen lingkungan virtual Python serta package manager (`pip`). Hal ini diperlukan agar dependensi proyek terisolasi dari lingkungan sistem utama.

Selanjutnya, dibuat direktori kerja `workspace-05`, pembuatan lingkungan virtual `.venv`, dan instalasi framework aplikasi web BlackSheep:

mkdir workspace-05
cd workspace-05/
python3 -m venv .venv
source .venv/bin/activate
pip install blacksheep blacksheep-cli

![Virtual Environment dan BlackSheep](path/to/gambar-3.2.png)
*Gambar 3.2: Aktivasi virtual environment dan instalasi paket blacksheep beserta blacksheep-cli.*

**Penjelasan Gambar 3.2:**
Setelah mengaktifkan lingkungan virtual via `source .venv/bin/activate` (ditandai dengan prompt `(.venv)`), perintah `pip install blacksheep blacksheep-cli` mengunduh kerangka kerja BlackSheep beserta modul pendukungnya seperti `uvicorn`, `rodi`, `guardpost`, `h11`, dan `h2`.

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
*Gambar 3.3: Proses inisialisasi struktur proyek blacksheep_lb menggunakan CLI.*

**Penjelasan Gambar 3.3:**
Perintah `blacksheep create` memicu wizard interaktif. Pilihan template `api` mengonstruksi struktur dasar proyek REST API berbasis arsitektur Controller dan OpenAPI (Swagger) dokumentasi.

Masuk ke direktori proyek dan install seluruh paket dependensi pendukung:

cd blacksheep_lb
pip install -r requirements.txt
python dev.py

![Menjalankan Dev Server](path/to/gambar-3.4.png)
*Gambar 3.4: Menjalankan aplikasi web secara lokal pada port 44777 via Uvicorn.*

**Penjelasan Gambar 3.4:**
Menjalankan `python dev.py` akan memulai ASGI Web Server (`Uvicorn`) pada alamat local `http://localhost:44777/docs/`. Fitur reloader otomatis (*StatReload*) diaktifkan untuk memantau perubahan kode sumber selama tahap pengembangan.

**Pengujian Aplikasi Secara Lokal:**
- Aplikasi berjalan lokal pada port `44777` (`http://localhost:44777/docs/`).
- Saat diakses via browser pada endpoint `/docs/`, Swagger UI menampilkan dokumentasi endpoint API yaitu `GET /api/examples` dan `POST /api/examples`.

![OpenAPI Swagger UI](path/to/gambar-3.5.png)
*Gambar 3.5: Halaman Swagger UI OpenAPI interaktif pada browser.*

**Penjelasan Gambar 3.5:**
Tampilan antarmuka Swagger UI memverifikasi bahwa aplikasi BlackSheep berhasil berjalan dan secara otomatis menghasilkan dokumentasi API untuk endpoint `GET /api/examples` dan `POST /api/examples`.

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
*Gambar 3.6: Script penyiapan direktori nginx, Dockerfile, nginx.conf, dan docker-compose.yml.*

**Penjelasan Gambar 3.6:**
Perintah heredoc (`cat << 'EOF'`) membuat file konfigurasi Nginx dan `docker-compose.yml`. Blok `upstream backend` mendefinisikan grup server aplikasi (`bs_app:80`), sementara `proxy_pass http://backend;` mengarahkan semua lalu lintas HTTP port 80 ke grup backend tersebut.

---

### 3.4 Eksekusi dan Pengujian Scaling (Load Balancing)
Menjalankan Docker Compose dengan melakukan scaling layanan aplikasi menjadi **2 instance**:

docker compose up --build -d --scale bs_app=2

![Build dan Scaling Instance](path/to/gambar-3.7.png)
*Gambar 3.7: Proses build image Docker dan scaling kontainer bs_app menjadi 2 instance.*

**Penjelasan Gambar 3.7:**
Perintah `docker compose up --build -d --scale bs_app=2` membangun image Docker untuk aplikasi Python (`bs_app`) dan Nginx. Parameter `--scale bs_app=2` secara otomatis mereplikasi kontainer aplikasi menjadi dua instance aktif: `workspace-05-bs_app-1` dan `workspace-05-bs_app-2`.

**Hasil Status Container:**
- Layanan aplikasi memicu jalannya kontainer `workspace-05-bs_app-1` dan `workspace-05-bs_app-2`.
- Kontainer `nginx` bertindak sebagai pintu masuk utama pada port 80.

---

## 4. Failure Detection & High Availability (Toleransi Kegagalan)

### 4.1 Simulasi High Availability (Pengujian Stop/Start Instance)

**1. Menghentikan Salah Satu Instance Aplikasi (`bs_app-1`):**

docker stop workspace-05-bs_app-1

![Menghentikan Instance Aplikasi](path/to/gambar-4.1.png)
*Gambar 4.1: Eksekusi perintah docker stop untuk mematikan salah satu kontainer aplikasi.*

**Penjelasan Gambar 4.1:**
Untuk menguji skenario kegagalan (*fault tolerance*), salah satu kontainer backend (`workspace-05-bs_app-1`) dihentikan secara paksa menggunakan `docker stop`. Ini mensimulasikan kegagalan server fisik atau crash aplikasi di lingkungan produksi.

**2. Pengujian Akses Browser (`http://localhost/docs/`):**
- **Hasil**: Halaman Dokumentasi API tetap dapat diakses dengan normal tanpa adanya eror (*Zero Downtime*).
- **Analisis**: Nginx mendeteksi kegagalan pada `bs_app-1` dan langsung mengalihkan seluruh trafik ke `bs_app-2`.

![Aplikasi Tetap Aktif Via Nginx](path/to/gambar-4.2.png)
*Gambar 4.2: Antarmuka Swagger API yang tetap responsif meskipun 1 instance backend mati.*

**Penjelasan Gambar 4.2:**
Browser mengakses `http://localhost/docs/` pada port 80. Walaupun `bs_app-1` telah dimatikan, Nginx sebagai Load Balancer membelokkan lalu lintas secara transparan ke instance `bs_app-2` yang masih hidup. Pengguna tidak mengalami downtime atau kegagalan akses (*High Availability*).

**3. Pengujian Endpoint API (GET & POST):**
- Request `GET /api/examples` menghasilkan respon status `200 OK`.
- Request `POST /api/examples` berhasil menambahkan data dengan respon status `200 OK`.

![Uji Coba GET API](path/to/gambar-4.3.png)
*Gambar 3.4.3: Pengujian pengiriman request GET /api/examples pada Swagger UI.*

![Uji Coba POST API](path/to/gambar-4.4.png)
*Gambar 3.4.4: Pengujian pengiriman request POST /api/examples pada Swagger UI.*

**Penjelasan Gambar 3.4.3 & 3.4.4:**
Eksekusi tombol *Try it out* pada endpoint `GET` dan `POST` mengembalikan status HTTP `200 OK` beserta payload data JSON. Ini membuktikan bahwa layanan backend yang tersisa (`bs_app-2`) berfungsi penuh dalam mengolah request read maupun write.

**4. Recovery Instance & Stopping Containers:**
Menyalakan kembali instance `bs_app-1` dan menghentikan seluruh layanan setelah pengujian selesai:

docker start workspace-05-bs_app-1
docker ps --format '{{.Image}} {{.Names}} {{.Status}}'
docker compose down

![Recovery dan Teardown Environment](path/to/gambar-4.5.png)
*Gambar 4.5: Menyalakan kembali kontainer (recovery) dan melakukan teardown environment.*

**Penjelasan Gambar 4.5:**
Perintah `docker start workspace-05-bs_app-1` mengembalikan kontainer ke status `Up` (*recovery*). Nginx secara otomatis memasukkan kembali instance tersebut ke perputaran pembagian beban. Akhirnya, `docker compose down` membersihkan semua kontainer dan jaringan virtual yang telah dibuat.

---

## 5. Kesimpulan

1. **Load Balancing**: Nginx sebagai Reverse Proxy berhasil membagi beban request HTTP secara seimbang ke beberapa instance aplikasi backend (`bs_app-1` dan `bs_app-2`).
2. **High Availability & Fault Tolerance**: Arsitektur ini membuktikan bahwa sistem memiliki kemampuan toleransi kegagalan. Jika salah satu server/instance aplikasi mati, Nginx secara fleksibel dan otomatis mengalihkan request ke server lain yang aktif tanpa mengganggu pengguna akhir (*Zero Downtime*).
