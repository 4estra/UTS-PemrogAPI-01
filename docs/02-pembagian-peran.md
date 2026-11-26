<div align="center">

[🏠 README](../README.md) •
[01 Soal](01-pemahaman-soal.md) •
[02 Peran](02-pembagian-peran.md) •
[03 Jebakan](03-jebakan-dan-solusi.md) •
[04 Spesifikasi](04-spesifikasi-api.md) •
[05 Postman](05-panduan-postman.md) •
[06 QA](06-checklist-qa-celah.md) •
[07 Pengumpulan](07-checklist-pengumpulan.md)

</div>

---

# 02. Pembagian Peran

Dibagi 5 tahap berurutan, dari fondasi sampai uji celah. Tahap 4 dan 5 baru bermakna setelah tahap 2 dan 3 selesai.

```text
[1 Database & Setup] → [2 Kategori] + [3 Produk] → [4 Postman & Error Handling] → [5 QA & Dokumentasi]
```

## Tahap 1: Database dan Setup

**Pemilik:** [nama]

| Tugas | Hasil |
| --- | --- |
| Instal PostgreSQL, buat `db_uts` | Database siap |
| Tulis `sql/schema.sql` persis sesuai tabel soal | `schema.sql` |
| Tulis `sql/seed.sql` (min. 3 kategori, 5 produk) | `seed.sql` |
| Setup project: `package.json`, `.env.example`, `.gitignore` | File dasar |
| `src/config/db.js` (pg Pool) | Koneksi |
| `src/app.js` dan `src/server.js` | Kerangka aplikasi |

Perhatikan: [jebakan database](03-jebakan-dan-solusi.md#d-database).

## Tahap 2: Kategori (induk)

**Pemilik:** [nama]

| Tugas | File |
| --- | --- |
| Middleware id | `middlewares/parseId.js` |
| Validasi kategori | `middlewares/validateKategori.js` |
| Repository (findAll, findById, create, remove) | `repositories/kategoriRepository.js` |
| Controller, termasuk 23505 → 409 dan 23503 → 409 | `controllers/kategoriController.js` |
| Route | `routes/kategoriRoutes.js` |

## Tahap 3: Produk (anak)

**Pemilik:** [nama]

| Tugas | File |
| --- | --- |
| Validasi produk (dua fungsi) | `middlewares/validateProduk.js` |
| Repository dengan filter `deleted_at IS NULL` di setiap query | `repositories/produkRepository.js` |
| Controller: GET (daftar, by id, nested), POST, PUT, PATCH, soft delete, restore | `controllers/produkController.js` |
| Route | `routes/produkRoutes.js` |
| *(nilai tambah)* permanent delete dan filter `?kategori_id` | repository + controller + route |

Titik rawan: 23503 → **422** pada POST/PUT/PATCH, nested route (404 vs 200 kosong), filter soft delete.

## Tahap 4: Postman dan Error Handling

**Pemilik:** [nama]

| Tugas | Hasil |
| --- | --- |
| Handler 404 "Route tidak ditemukan" dan error handler terpusat (JSON rusak → 400, lainnya → 500) | `app.js` |
| Collection, Environment, Folder 1-Alur Utama (S01-S15) lengkap dengan `pm.test` | `uts.postman_collection.json` |
| Memastikan Folder 1 hijau saat dijalankan berulang | Runner hijau |

Panduan: [05-panduan-postman](05-panduan-postman.md).

## Tahap 5: QA, Cari Celah, dan Dokumentasi

**Pemilik:** [nama]

| Tugas | Hasil |
| --- | --- |
| Folder 2-Skenario Error (N01-N14) | di collection |
| Uji celah di luar skenario soal | [06-checklist-qa-celah](06-checklist-qa-celah.md) |
| Screenshot Runner dan bukti database (Folder 3) | `docs/screenshots/` |
| Menulis dan merapikan `README.md` | README final |
| Pemeriksaan akhir sebelum submit | [07-checklist-pengumpulan](07-checklist-pengumpulan.md) |

## Aturan kerja bersama

- Satu branch per orang, gabungkan ke `main` lewat pull request atau merge setelah dicek.
- Jangan commit `.env` atau `node_modules`.
- Bentuk respons harus seragam: `{ status, message, data }`. Sepakati sebelum mulai koding.
- Setiap anggota menjalankan Folder 1 di Postman sebelum merge ke `main`.
