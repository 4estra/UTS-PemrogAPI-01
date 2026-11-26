<div align="center">

[🏠 README](https://github.com/FirmanSyah2078/UTS-PemrogAPI-01#readme) •
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

Lima peran, semua **kerja paralel** sejak awal — asalkan kontrak (nama tabel/kolom dari Database, bentuk respons, nama file) sudah dibekukan duluan. Urutan di bawah ini cuma urutan **push ke `main`**, bukan urutan mulai kerja.

```text
A Database → B Pondasi → C Kategori + D Produk (barengan) → E Middleware, Postman & Dokumentasi
```

## A — Database

**Pemilik:** Abbad

| Tugas | Hasil |
| --- | --- |
| Instal PostgreSQL, buat `db_uts` | Database siap |
| Tulis `sql/schema.sql` persis sesuai tabel soal | `schema.sql` |
| Tulis `sql/seed.sql` (min. 3 kategori, 5 produk) | `seed.sql` |

Paling kritis: anggota lain butuh nama tabel dan kolom dari sini sebelum bisa menulis repository. Perhatikan: [jebakan database](03-jebakan-dan-solusi.md#d-database).

## B — Pondasi

**Pemilik:** Firman

| Tugas | Hasil |
| --- | --- |
| Setup project: `package.json`, `.env.example`, `.gitignore` | File dasar |
| `src/config/db.js`, satu `Pool` dari `pg`, baca dari `.env` | Koneksi |
| `src/app.js`: `express.json()`, router, handler 404 "Route tidak ditemukan", error handler terpusat (JSON rusak → 400, lainnya → 500) | Kerangka aplikasi |
| `src/server.js`: cuma `listen` | Entry point |

## C — Kategori (induk)

**Pemilik:** Arganata

| Tugas | File |
| --- | --- |
| Repository (findAll, findById, create, remove) | `repositories/kategoriRepository.js` |
| Controller, termasuk 23505 → 409 dan 23503 → 409 | `controllers/kategoriController.js` |
| Route | `routes/kategoriRoutes.js` |

## D — Produk (anak)

**Pemilik:** Eko

| Tugas | File |
| --- | --- |
| Repository dengan filter `deleted_at IS NULL` di setiap query | `repositories/produkRepository.js` |
| Controller: GET (daftar, by id, nested), POST, PUT, PATCH, soft delete, restore | `controllers/produkController.js` |
| Route | `routes/produkRoutes.js` |
| *(nilai tambah)* permanent delete dan filter `?kategori_id` | repository + controller + route |

Titik rawan: 23503 → **422** pada POST/PUT/PATCH, nested route (404 vs 200 kosong), filter soft delete.

C dan D bisa jalan **barengan** karena file beda, asal nama tabel/kolom dari A sudah dikonfirmasi.

## E — Middleware, Postman, QA, dan Dokumentasi

**Pemilik:** Yusron

| Tugas | File / Hasil |
| --- | --- |
| `parseId`: id di URL harus bilangan bulat positif, kalau tidak 400 | `middlewares/parseId.js` |
| Validasi kategori (POST/PUT field lengkap) | `middlewares/validateKategori.js` |
| Validasi produk, dua fungsi (`validate...` untuk POST/PUT lengkap, `validatePatch` untuk field yang dikirim saja), kumpulkan semua error ke array `errors` | `middlewares/validateProduk.js` |
| Collection + Environment, Folder 1-Alur Utama (S01-S15) lengkap dengan `pm.test` | `uts.postman_collection.json` |
| Folder 2-Skenario Error (N01-N14) dan Folder 3-Bukti Delete | di collection |
| Uji celah di luar skenario soal | [06-checklist-qa-celah](06-checklist-qa-celah.md) |
| Screenshot Runner dan bukti database | `docs/screenshots/` |
| Menulis dan merapikan `README.md` | README final |
| Pemeriksaan akhir sebelum submit | [07-checklist-pengumpulan](07-checklist-pengumpulan.md) |

Panduan: [05-panduan-postman](05-panduan-postman.md).

## Urutan push ke `main`

Kerja paralel, tapi merge harus urut biar tidak konflik:

1. **A** push dulu — folder `sql/`
2. **B** push — `config/`, `app.js`, `server.js`
3. **C** dan **D** push, bisa barengan karena file beda
4. **E** push terakhir — `middlewares/`, `uts.postman_collection.json`, `README.md`

Kalau konflik: pull dulu, selesaikan di lokal, baru push.

## Aturan kerja bersama

- Satu branch per orang, gabungkan ke `main` lewat pull request atau merge setelah dicek.
- Jangan commit `.env` atau `node_modules`.
- Bentuk respons harus seragam: `{ status, message, data }` (gagal: `{ status, message, errors }`, `errors` hanya untuk kegagalan validasi — lihat [04-spesifikasi-api](04-spesifikasi-api.md)). Sepakati sebelum mulai koding.
- Setiap anggota menjalankan Folder 1 di Postman sebelum merge ke `main`.
