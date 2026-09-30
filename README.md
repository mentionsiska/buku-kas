# Buku Kas — Tracker Pemasukan & Pengeluaran Pribadi

Aplikasi web sederhana untuk mencatat pemasukan dan pengeluaran pribadi, dengan desain minimalis bergaya buku kas (ledger). Data disimpan di Firestore dan hanya bisa diakses oleh akun masing-masing pengguna.

Isi folder ini:
```
buku-kas/
├─ public/
│  └─ index.html        <- seluruh aplikasi (HTML, CSS, JS) dalam satu file
├─ firebase.json         <- konfigurasi Firebase Hosting
├─ firestore.rules       <- aturan keamanan Firestore
└─ README.md
```

## 1. Buat project Firebase

1. Buka [Firebase Console](https://console.firebase.google.com/) → **Add project** → ikuti langkahnya.
2. Di dalam project, buka **Build → Authentication → Get started**, lalu aktifkan provider **Google**. Isi "Project support email" dengan email kamu, lalu simpan.
3. Buka **Build → Firestore Database → Create database**. Pilih mode **Production**, pilih region terdekat (mis. `asia-southeast2` untuk Indonesia).
4. Buka **Project settings** (ikon gerigi) → tab **General** → scroll ke **Your apps** → klik ikon **Web (`</>`)** untuk mendaftarkan aplikasi web baru. Beri nama bebas, lalu Firebase akan menampilkan objek `firebaseConfig`.

> Catatan: setelah deploy ke Hosting (langkah 6), domain `NAMA-PROJECT.web.app` otomatis masuk ke daftar domain yang diizinkan untuk login Google. Kalau nanti pakai domain kustom, tambahkan manual di **Authentication → Settings → Authorized domains**.

## 2. Masukkan konfigurasi ke aplikasi

Buka `public/index.html`, cari bagian ini di dalam `<script type="module">`:

```js
const firebaseConfig = {
  apiKey: "GANTI_DENGAN_API_KEY",
  authDomain: "GANTI_DENGAN_PROJECT.firebaseapp.com",
  projectId: "GANTI_DENGAN_PROJECT_ID",
  storageBucket: "GANTI_DENGAN_PROJECT.appspot.com",
  messagingSenderId: "GANTI_DENGAN_SENDER_ID",
  appId: "GANTI_DENGAN_APP_ID"
};
```

Ganti seluruh nilainya dengan `firebaseConfig` yang didapat dari langkah 1.4 di atas, lalu simpan file.

## 3. Install Firebase CLI (jika belum ada)

```bash
npm install -g firebase-tools
firebase login
```

## 4. Hubungkan folder ini ke project Firebase

Dari dalam folder `buku-kas/`:

```bash
firebase use --add
```

Pilih project Firebase yang tadi dibuat, beri alias (mis. `default`). Ini akan membuat file `.firebaserc` — tidak perlu menjalankan `firebase init` lagi karena `firebase.json` dan `firestore.rules` sudah disediakan.

## 5. Deploy aturan keamanan Firestore

```bash
firebase deploy --only firestore:rules
```

## 6. Deploy aplikasi ke Firebase Hosting

```bash
firebase deploy --only hosting
```

Setelah selesai, CLI akan menampilkan URL seperti `https://NAMA-PROJECT.web.app` — buka URL itu untuk mulai memakai aplikasi.

## Coba dulu secara lokal (opsional)

```bash
firebase emulators:start --only hosting
```

lalu buka `http://localhost:5000`. Karena aplikasi tetap terhubung ke Authentication & Firestore project asli (bukan emulator), akun dan data yang dibuat akan sama seperti di versi yang sudah dideploy.

## Cara kerja aplikasi

- **Login** memakai akun Google (Firebase Authentication, provider Google via popup). Tidak perlu daftar manual — akun Google yang dipakai otomatis jadi akun aplikasi. Setiap akun hanya melihat datanya sendiri.
- **Tambah transaksi**: klik "+ Tambah transaksi", pilih Pemasukan/Pengeluaran, isi deskripsi, jumlah, tanggal, dan kategori.
- **Saldo, total masuk, dan total keluar** dihitung otomatis dan diperbarui secara real-time (Firestore `onSnapshot`).
- **Hapus transaksi** lewat tombol × di setiap baris.
- **Filter** daftar transaksi: Semua / Pemasukan / Pengeluaran.
- Data tersimpan di koleksi Firestore `transactions`, masing-masing dokumen memiliki field `uid` pemiliknya — inilah yang dijaga oleh `firestore.rules` agar pengguna lain tidak bisa membaca atau mengubah data orang lain.

## Kustomisasi

- **Kategori**: ubah daftar `<option>` pada elemen `#txCategory` di `index.html`.
- **Warna & font**: seluruh token desain ada di bagian `:root { ... }` pada `<style>` di awal `index.html` (warna latar, tinta, warna pemasukan/pengeluaran, dan jenis huruf).
- **Mata uang**: fungsi `rupiah()` di JavaScript memakai `Intl.NumberFormat('id-ID', { currency: 'IDR' })` — ganti locale/kode mata uang jika perlu.
