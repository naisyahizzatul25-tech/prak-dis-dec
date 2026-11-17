# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Modul 7: Cloud Computing & Containerized App (CA)

**Mata Kuliah:** Praktikum Sistem Terdistribusi dan Terdesentralisasi  
**Lingkungan Praktikum:** Windows Subsystem for Linux (WSL - Ubuntu) & Docker Desktop  
**Nama:** Naisyah Izzatul Jannah K.  
**NIM:** 255410040  
**Prodi:** Informatika (IF 1) - Universitas Teknologi Digital Indonesia (UTDI)  

---

## Tujuan Praktikum
1. Memahami konsep dasar **Cloud Computing** dengan pendekatan *Everything as a Service* (**XaaS**) beserta pembagian kategori layanannya (**SaaS**, **PaaS**, dan **IaaS**).
2. Memahami konsep dan arsitektur **Containerized App (CA)** dalam ekosistem komputasi awan.
3. Mampu menyiapkan lingkungan pengembangan Python terisolasi menggunakan **uv** di lingkungan WSL (Ubuntu).
4. Mampu membangun dan menjalankan aplikasi web berbasis **Python Flask** menjadi sebuah **Containerized App (CA)** menggunakan **Docker** melalui konfigurasi `Dockerfile`.

---

## Dasar Teori

### 1. Cloud Computing & XaaS (*Everything as a Service*)
**Cloud Computing** adalah model penyediaan sumber daya komputasi (seperti server, penyimpanan, basis data, jaringan, dan perangkat lunak) secara fleksibel melalui jaringan internet dengan pendekatan **XaaS** (*Everything as a Service*). Penyedia layanan (*provider*) mengelola infrastruktur di balik layar, sementara konsumen menggunakan sumber daya tersebut sesuai kebutuhan. Secara umum, XaaS dibagi menjadi tiga kategori utama:
* **IaaS (*Infrastructure as a Service*):** Layanan penyediaan infrastruktur komputasi dasar secara virtual (mesin virtual, penyimpanan, dan jaringan), di mana pengguna tetap mengelola sistem operasi hingga aplikasi di atasnya.
* **PaaS (*Platform as a Service*):** Layanan penyediaan platform siap pakai (sistem operasi, *runtime*, dan *middleware*) sehingga pengembang dapat langsung fokus membangun dan menjalankan kode aplikasi tanpa mengurus konfigurasi server.
* **SaaS (*Software as a Service*):** Layanan perangkat lunak jadi yang siap digunakan langsung oleh pengguna akhir melalui peramban web (*browser*) atau aplikasi klien tanpa perlu instalasi maupun pemeliharaan sistem.

### 2. Containerized App (CA) & Docker
**Containerized App (CA)** adalah aplikasi yang dikemas bersama seluruh dependensi, pustaka (*libraries*), *runtime*, dan konfigurasi lingkungannya ke dalam satu paket terisolasi yang disebut **Container**. Dengan pendekatan ini, aplikasi dapat berjalan secara konsisten di berbagai lingkungan komputasi tanpa mengalami masalah perbedaan konfigurasi (*"it works on my machine"*).

**Docker** merupakan salah satu platform *containerization* populer yang memanfaatkan berkas **`Dockerfile`**—berisi instruksi otomatis untuk membangun *container image*—sehingga proses distribusi (*deployment*) dan penskalaan aplikasi pada layanan *cloud* menjadi jauh lebih cepat, ringan, dan portabel.

---

## 0. Tugas 0: Analisis Layanan XaaS

### 1. Contoh Vendor untuk Setiap Kategori XaaS
* **SaaS (*Software as a Service*):** Google Workspace (sebelumnya G Suite) — *Vendor: Google*
* **PaaS (*Platform as a Service*):** Heroku — *Vendor: Salesforce*
* **IaaS (*Infrastructure as a Service*):** Amazon EC2 (*Elastic Compute Cloud*) — *Vendor: Amazon Web Services (AWS)*

### 2. Uraian Layanan (*Service*) dari Masing-Masing Vendor
* **Google Workspace (SaaS):**  
  Menyediakan perangkat lunak produktivitas dan kolaborasi berbasis *cloud* yang siap pakai langsung melalui *browser* atau aplikasi klien tanpa perlu proses instalasi maupun pengelolaan server oleh pengguna. Layanan meliputi surat elektronik bisnis (**Gmail**), penyimpanan awan (**Google Drive**), pengolah dokumen kolaboratif secara *real-time* (**Google Docs, Sheets, Slides**), serta konferensi video (**Google Meet**). Seluruh pembaruan fitur, keamanan data, hingga infrastruktur dikelola sepenuhnya oleh Google.
* **Heroku (PaaS):**  
  Menyediakan lingkungan *platform* siap pakai bagi *developer* untuk membangun, menjalankan, dan menskalakan aplikasi web atau API tanpa harus mengonfigurasi sistem operasi, *web server*, maupun jaringan secara manual. Layanan utamanya meliputi *runtime* terkelola (Python, Node.js, Java, Go, PHP), integrasi *deployment* otomatis dari repositori Git/GitHub, *add-ons* basis data terkelola (Heroku Postgres, Redis), serta penskalaan otomatis (*auto-scaling* pada *Dyno*).
* **Amazon EC2 (IaaS):**  
  Menyediakan sumber daya infrastruktur komputasi virtual yang fleksibel di atas pusat data AWS. Layanan meliputi penyewaan mesin virtual (**EC2 Instances**) dengan pilihan spesifikasi CPU, RAM, dan GPU tertentu, pemilihan sistem operasi (*Amazon Machine Images*), penyimpanan blok virtual (**Amazon EBS**), serta pengaturan jaringan virtual terisolasi (**Amazon VPC**), *firewall* (*Security Groups*), dan IP publik.

### 3. Arsitektur Umum Layanan Cloud Computing (XaaS)

![Arsitektur Umum Layanan Cloud Computing (XaaS)](images/00-arsitektur-xaas.png)

**Pembahasan Arsitektur:**  
Diagram di atas membandingkan pembagian tanggung jawab pengelolaan 9 lapisan sistem (*Networking, Storage, Servers, Virtualization, Operating System, Middleware, Runtime, Data,* dan *Applications*) antara model tradisional (*On-Premises*) dan layanan *Cloud*:
* **On-Premises:** Seluruh 9 lapisan dikelola sepenuhnya oleh pengguna/tim IT internal.
* **IaaS:** *Provider* mengelola lapisan fisik hingga virtualisasi (*Networking* s.d. *Virtualization*). Pengguna mengelola sistem operasi hingga aplikasi dan data.
* **PaaS:** *Provider* mengambil alih hingga lapisan *Runtime*. Pengguna hanya fokus mengelola **Data** dan kode **Aplikasi**.
* **SaaS:** *Provider* mengelola seluruh lapisan dari bawah hingga atas. Pengguna akhir langsung memakai perangkat lunak yang tersedia.

---

## Persiapan Source Code Aplikasi Flask di WSL

### Langkah 1: Membuat Direktori Kerja
Membuat direktori `$HOME/src` sebagai ruang kerja utama dan masuk ke dalamnya menggunakan perintah `mkdir -p $HOME/src && cd $HOME/src`.

![Membuat Direktori Kerja](images/01-buat-direktori.png)

### Langkah 2: Clone Repositori Flask dari GitHub
Mengunduh *source code* resmi Flask menggunakan `git clone https://github.com/pallets/flask`. Pada percobaan ini muncul pesan `destination path 'flask' already exists`, yang menandakan repositori `flask` sudah tersedia di dalam direktori `~/src`.

![Clone Repositori Flask](images/02-git-clone.png)

### Langkah 3: Menyalin Direktori Tutorial
Menyalin folder contoh aplikasi blog `examples/tutorial` menjadi folder proyek tersendiri bernama `flask-app` menggunakan perintah `cp -R ~/src/flask/examples/tutorial flask-app`.

![Salin Direktori Tutorial](images/03-copy-tutorial.png)

### Langkah 4: Verifikasi Isi Direktori & Masuk ke `flask-app`
Memeriksa daftar direktori dengan `ls -la` untuk memastikan folder `flask-app` berhasil terbentuk, kemudian masuk ke dalam direktori proyek menggunakan `cd flask-app`.

![Cek Isi Direktori](images/04-cek-direktori.png)

---

## 1. Membuat Virtual Environment Python 3.14.4

### Langkah 1: Membuat Environment dengan `uv`
Membuat *virtual environment* menggunakan manajer paket `uv` dengan interpreter **CPython 3.14.4** (`uv venv --python 3.14.4`). Karena `.venv` sudah ada sebelumnya, sistem menanyakan konfirmasi penggantian dan dijawab `yes`.

![Membuat Virtual Environment](images/05-uv-venv.png)

### Langkah 2: Mengaktifkan Virtual Environment
Mengaktifkan *virtual environment* dengan menjalankan `source .venv/bin/activate`. *Prompt* terminal berubah menjadi `(flask-app) acer@LAPTOP-8F38RLV8:~/src/flask-app$`, menandakan *environment* terisolasi telah aktif.

![Mengaktifkan Virtual Environment](images/06-activate-venv.png)

### Langkah 3: Memeriksa Struktur Direktori Proyek
Menjalankan `ls -la` untuk memverifikasi isi proyek. Terlihat folder `.venv`, `flaskr` (kode utama aplikasi), `pyproject.toml` (konfigurasi paket), `tests`, serta file pendukung lainnya.

![Struktur File Proyek](images/07-struktur-file.png)

---

## 2. Install Paket yang Diperlukan & Uji Aplikasi Secara Lokal

### Langkah 1: Instalasi Paket Proyek (`uv pip install -e .`)
Menginstal proyek `flaskr` beserta 8 dependensi utamanya (`flask 3.1.3`, `werkzeug 3.1.9`, `jinja2 3.1.6`, `click 8.5.1`, `blinker`, `itsdangerous`, `markupsafe`) dalam mode *editable* hanya dalam waktu 16ms.

![Instalasi Paket dengan uv](images/08-uv-pip-install.png)

### Langkah 2: Inisialisasi Database SQLite
Menjalankan perintah `flask --app flaskr init-db` untuk membuat skema tabel basis data SQLite. Terminal menampilkan pesan konfirmasi `Initialized the database.`.

![Inisialisasi Database](images/09-init-db.png)

### Langkah 3: Menjalankan Server Pengembangan Flask
Menjalankan aplikasi secara lokal menggunakan perintah `flask --app flaskr run`. Server aktif pada alamat lokal `http://127.0.0.1:5000`.

![Menjalankan Server Flask Lokal](images/10-flask-run-lokal.png)

### Langkah 4: Pengujian Aplikasi pada Browser
Membuka alamat `http://localhost:5000` di browser Windows untuk memverifikasi seluruh fitur aplikasi berjalan dengan baik:

* **Halaman Utama (Posts):** Menampilkan daftar postingan serta navigasi *Register* dan *Log In*.  
  ![Halaman Utama Localhost 5000](images/11-browser-5000-home.png)

* **Halaman Register (`/auth/register`):** Form pendaftaran akun pengguna baru (*Username* dan *Password*).  
  ![Halaman Register Localhost 5000](images/12-browser-5000-register.png)

* **Halaman Log In (`/auth/login`):** Form masuk pengguna yang telah terdaftar.  
  ![Halaman Login Localhost 5000](images/13-browser-5000-login.png)

* **Log HTTP Request di Terminal:** Terminal mencatat setiap permintaan `GET` ke halaman `/`, `/auth/register`, `/auth/login`, dan aset statis dengan kode status `200` (berhasil).  
  ![Log Terminal Server Lokal](images/14-log-server-lokal.png)

### Langkah 5: Menghentikan Server Lokal
Menghentikan proses server pengembangan Flask di terminal WSL dengan menekan kombinasi tombol **`Ctrl + C`** (`^C`) sebelum melanjutkan ke tahap *containerization*.

![Menghentikan Server Lokal](images/15-stop-server-lokal.png)

---

## 3. Membuat Aplikasi Menjadi Containerized App (CA) – Docker

### Langkah 3.1: Membuat `Dockerfile`
Membuat berkas konfigurasi `Dockerfile` di direktori *root* proyek menggunakan editor `nano Dockerfile`.

![Membuka Editor Nano Dockerfile](images/16-nano-dockerfile.png)

Isi konfigurasi `Dockerfile` yang digunakan untuk membangun *image*:

```dockerfile
FROM python:3.14

RUN mkdir /app
WORKDIR /app

ADD . /app/

RUN pip install -e .
RUN flask --app flaskr init-db

EXPOSE 5000
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]
```

![Isi Konfigurasi Dockerfile](images/17-isi-dockerfile.png)

**Penjelasan Instruksi Dockerfile:**
* `FROM python:3.14`: Menggunakan *base image* resmi Python 3.14.
* `RUN mkdir /app` & `WORKDIR /app`: Membuat dan menetapkan `/app` sebagai direktori kerja di dalam *container*.
* `ADD . /app/`: Menyalin seluruh kode proyek dari direktori lokal ke dalam `/app/` di *container*.
* `RUN pip install -e .` & `RUN flask --app flaskr init-db`: Menginstal dependensi Flask dan menginisialisasi database SQLite saat proses *build* berlangsung.
* `EXPOSE 5000`: Membuka port `5000` pada *container*.
* `CMD [...]`: Menjalankan aplikasi Flask pada semua antarmuka jaringan (`0.0.0.0`) di port `5000` saat *container* dijalankan.

### Langkah 3.2: Memastikan Layanan Docker Aktif
Karena menggunakan **Docker Desktop** di Windows yang sudah terintegrasi dengan WSL2, layanan *Docker daemon* langsung aktif tanpa perlu menjalankan `systemctl start docker`. Pengecekan dilakukan dengan perintah `docker ps` yang berhasil menampilkan status *container* yang sedang berjalan.

![Verifikasi Docker Aktif](images/18-docker-ps.png)

### Langkah 3.3: Membangun Docker Image (`docker build`)
Menjalankan perintah `docker build -f Dockerfile -t flaskr:1.0.0 .` untuk membangun *image* bernama `flaskr` dengan tag `1.0.0`. Seluruh 6 tahapan *build* selesai dengan status `FINISHED` dan menghasilkan *image* `docker.io/library/flaskr:1.0.0`.

![Proses Build Docker Image](images/19-docker-build.png)

### Langkah 3.4: Verifikasi Image Terbentuk
Memeriksa daftar *image* lokal menggunakan `docker image ls`. Terlihat *image* **`flaskr:1.0.0`** dengan ID `58b2fed8c574` dan ukuran `1.65GB` telah berhasil terdaftar.

![Daftar Docker Image](images/20-docker-image-ls.png)

---

## 4. Menjalankan Image Menjadi Container (`docker run`)

### Langkah 4.1: Menjalankan Container dengan Port Mapping
Menjalankan *image* `flaskr:1.0.0` menjadi *container* aktif menggunakan perintah `docker run -p 5001:5000 flaskr:1.0.0`. Perintah `-p 5001:5000` memetakan port `5000` di dalam *container* ke port `5001` pada *localhost* host Windows/WSL.

![Menjalankan Docker Container](images/21-docker-run.png)

### Langkah 4.2: Pengujian Containerized App pada Browser
Mengakses aplikasi melalui port hasil *mapping* yaitu **`http://localhost:5001`**:

* **Halaman Register (`http://localhost:5001/auth/register`):** Aplikasi berjalan normal dari dalam *container* Docker.  
  ![Halaman Register Container Port 5001](images/22-browser-5001-register.png)

* **Halaman Log In (`http://localhost:5001/auth/login`):** Halaman autentikasi dapat diakses dan merespons permintaan dengan baik.  
  ![Halaman Login Container Port 5001](images/23-browser-5001-login.png)

### Langkah 4.3: Log Akses Container & Menghentikan Container
Terminal menampilkan log akses HTTP dari IP *bridge network* Docker (`172.17.0.1`) dengan status `200`. Proses *container* kemudian dihentikan dengan menekan tombol **`Ctrl + C`** (`^C`).

![Log Akses dan Penghentian Container](images/24-stop-docker-container.png)

---

## Kesimpulan
Aplikasi web berbasis Python Flask (`flaskr`) berhasil dikemas menjadi *Containerized App* (CA) menggunakan Docker di lingkungan WSL (Ubuntu). Melalui *Dockerfile*, seluruh proses penyiapan lingkungan Python 3.14, instalasi paket, dan inisialisasi database dapat diotomatisasi ke dalam *image* `flaskr:1.0.0` yang portabel dan dapat dijalankan secara konsisten melalui pemetaan port `5001:5000`.
