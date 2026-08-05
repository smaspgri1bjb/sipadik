# PANDUAN INSTALASI & DEPLOYMENT — SIPADIK
**Sistem Pemutakhiran Data Peserta Didik (SMAS PGRI 1 Banjarbaru)**

---

## 1. PENDAHULUAN & ARSITEKTUR FILE

Aplikasi ini dibangun menggunakan arsitektur modular yang dapat dijalankan secara langsung di **Google Apps Script (GAS) Web App** maupun di-embed ke **Google Sites** atau di-host di **GitHub Pages**.

```
📁 SIPADIK
├── Konfigurasi.gs        → Metadata sekolah, koordinat, ID template Docs, dropdown
├── Kode.gs                → Web App router (doGet, doPost), templating HTML, API router
├── Utilitas.gs            → Proper case converter, XSS sanitizer, response helper
├── Validasi.gs            → Validasi server-side (NISN, NIK, KK, Email)
├── Lokasi.gs              → Algoritma Haversine (hitung jarak km & estimasi waktu tempuh)
├── Database.gs            → Otomatisasi Drive & Spreadsheet DataPD, LockService
├── Dokumen.gs             → Merge data ke Google Docs, convert PDF & simpan Drive
├── Admin.gs               → Autentikasi SHA-256 password admin & ganti password
├── index.html             → Landing page & Form 6-Tahap
├── style.html             → Modern CSS Design System & Responsive Rules
├── javascript.html        → Client-side logic, stepper 6-tahap, geolokasi, validasi
├── previewdokumen.html    → Pratinjau bukti pemutakhiran
├── adminlogin.html        → Form login admin
└── admindashboard.html    → Dashboard admin (Tabel, Filter, Chart.js, Export XLSX)
```

---

## 2. LANGKAH-LANGKAH DEPLOYMENT DI GOOGLE APPS SCRIPT

### Langkah 1: Buat Project Apps Script Baru
1. Buka [Google Apps Script Dashboard](https://script.google.com/).
2. Klik **New Project** / **Proyek Baru**.
3. Beri nama proyek: `SIPADIK_2026_SMAS_PGRI_1_BANJARBARU`.

### Langkah 2: Salin Seluruh File Proyek
Buat file di editor Apps Script satu per satu sesuai jenis ekstensi:

1. **File Script (`.gs`)**:
   - `Konfigurasi.gs`
   - `Kode.gs`
   - `Utilitas.gs`
   - `Validasi.gs`
   - `Lokasi.gs`
   - `Database.gs`
   - `Dokumen.gs`
   - `Admin.gs`

2. **File HTML (`.html`)**:
   - `index.html`
   - `style.html`
   - `javascript.html`
   - `previewdokumen.html`
   - `adminlogin.html`
   - `admindashboard.html`

> **PENTING**: Nama file `.gs` menggunakan huruf besar di awal (*Proper Case*), sedangkan nama file `.html` menggunakan huruf kecil semua (*lowercase*).

### Langkah 3: Pengaturan Template Google Docs
Buka file `Konfigurasi.gs` dan pastikan ID Template Google Docs milik Anda sudah disesuaikan (jika ingin menggunakan template sendiri):
- `TEMPLATE_DOC_NON_WALI_ID`: `1USksGErSIn_9LiCqUI5b452gR8X4x5EDrcDgNTT4YLc`
- `TEMPLATE_DOC_WALI_ID`: `1-qnJDSINVDpCoeppcgL8kdPAH8sdGDHU68WUeQVciw4`

Pastikan kedua berkas template tersebut telah diberi akses **Dapat Melihat (Viewer)** atau **Editor** untuk akun Google pengembang Apps Script.

### Langkah 4: Inisialisasi Otomatis Storage & Database
1. Pilih fungsi `initDatabaseStorage` di dropdown toolbar editor Apps Script.
2. Klik tombol **Run / Jalankan**.
3. Berikan **Izin Akses (Authorization)** ke Google Drive & Spreadsheet Anda saat popup konfirmasi Google muncul.
4. Sistem akan otomatis membuat:
   - Folder Drive: `01_SIPADIK_2026`
   - Spreadsheet DB: `pemutakhiran_db` (sheet `DataPD` dengan header)
   - Subfolder Arsip: `arsip_output_dokumen`
   - Password Admin Default: `admin123` (di-hash SHA-256)

### Langkah 5: Deploy Sebagai Web App
1. Klik tombol **Deploy** (di kanan atas editor) > **New deployment** / **Pengalaman baru**.
2. Klik ikon gerigi ⚙️ > pilih **Web App**.
3. Isi konfigurasi deployment:
   - **Description**: `SIPADIK v2.0 Production`
   - **Execute as**: `Me (Email Anda)`
   - **Who has access**: `Anyone` (Siapa saja - agar seluruh murid dapat mengakses form tanpa perlu login akun Google).
4. Klik **Deploy**.
5. Salin **Web App URL** (URL ini digunakan sebagai tautan form utama murid).

---

## 3. PENGGUNAAN HALAMAN APLIKASI

| Halaman | Akses URL | Keterangan |
|---|---|---|
| **Formulir Pemutakhiran (Siswa)** | `<WEB_APP_URL>` | Halaman utama pengisian formulir 6-tahap |
| **Login Admin** | `<WEB_APP_URL>?page=admin` | Halaman login administrator |
| **Dashboard Admin** | `<WEB_APP_URL>?page=dashboard` | Halaman dashboard statistik & tabel data |

---

## 4. OPSIONAL: DEPLOYMENT FRONTEND DI GITHUB PAGES

Jika Anda ingin menjalankan file `.html` dan `.js` di **GitHub Pages**:
1. Buat repository baru di akun GitHub Anda (misalnya `sipadik-web`).
2. Unggah file `index.html`, `style.html`, `javascript.html`, dsb.
3. Pada `javascript.html`, sesuaikan nilai `AppState.webAppUrl` dengan Web App URL Google Apps Script Anda:
   ```javascript
   AppState.webAppUrl = "https://script.google.com/macros/s/AKfycb.../exec";
   ```
4. Aktifkan **GitHub Pages** di repository **Settings > Pages > Branch: main**.

---

## 5. FITUR KEAMANAN & TESTING

1. **Anti-Duplicate Submit**: Menggunakan `LockService.getScriptLock()` untuk mencegah submitting ganda pada saat bersamaan.
2. **Password Admin Hashed**: Password admin tidak pernah disimpan secara plain text, melainkan di-hash dengan SHA-256 via `Utilities.computeDigest`.
3. **Validasi Ganda**: Input NISN (10 digit), NIK (16 digit), No KK (16 digit) divalidasi ketat pada client-side dan server-side.
