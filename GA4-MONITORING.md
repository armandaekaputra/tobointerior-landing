# GA4 Monitoring — TOBO Interior Landing Page

File khusus untuk sesi yang fokus pantau performa funnel via GA4 (terpisah
dari [CLAUDE.md](CLAUDE.md) yang isinya konteks project & Google Ads secara
keseluruhan). Baca file ini dulu di awal sesi baru, tidak perlu re-derive
dari nol.

## Konteks singkat

Landing page ini nyari konversi lewat 1 aksi: isi form di `/konsultasi` →
redirect WhatsApp. GA4 dipakai bukan buat optimasi campaign (itu tugas
Google Ads conversion tracking), tapi buat **diagnosis funnel** —
lihat berapa orang yang sampai ke `/konsultasi/` dan berapa lama mereka
di situ, karena Google Ads sendiri tidak bisa kasih detail itu.

## Cara akses (gotcha yang sering kejadian)

1. Buka `analytics.google.com` via Claude in Chrome — biasanya kebuka di
   akun default (`tobokoding@gmail.com`) dulu, harus **switch akun** dulu
   ke **`tobocreative@gmail.com`** lewat avatar kanan atas.
2. Setelah switch, otomatis masuk ke properti **"TOBO Interior - Situs
   Utama"** (property ID `548858288`, akun `403729391`).
3. **WAJIB filter dulu** sebelum baca angka apa pun — laporan level-properti
   menggabungkan SEMUA stream (landing page + `tobointerior.com` yang
   punya blog). Tanpa filter, angka jadi kecampur dan menyesatkan.

## Shortcut: langsung ke laporan yang sudah difilter

URL ini langsung buka laporan "Halaman dan layar" (belum ter-filter,
tinggal tambah filter manual sekali lagi — GA4 tidak simpan filter di URL
kalau dibuat lewat klik UI biasa):

```
https://analytics.google.com/analytics/web/?authuser=1#/a403729391p548858288/reports/explorer?params=_u..nav%3Dmaui&ruid=all-pages-and-screens,business-objectives,raise-brand-awareness&collectionId=business-objectives&r=all-pages-and-screens
```

Langkah pasang filter (selalu sama):
1. Klik "Tambahkan filter" di bawah judul laporan.
2. Dimensi → ketik/pilih **"Nama Host"**.
3. Jenis Pencocokan → **"sama persis"**.
4. Nilai → pilih **`landing.tobointerior.com`** (jangan pilih
   `tobointerior.com` atau `localhost`).
5. Klik "Terapkan".
6. Atur rentang tanggal custom sesuai kebutuhan (klik tanggal di kanan
   atas, pilih "Kustom").

## Metrik yang dipantau

Dari tabel hasil filter, fokus ke baris `/konsultasi/` dibanding baris `/`:
- **Tampilan & Pengguna aktif** — makin banyak yang sampai ke
  `/konsultasi/` (bukan cuma `/`), makin bagus.
- **Waktu engagement rata-rata** — form ada 5 field (Nama, Lokasi, Jenis
  ruang, Jumlah ruangan, Target mulai/selesai). Idealnya harus jauh lebih
  dari 10-15 detik kalau orang beneran isi form, bukan cuma buka-tutup.

## Riwayat data (buat lihat tren)

| Periode | Tampilan `/konsultasi/` | Pengguna bersih | Waktu engagement | Catatan |
|---|---|---|---|---|
| 28 Jul–24 Agu 2026 | 3 | ~1 (2 dari 3 kontaminasi pre-pause 16 Agu) | 7 dtk | Baseline sebelum fix copy |
| 25 Agu–16 Sep 2026 | 7 | 6 | 10 dtk | Setelah fix harga Rp100-300jt (commit `362a7e8`) + tambahan case study Park Regis |

**Cara isi baris baru**: ulangi langkah filter di atas dengan rentang
tanggal dari tanggal terakhir dicek sampai hari ini, catat 3 angka
(tampilan, pengguna, waktu engagement) untuk baris `/konsultasi/`, lalu
tambahkan baris ke tabel ini plus commit + push filenya.

## Keterbatasan yang perlu diingat

- **GA4 property BELUM ditautkan ke akun Google Ads** (dicek 25 Agu 2026,
  Admin > Penautan produk > Penautan Google Ads: 0 link di semua
  kategori). Jadi data konversi/atribusi Ads tidak muncul otomatis di
  GA4 — kalau butuh angka konversi Ads, itu tetap harus dicek langsung
  di `ads.google.com` (lihat keterbatasan akses di bawah), bukan dari GA4.
- **`ads.google.com` tidak bisa diakses konsisten oleh Claude in Chrome**
  — kadang jalan, kadang diblokir ("Navigation to this domain is not
  allowed"), sifatnya per-sesi bukan permanen. Kalau butuh data Ads,
  minta user screenshot manual (tabel Kampanye, kolom Konversi & Rasio
  konv.) daripada berulang kali coba akses otomatis.
- GA4 baru terpasang di landing page sejak **16 Agu 2026** — data sebelum
  itu tidak ada untuk landing page.

## Referensi

Detail lengkap campaign Google Ads, funnel, infra, dan histori keputusan
ada di [CLAUDE.md](CLAUDE.md) — buka itu juga kalau butuh konteks di luar
GA4 murni (misal mau tau kenapa harga Rp100-300jt di-scope ulang, atau
status webhook auto-deploy).
