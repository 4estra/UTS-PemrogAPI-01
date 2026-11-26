---
title: Pemahaman Soal
nav_order: 1
---

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

# 01. Pemahaman Soal

## Tujuan soal

Membangun **REST API toko online** (Express.js + PostgreSQL) untuk dua resource one-to-many, lalu **membuktikan kebenarannya dengan Postman**. Yang dinilai bukan hanya API jalan, tetapi juga ketepatan status code, arsitektur berlapis, dan keputusan desain yang dijelaskan di README.

## Empat hal inti

1. **Dua tabel berelasi:** `kategori` (induk) dan `produk` (anak).
2. **Dua jenis delete:** `kategori` hard delete, `produk` soft delete (`deleted_at`).
3. **Arsitektur berlapis:** route → middleware → controller → repository. SQL hanya di repository, semua query berparameter.
4. **Penanganan error tepat:** 400, 404, 409, 422, 500 sesuai kondisi, tanpa membocorkan pesan PostgreSQL.

## Rujukan dari kelas

Contoh acuan ada di `kelas-pemrog_api-main.zip`, folder `02.crud-api-mahasiswa`. Folder itu sudah berisi pola yang diminta: `config/db.js`, class repository yang diekspor satu instance, `parseId`, validasi dua fungsi (`validate...` dan `validatePatch`), `app.js` dengan handler 404 dan error handler terpusat, serta method `softDelete` dan `restore` yang **belum dihubungkan** ke route. Bagian yang belum dihubungkan itu yang harus diselesaikan di UTS.

## Rincian tugas

### A. Database
- Database `db_uts`, perintah di `sql/schema.sql`.
- Data awal di `sql/seed.sql`: minimal 3 baris `kategori` dan 5 baris `produk`.

### A1. Arsitektur dan koneksi
- `pg Pool` dibuat sekali di `config/db.js`, konfigurasi dari `.env`.
- Repository berupa class, diekspor satu instance.
- `app.js` menyusun aplikasi (`express.json()`, router, handler 404, error handler terpusat). `server.js` hanya `listen`.
- Middleware `parseId` memvalidasi id di URL. Middleware validasi memakai dua fungsi (`validate...` untuk POST/PUT, `validatePatch` untuk PATCH).
- JSON rusak → 400. Error tak terduga → 500 dengan pesan umum.

### A2. Endpoint induk (`kategori`)
GET daftar, GET by id, POST, DELETE (hard delete, 23503 → 409).

### A3. Endpoint anak (`produk`)
GET daftar (+count), GET by id, GET nested per kategori, POST, PUT, PATCH, DELETE (soft delete), PATCH restore.

### A4. Soft delete dan restore
- Soft delete = `UPDATE ... SET deleted_at = NOW()`, baris tetap ada.
- Filter `deleted_at IS NULL` wajib di GET daftar, GET by id, GET nested, PUT, PATCH, dan soft delete.
- Soft delete kedua pada id yang sama → 404.
- Restore hanya untuk baris yang sedang terhapus, selain itu 404.
- Hard delete kategori tetap 409 selama ada produk (termasuk yang di-soft delete). **Jelaskan di README.**
- `kode_produk` milik baris soft delete tetap memicu 409 bila dipakai lagi. **Jelaskan keputusan desain di README.**

### B. Pengujian Postman
- Collection `UTS Pemrograman API` + satu Environment (`baseUrl`, `kategoriId`, `produkId`, `kategoriSementaraId`).
- Folder: **1-Alur Utama** (S01-S15), **2-Skenario Error** (N01-N14), **3-Bukti Delete**.
- Folder 1 harus bisa dijalankan **berulang kali** di database yang sudah berisi seed.
- Bukti database (3 screenshot) dan hasil Runner (Folder 1 dan 2).

## Poin plus (maksimal 5 poin, total nilai tidak lebih dari 100)

- `DELETE /api/produk/:id/permanent`: hard delete pada produk, **hanya** untuk baris yang sudah di-soft delete (produk aktif dibalas 409).
- Filter `GET /api/produk?kategori_id=1`.

## Yang dikumpulkan

Satu repository GitHub, cabang `main`, tanpa `.env` dan `node_modules`: kode lengkap + `package.json` + `.env.example` + `.gitignore`, `sql/schema.sql`, `sql/seed.sql`, `uts.postman_collection.json`, `README.md`, dan tangkapan layar. Judul pengumpulan: `UTS-PemrogAPI-[namakelompok]`.

## Catatan dari typo di PDF soal

- "sg1/schema. sql" maksudnya `sql/schema.sql`.
- "lihat B.5" pada soft delete maksudnya bagian A4.
- URL "/api/kategori/:id/produ k" maksudnya `/api/kategori/:id/produk`.
