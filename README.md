# Try Out TKA Matematika SMP 2027 — Kabupaten Pacitan

Aplikasi try out mandiri (dua file HTML, tanpa CDN apa pun): `index.html`
(halaman siswa) dan `guru.html` (halaman guru). Desain bersih dan terang,
nyaman di HP, mengikuti gaya aplikasi Try Out TKA SD/MI sebelumnya.

Kredit pembuatan: **Design by [rasa.web.id](https://rasa.web.id)** — tampil di
kaki halaman pada semua tampilan dan di bawah judul pada tampilan awal,
di `index.html` maupun `guru.html`.

## Bank soal — 59 soal

Sumber: `TKA_MATEMATIKA_SMP_2027_VERSI_1.html` (30 soal) dan
`TKA_MATEMATIKA_SMP_2027_VERSI_2.html` (30 soal). Bank akhir: **59 soal**
(PG 29, MCMA 20, Benar/Salah 10) — lihat `../bank-soal.json`.

- **Dedup:** V2 q7 dibuang karena sama isi dengan V1 q5 (barisan aritmetika
  a = 15, b = 4, U9 = 47; hanya redaksi dan pengecohnya berbeda).
- **Audit kunci:** 59/59 kunci TERVERIFIKASI benar — tiap soal diselesaikan
  ulang dan dihitung ulang dengan Python, lalu dibandingkan dengan kunci dan
  dengan kesimpulan penjelasannya sendiri. **0 soal bermasalah.**
  Bukti hitung: `../work/audit-kunci.md`.
- **Gambar:** V2 memakai 30 diagram SVG inline — dipertahankan apa adanya.
  V1 memakai 30 foto Unsplash: 29 dihapus (murni dekoratif, tidak dirujuk
  batang soal), 1 dibuat ulang sebagai diagram SVG (V1 q28, diagram lingkaran
  penjualan HP: Samsung 30%, Apple 25%, Xiaomi 20%, Vivo 15%, Oppo 10%).
  Temuan: foto Unsplash yang terpasang di V1 q28 ternyata BUKAN diagram
  lingkaran (foto ikon media sosial) — menyesatkan, sudah diganti SVG.
  **Tidak ada satu pun URL gambar eksternal yang dipertahankan.**
  Catatan: soal V1 q9 (tangga Pythagoras), q10 (taman + kolam), dan q27
  (garis sejajar) secara substansi akan terbantu diagram, tetapi batangnya
  tidak merujuk gambar — kandidat penambahan SVG bila dikehendaki nanti.

## Aturan sesi (resmi TKA)

- Setiap siswa mendapat **30 soal acak**: 15 PG (dari 29), 10 MCMA (dari 20),
  5 Benar/Salah (dari 10). Urutan soal DAN urutan pilihan PG/MCMA diacak
  (penilaian memakai identitas pilihan, bukan huruf).
- Waktu: **75 menit total** per sesi (hitung mundur, tanpa batas per soal).
  Waktu habis → otomatis dikumpulkan dan dinilai dari jawaban yang ada.
- Palet nomor soal 1–30 (sudah/belum dijawab) + tombol navigasi.
- Identitas: Nama, Kelas IX + Rombel (teks bebas, mis. IX-A), pilih
  Kecamatan dulu (12 kecamatan) lalu pilih Sekolah dari daftar
  **72 SMP/MTs Kabupaten Pacitan + NPSN** (sumber: daftar guru repo
  `masgurubudi/tryout-tka-kabupaten`; kecamatan dari rekap TO 5 Nov 2026
  + verifikasi direktori Kemendikdasmen untuk 4 sekolah: Al-Istiqomah →
  Pacitan, PGRI Sugihwaras → Pringkuku, PGRI Montongan → Tulakan,
  Muhammadiyah 3 → Tegalombo — untuk yang terakhir ada satu sumber lama yang
  menulis alamat di Kec. Pacitan, tetapi 3 sumber + Wikipedia Desa Tegalombo
  + koordinat menunjuk Tegalombo).

## Penilaian

- PG: 2 poin; MCMA: 3,5 poin (cocok persis); Benar/Salah: 3,5 poin dibagi
  rata per pernyataan (kredit sebagian).
- Skor mentah maks **82,5** → normalisasi 0–100 (satu desimal).
- Setelah selesai: skor + rincian benar/salah per soal + pembahasan tiap soal.

## Penyimpanan nilai

- Firebase project `database-mpi-spendaka`, koleksi **baru**
  `tka_matematika_2027` (konfigurasi diambil dari aplikasi SD/MI yang sudah
  ada — tidak dikarang). Disimpan per percobaan: nama, kelas, rombel,
  sekolah, NPSN, kecamatan, skor, benar, total, rincian per soal
  (id soal + benar/salah + poin), durasi, waktu.
- Komunikasi memakai **Firestore REST API** (tanpa SDK/CDN) supaya berkas
  tetap mandiri.
- Cadangan: jawaban + sisa waktu tersimpan ke localStorage secara berkala.
  Bila halaman dimuat ulang saat sesi berjalan, ditawarkan **Lanjutkan Sesi**
  atau **Buang & Mulai Baru**. Bila simpan ke Firestore gagal (offline),
  record diantrekan di localStorage dan **dikirim ulang otomatis** pada
  kunjungan berikut. Skor tetap ditampilkan apa pun yang terjadi.

## Halaman guru (`guru.html`)

- Masuk dengan **PIN master `112233`** (melihat semua sekolah) ATAU **NPSN
  sekolah** (hanya melihat nilai sekolah sendiri). PIN hanya ada di
  `guru.html` — tidak ada di `index.html`.
- Tab: **Matriks Nilai** (satu baris per siswa, skor terbaik, kolom
  Kecamatan), **Peringkat** (peringkat sama untuk nilai sama),
  **Semua Percobaan**, **Rekap Kabupaten** (peserta, percobaan, sekolah ikut,
  rata-rata, tertinggi/terendah, sebaran nilai, rata-rata per kecamatan).
- Filter kecamatan + sekolah + nama. **Ekspor CSV** dari tampilan yang aktif
  mengikuti filter: pemisah titik koma + BOM (terbuka rapi di Excel Indonesia).

## Aturan Firestore (`firestore-rules.txt`)

Tempel blok `match /tka_matematika_2027/{docId}` ke aturan Firestore proyek
`database-mpi-spendaka` (di samping blok `tryout_tka_sd` dan
`tryout_tka_kabupaten` yang sudah ada), lalu terbitkan dari Firebase Console:

```
match /tka_matematika_2027/{docId} {
  allow read, create: if true;
  allow update, delete: if false;
}
```

Aturan ini TIDAK diterbitkan oleh pembuat aplikasi — user yang menerbitkannya
sendiri dari Firebase Console.

## Penerbitan

Unggah `index.html` dan `guru.html` ke hosting yang sama (mis. GitHub Pages
pada repo yang sama dengan aplikasi try out sebelumnya). Tidak ada berkas
lain yang dibutuhkan — semua CSS, soal, dan daftar sekolah sudah di dalam
kedua berkas.

## Verifikasi (jsdom + Node, `build-app/test.js`)

**42/42 uji lulus**, antara lain:

- Komposisi undian tepat 15/10/5, tanpa id ganda, tiga sesi berurutan
  menghasilkan undian berbeda, opsi PG teracak.
- Timer 75:00 dan auto-submit saat waktu habis.
- Skor: semua benar = 100,0 (82,5 poin); hanya PG benar = 36,4;
  kredit sebagian Benar/Salah (2/3 pernyataan) = 2,8.
- Resume dari localStorage (tawaran lanjutkan sesi).
- Gagal simpan → antre di localStorage; kunjungan berikut mengirim ulang.
- Payload Firestore: nama, NPSN, kecamatan, rincian 30 soal ke koleksi
  `tka_matematika_2027`.
- Guru: PIN salah ditolak; PIN master melihat semua; NPSN hanya sekolah
  sendiri; matriks memakai skor terbaik; peringkat benar; CSV diawali BOM,
  pemisah titik koma, mengikuti filter aktif.

## Perbaikan dari versi 1/2 yang dimasukkan

Dibanding berkas HTML asal: timer total 75 menit (bukan 4 menit per soal);
hasil tersimpan ke Firestore proyek sendiri (bukan config lingkungan
artifact); simpan jawaban otomatis + lanjutkan sesi; antrean offline;
foto Unsplash dekoratif dihapus (V1) / diganti SVG (q28); kunci guru dan
kode guru tidak ada di berkas siswa; tanpa CDN (tanpa Tailwind, Google Fonts,
gambar eksternal); identitas skala kabupaten (kecamatan + 72 sekolah + NPSN).
