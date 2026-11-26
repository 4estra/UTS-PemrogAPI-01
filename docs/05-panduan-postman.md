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

# 05. Panduan Postman

## Persiapan

1. Jalankan PostgreSQL dan server (`npm run dev`) serta pastikan `seed.sql` sudah dimuat.
2. Buat **Environment** (mis. `UTS Local`) dengan variabel:

   | Variabel | Initial value |
   | --- | --- |
   | `baseUrl` | `http://localhost:3000` |
   | `kategoriId` | (kosong) |
   | `produkId` | (kosong) |
   | `kategoriSementaraId` | (kosong) |

3. Buat Collection **`UTS Pemrograman API`** dengan tiga folder: `1-Alur Utama`, `2-Skenario Error`, `3-Bukti Delete`.
4. Semua URL memakai `{{baseUrl}}`, tidak boleh ada alamat yang ditulis langsung.
5. Untuk request yang mengirim body: **Body → raw → JSON**.
6. Pilih Environment di pojok kanan atas sebelum menjalankan apa pun.

## Aturan penulisan body (penting)

```json
{
  "kategori_id": {{kategoriId}},
  "kode_produk": "P{{$timestamp}}",
  "nama_produk": "Produk Uji",
  "harga": 15000,
  "stok": 10
}
```

- `{{kategoriId}}` **tanpa kutip** (angka). Dengan kutip jadi string dan ditolak 400.
- `kode_produk` maksimal 20 karakter.
- Gunakan prefiks berbeda untuk nilai unik di request berbeda (lihat jebakan C3 di [03](03-jebakan-dan-solusi.md)).

## Folder 1: Alur Utama (S01-S15)

Jalankan berurutan. Seluruh test harus hijau, dan bisa diulang berkali-kali.

| No | Request | Body / catatan | Hasil yang diperiksa |
| --- | --- | --- | --- |
| S01 | POST `/api/kategori` | `{"nama_kategori": "Kat-{{$timestamp}}", "deskripsi": "Uji Postman"}` | 201, simpan `kategoriId` |
| S02 | POST `/api/produk` | body di atas | 201, simpan `produkId`, `deleted_at` = null |
| S03 | GET `/api/produk/{{produkId}}` | - | 200, `data.id` = produkId |
| S04 | GET `/api/kategori/{{kategoriId}}/produk` | - | 200, `count` ≥ 1, memuat produk S02 |
| S05 | PUT `/api/produk/{{produkId}}` | 5 field lengkap, nilai baru (`kode_produk`: `PU{{$timestamp}}`) | 200, semua field sesuai body |
| S06 | PATCH `/api/produk/{{produkId}}` | `{"stok": 40}` | 200, stok = 40, field lain sama seperti S05 |
| S07 | DELETE `/api/produk/{{produkId}}` | - | 204, body kosong |
| S08 | GET `/api/produk/{{produkId}}` | - | 404 |
| S09 | GET `/api/produk` | - | 200, tidak memuat produkId |
| S10 | PATCH `/api/produk/{{produkId}}/restore` | - | 200, `deleted_at` = null |
| S11 | GET `/api/produk/{{produkId}}` | - | 200 |
| S12 | DELETE `/api/kategori/{{kategoriId}}` | - | 409 (masih dipakai produk) |
| S13 | POST `/api/kategori` | `{"nama_kategori": "Sementara-{{$timestamp}}"}` | 201, simpan `kategoriSementaraId` |
| S14 | DELETE `/api/kategori/{{kategoriSementaraId}}` | - | 204 |
| S15 | GET `/api/kategori/{{kategoriSementaraId}}` | - | 404 |

### Skrip Tests yang bisa dipakai

**S01** (simpan id):

```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
const json = pm.response.json();
pm.test("Kategori dibuat", () => pm.expect(json.data.id).to.be.a("number"));
pm.environment.set("kategoriId", json.data.id);
```

**S02** (simpan id, deleted_at null):

```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
const json = pm.response.json();
pm.test("deleted_at bernilai null", () => pm.expect(json.data.deleted_at).to.eql(null));
pm.environment.set("produkId", json.data.id);
```

**S03** (id sama; perhatikan `Number`):

```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("id sama dengan produkId", () => {
  pm.expect(pm.response.json().data.id).to.eql(Number(pm.environment.get("produkId")));
});
```

**S04**:

```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
const json = pm.response.json();
pm.test("count minimal 1", () => pm.expect(json.count).to.be.at.least(1));
pm.test("memuat produk S02", () => {
  const id = Number(pm.environment.get("produkId"));
  pm.expect(json.data.some((p) => p.id === id)).to.be.true;
});
```

**S06** (hanya stok yang berubah; contoh membandingkan dengan nilai yang dikirim di S05, sesuaikan angkanya):

```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
const d = pm.response.json().data;
pm.test("stok = 40", () => pm.expect(d.stok).to.eql(40));
pm.test("field lain tidak berubah", () => {
  pm.expect(d.nama_produk).to.eql("Produk Uji Revisi"); // samakan dengan body S05
  pm.expect(d.harga).to.eql(25000);                      // samakan dengan body S05
});
```

**S07** (204 tanpa body, jangan panggil `.json()`):

```javascript
pm.test("Status 204", () => pm.response.to.have.status(204));
pm.test("Tanpa body", () => pm.expect(pm.response.text()).to.eql(""));
```

**S09** (tidak memuat produkId):

```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("tidak memuat produkId", () => {
  const id = Number(pm.environment.get("produkId"));
  pm.expect(pm.response.json().data.some((p) => p.id === id)).to.be.false;
});
```

**S10**:

```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("deleted_at kembali null", () => pm.expect(pm.response.json().data.deleted_at).to.eql(null));
```

**S13** (simpan id sementara):

```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
pm.environment.set("kategoriSementaraId", pm.response.json().data.id);
```

**Request yang mengharapkan 404/409** (S08, S12, S15):

```javascript
pm.test("Status 404", () => pm.response.to.have.status(404)); // ganti angkanya sesuai skenario
```

## Folder 2: Skenario Error (N01-N14)

| No | Request | Body / catatan | Status |
| --- | --- | --- | --- |
| N01 | POST `/api/kategori` | `{"nama_kategori": ""}` | 400 |
| N02 | POST `/api/kategori` | `nama_kategori` yang sudah ada di seed | 409 |
| N03 | POST `/api/produk` | `harga: -5`, `stok: "banyak"`; **`errors` berisi 2 pesan** | 400 |
| N04 | POST `/api/produk` | `kategori_id: 99999` | 422 |
| N05 | POST `/api/produk` | `kode_produk` milik produk seed | 409 |
| N06 | PUT `/api/produk/{{produkId}}` | `{"nama_produk": "Baru"}` | 400 |
| N07 | PATCH `/api/produk/{{produkId}}` | `{}` | 400 |
| N08 | PATCH `/api/produk/{{produkId}}` | `{"kode_produk": "<kode produk seed>"}` | 409 |
| N09 | GET `/api/produk/abc` | - | 400 |
| N10 | GET `/api/produk/99999` | - | 404 |
| N11 | DELETE `/api/produk/99999` | - | 404 |
| N12a | DELETE `/api/produk/{{produkId}}` | pertama: 204 | 204 |
| N12b | DELETE `/api/produk/{{produkId}}` | **kedua** | 404 |
| N13a | PATCH `/api/produk/{{produkId}}/restore` | pertama: 200 | 200 |
| N13b | PATCH `/api/produk/{{produkId}}/restore` | **kedua** | 404 |
| N14 | POST `/api/produk` | Body raw JSON rusak: `{"nama_produk": "x",}` | 400 |

Urutan di atas saling cocok: setelah S11 produk aktif, N12a menghapusnya, N13a memulihkannya.

N08 dijalankan saat produk S02 aktif (sebelum N12). Pastikan kode seed yang dipakai berbeda dengan kode produk S05.

Untuk N12 dan N13 (soal bilang "periksa request kedua"), yang menentukan adalah test di **N12b** dan **N13b**.

### Pemeriksaan isi respons error (wajib untuk semua skenario error)

Pasang di tab **Tests** tiap request error (atau di level folder 2 dengan guard `>= 400`):

```javascript
if (pm.response.code >= 400) {
  pm.test("Body error valid", () => {
    const j = pm.response.json();
    pm.expect(j.status).to.eql("error");
    pm.expect(j.message).to.be.a("string").and.not.empty;
  });
}
```

Untuk N03 tambahkan:

```javascript
pm.test("errors berisi dua pesan", () => {
  pm.expect(pm.response.json().errors).to.have.lengthOf(2);
});
```

## Folder 3: Bukti Delete

Isi folder ini belum dijelaskan rinci di soal. Tanyakan dosen bila ragu. Usulan: mini-alur mandiri berisi request membuat kategori dan produk, soft delete, restore, dan hard delete kategori sementara.

Bukti database diambil **manual** (tidak lewat Runner otomatis), dengan pgAdmin atau psql terbuka:

| Momen | Query | Yang harus terlihat |
| --- | --- | --- |
| Setelah S07 | `SELECT id, kode_produk, deleted_at FROM produk WHERE id = <produkId>;` | Baris **masih ada**, `deleted_at` terisi waktu |
| Setelah S10 | query yang sama | `deleted_at` **NULL** |
| Setelah S14 | `SELECT * FROM kategori WHERE id = <kategoriSementaraId>;` | **0 baris** (hilang permanen) |

Cara: klik Send satu per satu sampai S07 → screenshot DB → lanjut S08-S10 → screenshot → lanjut S11-S14 → screenshot. Cantumkan id yang dipakai di query agar terlihat cocok dengan Postman.

## Hasil Collection Runner (B5)

- Jalankan Folder 1 dan Folder 2 dengan Collection Runner (environment terpilih).
- Screenshot ringkasan yang menampilkan jumlah test **lulus dan gagal**.
- **Jangan mengubah screenshot.** Nilai dihitung dari skenario yang benar-benar lulus. Kalau ada yang gagal, perbaiki kodenya.

## Ekspor

- Collection: `uts.postman_collection.json` (Collection v2.1) di root repo.
- Environment: ekspor juga (mis. `uts.postman_environment.json`) agar penguji bisa memakai `baseUrl` yang sama.

