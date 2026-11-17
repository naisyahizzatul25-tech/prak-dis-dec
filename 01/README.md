# Laporan Praktikum Sistem Terdistribusi dan Terdesentralisasi
## Minggu 01: Pengenalan Sistem Terdistribusi dan Terdesentralisasi - Git dan GitHub

### Identitas Mahasiswa
* **Nama:** Naisyah Izzatul Jannah K.
* **NIM:** 255410040
* **Kelas:** Informatika

---

AI yang digunakan: Claude (Anthropic), model Claude Sonnet 5.5

Prompt: "Buatkan rangkuman materi dari file PDF dalam format markdown untuk README.md." 

Bagian yang disesuaikan dari keluaran prompt ada pada: *(isi sendiri, misalnya: bagian Perbedaan Terdistribusi vs Terdesentralisasi saya tambah contoh; bagian CAP Theorem saya periksa ulang ke sumber lain)*

---

# Pengenalan Sistem Terdistribusi dan Terdesentralisasi

Mata kuliah: Sistem Terdistribusi dan Terdesentralisasi (FIF25002T, 3 SKS), Prodi Informatika, Fakultas Teknologi Informasi UTDI. Dosen: Dr. Bambang Purnomosidi D. P.

## Tujuan Pembelajaran

- Menjelaskan konsep sistem terdistribusi.
- Menjelaskan konsep sistem terdesentralisasi.
- Menjelaskan perbedaan keduanya.
- Menyebutkan contoh penerapan nyata.
- Menyebutkan tantangan dan tren terbaru.

## 1. Arti Penting Sistem Terdistribusi dan Terdesentralisasi

Layanan modern menghadapi data dan pengguna dalam jumlah sangat besar, sehingga membutuhkan:

- **Skalabilitas**: kemampuan sistem berkembang dan menangani peningkatan beban kerja, pengguna, atau permintaan tanpa mengorbankan kinerja atau kualitas.
- **Fault tolerance**: sistem tetap beroperasi normal walaupun sebagian komponen (hardware maupun software) gagal.
- **Performa dan ketersediaan 24/7**.

Tanpa sistem terdistribusi dan terdesentralisasi, layanan seperti Google, Netflix, dan blockchain tidak mungkin ada.

## 2. Pengertian Sistem Terdistribusi

Menurut Tanenbaum dan van Steen (2007), sistem terdistribusi adalah *a collection of independent computers that appears to its users as a single coherent system*.

Ciri utamanya:

- Komponen bersifat independen.
- Berkomunikasi melalui jaringan.
- Menyediakan **transparansi**: lokasi, akses, replikasi, dan kegagalan (pengguna tidak perlu tahu di mana data berada, bagaimana diakses, berapa salinannya, atau ada komponen yang gagal).

Contoh arsitektur: aplikasi mobile dan browser mengakses API Gateway / Storefront WebApp, yang meneruskan permintaan ke layanan terpisah (Account, Inventory, Shipping), masing-masing dengan database sendiri.

## 3. Arsitektur Sistem Dasar

| Arsitektur | Penjelasan | Contoh |
|---|---|---|
| **Client-Server** | Banyak client terhubung lewat internet ke server pusat yang melayani permintaan. | Web, aplikasi mobile |
| **Peer-to-Peer (P2P)** | Setiap node setara, saling terhubung langsung tanpa server pusat. | BitTorrent, Bitcoin, Ethereum, dan kripto lain |
| **Hybrid** | Gabungan keduanya, menggunakan *supernode* sebagai penghubung antar kelompok peer. | WhatsApp, Telegram, Signal (sesuai ilustrasi slide) |

## 4. Karakteristik Sistem Terdistribusi

- Transparansi lokasi
- Skalabilitas
- Fault tolerance
- Kinerja tinggi
- Heterogenitas (komponen dapat berbeda hardware, OS, dan teknologi)

## 5. Pengertian Sistem Terdesentralisasi

- **Sentralisasi**: satu titik kontrol.
- **Desentralisasi**: kontrol didistribusikan, sehingga tidak ada *single point of failure*.
- Keputusan diambil secara kolektif melalui **konsensus**.
- Contoh: Blockchain, DNS, BitTorrent.

Paul Baran (1964) menggambarkan tiga topologi jaringan:

1. **Centralized**: semua node terhubung ke satu pusat.
2. **Decentralized**: ada beberapa pusat (hub) yang saling terhubung.
3. **Distributed**: node saling terhubung dalam bentuk jaring tanpa pusat dominan.

## 6. Sistem Terdistribusi vs Sistem Terdesentralisasi

| Aspek | Terdistribusi | Terdesentralisasi |
|---|---|---|
| Fokus | Pembagian **beban kerja** | Pembagian **kontrol** |
| Contoh | Google File System, MinIO Object Storage | Blockchain |

Keduanya saling terkait tetapi berbeda fokus.

## 7. Studi Kasus Sistem Terdistribusi

- Google File System / HDFS: penyimpanan data berskala besar.
- Apache Kafka dan Apache Pulsar: messaging dan streaming.
- Netflix, Twitch, YouTube: layanan streaming video.
- Instagram, Threads, Facebook, TikTok: media sosial berskala global.

## 8. Studi Kasus Sistem Terdesentralisasi

- Blockchain (Bitcoin, Ethereum, Algorand, Sui, Near, Aptos, dll.)
- DNS (Domain Name System)
- BitTorrent

## 9. CAP Theorem (Eric Brewer, 2000)

Sering disebut Brewer's Theorem. Tiga properti:

- **Consistency (C)**: setiap operasi baca mengembalikan penulisan terbaru atau error.
- **Availability (A)**: setiap permintaan ke node yang tidak gagal mendapat respons non-error, meskipun bukan data terbaru.
- **Partition Tolerance (P)**: sistem tetap berfungsi walaupun komunikasi antar node terputus atau terfragmentasi.

Pada distributed data store, partisi jaringan pasti bisa terjadi, sehingga Consistency dan Availability tidak bisa dijamin bersamaan saat partisi. Perancang sistem harus memilih:

- **AP (Availability + Partition Tolerance)**: permintaan selalu dilayani, dengan risiko data tidak sinkron (mengorbankan konsistensi). Contoh pada slide: MongoDB.
- **CP (Consistency + Partition Tolerance)**: permintaan read maupun write ditolak demi menjaga konsistensi (mengorbankan availability). Contoh pada slide: Apache Cassandra.

> Catatan: klasifikasi AP/CP pada produk nyata bergantung pada konfigurasi. Contoh di atas mengikuti slide kuliah dan sebaiknya diverifikasi ke dokumentasi resmi masing-masing produk.

## 10. Konsistensi Data

- **Strong consistency**: hasil baca selalu sama (data terbaru). Contoh: bank.
- **Eventual consistency**: data akhirnya menjadi konsisten setelah beberapa waktu. Contoh: media sosial.

## 11. Tantangan

- **Latency** (berapa lama proses) dan **throughput** (berapa banyak yang diproses).
- **Keamanan**: DDoS dan Sybil attack.
- Kompleksitas manajemen node.
- **Governance** (tata kelola) pada sistem terdesentralisasi.

**Sybil attack**: serangan siber dengan membuat banyak identitas palsu (akun, node, atau pseudonim, bisa sampai ribuan) sekaligus untuk mendominasi pengaruh, suara, atau reputasi di jaringan. Umumnya menyerang sistem terdesentralisasi seperti P2P dan blockchain. Nama "Sybil" berasal dari buku tahun 1973 tentang seorang perempuan dengan *identity disorder*.

## 12. Tren Masa Depan

- Cloud-native (Kubernetes, Docker)
- Edge dan Fog Computing
- Web3 dan DeFi
- Federated Learning (AI terdistribusi)

## 13. Ringkasan

- Sistem terdistribusi = berbagi **beban kerja**.
- Sistem terdesentralisasi = berbagi **kontrol**.
- Keduanya saling terkait tetapi berbeda fokus.
- Keduanya menjadi fondasi teknologi modern.

## 14. Diskusi

- Mengapa dunia semakin bergerak menuju distribusi dan desentralisasi?
- Apa contoh sehari-hari yang paling terasa bagi Anda?

## Referensi

- Slide kuliah: Pengenalan Sistem Terdistribusi dan Terdesentralisasi, Dr. Bambang Purnomosidi D. P., UTDI.
- Tanenbaum, A. S. dan van Steen, M. (2007). *Distributed Systems: Principles and Paradigms*.
- Baran, P. (1964). *On Distributed Communications*, Memorandum RM-3420-PR.
