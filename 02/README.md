# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Modul: Manajemen Proses Linux (WSL) & Implementasi GraphQL Server

### Identitas Mahasiswa
- **Nama:** Naisyah Izzatul Jannah K.
- **NIM:** 255410040
- **Kelas:** Informatika IF 1 (Semester 3)
- **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)

---

### A. Tujuan
1. Memahami dan mengonfigurasi lingkungan sistem operasi Linux di Windows menggunakan *Windows Subsystem for Linux* (WSL) serta memantau proses sistem secara *real-time* menggunakan utilitas `htop`.
2. Membangun server API modern menggunakan arsitektur **GraphQL** berbasis Python dengan pustaka `strawberry-graphql`.
3. Mengelola lingkungan virtual (*virtual environment*) dan dependensi menggunakan manajer paket cepat `uv` serta menguji kueri API melalui *GraphiQL Playground*.

---

### B. Dasar Teori
- **WSL (Windows Subsystem for Linux):** Fitur sistem operasi Windows yang memungkinkan pengembang menjalankan lingkungan Linux (termasuk perintah baris dan utilitas berbasis Linux) secara langsung tanpa perantara mesin virtual yang berat.
- **GraphQL:** Bahasa kueri modern untuk API yang memungkinkan klien menentukan struktur data yang dibutuhkan secara presisi, sehingga menghindari pengiriman data berlebih (*over-fetching*).
- **Strawberry GraphQL:** Kerangka kerja (*framework*) berbasis Python yang memanfaatkan atribut *type hints* standar Python untuk mendefinisikan skema, kueri, dan resolver GraphQL secara intuitif.

---

### C. Langkah Kerja & Bukti Praktikum

#### BAGIAN I: MANAJEMEN PROSES LINUX & LINGKUNGAN WSL

##### 1. Pengenalan & Inisialisasi Lingkungan WSL (Nomor 1)
- **Tujuan:** Mengunduh, memasang, dan memverifikasi lingkungan Linux Ubuntu pada sistem host Windows menggunakan WSL.
- **Langkah Kerja:** Menjalankan perintah pengunduhan dan instalasi WSL melalui antarmuka baris perintah Windows.
- **Bukti Praktikum:**
  <img width="800" alt="Instalasi WSL" src="https://github.com/user-attachments/assets/1191c5f4-395a-4224-85d0-722c24f4c127" />

##### 2. Pengenalan Utilitas Sistem & Kernel Linux (Nomor 2)
- **Tujuan:** Memahami cara kernel Linux mengelola alokasi memori, instruksi CPU, dan penomoran identifikasi proses (*Process ID / PID*).
- **Langkah Kerja:** Membaca informasi sistem dan panduan antarmuka WSL.
- **Bukti Praktikum:**
  <img width="800" alt="Welcome to WSL" src="https://github.com/user-attachments/assets/a8b77bd2-fc73-4c0b-99bf-fcce36cb4747" />

##### 3. Pemantauan Proses Sistem Secara Real-Time dengan `htop` (Nomor 3)
- **Tujuan:** Memantau penggunaan sumber daya hardware (CPU, RAM, Swap) serta daftar proses aktif di terminal Ubuntu.
- **Perintah Terminal:**
```bash
htop
Penjelasan: Perintah htop menampilkan statistik penggunaan CPU per inti (core), persentase konsumsi memori, serta rincian proses aktif beserta PID dan penggunaan CPU/RAM masing-masing.

Bukti Praktikum:

4. Navigasi Awal & Kesiapan Terminal (Nomor 4)
Tujuan: Memastikan terminal WSL aktif, mengecek direktori kerja, dan mempersiapkan sistem untuk masuk ke modul pengembangan berikutnya.

Perintah Terminal:

Bash
pwd
ls -la
Penjelasan: Memastikan posisi direktori aktif berada di dalam lingkungan Linux WSL sebelum membuat folder proyek baru.

Bukti Praktikum:

BAGIAN II: MEMBANGUN GRAPHQL SERVER DENGAN strawberry-graphql
1. Inisialisasi Direktori Kerja / Workspace (Nomor 1)
Tujuan: Membuat direktori proyek terisolasi bernama workspace-01 untuk menyimpan seluruh berkas server GraphQL.

Perintah Terminal:

Bash
mkdir workspace-01
cd workspace-01
Penjelasan: Perintah mkdir membuat direktori baru, dan cd digunakan untuk berpindah ke dalam lokasi direktori tersebut.

Bukti Praktikum:

2. Pengaturan Lingkungan Virtual Menggunakan uv (Nomor 2)
Tujuan: Mengisolasi dependensi Python proyek menggunakan pengelola paket berkinerja tinggi uv.

Perintah Terminal:

Bash
uv venv
source .venv/bin/activate
Penjelasan: uv venv menginisialisasi lingkungan virtual lokal pada direktori .venv, dan source mengaktifkannya agar pustaka yang diinstal tidak mengganggu sistem utama.

Bukti Praktikum:

3. Instalasi Pustaka strawberry-graphql (Nomor 3)
Tujuan: Memasang pustaka Strawberry GraphQL beserta perkakas antarmuka baris perintah (CLI).

Perintah Terminal:

Bash
uv pip install 'strawberry-graphql[cli]'
Penjelasan: Memasang modul strawberry-graphql beserta server ASGI uvicorn dan dependensi pendukungnya ke dalam virtual environment.

Bukti Praktikum:

4. Pembuatan Berkas Skema Server schema.py (Nomor 4)
Tujuan: Membuat dan membuka berkas kode sumber schema.py menggunakan teks editor berbasis terminal nano.

Perintah Terminal:

Bash
nano schema.py
Bukti Praktikum:

5. Penulisan Kode Sumber Skema GraphQL (Nomor 5)
Tujuan: Mendefinisikan struktur objek data Book serta root query untuk menyajikan data sampel buku.

Kode Sumber (schema.py):

Python
import strawberry

@strawberry.type
class Book:
    title: str
    author: str

@strawberry.type
class Query:
    @strawberry.field
    def books(self) -> list[Book]:
        return [
            Book(
                title="The Great Gatsby",
                author="F. Scott Fitzgerald",
            )
        ]

schema = strawberry.Schema(query=Query)
Penjelasan Kode:

import strawberry: Memuat kerangka kerja Strawberry.

@strawberry.type class Book: Mendefinisikan tipe data objek GraphQL Book yang memiliki atribut title dan author bertipe string.

@strawberry.type class Query: Mendefinisikan titik masuk kueri utama.

def books(...) -> list[Book]: Fungsi resolver yang mengembalikan daftar objek data buku tiruan (mock data).

schema = strawberry.Schema(query=Query): Menggabungkan tipe data dan kueri menjadi skema GraphQL aktif.

Bukti Praktikum:

6. Menjalankan Server Pengembangan GraphQL (Nomor 6)
Tujuan: Menjalankan peladen (server) lokal di lingkungan WSL agar API GraphQL dapat menerima permintaan HTTP.

Perintah Terminal:

Bash
strawberry dev schema
Penjelasan: Perintah ini meluncurkan development server Strawberry yang secara otomatis memperbarui perubahan kode dan dapat diakses pada alamat http://localhost:8000/graphql.

Bukti Praktikum:

7. Pengujian Kueri di GraphiQL Playground (Nomor 7)
Tujuan: Menguji keabsahan API GraphQL dengan mengeksekusi kueri melalui peramban web (browser).

Kueri Uji Coba:

GraphQL
{
  books {
    title
    author
  }
}
Hasil Response JSON:

JSON
{
  "data": {
    "books": [
      {
        "title": "The Great Gatsby",
        "author": "F. Scott Fitzgerald"
      }
    ]
  }
}
Penjelasan: Kueri dikirimkan dari panel kiri GraphiQL Playground, dan peladen berhasil mengembalikan respons JSON berisi judul dan penulis buku sesuai dengan logika resolver di schema.py.

Bukti Praktikum:

D. Kesimpulan
Seluruh rangkaian praktikum Sistem Terdistribusi dan Terdesentralisasi minggu ini telah berhasil diselesaikan dengan baik:

Bagian I: Pemahaman lingkungan sistem Linux di Windows melalui WSL serta pengelolaan dan pemantauan proses sistem berbasis baris perintah (htop) berhasil dipraktikkan secara lengkap dari nomor 1 hingga 4.

Bagian II: Pembangunan API modern berbasis GraphQL menggunakan Python dan kerangka kerja strawberry-graphql berhasil diimplementasikan secara menyeluruh dari nomor 1 sampai 7, mulai dari pengisolasian lingkungan uv, penulisan kode schema.py, hingga pengujian eksekusi kueri pada GraphiQL Playground dengan keluaran JSON yang valid.
