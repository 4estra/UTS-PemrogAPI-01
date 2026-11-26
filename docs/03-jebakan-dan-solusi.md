---
title: Jebakan dan Solusi
nav_order: 3
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

# 03. Jebakan dan Solusi

Jebakan dibagi per area. Tingkat: 🔴 bisa menggagalkan skenario/nilai, 🟡 mengurangi kualitas, 🟢 saran.

## A. Jebakan dari kode contoh (`02.crud-api-mahasiswa`)

| # | Tingkat | Masalah di kode contoh | Solusi untuk UTS |
| --- | --- | --- | --- |
| A1 | 🔴 | `findById`, `replace`, `update` **tidak** memakai `deleted_at IS NULL` | Tambahkan `AND deleted_at IS NULL` di semua query produk (kecuali `restore`) |
| A2 | 🔴 | Error 23505 ditangani **global** di `app.js` dengan pesan "NIM atau email..." | Tangani di controller tiap resource. Handler global hanya untuk JSON rusak dan 500 |
| A3 | 🔴 | Validasi teks `Boolean(v)` meloloskan `"   "` (spasi saja) | `typeof v === "string" && v.trim() !== ""` |
| A4 | 🔴 | `parseId` memakai `Number()`: `1e2`, `1.0`, `0x10`, ` 5 ` lolos | Pakai regex `^\d+$` |
| A5 | 🔴 | `parseId` tanpa batas atas: id sangat besar → PG error 22003 → **500** | Tolak id > 2147483647 dengan 400 |
| A6 | 🟡 | `getAll` tanpa `message`; PUT/PATCH tanpa `message` | Semua respons sukses memakai `{status, message, data}` (+`count` untuk daftar) |
| A7 | 🟡 | Controller menghitung ulang `Number(req.params.id)` | Pakai `req.id` dari `parseId` |
| A8 | 🔴 | Skema contoh `TIMESTAMPTZ` | Ikuti soal: `TIMESTAMP` |
| A9 | 🔴 | Validasi di folder 01 membalas **422** untuk field kosong | UTS: validasi → **400**. 422 khusus `kategori_id` tidak ada |
| A10 | 🔴 | `.env` (berisi password) ikut di zip, tidak ada `.gitignore` | Buat `.gitignore` (`node_modules`, `.env`) dan `.env.example` tanpa password |
| A11 | 🔴 | Whitelist kolom di `update()` | Jangan masukkan `id`, `created_at`, `deleted_at` ke daftar yang boleh diubah |
| A12 | 🟢 | `testConnection.js` tidak ada di struktur soal | Tidak perlu disalin ke repo UTS |

## B. Jebakan dari soal

| # | Tingkat | Jebakan | Solusi |
| --- | --- | --- | --- |
| B1 | 🔴 | **23503 punya dua arti.** Hapus kategori yang dipakai → 409. INSERT/UPDATE produk dengan `kategori_id` tidak ada → 422 | Tangani di controller sesuai operasi |
| B2 | 🔴 | FK dengan `ON DELETE CASCADE` | **Jangan.** Pakai FK biasa agar hapus kategori ditolak |
| B3 | 🔴 | Partial unique index untuk `kode_produk` | **Jangan.** Soal minta `UNIQUE` biasa dan 409 tetap berlaku untuk baris soft delete |
| B4 | 🔴 | `GET /api/kategori/:id/produk`: kategori tidak ada vs kategori ada tapi kosong | Cek kategori dulu: tidak ada → 404, ada tanpa produk → 200 dengan `data: []` |
| B5 | 🔴 | Soft delete kedua pada id yang sama harus 404 | `UPDATE ... WHERE id=$1 AND deleted_at IS NULL`, lalu cek `rowCount` |
| B6 | 🔴 | Restore pada produk aktif atau tidak ada harus 404 | `UPDATE ... SET deleted_at = NULL WHERE id=$1 AND deleted_at IS NOT NULL RETURNING *` |
| B7 | 🔴 | PUT kurang field → 400. PATCH `{}` → 400 | `validateProduk` (semua wajib) dan `validatePatch` (minimal satu field) |
| B8 | 🔴 | Dua kesalahan sekaligus (N03) harus menghasilkan `errors` berisi dua pesan | Kumpulkan semua error sebelum membalas, jangan berhenti di yang pertama |
| B9 | 🟡 | Panjang kolom (`nama_kategori` ≤ 50, `kode_produk` ≤ 20, dll.) tidak ada di aturan validasi | Tambah validasi panjang → 400, kalau tidak PG error 22001 → 500 |
| B10 | 🟡 | `deskripsi` kategori boleh kosong | Boleh tidak dikirim atau `null`; kalau ada, string maks 200 |
| B11 | 🟡 | Angka melebihi integer PG (`harga`, `stok`, `kategori_id`) → 22003 → 500 | Batasi maksimum 2147483647 di validasi |
| B12 | 🟡 | Pesan PostgreSQL tidak boleh bocor | Jangan pernah mengirim `err.message` ke klien |

## C. Jebakan Postman

| # | Tingkat | Jebakan | Solusi |
| --- | --- | --- | --- |
| C1 | 🔴 | `"kategori_id": "{{kategoriId}}"` (pakai kutip) → string → 400 | Tulis tanpa kutip: `"kategori_id": {{kategoriId}}` |
| C2 | 🔴 | `pm.environment.get()` selalu string, jadi `id === produkId` gagal | `Number(pm.environment.get("produkId"))` |
| C3 | 🔴 | `{{$timestamp}}` dalam **detik**. S01 dan S13 sama-sama membuat kategori dalam detik yang sama → 409 | Beda prefiks: `Kat-{{$timestamp}}` dan `Sementara-{{$timestamp}}` |
| C4 | 🔴 | `kode_produk` maks 20 karakter. `PRODUK-UTS-{{$timestamp}}` = 21 | `P{{$timestamp}}` (11 karakter) |
| C5 | 🔴 | Respons 204 tanpa body: `pm.response.json()` melempar error | Cek `pm.response.text()` kosong |
| C6 | 🟡 | N12 dan N13 ("dua kali berturut-turut") | Buat dua request terpisah (a dan b), periksa hasil request **kedua** |
| C7 | 🟡 | N02, N05, N08 butuh nilai milik data lain | Pakai nilai tetap dari seed (`seed.sql` tidak boleh berubah) |
| C8 | 🟡 | S06 "hanya stok yang berubah" | Bandingkan juga field lain dengan nilai dari S05 |
| C9 | 🔴 | Lupa memilih Environment di Runner | Pilih Environment (pojok kanan atas) sebelum Run |
| C10 | 🟡 | Export environment memakai *initial value* | Isi *initial value* `baseUrl` = `http://localhost:3000` |
| C11 | 🟡 | Folder 3 butuh screenshot di momen tertentu, tidak bisa di tengah Runner otomatis | Jalankan S01-S07 manual (Send), screenshot DB, lanjut S08-S10, screenshot, dst. |

## D. Database

| # | Tingkat | Jebakan | Solusi |
| --- | --- | --- | --- |
| D1 | 🔴 | Seed mengisi `id` manual (1, 2, 3...) → sequence SERIAL tidak maju → POST pertama bentrok `produk_pkey` (23505) → **409 palsu** | Jangan isi `id`. Biarkan SERIAL, dan ambil `kategori_id` lewat subquery |
| D2 | 🟡 | Seed dijalankan dua kali → UNIQUE bentrok | Awali seed dengan `TRUNCATE produk, kategori RESTART IDENTITY CASCADE;` |
| D3 | 🟡 | `schema.sql` tidak bisa dijalankan ulang | Awali dengan `DROP TABLE IF EXISTS produk; DROP TABLE IF EXISTS kategori;` (anak dulu) |
| D4 | 🟡 | `CREATE DATABASE` tidak bisa jalan di dalam transaksi / satu eksekusi di pgAdmin | Jalankan terpisah, atau pakai `psql -f` dengan `\c db_uts` |
| D5 | 🟢 | Sisakan satu kategori tanpa produk di seed | Berguna untuk demo 200-kosong dan hard delete |

Contoh pola seed yang aman:

```sql
INSERT INTO produk (kategori_id, kode_produk, nama_produk, harga, stok)
VALUES (
  (SELECT id FROM kategori WHERE nama_kategori = 'Elektronik'),
  'ELK-001', 'Mouse Wireless', 85000, 20
);
```

## E. Query inti (acuan untuk repository)

```sql
-- daftar produk aktif
SELECT * FROM produk WHERE deleted_at IS NULL ORDER BY id;

-- detail produk aktif
SELECT * FROM produk WHERE id = $1 AND deleted_at IS NULL;

-- produk aktif milik kategori
SELECT * FROM produk WHERE kategori_id = $1 AND deleted_at IS NULL ORDER BY id;

-- PUT (ganti semua)
UPDATE produk
SET kategori_id=$1, kode_produk=$2, nama_produk=$3, harga=$4, stok=$5
WHERE id=$6 AND deleted_at IS NULL RETURNING *;

-- soft delete
UPDATE produk SET deleted_at = NOW() WHERE id = $1 AND deleted_at IS NULL;

-- restore
UPDATE produk SET deleted_at = NULL WHERE id = $1 AND deleted_at IS NOT NULL RETURNING *;

-- hard delete kategori
DELETE FROM kategori WHERE id = $1;

-- (nilai tambah) hard delete produk, hanya yang sudah di-soft delete
DELETE FROM produk WHERE id = $1 AND deleted_at IS NOT NULL;
```

Untuk permanent delete, bedakan "id tidak ada" (404) dengan "ada tapi masih aktif" (409). Kalau `rowCount` 0, cek dulu apakah barisnya ada.

## F. Peta penanganan error

```text
23505 (UNIQUE)               → controller → 409
23503 saat hapus kategori    → controller kategori → 409
23503 saat INSERT/UPDATE     → controller produk   → 422
entity.parse.failed          → error handler app.js → 400
lainnya                      → console.error + 500 "Terjadi kesalahan pada server"
```
