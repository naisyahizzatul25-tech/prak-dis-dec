# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Minggu 02: Komunikasi Antar Proses pada Sistem Terdistribusi

### Identitas Mahasiswa
* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika IF 1

---

### A. Tujuan
1. Memahami konsep proses pada satu *node* yang dikelola oleh sistem operasi, serta cara menampilkan, menjalankan, me-*restart*, dan mematikan proses.
2. Memahami perbedaan komunikasi antar proses pada satu *node* dengan komunikasi antar proses pada sistem terdistribusi.
3. Membangun GraphQL server menggunakan Python (paket Strawberry) dengan `uv`, lalu mengujinya melalui GraphiQL.

---

### B. Dasar Teori
* **Proses** adalah hasil eksekusi program/aplikasi yang bersifat *executable*. Proses dikelola oleh sistem operasi dan terdiri atas *executable code*, data, *resources*, serta informasi *state* (*stack* dan *heap*).
* **Proses pada satu node** sepenuhnya berada dalam kendali sistem operasi (eksekusi, alokasi *resources*, pengelolaan, dan komunikasi antar proses) dan bersifat transparan bagi pengguna. Semua proses berjalan pada *clock* yang sama dan dapat memakai *shared memory*.
* **Komunikasi antar proses pada sistem terdistribusi** berbeda karena antar *node* tidak berbagi *clock* maupun *shared memory*, sehingga dibutuhkan mekanisme khusus. Salah satunya **GraphQL**, di mana server melayani *query* dari *client* sesuai spesifikasi GraphQL dan kedua sisi dapat dikembangkan dengan peranti yang berbeda.
* **Strawberry** adalah paket Python untuk membuat GraphQL server. **uv** adalah pengelola paket dan *virtual environment* Python.

---

### C. Langkah Kerja & Bukti Praktikum

## BAGIAN I: Proses pada Satu Node

#### 0. Persiapan Lingkungan Linux (WSL)
Praktikum dikerjakan di Ubuntu pada Windows menggunakan WSL, dimulai dengan pengunduhan dan pemasangan WSL.
<img width="800" alt="Download WSL" src="images/01-wsl-download.png" />

Setelah terpasang, muncul jendela *Welcome to Windows Subsystem for Linux*.
<img width="800" alt="Welcome to WSL" src="images/03-welcome-wsl.png" />

#### 1. Menampilkan Proses yang Ada pada Komputer
Menampilkan seluruh proses pada Linux (WSL) menggunakan `htop`, berisi penggunaan CPU per inti, memori, *swap*, serta daftar proses lengkap dengan PID, pengguna, dan perintah yang dijalankan.
   * Perintah: `htop`

<img width="800" alt="Daftar proses htop" src="images/02-htop.png" />

#### 2. Menjalankan Aplikasi dan Melihat Proses yang Dimunculkan
Menjalankan salah satu aplikasi, lalu mencari proses yang dimunculkannya melalui `htop` (atau `ps`).
   * Aplikasi yang dijalankan: **(isi nama aplikasi)**
   * Perintah: `ps aux | grep <nama-aplikasi>`

<img width="800" alt="Proses aplikasi yang dijalankan" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 3. Me-restart dan Mematikan Proses
Petunjuk yang digunakan:
   * **Mematikan proses:** `kill <PID>` (sinyal SIGTERM, meminta proses berhenti dengan baik). Jika tidak mau berhenti: `kill -9 <PID>` (SIGKILL, paksa berhenti). Alternatif: `pkill <nama-proses>`, atau di `htop` pilih proses lalu tekan `F9` (Kill).
   * **Me-restart proses:** Proses biasa dimatikan lalu dijalankan kembali. Untuk layanan (*service*) digunakan `sudo systemctl restart <nama-service>`, sedangkan `kill -HUP <PID>` meminta proses membaca ulang konfigurasinya.

Proses dari aplikasi yang dijalankan dimatikan menggunakan perintah `kill` (bukan perintah keluar dari aplikasi):

<img width="800" alt="Mematikan proses dengan kill" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 4. Penjelasan
Aplikasi yang dijalankan otomatis menjadi proses yang diberi PID oleh kernel dan tampil pada `htop`. Proses tersebut dimatikan dengan mengirim sinyal ke PID-nya, sehingga sistem operasi menghentikan proses tanpa melalui menu keluar aplikasi. Seluruh pengelolaan ini ditangani sistem operasi pada satu *node* dan bersifat transparan bagi pengguna.

---

## BAGIAN II: Komunikasi Antar Proses pada Sistem Terdistribusi (GraphQL Server dengan Strawberry)

#### 1. Mempelajari `uv` dan Membuat Workspace `workspace-01`
Membuat workspace bernama `workspace-01`, lalu berpindah ke dalamnya.
   * Perintah: `mkdir workspace-01`
   * Perintah: `cd workspace-01`

Saat `uv` dijalankan, muncul pesan `uv: command not found` karena `uv` belum terpasang, sehingga `uv` dipasang terlebih dahulu.
   * Perintah: `curl -LsSf https://astral.sh/uv/install.sh | sh`
   * Perintah: `source $HOME/.local/bin/env` (menambahkan `uv` ke PATH)

<img width="800" alt="Workspace dan instalasi uv" src="images/04-workspace-uv-venv.png" />

#### 2. Membuat dan Mengaktifkan Environment
`uv` menggunakan Python 3.14.4 yang ada pada sistem (`/usr/bin/python3`) untuk membuat *virtual environment* di `.venv`, lalu environment diaktifkan (muncul penanda `(workspace-01)` pada *prompt*).
   * Perintah: `uv venv`
   * Perintah: `source .venv/bin/activate`

<img width="800" alt="Membuat dan mengaktifkan venv" src="images/04-workspace-uv-venv.png" />

#### 3. Instalasi Paket yang Diperlukan
Memasang Strawberry GraphQL beserta CLI. Sebanyak 24 paket terpasang, di antaranya `strawberry-graphql`, `graphql-core`, `starlette`, dan `uvicorn`.
   * Perintah: `uv pip install 'strawberry-graphql[cli]'`

<img width="800" alt="Instalasi strawberry-graphql" src="images/05-install-strawberry.png" />

#### 4. Membuat Berkas `schema.py`
Membuka editor `nano` untuk menulis source code server.
   * Perintah: `nano schema.py`

<img width="800" alt="nano schema.py kosong" src="images/06-nano-kosong.png" />

#### 5. Menuliskan Source Code `schema.py`
Mendefinisikan tipe `Book`, *root query* `books`, dan skema GraphQL.

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
   * `def books(...) -> list[Book]`: *resolver* yang mengembalikan daftar data buku.
   * `strawberry.Schema(query=Query)`: menggabungkan tipe dan kueri menjadi skema GraphQL.

<img width="800" alt="Source code schema.py" src="images/07-nano-kode.png" />

Berkas disimpan dengan `Ctrl+O` (*Write Out*), lalu keluar dengan `Ctrl+X`.

<img width="800" alt="Menyimpan schema.py" src="images/08-nano-simpan.png" />

#### 6. Menjalankan Server
Menjalankan server pengembangan Strawberry, yang dapat diakses di `http://localhost:8000/graphql` (alamat `0.0.0.0` dapat diganti `localhost` atau `127.0.0.1`).
   * Perintah: `strawberry dev schema`

<img width="800" alt="Server strawberry berjalan" src="TEMPEL_URL_GAMBAR_DI_SINI" />

#### 7. Menjalankan Query di GraphiQL
Membuka `http://localhost:8000/graphql` di peramban. Bagian kiri untuk menuliskan *query*, bagian kanan menampilkan hasil. Query dijalankan dengan tombol *Run*.

Query:
```graphql
{
  books {
    title
    author
  }
}
```

Hasil:
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

<img width="800" alt="Hasil query di GraphiQL" src="images/09-graphiql-hasil.png" />

Server dimatikan dengan menekan `Ctrl+C` pada *shell* tempat `strawberry` dijalankan.

---

## TUGAS: Client untuk GraphQL Server
Membuat *client* dengan Python (hanya pustaka standar) yang mengirim *query* ke GraphQL server di atas. Server harus berjalan terlebih dahulu (`strawberry dev schema`), lalu *client* dijalankan di terminal lain.

Berkas `client.py`:
```python
import json
import urllib.request

URL = "http://localhost:8000/graphql"
QUERY = "{ books { title author } }"

request = urllib.request.Request(
    URL,
    data=json.dumps({"query": QUERY}).encode("utf-8"),
    headers={"Content-Type": "application/json"},
)

with urllib.request.urlopen(request) as response:
    result = json.load(response)

for book in result["data"]["books"]:
    print(f'{book["title"]} - {book["author"]}')
```

   * Perintah: `python client.py`

<img width="800" alt="Hasil client GraphQL" src="TEMPEL_URL_GAMBAR_DI_SINI" />

*Client* (peminta layanan) dan server (pelayan) merupakan dua proses terpisah yang berkomunikasi melalui HTTP dengan format *query* GraphQL, sehingga contoh ini menunjukkan komunikasi antar proses pada sistem terdistribusi.

---

### D. Kesimpulan
Praktikum minggu ini menunjukkan bahwa pada satu *node*, proses dikelola penuh oleh sistem operasi dan dapat ditampilkan, dijalankan, serta dimatikan menggunakan `htop` dan `kill`. Pada sistem terdistribusi, komunikasi antar proses membutuhkan mekanisme khusus karena tidak ada *clock* dan *shared memory* bersama. GraphQL server berbasis Strawberry berhasil dibangun dengan `uv` dan diuji melalui GraphiQL (hasil JSON valid), kemudian diakses oleh *client* sebagai contoh komunikasi antar proses.
