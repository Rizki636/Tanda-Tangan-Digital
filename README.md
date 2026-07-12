# Registri — Sistem Tanda Tangan Digital Pribadi

Sistem tanda tangan digital untuk penggunaan pribadi, dibangun dari GitHub Pages +
Google Sheets + Google Apps Script. Setiap kali Anda menandatangani dokumen, sistem
mencatat entri ke "buku register" (Google Sheet) berupa ID unik, waktu, judul dokumen,
identitas Anda, dan sidik (hash) dokumen akhirnya. Yang dibubuhkan ke PDF, secara
default, **murni kode QR saja** — tidak ada ID, nama, atau keterangan apa pun tercetak,
karena semua itu sudah bisa dilihat begitu QR-nya dipindai. Anda juga bisa menambahkan
teks bebas sendiri di atas/bawah QR (mis. gaya "Ditetapkan di ... / Pada tanggal ... /
Jabatan / Instansi / Nama") dan mengatur ukuran QR-nya sendiri.

## Isi berkas

```
index.html      → halaman depan
sign.html       → alat menandatangani (privat, perlu kata sandi)
verify.html     → alat verifikasi (publik)
Code.gs         → backend Google Apps Script
assets/style.css
```

## Langkah pemasangan

### 1. Buat Google Sheet
1. Buat Google Sheet baru, beri nama bebas (mis. "Registri Tanda Tangan").
2. Buka **Extensions → Apps Script**.
3. Hapus isi `Code.gs` bawaan, ganti dengan isi `Code.gs` dari paket ini.
4. Simpan project (beri nama, mis. "Registri Backend").

### 2. Atur kata sandi rahasia & identitas Anda
1. Di editor Apps Script: **Project Settings** (ikon gerigi di kiri) →
   **Script Properties** → **Add script property**.
2. Tambahkan tiga baris berikut:
   - Key `PASSPHRASE` → kata sandi rahasia pilihan Anda.
   - Key `SIGNER_NAME` → nama lengkap Anda, tampil saat orang memverifikasi.
   - Key `SIGNER_ORG` → nama instansi/perusahaan Anda (boleh dikosongkan).
3. Simpan.

### 3. Deploy sebagai Web App
1. Di editor Apps Script: **Deploy → New deployment**.
2. Pilih tipe **Web app**.
3. **Execute as**: Me (akun Anda).
4. **Who has access**: Anyone.
5. Deploy. Google akan meminta Anda **mengizinkan akses ke Google Drive** (dipakai
   untuk fitur "lihat file digital" opsional) — setujui saja, karena berjalan di
   bawah akun Anda sendiri.
6. Salin **URL Web App** yang muncul (`https://script.google.com/macros/s/…/exec`).
7. Setiap kali Anda mengubah `Code.gs`, buat **New deployment** lagi (atau
   "Manage deployments → Edit → New version") agar perubahan aktif.

### 4. Buat repository GitHub Pages
1. Buat repository baru di GitHub (boleh publik).
2. Unggah `index.html`, `sign.html`, `verify.html`, folder `assets/`.
3. **Settings → Pages** → aktifkan dari branch `main` folder root.
4. Catat URL situsnya, mis. `https://USERNAME.github.io/NAMA-REPO`.

### 5. Sambungkan konfigurasi
Buka `sign.html` dan `verify.html`, cari bagian `CONFIG` di bagian atas `<script>`,
lalu isi:

```js
const CONFIG = {
  APPS_SCRIPT_URL: "https://script.google.com/macros/s/XXXXX/exec", // dari langkah 3
  SITE_BASE_URL:   "https://USERNAME.github.io/NAMA-REPO"           // dari langkah 4
};
```

Unggah ulang kedua berkas ini ke GitHub setelah diisi.

### 6. Coba
1. Buka `sign.html`, isi kata sandi + judul dokumen, unggah PDF.
2. (Opsional) Di bagian "Ukuran & teks di sekitar QR": atur ukuran QR kalau mau,
   dan isi teks bebas di atas/bawah QR kalau Anda mau gaya seperti stempel resmi
   (kosongkan kalau mau murni QR saja).
3. Pilih halaman, lalu geser kotak merah (stempel) ke posisi yang Anda mau —
   coba juga tombol preset "Kiri atas/Kanan atas/dst" sebagai titik awal cepat.
4. (Opsional) Centang "Izinkan siapa pun yang memindai QR ini membuka salinan
   dokumennya" kalau Anda memang ingin dokumen ini bisa dilihat isinya lewat QR.
5. Klik **Catat & bubuhkan tanda tangan**, lalu unduh PDF hasilnya dan pastikan
   stempelnya muncul di posisi yang Anda pilih.
6. Buka `verify.html`, salin ID dari `sign.html`, klik **Periksa** — pastikan
   status "Tercatat Sah" muncul dengan nama & judul dokumen yang benar. Kalau
   langkah 4 Anda centang, tombol **"📄 Lihat file digital"** akan muncul dan
   bisa dibuka/ditutup.
7. Di kotak "Cek keutuhan dokumen", unggah PDF hasil unduhan tadi — pastikan
   muncul "✓ Cocok".
8. Cek Google Sheet Anda — baris baru harus muncul di tab `SignatureLog`.

## Cara kerja sistem anti-penyalahgunaan

- **Tidak ada apa pun tercetak selain QR, secara default.** Tidak ada gambar tanda
  tangan, tidak ada ID atau nama otomatis tercetak — semua detail (nama, instansi,
  judul dokumen, waktu, status) baru muncul setelah QR-nya benar-benar dipindai dan
  diperiksa ke buku register, bukan bisa dibaca langsung dari dokumennya. Kalau Anda
  mau, Anda bisa menambahkan teks bebas sendiri (yang Anda ketik manual) di atas/bawah
  QR, dan mengatur ukuran QR-nya.
- **Dua hash dicatat per entri**: `OriginalHash` (sebelum distempel, arsip
  internal) dan `FinalHash` (setelah distempel — inilah yang dibandingkan lewat
  fitur "Cek keutuhan dokumen" di `verify.html`). Kalau seseorang mengedit
  dokumen atau memindahkan stempel QR ke dokumen lain, hash-nya tidak akan cocok.
- **Posisi stempel bebas digeser** (preset + drag manual pakai jari/mouse),
  disimpan sebagai posisi relatif terhadap halaman sehingga tetap akurat di
  berbagai ukuran kertas.
- **Identitas penanda tangan tercantum** saat verifikasi: nama, instansi (jika
  diisi), jabatan/alasan (jika diisi) — bukan sekadar status valid/tidak.
- **Fitur "Lihat file digital" bersifat opt-in per dokumen.** Defaultnya OFF.
  Kalau Anda aktifkan untuk suatu dokumen, file (versi sudah distempel)
  diunggah ke Google Drive Anda dengan akses "siapa saja yang punya link dapat
  melihat", lalu tombol "📄 Lihat file digital" muncul di halaman verifikasi
  untuk dokumen itu saja — bisa dibuka/ditutup oleh siapa pun yang memeriksa.
- **Tombol cabut (revoke).** Panggil `action=revoke` dengan kata sandi Anda
  untuk mengubah status suatu entri jadi "Revoked" — `verify.html` langsung
  menunjukkan status tidak berlaku.

### Batasan & hal yang perlu Anda sadari
- **Privasi dokumen kalau fitur "lihat file digital" diaktifkan**: siapa pun
  yang memiliki QR/link/ID tersebut bisa membuka ISI dokumennya, bukan hanya
  penerima yang Anda maksud. Aktifkan hanya untuk dokumen yang memang boleh
  dibaca siapa saja yang memegang link/QR-nya.
- **Ukuran file**: dokumen dikirim sebagai teks base64 lewat POST — sebaiknya
  jaga ukuran PDF beberapa MB saja (bukan puluhan MB) kalau fitur "lihat file
  digital" diaktifkan, supaya prosesnya tidak lambat atau gagal.
- Ini **bukan tanda tangan elektronik tersertifikasi secara hukum** (bukan PKI/
  sertifikat digital resmi). Untuk dokumen yang perlu kekuatan hukum formal di
  Indonesia, gunakan penyelenggara tersertifikasi Kominfo (mis. Privy, Digisign,
  PrivyID, dll).
- Siapa pun yang tahu kata sandi Anda bisa membuat entri baru — jaga kata sandi
  seperti menjaga kunci meterai.
- Apps Script Web App dengan akses "Anyone" berarti endpoint verifikasi bisa
  diakses publik (memang harus, agar orang lain bisa memverifikasi) — tapi
  endpoint `sign`, `finalize`, dan `revoke` tetap terkunci di balik kata sandi
  Anda.
