# 07. Checklist Pengumpulan

Cek semuanya sebelum submit. Judul pengumpulan: **`UTS-PemrogAPI-[namakelompok]`**.

## Repository

- [ ] Satu repository GitHub, cabang **`main`**
- [ ] Nama repo / judul pengumpulan: `UTS-PemrogAPI-[namakelompok]`
- [ ] **Tidak ada** `.env` di repo (cek `git ls-files | findstr .env`)
- [ ] **Tidak ada** `node_modules` di repo
- [ ] `.gitignore` memuat `node_modules` dan `.env`
- [ ] `.env.example` ada dan **tanpa password asli**
- [ ] `package.json` memuat dependency (`express`, `pg`, `dotenv`) dan script `start` / `dev`
- [ ] `npm install` dari clone baru berjalan tanpa error

## Struktur kode

- [ ] Struktur sesuai pohon di soal (`src/`, `sql/`, dll.)
- [ ] `server.js` hanya menjalankan `listen`
- [ ] `app.js` berisi `express.json()`, router, handler 404, error handler terpusat
- [ ] `config/db.js`: pg Pool dibuat sekali, konfigurasi dari `.env`
- [ ] Repository berupa class dan diekspor satu instance
- [ ] **Tidak ada SQL** di controller
- [ ] Semua query berparameter (`$1`, `$2`, ...)
- [ ] `parseId` dipakai di semua route yang memakai `:id`
- [ ] Validasi memakai dua fungsi (`validate...` dan `validatePatch`)

## Database

- [ ] `sql/schema.sql` persis sesuai tabel soal (tipe, constraint, `TIMESTAMP`)
- [ ] FK **tanpa** `ON DELETE CASCADE`
- [ ] `kode_produk` `UNIQUE` biasa (bukan partial index)
- [ ] `sql/seed.sql`: minimal 3 kategori dan 5 produk
- [ ] Seed tidak mengisi `id` manual
- [ ] Schema dan seed bisa dijalankan ulang tanpa error

## Fungsional

- [ ] Semua endpoint A2 dan A3 berjalan dengan status sesuai tabel
- [ ] Filter `deleted_at IS NULL` ada di GET daftar, GET by id, GET nested, PUT, PATCH, soft delete
- [ ] Soft delete kedua → 404; restore pada produk aktif → 404
- [ ] 23503 → 409 saat hapus kategori, 422 saat INSERT/UPDATE produk
- [ ] 23505 → 409
- [ ] JSON rusak → 400, error tak terduga → 500 pesan umum
- [ ] Respons error tidak membocorkan pesan PostgreSQL
- [ ] URL tidak dikenal → 404 `Route tidak ditemukan`

## Postman

- [ ] Collection bernama **`UTS Pemrograman API`**, diekspor sebagai **`uts.postman_collection.json`**
- [ ] Environment: `baseUrl`, `kategoriId`, `produkId`, `kategoriSementaraId`
- [ ] Semua URL memakai `{{baseUrl}}`
- [ ] Tiga folder: `1-Alur Utama`, `2-Skenario Error`, `3-Bukti Delete`
- [ ] S01-S15 dan N01-N14 lengkap, tiap request punya minimal satu `pm.test` status
- [ ] Skenario yang mengubah data memeriksa isi `data`
- [ ] Skenario error memeriksa `status === "error"` dan `message` tidak kosong
- [ ] Folder 1 hijau saat dijalankan **berulang** pada DB yang sudah berisi seed

## README.md

- [ ] Cara menjalankan server
- [ ] Rancangan tabel dan relasinya
- [ ] Daftar endpoint beserta **contoh body**
- [ ] Alasan memilih **PUT dan PATCH**
- [ ] Perilaku soft delete terhadap **UNIQUE** (`kode_produk`)
- [ ] Perilaku soft delete terhadap **foreign key** (hard delete kategori tetap 409)
- [ ] Alasan **hard delete pada tabel induk**
- [ ] Nama anggota dan peran sudah diisi

## Tangkapan layar (`docs/screenshots/`)

- [ ] Runner Folder 1 (jumlah lulus / gagal terlihat)
- [ ] Runner Folder 2 (jumlah lulus / gagal terlihat)
- [ ] DB setelah S07: baris ada, `deleted_at` terisi
- [ ] DB setelah S10: `deleted_at` NULL
- [ ] DB setelah S14: baris kategori sementara tidak ada
- [ ] Screenshot **tidak diedit**

## Nilai tambah (opsional, maks 5 poin)

- [ ] `DELETE /api/produk/:id/permanent` (hanya produk yang sudah di-soft delete; aktif → 409)
- [ ] `GET /api/produk?kategori_id=1`
- [ ] Dua hal di atas tercantum di README (kalau tidak dikerjakan, hapus dari README)

## Terakhir

- [ ] Clone repo ke folder baru, ikuti README dari nol, pastikan berjalan
- [ ] Semua anggota sudah mengecek cabang `main`
- [ ] Link repo dikumpulkan dengan judul yang benar