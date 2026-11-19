# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi

## Modul 7: Cloud Computing & Containerized App (CA)
 
**Nama:** Naisyah Izzatul Jannah K.  
**NIM:** 255410040  
**Prodi:** Informatika (IF 1) - Universitas Teknologi Digital Indonesia (UTDI)  
**Mata Kuliah:** Praktikum Sistem Terdistribusi dan Terdesentralisasi  

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

<img width="940" height="592" alt="image" src="https://github.com/user-attachments/assets/5a22db92-06f0-4006-b759-3d27b7d55c11" />

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

<img width="726" height="116" alt="image" src="https://github.com/user-attachments/assets/016d669e-698b-4767-b6be-3f0edd40afcd" />

### Langkah 2: Clone Repositori Flask dari GitHub
Mengunduh *source code* resmi Flask menggunakan `git clone https://github.com/pallets/flask`. Pada percobaan ini muncul pesan `destination path 'flask' already exists`, yang menandakan repositori `flask` sudah tersedia di dalam direktori `~/src`.

<img width="940" height="86" alt="image" src="https://github.com/user-attachments/assets/80f055f5-0104-48c0-835e-2f267a873f29" />

### Langkah 3: Menyalin Direktori Tutorial
Menyalin folder contoh aplikasi blog `examples/tutorial` menjadi folder proyek tersendiri bernama `flask-app` menggunakan perintah `cp -R ~/src/flask/examples/tutorial flask-app`.

<img width="940" height="95" alt="image" src="https://github.com/user-attachments/assets/e51e58a0-306f-47ac-96dd-80ef8bd4a43a" />

### Langkah 4: Verifikasi Isi Direktori & Masuk ke `flask-app`
Memeriksa daftar direktori dengan `ls -la` untuk memastikan folder `flask-app` berhasil terbentuk, kemudian masuk ke dalam direktori proyek menggunakan `cd flask-app`.

<img width="864" height="297" alt="image" src="https://github.com/user-attachments/assets/5ad3063f-740d-4db0-b482-f3a81a607ebe" />

---

## 1. Membuat Virtual Environment Python 3.14.4

### Langkah 1: Membuat Environment dengan `uv`
Membuat *virtual environment* menggunakan manajer paket `uv` dengan interpreter **CPython 3.14.4** (`uv venv --python 3.14.4`). Karena `.venv` sudah ada sebelumnya, sistem menanyakan konfirmasi penggantian dan dijawab `yes`.

<img width="940" height="165" alt="image" src="https://github.com/user-attachments/assets/6ad0a08e-66df-4a78-9126-0d4ad9a38057" />

### Langkah 2: Mengaktifkan Virtual Environment
Mengaktifkan *virtual environment* dengan menjalankan `source .venv/bin/activate`. *Prompt* terminal berubah menjadi `(flask-app) acer@LAPTOP-8F38RLV8:~/src/flask-app$`, menandakan *environment* terisolasi telah aktif.

<img width="940" height="79" alt="image" src="https://github.com/user-attachments/assets/c5fd1985-73bb-40fb-ae06-174080f71f6c" />

### Langkah 3: Memeriksa Struktur Direktori Proyek
Menjalankan `ls -la` untuk memverifikasi isi proyek. Terlihat folder `.venv`, `flaskr` (kode utama aplikasi), `pyproject.toml` (konfigurasi paket), `tests`, serta file pendukung lainnya.

<img width="940" height="374" alt="image" src="https://github.com/user-attachments/assets/eca21819-e65d-4278-8d40-716c05acb8a9" />

---

## 2. Install Paket yang Diperlukan & Uji Aplikasi Secara Lokal

### Langkah 1: Instalasi Paket Proyek (`uv pip install -e .`)
Menginstal proyek `flaskr` beserta 8 dependensi utamanya (`flask 3.1.3`, `werkzeug 3.1.9`, `jinja2 3.1.6`, `click 8.5.1`, `blinker`, `itsdangerous`, `markupsafe`) dalam mode *editable* hanya dalam waktu 16ms.

<img width="940" height="384" alt="image" src="https://github.com/user-attachments/assets/fa483aa7-82ec-4bf3-a88c-426e2af907b5" />

### Langkah 2: Inisialisasi Database SQLite
Menjalankan perintah `flask --app flaskr init-db` untuk membuat skema tabel basis data SQLite. Terminal menampilkan pesan konfirmasi `Initialized the database.`.

<img width="940" height="83" alt="image" src="https://github.com/user-attachments/assets/bf05d0b1-5ecc-4eb9-b59d-7fc4c2946b6b" />

### Langkah 3: Menjalankan Server Pengembangan Flask
Menjalankan aplikasi secara lokal menggunakan perintah `flask --app flaskr run`. Server aktif pada alamat lokal `http://127.0.0.1:5000`.

<img width="940" height="102" alt="image" src="https://github.com/user-attachments/assets/18d07907-845a-4a04-b9e8-4ba3da3ca06f" />

### Langkah 4: Pengujian Aplikasi pada Browser
Membuka alamat `http://localhost:5000` di browser Windows untuk memverifikasi seluruh fitur aplikasi berjalan dengan baik:

* **Halaman Utama (Posts):** Menampilkan daftar postingan serta navigasi *Register* dan *Log In*.  
 <img width="940" height="142" alt="image" src="https://github.com/user-attachments/assets/d2e58e24-f54b-44f9-9f46-9c7abb1b3b04" />

* **Halaman Register (`/auth/register`):** Form pendaftaran akun pengguna baru (*Username* dan *Password*).  
  <img width="940" height="330" alt="image" src="https://github.com/user-attachments/assets/8f162d71-7e6c-4704-9969-0883281d4dc2" />

* **Halaman Log In (`/auth/login`):** Form masuk pengguna yang telah terdaftar.  
<img width="940" height="319" alt="image" src="https://github.com/user-attachments/assets/03e02a00-c5e7-42f5-8461-5ce880413db6" />

* **Log HTTP Request di Terminal:** Terminal mencatat setiap permintaan `GET` ke halaman `/`, `/auth/register`, `/auth/login`, dan aset statis dengan kode status `200` (berhasil).  
 <img width="940" height="310" alt="image" src="https://github.com/user-attachments/assets/32ece980-53ce-41f0-8991-d8f542864452" />

### Langkah 5: Menghentikan Server Lokal
Menghentikan proses server pengembangan Flask di terminal WSL dengan menekan kombinasi tombol **`Ctrl + C`** (`^C`) sebelum melanjutkan ke tahap *containerization*.

<img width="940" height="323" alt="image" src="https://github.com/user-attachments/assets/925a7e97-b9df-474f-876d-c77d60e7bd2c" />

---

## 3. Membuat Aplikasi Menjadi Containerized App (CA) – Docker

### Langkah 3.1: Membuat `Dockerfile`
Membuat berkas konfigurasi `Dockerfile` di direktori *root* proyek menggunakan editor `nano Dockerfile`.

<img width="940" height="53" alt="image" src="https://github.com/user-attachments/assets/9a3e46dd-e746-48c7-9afc-0f5a1f8ff0a8" />


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

<img width="940" height="479" alt="image" src="https://github.com/user-attachments/assets/83faac6a-829e-4647-8b22-3b3d034fbd47" />

**Penjelasan Instruksi Dockerfile:**
* `FROM python:3.14`: Menggunakan *base image* resmi Python 3.14.
* `RUN mkdir /app` & `WORKDIR /app`: Membuat dan menetapkan `/app` sebagai direktori kerja di dalam *container*.
* `ADD . /app/`: Menyalin seluruh kode proyek dari direktori lokal ke dalam `/app/` di *container*.
* `RUN pip install -e .` & `RUN flask --app flaskr init-db`: Menginstal dependensi Flask dan menginisialisasi database SQLite saat proses *build* berlangsung.
* `EXPOSE 5000`: Membuka port `5000` pada *container*.
* `CMD [...]`: Menjalankan aplikasi Flask pada semua antarmuka jaringan (`0.0.0.0`) di port `5000` saat *container* dijalankan.

### Langkah 3.2: Memastikan Layanan Docker Aktif
Karena menggunakan **Docker Desktop** di Windows yang sudah terintegrasi dengan WSL2, layanan *Docker daemon* langsung aktif tanpa perlu menjalankan `systemctl start docker`. Pengecekan dilakukan dengan perintah `docker ps` yang berhasil menampilkan status *container* yang sedang berjalan.

<img width="940" height="129" alt="image" src="https://github.com/user-attachments/assets/8bcc98a8-99d0-4953-9e21-efae4e9db962" />

### Langkah 3.3: Membangun Docker Image (`docker build`)
Menjalankan perintah `docker build -f Dockerfile -t flaskr:1.0.0 .` untuk membangun *image* bernama `flaskr` dengan tag `1.0.0`. Seluruh 6 tahapan *build* selesai dengan status `FINISHED` dan menghasilkan *image* `docker.io/library/flaskr:1.0.0`.

<img width="940" height="535" alt="image" src="https://github.com/user-attachments/assets/d099787f-84b2-4055-9971-006f54ba648e" />

### Langkah 3.4: Verifikasi Image Terbentuk
Memeriksa daftar *image* lokal menggunakan `docker image ls`. Terlihat *image* **`flaskr:1.0.0`** dengan ID `58b2fed8c574` dan ukuran `1.65GB` telah berhasil terdaftar.

<img width="940" height="111" alt="image" src="https://github.com/user-attachments/assets/cd3ee2db-76ad-497d-8868-5d0276561817" />

---

## 4. Menjalankan Image Menjadi Container (`docker run`)

### Langkah 4.1: Menjalankan Container dengan Port Mapping
Menjalankan *image* `flaskr:1.0.0` menjadi *container* aktif menggunakan perintah `docker run -p 5001:5000 flaskr:1.0.0`. Perintah `-p 5001:5000` memetakan port `5000` di dalam *container* ke port `5001` pada *localhost* host Windows/WSL.

<img width="940" height="131" alt="image" src="https://github.com/user-attachments/assets/8e7b9254-0066-4e0e-a77e-20561ddc98aa" />

### Langkah 4.2: Pengujian Containerized App pada Browser
Mengakses aplikasi melalui port hasil *mapping* yaitu **`http://localhost:5001`**:

* **Halaman Register (`http://localhost:5001/auth/register`):** Aplikasi berjalan normal dari dalam *container* Docker.  
  <img width="940" height="274" alt="image" src="https://github.com/user-attachments/assets/08381188-a856-4978-97b7-7b5eb3f63d31" />

* **Halaman Log In (`http://localhost:5001/auth/login`):** Halaman autentikasi dapat diakses dan merespons permintaan dengan baik.  
 <img width="940" height="271" alt="image" src="https://github.com/user-attachments/assets/2023a9f2-fda9-4892-aafb-81485b085ef8" />

### Langkah 4.3: Log Akses Container & Menghentikan Container
Terminal menampilkan log akses HTTP dari IP *bridge network* Docker (`172.17.0.1`) dengan status `200`. Proses *container* kemudian dihentikan dengan menekan tombol **`Ctrl + C`** (`^C`).

<img width="940" height="505" alt="image" src="https://github.com/user-attachments/assets/d125946c-cded-472b-8038-4ab03f077475" />


---

## Kesimpulan
Aplikasi web berbasis Python Flask (`flaskr`) berhasil dikemas menjadi *Containerized App* (CA) menggunakan Docker di lingkungan WSL (Ubuntu). Melalui *Dockerfile*, seluruh proses penyiapan lingkungan Python 3.14, instalasi paket, dan inisialisasi database dapat diotomatisasi ke dalam *image* `flaskr:1.0.0` yang portabel dan dapat dijalankan secara konsisten melalui pemetaan port `5001:5000`.
