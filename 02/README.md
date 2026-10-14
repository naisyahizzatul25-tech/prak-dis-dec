# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Minggu 02: Manajemen Proses Linux (WSL) dan Implementasi GraphQL Server

### Identitas Mahasiswa
* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1

---

### A. Tujuan
1. Memahami dan mengonfigurasi lingkungan Linux di Windows menggunakan *Windows Subsystem for Linux* (WSL) serta memantau proses sistem secara *real-time* dengan `htop`.
2. Membangun server API modern berbasis **GraphQL** menggunakan Python dan pustaka `strawberry-graphql`.
3. Mengelola *virtual environment* dan dependensi dengan `uv`, serta menguji kueri API melalui *GraphiQL Playground*.

---

### B. Dasar Teori
* **WSL (Windows Subsystem for Linux)** adalah fitur Windows yang memungkinkan pengembang menjalankan lingkungan Linux dan utilitas baris perintahnya langsung tanpa mesin virtual yang berat.
* **GraphQL** adalah bahasa kueri untuk API yang memungkinkan klien menentukan struktur data yang dibutuhkan secara presisi, sehingga menghindari *over-fetching*.
* **Strawberry GraphQL** adalah *framework* Python yang memanfaatkan *type hints* standar untuk mendefinisikan skema, kueri, dan resolver GraphQL.

---

### C. Langkah Kerja & Bukti Praktikum

## BAGIAN I: Manajemen Proses Linux & Lingkungan WSL

#### 1. Instalasi & Inisialisasi WSL
Mengunduh, memasang, dan memverifikasi Linux Ubuntu pada Windows menggunakan WSL melalui antarmuka baris perintah Windows.
<img width="800" alt="Instalasi WSL" src="https://github.com/user-attachments/assets/1191c5f4-395a-4224-85d0-722c24f4c127" />

#### 2. Pengenalan Utilitas Sistem & Kernel Linux
Membaca informasi sistem dan panduan antarmuka WSL untuk memahami pengelolaan memori, instruksi CPU, dan penomoran proses (*Process ID / PID*) oleh kernel Linux.
<img width="800" alt="Welcome to WSL" src="https://github.com/user-attachments/assets/a8b77bd2-fc73-4c0b-99bf-fcce36cb4747" />

#### 3. Pemantauan Proses dengan `htop`
Memantau penggunaan CPU per inti, RAM, Swap, serta daftar proses aktif beserta PID-nya.
   * Perintah: `htop`

   <img width="800" alt="htop" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 4. Navigasi Awal & Kesiapan Terminal
Memastikan terminal WSL aktif dan posisi direktori berada di lingkungan Linux sebelum membuat folder proyek.
   * Perintah: `pwd` dan `ls -la`

   <img width="800" alt="navigasi terminal" src="TEMPEL_URL_GAMBAR_DI_SINI" />

---

## BAGIAN II: Membangun GraphQL Server dengan `strawberry-graphql`

#### 1. Membuat Direktori Kerja
Membuat direktori proyek terisolasi bernama `workspace-01` untuk menyimpan seluruh berkas server.
   * Perintah: `mkdir workspace-01` lalu `cd workspace-01`

   <img width="800" alt="pembuatan direktori" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 2. Membuat & Mengaktifkan Virtual Environment dengan `uv`
Mengisolasi dependensi Python proyek agar tidak mengganggu sistem utama.
   * Perintah: `uv venv`
   * Perintah: `source .venv/bin/activate`

   <img width="800" alt="aktivasi venv" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 3. Instalasi Pustaka `strawberry-graphql`
Memasang Strawberry GraphQL beserta CLI, server ASGI `uvicorn`, dan dependensi pendukungnya.
   * Perintah: `uv pip install 'strawberry-graphql[cli]'`

   <img width="800" alt="instalasi" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 4. Membuat Berkas `schema.py`
Membuka editor teks `nano` untuk membuat berkas skema server.
   * Perintah: `nano schema.py`

   <img width="800" alt="editor nano" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 5. Menulis Kode Skema GraphQL
Mendefinisikan tipe data `Book` dan *root query* untuk menyajikan data sampel buku.

```python
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
```

   * `@strawberry.type class Book`: tipe objek GraphQL dengan atribut `title` dan `author` bertipe string.
   * `@strawberry.type class Query`: titik masuk kueri utama.
   * `def books(...) -> list[Book]`: *resolver* yang mengembalikan daftar data buku tiruan (*mock data*).
   * `strawberry.Schema(query=Query)`: menggabungkan tipe dan kueri menjadi skema GraphQL aktif.

   <img width="800" alt="kode sumber" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 6. Menjalankan Server Pengembangan
Menjalankan server lokal agar API menerima permintaan HTTP di `http://localhost:8000/graphql`.
   * Perintah: `strawberry dev schema`

   <img width="800" alt="server berjalan" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 7. Pengujian Kueri di GraphiQL Playground
Mengeksekusi kueri melalui peramban untuk menguji keabsahan API.

Kueri:
```graphql
{
  books {
    title
    author
  }
}
```

Respons JSON:
```json
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
```

   <img width="800" alt="Playground & JSON Response" src="TEMPEL_URL_GAMBAR_DI_SINI" />

---

### D. Kesimpulan
Praktikum minggu ini telah berhasil menyelesaikan seluruh rangkaian kegiatan. Pada Bagian I, lingkungan Linux di Windows melalui WSL serta pemantauan proses dengan `htop` berhasil dipraktikkan. Pada Bagian II, API berbasis GraphQL dengan Python dan `strawberry-graphql` berhasil dibangun, mulai dari isolasi lingkungan dengan `uv`, penulisan `schema.py`, hingga pengujian kueri pada GraphiQL Playground dengan keluaran JSON yang valid.
