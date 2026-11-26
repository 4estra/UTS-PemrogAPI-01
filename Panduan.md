# UTS Pemrograman API

REST API Express.js + PostgreSQL. Dua tabel: `kategori` (hard delete) dan `produk` (soft delete).

---

## Kontrak — Bekukan Dulu Sebelum Coding

Semua orang kerja paralel, jadi 4 hal ini harus sama sejak awal. Tanpa ini, merge jadi ribut.

**1. Variable .env**

    PORT=3000
    DB_HOST=localhost
    DB_PORT=5432
    DB_USER=postgres
    DB_PASSWORD=***
    DB_NAME=db_uts

**2. Bentuk response**

Sukses: `{ "status": "success", "message": "...", "data": ... }`

Gagal: `{ "status": "error", "message": "...", "errors": [] }`

Daftar nambah `count`. Delete sukses = 204 tanpa body.

**3. Kode error PostgreSQL**

- `23505` (UNIQUE) jadi 409
- `23503` (FK) di DELETE kategori jadi 409
- `23503` (FK) di POST/PUT produk jadi 422

**4. Nama file, jangan diubah**

- Repository: `kategoriRepository.js`, `produkRepository.js`
- Controller: `kategoriController.js`, `produkController.js`
- Route: `kategoriRoutes.js`, `produkRoutes.js`
- Middleware: `parseId.js`, `validateKategori.js`, `validateProduk.js`

---

## Struktur Folder

    UTS-PemrogAPI-01/
    ├── .env.example
    ├── .gitignore
    ├── package.json
    ├── README.md
    ├── sql/
    │   ├── schema.sql
    │   └── seed.sql
    └── src/
        ├── app.js
        ├── server.js
        ├── config/db.js
        ├── routes/
        ├── middlewares/
        ├── controllers/
        └── repositories/

---

## Pembagian Tugas — 5 Orang, Semua Paralel

Semua bisa mulai barengan. Yang penting: kontrak di atas sudah dibekukan.

### A — Database (Abbad)

File: `sql/schema.sql`, `sql/seed.sql`

- Buat database `db_uts`
- Tabel `kategori`: id, nama_kategori, deskripsi, created_at
- Tabel `produk`: id, kategori_id, kode_produk, nama_produk, harga, stok, created_at, deleted_at
- Seed minimal 3 kategori, 5 produk
- Paling kritis, orang lain butuh nama tabel dan kolom kamu

### B — Pondasi (Firman)

File: `src/config/db.js`, `src/app.js`, `src/server.js`

- `db.js`: satu Pool dari pg, baca dari `.env`
- `app.js`: `express.json()`, router, handler 404, error handler
- `server.js`: cuma listen

### C — Kategori, 4 endpoint (Arganata)

File: `kategoriRepository.js`, `kategoriController.js`, `kategoriRoutes.js`

- GET /api/kategori
- GET /api/kategori/:id
- POST /api/kategori
- DELETE /api/kategori/:id, hard delete, tangkap 23503 jadi 409

### D — Produk, 8 endpoint (Eko)

File: `produkRepository.js`, `produkController.js`, `produkRoutes.js`

- GET /api/produk, filter `deleted_at IS NULL` plus count
- GET /api/produk/:id
- GET /api/kategori/:id/produk
- POST /api/produk
- PUT /api/produk/:id
- PATCH /api/produk/:id
- DELETE /api/produk/:id, soft delete
- PATCH /api/produk/:id/restore

Filter `deleted_at IS NULL` wajib di 6 tempat: GET list, GET by id, GET nested, PUT, PATCH, soft delete.

### E — Middleware, Postman, README (Yusron)

File: `middlewares/*`, `uts.postman_collection.json`, `README.md`

- `parseId`: id di URL harus bilangan bulat positif, kalau tidak 400
- `validateKategori` dan `validatePatch`: POST/PUT field lengkap, PATCH cuma yang dikirim
- `validateProduk`: kumpulkan semua error ke array `errors`
- Postman 3 folder: Alur Utama S01-S15, Skenario Error N01-N14, Bukti Delete

---

## Urutan Merge ke Git

Kerja paralel, tapi merge harus urut biar nggak konflik.

1. **A** push dulu, folder `sql/`
2. **B** push, `config/`, `app.js`, `server.js`
3. **C** dan **D** push, bisa barengan karena file beda
4. **E** push, `middlewares/`, postman, README

Kalau konflik: pull dulu, selesaikan di lokal, baru push.

---

## Cara Menjalankan

    npm install
    cp .env.example .env

Edit `.env`, isi password postgres kamu.

    psql -U postgres -f sql/schema.sql
    psql -U postgres -f sql/seed.sql
    npm start

Server jalan di `http://localhost:3000`.

---

## Rancangan Database

Relasi: satu kategori punya banyak produk.

**kategori, induk, hard delete**

- `id` SERIAL PRIMARY KEY
- `nama_kategori` VARCHAR(50) NOT NULL UNIQUE
- `deskripsi` VARCHAR(200)
- `created_at` TIMESTAMP NOT NULL DEFAULT NOW()

**produk, anak, soft delete**

- `id` SERIAL PRIMARY KEY
- `kategori_id` INTEGER NOT NULL, FOREIGN KEY ke kategori(id)
- `kode_produk` VARCHAR(20) NOT NULL UNIQUE
- `nama_produk` VARCHAR(100) NOT NULL
- `harga` INTEGER NOT NULL CHECK (harga > 0)
- `stok` INTEGER NOT NULL DEFAULT 0 CHECK (stok >= 0)
- `created_at` TIMESTAMP NOT NULL DEFAULT NOW()
- `deleted_at` TIMESTAMP NULL, NULL berarti aktif

---

## Daftar Endpoint

**Kategori, hard delete**

- GET /api/kategori, 200
- GET /api/kategori/:id, 200 400 404
- POST /api/kategori, 201 400 409
- DELETE /api/kategori/:id, 204 400 404 409

Body POST kategori:

    { "nama_kategori": "Elektronik", "deskripsi": "Barang elektronik" }

**Produk, soft delete**

- GET /api/produk, 200
- GET /api/produk/:id, 200 400 404
- GET /api/kategori/:id/produk, 200 400 404
- POST /api/produk, 201 400 409 422
- PUT /api/produk/:id, 200 400 404 409 422
- PATCH /api/produk/:id, 200 400 404 409 422
- DELETE /api/produk/:id, 204 400 404
- PATCH /api/produk/:id/restore, 200 400 404

Body POST produk:

    {
      "kategori_id": 1,
      "kode_produk": "ELK-001",
      "nama_produk": "Televisi 32 inci",
      "harga": 2500000,
      "stok": 10
    }

**Nilai tambah**

- DELETE /api/produk/:id/permanent, hard delete baris yang sudah di-soft delete
- GET /api/produk?kategori_id=1, filter

---

## Keputusan Desain

**1. Kenapa PUT dan PATCH dua-duanya**

PUT ganti seluruh data, semua field wajib. PATCH ubah sebagian, body `{}` ditolak 400. Validasi beda: `validate` untuk POST/PUT tuntut lengkap, `validatePatch` untuk PATCH cuma cek field yang dikirim.

**2. Soft delete vs UNIQUE**

Soft delete cuma isi `deleted_at`, baris tetap ada. Jadi index UNIQUE `kode_produk` masih berlaku. Kirim kode yang sama dengan baris terhapus, PostgreSQL lempar 23505, API balas 409. Alasannya: kode produk itu identitas bisnis, harus unik selamanya. Mau pakai ulang kode? Hapus permanen dulu.

**3. Soft delete vs foreign key**

Produk terhapus masih merujuk kategori_id. Foreign key di database nggak tahu soal `deleted_at`, yang dicek cuma barisnya ada atau nggak. Jadi DELETE kategori tetap ditolak 409 selama masih ada produk yang merujuk, aktif maupun terhapus. Konsekuensi: hapus permanen produk dulu, baru kategori.

**4. Kenapa hard delete di tabel induk**

Kategori itu data referensi, bukan transaksional. Nggak punya riwayat yang perlu disimpan. Aman karena foreign key tetap mencegah hapus selama masih dirujuk produk. Sedangkan produk pakai soft delete karena punya harga, stok, dan riwayat yang lebih baik dipertahankan.

---

## Pengumpulan

Satu repo GitHub, branch `main`, tanpa `.env` dan `node_modules`.

1. Kode sumber plus `package.json`, `.env.example`, `.gitignore`
2. `sql/schema.sql` dan `sql/seed.sql`
3. `uts.postman_collection.json`
4. `README.md`
5. Screenshot: Collection Runner folder 1 dan 2, plus bukti database folder 3
6. Judul: `UTS-PemrogAPI-[namakelompok]`
