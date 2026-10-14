# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Modul: Manajemen Proses Linux (WSL) & Implementasi GraphQL Server

### Identitas Mahasiswa
- **Nama:** Naisyah Izzatul Jannah K.[cite: 1]
- **NIM:** 255410040[cite: 1]
- **Kelas:** Informatika IF 1 (Semester 3)[cite: 1]
- **Universitas:** Universitas Teknologi Digital Indonesia (UTDI)[cite: 1]

---

### A. Tujuan
1. Memahami dan mengonfigurasi lingkungan sistem operasi Linux di Windows menggunakan *Windows Subsystem for Linux* (WSL) serta memantau proses sistem secara *real-time* menggunakan utilitas `htop`[cite: 1].
2. Membangun server API modern menggunakan arsitektur **GraphQL** berbasis Python dengan pustaka `strawberry-graphql`[cite: 1].
3. Mengelola lingkungan virtual (*virtual environment*) dan dependensi menggunakan manajer paket cepat `uv` serta menguji kueri API melalui *GraphiQL Playground*[cite: 1].

---

### B. Dasar Teori
- **WSL (Windows Subsystem for Linux):** Fitur sistem operasi Windows yang memungkinkan pengembang menjalankan lingkungan Linux (termasuk perintah baris dan utilitas berbasis Linux) secara langsung tanpa perantara mesin virtual yang berat[cite: 1].
- **GraphQL:** Bahasa kueri modern untuk API yang memungkinkan klien menentukan struktur data yang dibutuhkan secara presisi, sehingga menghindari pengiriman data berlebih (*over-fetching*)[cite: 1].
- **Strawberry GraphQL:** Kerangka kerja (*framework*) berbasis Python yang memanfaatkan atribut *type hints* standar Python untuk mendefinisikan skema, kueri, dan resolver GraphQL secara intuitif[cite: 1].

---

### C. Langkah Kerja & Bukti Praktikum

#### BAGIAN I: MANAJEMEN PROSES LINUX & LINGKUNGAN WSL

##### 1. Pengenalan & Inisialisasi Lingkungan WSL (Nomor 1)
- **Tujuan:** Mengunduh, memasang, dan memverifikasi lingkungan Linux Ubuntu pada sistem host Windows menggunakan WSL[cite: 1].
- **Langkah Kerja:** Menjalankan perintah pengunduhan dan instalasi WSL melalui antarmuka baris perintah Windows[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Instalasi WSL" src="images/01-wsl-download.png" />[cite: 1, 6]

##### 2. Pengenalan Utilitas Sistem & Kernel Linux (Nomor 2)
- **Tujuan:** Memahami cara kernel Linux mengelola alokasi memori, instruksi CPU, dan penomoran identifikasi proses (*Process ID / PID*)[cite: 1].
- **Langkah Kerja:** Membaca informasi sistem dan panduan antarmuka WSL[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Welcome to WSL" src="images/03-welcome-wsl.png" />[cite: 1, 6]

##### 3. Pemantauan Proses Sistem Secara Real-Time dengan `htop` (Nomor 3)
- **Tujuan:** Memantau penggunaan sumber daya hardware (CPU, RAM, Swap) serta daftar proses aktif di terminal Ubuntu[cite: 1].
- **Perintah Terminal:**
  bash
htop
[cite: 1]
- **Penjelasan:** Perintah `htop` menampilkan statistik penggunaan CPU per inti (*core*), persentase konsumsi memori, serta rincian proses aktif beserta PID dan penggunaan CPU/RAM masing-masing[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Pemantauan Proses htop" src="images/02-htop.png" />[cite: 1, 6]

##### 4. Navigasi Awal & Kesiapan Terminal (Nomor 4)
- **Tujuan:** Memastikan terminal WSL aktif, mengecek direktori kerja, dan mempersiapkan sistem untuk masuk ke modul pengembangan berikutnya[cite: 1].
- **Perintah Terminal:**
bash
pwd
ls -la
`[cite: 1]
- **Penjelasan:** Memastikan posisi direktori aktif berada di dalam lingkungan Linux WSL sebelum membuat folder proyek baru[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Navigasi Terminal" src="images/03-welcome-wsl.png" />[cite: 1, 6]

---

#### BAGIAN II: MEMBANGUN GRAPHQL SERVER DENGAN `strawberry-graphql`

##### 1. Inisialisasi Direktori Kerja / Workspace (Nomor 1)
- **Tujuan:** Membuat direktori proyek terisolasi bernama `workspace-01` untuk menyimpan seluruh berkas server GraphQL[cite: 1].
- **Perintah Terminal:**
```bash
mkdir workspace-01
cd workspace-01
```[cite: 1]
- **Penjelasan:** Perintah `mkdir` membuat direktori baru, dan `cd` digunakan untuk berpindah ke dalam lokasi direktori tersebut[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Inisialisasi Workspace" src="images/04-workspace-uv-venv.png" />[cite: 1, 6]

##### 2. Pengaturan Lingkungan Virtual Menggunakan `uv` (Nomor 2)
- **Tujuan:** Mengisolasi dependensi Python proyek menggunakan pengelola paket berkinerja tinggi `uv`[cite: 1].
- **Perintah Terminal:**
```bash
uv venv
source .venv/bin/activate
```[cite: 1]
- **Penjelasan:** `uv venv` menginisialisasi lingkungan virtual lokal pada direktori `.venv`, dan `source` mengaktifkannya agar pustaka yang diinstal tidak mengganggu sistem utama[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Aktivasi Virtual Environment" src="images/04-workspace-uv-venv.png" />[cite: 1, 6]

##### 3. Instalasi Pustaka `strawberry-graphql` (Nomor 3)
- **Tujuan:** Memasang pustaka Strawberry GraphQL beserta perkakas antarmuka baris perintah (CLI)[cite: 1].
- **Perintah Terminal:**
```bash
uv pip install 'strawberry-graphql[cli]'
```[cite: 1]
- **Penjelasan:** Memasang modul `strawberry-graphql` beserta server ASGI `uvicorn` dan dependensi pendukungnya ke dalam virtual environment[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Instalasi Strawberry GraphQL" src="images/05-install-strawberry.png" />[cite: 1, 6]

##### 4. Pembuatan Berkas Skema Server `schema.py` (Nomor 4)
- **Tujuan:** Membuat dan membuka berkas kode sumber `schema.py` menggunakan teks editor berbasis terminal `nano`[cite: 1].
- **Perintah Terminal:**
```bash
nano schema.py
```[cite: 1]
- **Bukti Praktikum:**
  <img width="800" alt="Pembuatan Berkas schema.py" src="images/06-nano-kosong.png" />[cite: 1, 6]

##### 5. Penulisan Kode Sumber Skema GraphQL (Nomor 5)
- **Tujuan:** Mendefinisikan struktur objek data `Book` serta root query untuk menyajikan data sampel buku[cite: 1].
- **Kode Sumber (`schema.py`):**
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
```[cite: 1]
- **Penjelasan Kode:**
  - `import strawberry`: Memuat kerangka kerja Strawberry[cite: 1].
  - `@strawberry.type class Book`: Mendefinisikan tipe data objek GraphQL `Book` yang memiliki atribut `title` dan `author` bertipe string[cite: 1].
  - `@strawberry.type class Query`: Mendefinisikan titik masuk kueri utama[cite: 1].
  - `def books(...) -> list[Book]`: Fungsi resolver yang mengembalikan daftar objek data buku tiruan (*mock data*)[cite: 1].
  - `schema = strawberry.Schema(query=Query)`: Menggabungkan tipe data dan kueri menjadi skema GraphQL aktif[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Kode Sumber schema.py" src="images/07-nano-kode.png" />[cite: 1, 6]

##### 6. Menjalankan Server Pengembangan GraphQL (Nomor 6)
- **Tujuan:** Menjalankan peladen (*server*) lokal di lingkungan WSL agar API GraphQL dapat menerima permintaan HTTP[cite: 1].
- **Perintah Terminal:**
```bash
strawberry dev schema
```[cite: 1]
- **Penjelasan:** Perintah ini meluncurkan *development server* Strawberry yang secara otomatis memperbarui perubahan kode dan dapat diakses pada alamat `http://localhost:8000/graphql`[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Server GraphQL Berjalan" src="images/08-nano-simpan.png" />[cite: 1, 6]

##### 7. Pengujian Kueri di GraphiQL Playground (Nomor 7)
- **Tujuan:** Menguji keabsahan API GraphQL dengan mengeksekusi kueri melalui peramban web (*browser*)[cite: 1].
- **Kueri Uji Coba:**
```graphql
{
  books {
    title
    author
  }
}
```[cite: 1]
- **Hasil Response JSON:**
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
```[cite: 1]
- **Penjelasan:** Kueri dikirimkan dari panel kiri GraphiQL Playground, dan peladen berhasil mengembalikan respons JSON berisi judul dan penulis buku sesuai dengan logika resolver di `schema.py`[cite: 1].
- **Bukti Praktikum:**
  <img width="800" alt="Pengujian GraphiQL Playground" src="images/09-graphiql-hasil.png" />[cite: 1, 6]

---

### D. Kesimpulan
Seluruh rangkaian praktikum Sistem Terdistribusi dan Terdesentralisasi minggu ini telah berhasil diselesaikan dengan baik:
1. **Bagian I:** Pemahaman lingkungan sistem Linux di Windows melalui WSL serta pengelolaan dan pemantauan proses sistem berbasis baris perintah (`htop`) berhasil dipraktikkan secara lengkap dari nomor 1 hingga 4[cite: 1].
2. **Bagian II:** Pembangunan API modern berbasis GraphQL menggunakan Python dan kerangka kerja `strawberry-graphql` berhasil diimplementasikan secara menyeluruh dari nomor 1 sampai 7, mulai dari pengisolasian lingkungan `uv`, penulisan kode `schema.py`, hingga pengujian eksekusi kueri pada GraphiQL Playground dengan keluaran JSON yang valid[cite: 1].
