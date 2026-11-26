---
title: Checklist QA
nav_order: 6
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

# 06. Checklist QA dan Uji Celah

Skenario di luar S01-S15 dan N01-N14 untuk menemukan bug sebelum dosen menemukannya. Tandai hasilnya dengan ✅ / ❌.

## Validasi input

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q01 | POST kategori: `nama_kategori` hanya spasi `"   "` | 400 | |
| Q02 | POST kategori: `nama_kategori` 51 karakter | 400 (bukan 500) | |
| Q03 | POST kategori: `deskripsi` 201 karakter | 400 | |
| Q04 | POST kategori: `nama_kategori` berupa angka `123` atau `true` | 400 | |
| Q05 | POST produk: `harga: 10.5` | 400 | |
| Q06 | POST produk: `harga: "5000"` (string) | 400 | |
| Q07 | POST produk: `harga: 0` dan `harga: -1` | 400 | |
| Q08 | POST produk: `stok: -1` | 400 | |
| Q09 | POST produk: `stok: 0` | 201 (0 diperbolehkan) | |
| Q10 | POST produk: `harga: 99999999999` | 400 (bukan 500) | |
| Q11 | POST produk: `kategori_id: 0`, `-1`, `1.5`, `"1"` | 400 | |
| Q12 | POST produk: `kategori_id: 99999999999` | 400 (bukan 422/500) | |
| Q13 | POST produk: `kode_produk` 21 karakter | 400 (bukan 500) | |
| Q14 | POST produk tanpa body / tanpa header JSON | 400 | |
| Q15 | Body `[]`, `"abc"`, `123`, `null` | 400 | |
| Q16 | Semua field produk kosong sekaligus | 400, `errors` memuat semua field | |

## Id di URL

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q20 | `/api/produk/0`, `/api/produk/-1` | 400 | |
| Q21 | `/api/produk/1e2`, `/api/produk/1.0`, `/api/produk/0x10` | 400 | |
| Q22 | `/api/produk/99999999999` | 400 (bukan 500) | |
| Q23 | `/api/kategori/abc`, `/api/kategori/abc/produk` | 400 | |
| Q24 | `/api/kategori/99999/produk` | 404 | |
| Q25 | Kategori ada, tanpa produk: `/api/kategori/<id>/produk` | 200, `data: []`, `count: 0` | |

## Soft delete dan restore

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q30 | PUT / PATCH pada produk yang sudah di-soft delete | 404 | |
| Q31 | GET by id, GET daftar, GET nested untuk produk terhapus | tidak muncul | |
| Q32 | PATCH `{"deleted_at": null}` pada produk terhapus | 404 (dan tidak ter-restore) | |
| Q33 | PATCH `{"deleted_at": null}` pada produk aktif | diabaikan / 400 (`deleted_at` tidak berubah) | |
| Q34 | POST produk dengan field tambahan `id`, `deleted_at`, `created_at` | field tambahan diabaikan | |
| Q35 | Soft delete lalu POST kode_produk yang sama | 409 | |
| Q36 | Soft delete lalu hard delete kategorinya | 409 | |
| Q37 | Restore produk yang `kategori_id`-nya masih ada | 200 | |

## Unik dan foreign key

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q40 | PUT produk dengan data **sama persis** dirinya sendiri | 200 (bukan 409) | |
| Q41 | PATCH `kode_produk` ke nilai sendiri | 200 | |
| Q42 | PUT / PATCH `kategori_id` ke kategori yang tidak ada | 422 | |
| Q43 | PATCH hanya `stok` (tidak menyentuh kategori) | 200 | |
| Q44 | Hapus kategori yang punya produk soft delete saja | 409 | |
| Q45 | Hapus kategori tidak ada | 404 | |
| Q46 | Hapus kategori yang sama dua kali | 204 lalu 404 | |

## Keamanan dan kebocoran

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q50 | `nama_produk`: `'; DROP TABLE produk;--` | Tersimpan sebagai teks biasa, tabel aman | |
| Q51 | Cari di semua respons error teks `duplicate key`, `violates`, `constraint`, `relation` | Tidak ada | |
| Q52 | Matikan PostgreSQL, panggil `GET /api/produk` | 500 pesan umum | |
| Q53 | Kirim JSON rusak ke semua endpoint POST/PUT/PATCH | 400 `JSON tidak valid` | |
| Q54 | Method tidak didefinisikan (`PUT /api/kategori/1`) | 404 `Route tidak ditemukan` | |
| Q55 | URL acak `/api/xyz`, `/` | 404 `Route tidak ditemukan` | |

## Nilai tambah (jika dikerjakan)

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q60 | `DELETE /api/produk/:id/permanent` pada produk **aktif** | 409 | |
| Q61 | `DELETE /api/produk/:id/permanent` pada produk **sudah di-soft delete** | 204 | |
| Q62 | `DELETE /api/produk/:id/permanent` pada id tidak ada | 404 | |
| Q63 | Permanent delete lalu hapus kategorinya (jika tak ada produk lain) | 204 | |
| Q64 | `GET /api/produk?kategori_id=1` | 200, hanya milik kategori 1, hanya yang aktif | |
| Q65 | `GET /api/produk?kategori_id=abc` | 400 | |
| Q66 | `GET /api/produk?kategori_id=99999` | 200, `data: []` | |

## Pengulangan Runner

| # | Percobaan | Harapan | Hasil |
| --- | --- | --- | --- |
| Q70 | Jalankan Folder 1 tiga kali berturut-turut tanpa reset DB | Semua hijau | |
| Q71 | Jalankan Folder 1 → Folder 2 berurutan | Semua hijau | |
| Q72 | Reset DB (schema + seed), jalankan lagi dari awal | Semua hijau | |

## Pencatatan bug

| ID | Ditemukan oleh | Langkah | Seharusnya | Sebenarnya | Status |
| --- | --- | --- | --- | --- | --- |
| | | | | | |
