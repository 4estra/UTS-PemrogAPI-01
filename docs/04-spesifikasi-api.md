---
title: Spesifikasi API
nav_order: 4
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

# 04. Spesifikasi API

## Format respons

| Situasi | Bentuk |
| --- | --- |
| Sukses | `{ "status": "success", "message": "...", "data": ... }` |
| Sukses daftar | seperti di atas ditambah `"count": <jumlah>` |
| Soft delete / hard delete sukses | `204` tanpa body |
| Gagal | `{ "status": "error", "message": "..." }` |
| Gagal validasi | `{ "status": "error", "message": "Validasi gagal", "errors": ["..."] }` |
| URL tidak dikenal | `404` `{ "status": "error", "message": "Route tidak ditemukan" }` |
| Error server | `500` `{ "status": "error", "message": "Terjadi kesalahan pada server" }` |

Field `errors` **hanya** untuk kegagalan validasi.

## Aturan validasi

| Resource | Field | Aturan |
| --- | --- | --- |
| kategori | `nama_kategori` | wajib, string tidak kosong (maks 50 karakter) |
| kategori | `deskripsi` | opsional, boleh `null`, string maks 200 karakter |
| produk | `kategori_id` | bilangan bulat ≥ 1 |
| produk | `kode_produk` | wajib, string tidak kosong (maks 20 karakter) |
| produk | `nama_produk` | wajib, string tidak kosong (maks 100 karakter) |
| produk | `harga` | bilangan bulat > 0 |
| produk | `stok` | bilangan bulat ≥ 0 |

Aturan tambahan (di luar tabel soal, untuk mencegah error 500):

- String dianggap kosong jika hanya berisi spasi.
- Angka dibatasi maksimum 2.147.483.647 (batas integer PostgreSQL).
- POST dan PUT: **semua** field produk wajib. PATCH: hanya field yang dikirim diperiksa, minimal satu field.
- Field di luar daftar di atas diabaikan.

## Validasi id (`parseId`)

- Harus bilangan bulat positif (hanya digit), dan tidak melebihi 2147483647.
- Tidak valid → `400` `ID harus bilangan bulat positif`.

## Kategori

| Method | URL | Sukses | Gagal |
| --- | --- | --- | --- |
| GET | `/api/kategori` | 200 | - |
| GET | `/api/kategori/:id` | 200 | 400 id tidak valid, 404 tidak ada |
| POST | `/api/kategori` | 201 | 400 validasi, 409 `nama_kategori` sudah ada |
| DELETE | `/api/kategori/:id` | 204 (tanpa body) | 400 id, 404 tidak ada, 409 masih dipakai produk |

## Produk

| Method | URL | Sukses | Gagal |
| --- | --- | --- | --- |
| GET | `/api/produk` | 200 + `count` | - |
| GET | `/api/produk/:id` | 200 | 400, 404 (tidak ada **atau** sudah di-soft delete) |
| GET | `/api/kategori/:id/produk` | 200 (boleh `data: []`) | 400, 404 kategori tidak ada |
| POST | `/api/produk` | 201 | 400 validasi, 409 `kode_produk` kembar, 422 `kategori_id` tidak ada |
| PUT | `/api/produk/:id` | 200 | 400 (id / field kurang), 404, 409, 422 |
| PATCH | `/api/produk/:id` | 200 | 400 (id / body kosong / invalid), 404, 409, 422 |
| DELETE | `/api/produk/:id` | 204 (soft delete) | 400, 404 (tidak ada atau sudah terhapus) |
| PATCH | `/api/produk/:id/restore` | 200 | 400, 404 (tidak ada atau tidak sedang terhapus) |
| DELETE | `/api/produk/:id/permanent` *(plus)* | 204 | 400, 404, 409 (masih aktif) |
| GET | `/api/produk?kategori_id=1` *(plus)* | 200 | 400 (nilai filter tidak valid) |

## Arti kode status yang dipakai

| Status | Dipakai untuk |
| --- | --- |
| 200 | Berhasil mengambil / mengubah / memulihkan |
| 201 | Berhasil membuat |
| 204 | Berhasil menghapus (tanpa body) |
| 400 | Id tidak valid, validasi gagal, JSON rusak, PUT tidak lengkap, PATCH kosong |
| 404 | Data tidak ditemukan (termasuk produk yang sudah di-soft delete) atau route tidak dikenal |
| 409 | Pelanggaran UNIQUE (23505); hapus kategori yang masih dirujuk produk (23503); permanent delete pada produk aktif |
| 422 | `kategori_id` tidak ada di tabel kategori (23503 pada INSERT/UPDATE) |
| 500 | Error tak terduga, pesan umum |

## Urutan pemeriksaan pada satu request

```text
parseId (400) → validasi body (400) → repository → 404 bila tidak ditemukan
              → error DB dipetakan di controller (409 / 422) → selain itu 500
```

Artinya, jika id valid tetapi body tidak valid, hasilnya 400 (bukan 404), karena validasi dijalankan sebelum query.
