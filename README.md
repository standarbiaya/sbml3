# Dashboard SBML - paket Cloudflare diperbaiki

Perbaikan dari masalah `The directory specified by the assets.directory field ... does not exist: /opt/buildhome/repo/public`.

## PENTING: struktur root repository

Saat file ZIP diekstrak, **unggah ISI folder hasil ekstrak ke root (folder utama) repository GitHub**, bukan ZIP-nya dan bukan folder pembungkus. Pastikan tampilan root repository menjadi:

```
<root-repository>/
├── public/
│   └── index.html
├── worker.js
├── wrangler.jsonc
├── package.json
├── README.md
├── Dashboard-SBML-Langsung-Buka.html
└── contoh-data/
    └── template-sbml.csv
```

`wrangler.jsonc` menggunakan `"assets": { "directory": "./public" }`. Karena itu folder `public` **wajib sejajar dengan** file `wrangler.jsonc` saat proses build.

### Jika repo GitHub terlanjur mempunyai folder bertingkat

Misalnya yang ada adalah `dashboard-sbml-siap-pakai/public/index.html`, bukan `public/index.html`, pilih **satu** cara:

A. **Disarankan:** pindahkan ISI `dashboard-sbml-siap-pakai/` ke root repository, sehingga `public/` berada di root; atau

B. Atur **Root directory** proyek Cloudflare ke `dashboard-sbml-siap-pakai` (jika pengaturan tersebut tersedia) dan pastikan `wrangler.jsonc` terletak dalam folder yang sama.

Jangan mengganti `assets.directory` menjadi `.` karena itu dapat mempublikasikan file backend dan konfigurasi sebagai aset statis.

## Deploy dari GitHub melalui Cloudflare Workers Builds

1. Buat/ buka repository GitHub, unggah semua isi ZIP yang diekstrak ke root.
2. Periksa pada GitHub: `public/index.html`, `worker.js`, `wrangler.jsonc`, `package.json` tersedia pada lokasi di atas.
3. Di Cloudflare Workers & Pages, pilih Worker yang terhubung ke GitHub dan buka pengaturan **Build / Deploy**.
4. Set `Root directory` ke root repository (kosong atau `/`, sesuai pilihan antarmuka), dan `Deploy command` ke `npx wrangler deploy`. Aplikasi ini tidak membutuhkan build frontend terpisah; bila kolom **Build command** bersifat opsional, biarkan kosong.
5. Simpan, lalu **Retry deployment**. Jika sebelumnya repo mengandung struktur salah, pastikan perubahan struktur sudah di-commit/push ke branch yang terhubung.
6. Uji alamat `https://<nama-worker>.<subdomain>.workers.dev`. Dashboard tampil dengan data demo; Google Sheets belum terhubung jika secrets belum diset.

> Dalam antarmuka Cloudflare yang berbeda, letak dan istilah menu bisa berubah. Prinsip utamanya: command dijalankan dari direktori yang berisi `wrangler.jsonc` dan `public/`.

## Deploy dengan terminal lokal (alternatif)

Buka terminal **di dalam folder yang berisi** `wrangler.jsonc` dan `public` lalu:

```
npm install
npx wrangler login
npm run deploy
```

## Google Sheets (baca saja)

Aplikasi tetap bisa digunakan tanpa Google Sheets. Untuk integrasi, buat tab `SBML` dengan 9 header:

```
No,Nama KL,Tahun,No Surat / Tgl,Perihal,Surat Menkeu / Tgl,Karakteristik K/L,Jenis SBML,Status
```

Aktifkan Google Sheets API di Google Cloud; buat Service Account, lalu bagikan spreadsheet ke email Service Account sebagai Viewer. Isikan Worker secrets secara aman (jangan taruh di repository):

```
npx wrangler secret put SHEET_ID
npx wrangler secret put GOOGLE_CLIENT_EMAIL
npx wrangler secret put GOOGLE_PRIVATE_KEY
```

`SHEET_ID` adalah bagian URL di antara `/d/` dan `/edit`. `GOOGLE_CLIENT_EMAIL` dan `GOOGLE_PRIVATE_KEY` berasal dari JSON key Service Account. Setelah konfigurasi, deploy ulang dan klik Refresh. Google Sheets disinkronkan **baca saja**; perubahan data di dashboard lokal tidak menulis balik ke Sheet.

**Keamanan:** Jika berisi surat/keputusan internal K/L, jangan publikasikan dashboard dan endpoint `/api/sbml` tanpa Cloudflare Access atau autentikasi organisasi. Endpoint API **tidak** memiliki autentikasi sendiri.

## Grafik pie dan fitur

Grafik pie hanya membandingkan **Disetujui** dan **Ditolak**, sedangkan status lain (misalnya Dalam Proses) ditampilkan terpisah. Tersedia filter status, tahun, K/L, jenis SBML, pencarian seluruh kolom, tambah/ubah/hapus data lokal, impor CSV/JSON, ekspor CSV/JSON dan cetak. Data contoh bersifat **fiktif**.

## Uji cepat

- Akses `/` => halaman Dashboard SBML ditampilkan.
- Akses `/api/sbml` sebelum konfigurasi => HTTP 503 `Google Sheets belum dikonfigurasi` (normal).
- Setelah mengisi secrets dengan benar, `/api/sbml` mengembalikan data JSON dari spreadsheet.
