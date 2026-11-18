# Jawaban Praktikum Integratif P13–P15 — TokoKita

Nama : Siti Diana
NPM : 24312118
Prodi : Informatika — Universitas Teknokrat Indonesia
Mata kuliah : Pemrograman Web 2 (INF231102)

---

## SESI 1 — Blade (P13)

### Tugas 1 — Layout induk
- **TODO 1:** slot konten utama diisi `@yield('content')` di dalam `<div class="container mt-4">`.
- **Beda `@yield` dan `@section` (parent vs child):**
  - **Parent (layout)** memakai `@yield('nama')` untuk *membuat lubang/slot* yang bisa diisi. Bila tidak diisi, dipakai nilai default (contoh `@yield('title', 'TokoKita')`).
  - **Child (view anak)** memakai `@extends('layouts.app')` lalu `@section('nama') ... @endsection` untuk *mengisi* slot tersebut.
  - Singkatnya: `@yield` = tempat kosong di induk, `@section` = isi yang dikirim anak ke tempat kosong itu.

### Tugas 2 — Komponen kartu produk
- TODO 2a `{{ $nama }}`, 2b `Rp {{ number_format($harga, 0, ',', '.') }}`, 2c badge hijau bila `stok > 0`, merah dan teks `HABIS` bila 0.

### Tugas 3 — `@forelse` dan `$loop`
- **Keuntungan `@forelse` dibanding `@foreach` saat array kosong:** `@forelse` punya blok `@empty` bawaan, jadi tampilan "Belum ada produk tersedia." muncul tanpa perlu `@if (count($barangs))` terpisah. Dengan `@foreach` biasa halaman akan kosong tanpa pesan.
- TODO 3b: jumlah produk memakai `$loop->count`. Variabel `$loop` hanya ada **di dalam** perulangan, jadi teks "Total produk" ditaruh di dalam `@forelse` dengan `@if ($loop->last)`.

### Tugas 4 — Alert
- `{{ $message }}` ditampilkan di `components/alert-status.blade.php`; tombol `btn-close` + `data-bs-dismiss="alert"` menutup alert.

---

## SESI 2 — Migration, Seeder, Query Builder (P14)

### Tugas 6 — Migration
- **6a – kenapa `unique`?** Nama kategori tidak boleh kembar (mencegah "Minuman" tersimpan dua kali yang membuat data ambigu).
- **6b – kenapa `nullable`?** Deskripsi bersifat opsional; kategori tetap valid tanpa deskripsi, dan kolom `NOT NULL` akan menolak insert tanpa deskripsi.
- **6c –** `foreignId('kategori_id')->constrained()` membuat kolom `kategori_id` (unsigned bigint) beserta foreign key ke `kategoris.id`, sehingga barang tidak bisa menunjuk kategori yang tidak ada.
- **Bila `down()` tidak ditulis:** `php artisan migrate:rollback` / `migrate:fresh` akan error (method `down` tidak ada / tidak melakukan apa-apa), tabel tidak bisa dibatalkan lewat migration, dan perubahan harus dihapus manual. `down()` adalah "tombol undo" sebuah migration (di proyek ini `Schema::dropIfExists(...)`).

### Tugas 7 — Migrate
- Urutan file: `create_kategoris_table` (…122407) dibuat **sebelum** `create_barangs_table` (…122410) karena `barangs` memakai foreign key ke `kategoris`.
- `php artisan migrate:status` → semua migration berstatus **Ran**.
- Eksperimen `migrate:rollback`: hanya batch terakhir yang dibatalkan, yaitu tabel `barangs` dan `kategoris` hilang (tabel `users`, dll. dari batch 1 tetap ada di MySQL). `php artisan migrate` mengembalikannya (kosong).

### Tugas 8 — Seeder
- Ditambahkan 2 barang: `Keripik Singkong` (MK-001, kategori Makanan) dan `Beras Premium 5kg` (BP-002, kategori Bahan Pokok). Total 3 kategori & 5 barang.
- **Beda `db:seed` dan `migrate:fresh --seed`:**
  - `db:seed` hanya **menjalankan seeder** pada tabel yang sudah ada; bila dijalankan dua kali data dobel (atau error karena `unique` pada `sku`/`nama`).
  - `migrate:fresh --seed` **menghapus semua tabel**, membuat ulang dari migration, lalu menjalankan seeder, jadi database kembali bersih. Hati-hati: seluruh data lama hilang.

### Tugas 9 — Query Builder (hasil Tinker)

| Query | Output |
|---|---|
| `DB::table('barangs')->get()` | 5 baris (semua barang) |
| `where('stok', '>', 0)->get()` (**9a**) | 4 barang: Kopi Arabika, Gula Pasir 1kg, Keripik Singkong, Beras Premium 5kg (Teh Melati stok 0 tidak ikut) |
| `where('sku', 'KM-001')->first()` | 1 objek: `{id:1, nama:"Kopi Arabika", sku:"KM-001", harga:75000, stok:12, kategori_id:1}` |
| `where('harga', '<', 30000)->count()` (**9b**) | **3** (Teh Melati 25.000, Gula Pasir 18.000, Keripik 12.000) |
| `orderBy('harga','desc')->limit(3)->get()` | Kopi Arabika (75.000), Beras Premium 5kg (68.000), Teh Melati (25.000) |

**Beda `get()`, `first()`, `count()`:**
- `get()` → mengembalikan **koleksi** semua baris yang cocok (bisa kosong).
- `first()` → mengembalikan **satu** baris (objek) pertama, atau `null` bila tidak ada.
- `count()` → mengembalikan **angka** jumlah baris, tanpa mengambil datanya.

---

## SESI 3 — Eloquent ORM & Relasi (P15)

### Tugas 11 — Model & mass assignment
- `Barang`: `$fillable = ['nama','sku','harga','stok','kategori_id']`; `Kategori`: `$fillable = ['nama','deskripsi']`.
- **Risiko bila `$fillable` dikosongkan semua / diganti `$guarded = []`:** semua kolom boleh diisi massal, sehingga pengguna jahat dapat menyisipkan field tambahan lewat request (mis. `id`, `stok`, `kategori_id`, atau kolom sensitif seperti `is_admin`) dan mengubah data yang tidak seharusnya — kerentanan **mass assignment**. (Catatan: bila `$fillable` kosong dan `$guarded` bawaan, `create()` justru menolak semua field.)

### Tugas 12 — Relasi
- `Kategori::barangs()` → `hasMany(Barang::class)`; `Barang::kategori()` → `belongsTo(Kategori::class)`.
- Hasil Tinker: `Barang::where('sku','KM-001')->first()->kategori->nama` → **"Minuman"**; `Kategori::where('nama','Minuman')->first()->barangs` → koleksi **Kopi Arabika, Teh Melati**.
- **Bagaimana Laravel tahu kolom penghubungnya?** Lewat **konvensi penamaan**: `belongsTo(Kategori::class)` mencari kolom `<nama_method>_id` = `kategori_id` di tabel `barangs` dan mencocokkannya ke primary key `id` di `kategoris`. `hasMany(Barang::class)` mencari `<nama_model_induk>_id` = `kategori_id` di tabel `barangs`. Bila nama kolom berbeda, tulis eksplisit: `belongsTo(Kategori::class, 'kolom_fk')`.

### Tugas 13 — Controller
- `BarangController::index()` memakai `Barang::with('kategori')->get()` (Eloquent, bukan `DB::table`). Rute `/katalog` mengarah ke `[BarangController::class, 'index']`. Kartu menampilkan nama kategori lewat props baru `kategori` (default `'-'`).

### Tugas 14 — N+1 Problem

**Penjelasan:** N+1 terjadi saat 1 query mengambil daftar induk (barang), lalu setiap baris memicu 1 query lagi untuk mengambil relasinya (kategori) saat diakses di perulangan — total **1 + N** query. Pada data besar ini sangat lambat karena banyak round-trip ke database. *Eager loading* (`with('kategori')`) mengambil semua kategori yang dibutuhkan sekaligus dengan satu query `WHERE id IN (...)`, sehingga total tetap **2 query**.

**Hasil eksperimen (5 barang, dicatat di `storage/logs/laravel.log` lewat `DB::listen`):**

| Versi | Kode | Jumlah query |
|---|---|---|
| Bermasalah | `Barang::all()` + `foreach { $b->kategori->nama }` | **6** (1 + 5) |
| Eager loading | `Barang::with('kategori')->get()` | **2** (tetap, berapa pun jumlah barang) |

Cuplikan log (eager loading):
```
select * from "barangs"
select * from "kategoris" where "kategoris"."id" in (1, 2, 3)
```
Cuplikan log (N+1): 1 query `select * from "barangs"` diikuti 5 query `select * from "kategoris" where "id" = ? limit 1`.

---

## Refleksi
- Hal paling membantu: komponen Blade (`card-barang`, `alert-status`) dipakai ulang di Sesi 1 sampai 3 tanpa mengubah markup.
- Catatan lingkungan: `.env` proyek memakai `DB_DATABASE=tokokita` (sesuai dump `tokokita.sql`), sedangkan modul menyebut `tokokita_db`; pakai salah satu secara konsisten.
- Perbaikan kecil: `use App\http\Controllers\ProdukController` diperbaiki menjadi `App\Http\...` (huruf besar) agar tetap jalan di Linux/hosting.
