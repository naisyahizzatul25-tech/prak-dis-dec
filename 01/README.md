# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Minggu 01: Pengenalan Sistem Terdistribusi dan Terdesentralisasi - Git dan GitHub

### Identitas Mahasiswa
* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika

---

### A. Tujuan
1. Memahami konsep dasar Version Control System (VCS) dan platform kolaborasi GitHub.
2. Menguasai alur instalasi serta konfigurasi opsi-opsi utama Git di sistem operasi Windows.
3. Mampu mengelola repository lokal, repository akun pribadi, repository organisasi, serta melakukan alur kerja kolaborasi (*fork*, *clone*, *branching*, *pull request*).

---

### B. Dasar Teori
* **Git** adalah Distributed Version Control System (DVCS) yang mencatat riwayat perubahan berkas secara lokal maupun terdistribusi.
* **GitHub** adalah platform web penyimpan repository Git yang menyediakan fitur manajemen proyek, kolaborasi tim, organisasi, serta penggabungan kode via *Pull Request*.

---

### C. Langkah Kerja & Bukti Praktikum

#### 1. Instalasi Aplikasi Git (`01-install-git.md`)
Proses instalasi Git for Windows dilakukan melalui wizard setup langkah demi langkah:
# 01. Panduan Instalasi Git untuk Windows

Dokumen ini berisi panduan langkah demi langkah proses instalasi Git di sistem operasi Windows berdasarkan wizard instalasi resmi Git.

---

## Langkah-Langkah Instalasi Git

### 1. Lisensi Penggunaan (GNU General Public License)
<img width="781" height="579" alt="image" src="https://github.com/user-attachments/assets/e97bc6d3-737f-4498-8b22-1ed7cb4b99e4" />

* Baca informasi lisensi yang ditampilkan.
* Klik tombol **Next** untuk melanjutkan.

### 2. Memilih Komponen (Select Components)
<img width="769" height="589" alt="image" src="https://github.com/user-attachments/assets/924d77d6-a8d6-4de2-982e-351fc25ba9a9" />

* Pilih komponen yang ingin diinstal (pengaturan default sudah direkomendasikan):
  * `Windows Explorer integration` (Open Git Bash / Open Git GUI)
  * `Git LFS (Large File Support)`
  * `Associate .git* configuration files`
  * `Associate .sh files to be run with Bash`
  * `Scalar (Git add-on to manage large-scale repositories)`
* Klik tombol **Next**.

### 3. Memilih Editor Bawaan (Choosing the default editor used by Git)
<img width="797" height="613" alt="image" src="https://github.com/user-attachments/assets/d4faafb9-5100-4a53-89f9-45ff5f2df25a" />

* Tentukan editor yang akan digunakan oleh Git (misalnya: *Vim*, *VS Code*, dll.).
* Klik tombol **Next**.

### 4. Mengatur Nama Branch Utama (Adjusting the name of the initial branch in new repositories)
<img width="798" height="609" alt="image" src="https://github.com/user-attachments/assets/1de86679-0c18-4d82-9013-cca60fcaa550" />

* Pilih **Override the default branch name for new repositories**.
* Ketikkan nama branch utama yang diinginkan, yaitu `main`.
* Klik tombol **Next**.

### 5. Mengatur Environment PATH (Adjusting your PATH environment)
<img width="761" height="585" alt="image" src="https://github.com/user-attachments/assets/560902f5-7674-410d-a42b-7c893c55e902" />

* Pilih opsi **Git from the command line and also from 3rd-party software** (Direkomendasikan agar Git bisa diakses dari Command Prompt, PowerShell, maupun software pihak ketiga).
* Klik tombol **Next**.

### 6. Memilih Executable SSH (Choosing the SSH executable)
<img width="775" height="593" alt="image" src="https://github.com/user-attachments/assets/1da8ab12-ab16-4637-882c-be95b0b3d638" />

* Pilih opsi **Use external OpenSSH** (Menggunakan SSH eksternal bawaan sistem).
* Klik tombol **Next**.

### 7. Memilih Transport Backend HTTPS (Choosing HTTPS transport backend)
<img width="810" height="621" alt="image" src="https://github.com/user-attachments/assets/ec981172-5ae3-4725-b27c-b0d0a5fda247" />

* Pilih opsi **Use the native Windows Secure Channel library** (Menggunakan sertifikat keamanan dari Windows Certificate Stores).
* Klik tombol **Next**.

### 8. Konversi Line Ending (Configuring the line ending conversions)
<img width="820" height="627" alt="image" src="https://github.com/user-attachments/assets/d53c75b9-6da9-4e1d-b479-d40e14499a96" />

* Pilih opsi **Checkout Windows-style, commit Unix-style line endings** (`core.autocrlf = true`).
* Klik tombol **Next**.

### 9. Memilih Terminal Emulator untuk Git Bash (Configuring the terminal emulator to use with Git Bash)
<img width="806" height="621" alt="image" src="https://github.com/user-attachments/assets/7b219355-6634-4410-80a7-9689a731ab8f" />

* Pilih opsi **Use MinTTY (the default terminal of MSYS2)**.
* Klik tombol **Next**.

### 10. Menentukan Perilaku `git pull` (Choose the default behavior of `git pull`)
<img width="818" height="629" alt="image" src="https://github.com/user-attachments/assets/da8c44d7-6a62-4381-9d6b-6407f9b265d6" />

* Pilih opsi **Merge** (Membuat commit merge saat melakukan pull).
* Klik tombol **Next**.

### 11. Memilih Credential Helper (Choose a credential helper)
<img width="801" height="616" alt="image" src="https://github.com/user-attachments/assets/3433d74a-48f1-44cf-ae96-603f31a3c646" />

* Pilih opsi **Git Credential Manager** (Untuk mempermudah autentikasi akun GitHub/Git provider lainnya).
* Klik tombol **Next**.

### 12. Konfigurasi Opsi Tambahan (Configuring extra options)
<img width="814" height="617" alt="image" src="https://github.com/user-attachments/assets/aa3983fb-a569-4e44-b52a-e888aaccfe26" />

* Centang opsi **Enable file system caching** (Meningkatkan performa pembacaan data).
* Klik tombol **Install**.

### 13. Proses Instalasi (Installing)
<img width="632" height="484" alt="image" src="https://github.com/user-attachments/assets/1191c5f4-395a-4224-85d0-722c24f4c127" />

* Tunggu beberapa saat hingga proses pengekstrakan dan pemasangan berkas selesai.

### 14. Menyelesaikan Instalasi (Completing the Git Setup Wizard)
<img width="797" height="607" alt="image" src="https://github.com/user-attachments/assets/a8b77bd2-fc73-4c0b-99bf-fcce36cb4747" />

* Hilangkan centang pada *Launch Git Bash* jika tidak ingin langsung membukanya.
* Klik tombol **Finish** untuk menutup wizard instalasi.

---

## Verifikasi Hasil Instalasi

Setelah proses instalasi selesai, buka **Command Prompt (CMD)** atau **Terminal**, lalu jalankan perintah berikut untuk memastikan Git sudah terpasang dengan benar:

### 1. Cek Perintah dan Opsi Git
<img width="940" height="634" alt="image" src="https://github.com/user-attachments/assets/c6bc22af-c9bc-463e-9f6b-2d2ff5ca8c94" />


---

# 02. Konfigurasi Awal Git

Melakukan pengaturan identitas global pengguna pada terminal/Git Bash:
1. **Inisialisasi Repository Lokal:**
   * Perintah: `git init`

![git init](https://github.com/user-attachments/assets/de007b55-7a7e-4051-8b3a-db79bbca1336)

2. **Memeriksa Status Pelacakan:**
   * Perintah: `git status`

![git status](https://github.com/user-attachments/assets/93fdd9a5-c95f-4d5c-875f-a84203d8dac6)

3. **Menambahkan File ke Staging Area:**
   * Perintah: `git add .`

![git add](https://github.com/user-attachments/assets/b3f6357b-042a-441d-8b96-6b4c5c033f3e)

---

# 3. Mengelola Repo Sendiri di Account Sendiri
Melakukan pendaftaran akun dan latihan perintah dasar Git pada repository lokal:
1. **Pendaftaran Akun GitHub:**
   Pengisian form pendaftaran akun baru pada situs GitHub dengan username `naisyah-jannah22`. Terjadi kendala aturan penulisan username yang kemudian berhasil diselesaikan hingga valid.
   <img width="958" height="538" alt="image" src="https://github.com/user-attachments/assets/67b27443-cac6-4cd0-9545-c7c23c3edc5d" />

2. **Inisialisasi Repository Lokal:**
   * Perintah: `git init`
   <img width="504" height="241" alt="image" src="https://github.com/user-attachments/assets/de007b55-7a7e-4051-8b3a-db79bbca1336" />

3. **Memeriksa Status Pelacakan:**
   * Perintah: `git status`
  <img width="546" height="259" alt="image" src="https://github.com/user-attachments/assets/93fdd9a5-c95f-4d5c-875f-a84203d8dac6" />

4. **Menambahkan File ke Staging Area:**
   * Perintah: `git add .`
   <img width="346" height="186" alt="image" src="https://github.com/user-attachments/assets/b3f6357b-042a-441d-8b96-6b4c5c033f3e" />

5. **Menyimpan Perubahan (Commit):**
   * Perintah: `git commit -m "Membuat laporan minggu 01"`
  <img width="724" height="206" alt="image" src="https://github.com/user-attachments/assets/8879925b-e661-429b-9a98-c925f055fb2d" />

6. **Melihat Riwayat Commit:**
   * Perintah: `git log --oneline`
   <img width="682" height="241" alt="image" src="https://github.com/user-attachments/assets/119e0737-51b0-4cb4-8bc5-8adda0491d35" />

---

# 4. Mengelola Repository Sendiri - Organisasi
Membuat dan mengelola repository di bawah naungan Organisasi GitHub:

1. **Membuat/Masuk ke Organisasi GitHub:**
   Pengecekan status keanggotaan organisasi pada menu pengaturan akun GitHub.
   <img width="940" height="494" alt="image" src="https://github.com/user-attachments/assets/4ed2c127-c790-4a87-84ff-c58a13774bff" />

2. **Pengisian Detail Organisasi Baru:**
   Pengisian form nama organisasi (`prak-disdec-utdi`), email kontak, serta kepemilikan organisasi.
   <img width="940" height="511" alt="image" src="https://github.com/user-attachments/assets/d973ece8-ec9e-4978-923b-0e1016ecc1a9" />

3. **Penambahan Anggota Organisasi:**
   Halaman konfirmasi awal untuk menambahkan anggota tim ke dalam organisasi.
   <img width="940" height="471" alt="image" src="https://github.com/user-attachments/assets/2b014880-6761-4c35-9b46-b466a31f7541" />

4. **Dashboard Utama Organisasi:**
   Tampilan utama *Overview* dari organisasi `prak-disdec-utdi` setelah pengaturan selesai dibuat.
<img width="940" height="491" alt="image" src="https://github.com/user-attachments/assets/700edbdc-394d-444e-84d3-37c9bcdedaa8" />

---

# 5. Kolaborasi & Workflow GitHub (`04-kolaborasi.md`)
Melakukan simulasi dan praktik alur kerja kolaborasi proyek open-source/tim di GitHub:
1. **Fork Repository:**
   Membuat duplikasi (*fork*) dari repository pengguna lain/utama ke akun pribadi.
 <img width="577" height="303" alt="image" src="https://github.com/user-attachments/assets/c8c2a158-3e4c-4366-956e-6ea6c1c7abec" />

2. **Clone Repository Lokal:**
   Mendownload repository hasil fork ke komputer lokal.
   * Perintah: `git clone <URL_Repository>`
   <img width="571" height="234" alt="image" src="https://github.com/user-attachments/assets/c43e90bf-52d4-471c-a8cd-83421acc2efc" />

3. **Membuat Branch Baru:**
   Membuat dan berpindah ke cabang fitur (*feature branch*) baru untuk pengerjaan tugas.
   * Perintah: `git checkout -b fitur-baru`
  <img width="575" height="140" alt="image" src="https://github.com/user-attachments/assets/c06215d7-840d-4511-8524-f0867c561b79" />

4. **Push Changes ke Remote Branch:**
   Mengunggah perubahan dari branch lokal ke GitHub.
   * Perintah: `git push origin fitur-baru`
  <img width="579" height="239" alt="image" src="https://github.com/user-attachments/assets/ce8fa92b-320f-444d-9dc9-f59ac0526320" />

5. **Membuat & Menggabungkan Pull Request (PR):**
   Mengajukan *Pull Request* pada halaman GitHub dan melakukan proses *Merge* perubahan ke branch utama.
  <img width="959" height="500" alt="image" src="https://github.com/user-attachments/assets/82aaa849-933a-423e-bf44-31d35bba6d0b" />


---

### D. Hasil Praktikum & Link Pengumpulan
* **URL Repository Utama:** `https://github.com/naisyah-jannah22/prak-dis-dec`
* **URL Laporan Minggu 1:** `https://github.com/naisyah-jannah22/prak-dis-dec/tree/main/01`

---

### E. Kesimpulan
Praktikum minggu ke-1 telah berhasil menyelesaikan seluruh alur kerja Git dan GitHub, mulai dari instalasi bertahap, konfigurasi identitas pengguna, pengelolaan repository lokal pada akun pribadi maupun organisasi, hingga alur kerja kolaborasi terdistribusi (*fork*, *clone*, *branching*, dan *pull request*).
