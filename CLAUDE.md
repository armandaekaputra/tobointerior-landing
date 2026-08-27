# TOBO Interior — Landing Page

Landing page Astro untuk TOBO Interior, jasa renovasi interior kantor & ruang
komersial di Jakarta Selatan dan Tangerang Selatan. Dibuat untuk menampung
traffic dari campaign Google Ads.

## Stack & struktur

- **Astro 5** + **Tailwind CSS v4**, output static (`output: "static"`, tanpa
  adapter SSR — jangan tambahkan `sharp` atau image-optimization astro:assets,
  sudah pernah dicoba dan dihapus karena membebani build di VPS RAM kecil).
- Deploy via **Dockerfile** custom (build stage `node:22-alpine`, serve stage
  `nginx:alpine`) — bukan Nixpacks (Nixpacks terlalu lambat, pernah dicoba
  dan diganti).
- `nginx.conf` custom di root — pasang security header (HSTS,
  X-Content-Type-Options, X-Frame-Options, Referrer-Policy,
  Permissions-Policy, `server_tokens off`) dan `absolute_redirect off` (WAJIB
  ada — tanpa ini, redirect trailing-slash nginx men-downgrade HTTPS ke HTTP
  karena nginx di belakang Traefik/Coolify tidak tahu request asli HTTPS).

## Infrastruktur

- **Hosting**: VPS pribadi, dikelola lewat **Coolify** (self-hosted PaaS).
  RAM VPS kecil (~1.9GB) — hindari proses build yang berat.
- **Domain**: `landing.tobointerior.com` (subdomain). DNS dikelola di
  **Hostinger hPanel** (A record ke IP VPS). Domain utama `tobointerior.com`
  adalah situs terpisah (web utama, tidak dikelola di sesi/project ini).
- **Repo GitHub**: `armandaekaputra/tobointerior-landing` — **public**
  (dikonfirmasi 27 Agu 2026 via `gh api`, bukan private seperti catatan lama).
  (Sempat bernama `tobo-landing`, di-rename user; kalau ada referensi lama ke
  nama itu, itu histori, remote sudah diupdate ke nama baru.)
- Folder lokal: `/Users/arman/Documents/tobo-interior-lp` (di-rename dari
  `tobo-lp` oleh user — nama folder lokal tidak memengaruhi nama repo GitHub).
- **Auto-deploy webhook aktif sejak 27 Agu 2026** — Coolify → GitHub
  (`.../webhooks/source/github/events/manual`, event `push`). Sebelumnya
  toggle "Auto Deploy" di Coolify sudah nyala tapi TIDAK cukup — Coolify
  (non-GitHub-App integration) butuh webhook GitHub didaftarkan manual
  lewat repo Settings > Webhooks, pakai Payload URL & Secret dari halaman
  Coolify: Application > Configuration > Advanced > Webhooks. Gotcha:
  halaman GitHub Settings itu 404 kalau akun yang login bukan **Admin** di
  repo (collaborator biasa tidak cukup) — pastikan login sebagai
  `armandaekaputra` (owner), bukan akun lain. Verifikasi dari CLI:
  `gh api repos/armandaekaputra/tobointerior-landing/hooks`.

## Alur konversi (tanpa backend)

CTA "Diskusikan Rencana Proyek Anda" di homepage → `/konsultasi` (form: Nama,
Lokasi, Jenis ruang, Jumlah ruangan, Target mulai/selesai) → JS client-side
menyusun pesan dan redirect ke `wa.me/6281190000281?text=...` — **tidak ada
database atau backend**, semua di browser. User harus manual tekan "Kirim" di
WhatsApp (di luar kendali kita, keterbatasan bawaan wa.me).

## Tracking (di `src/layouts/Base.astro`)

- **Google Ads Conversion**: `AW-18371371265` / label `5UjdCObQ9tscEIHCk7hE`
  — fire di klik "Lanjutkan ke WhatsApp" ([konsultasi.astro](src/pages/konsultasi.astro)),
  pakai `event_callback` + fallback timeout 1 detik supaya redirect tidak
  balapan dengan pengiriman event.
- **GA4**: `G-46X8FFZG2Z` — stream baru "TOBO Interior Landing Page" yang
  ditambahkan ke properti GA4 **yang sudah ada** ("TOBO Interior - Situs
  Utama", isinya juga stream untuk `tobointerior.com`). Dipasang manual di
  kode (BUKAN lewat opsi auto-install Google) supaya tidak menimpa
  konfigurasi tag Ads yang sudah jalan.
- **Penting**: laporan GA4 level-properti menggabungkan SEMUA stream secara
  default. Untuk lihat data landing page saja, filter dimensi **"Nama Host"
  sama persis `landing.tobointerior.com`**.
- Akun Google Ads & GA4 ada di **tobocreative@gmail.com** — akun ini kelola
  banyak brand (TOBO Koding, TOBO Creative, TOBO Interior, dll), harus
  switch akun dulu via avatar kanan atas kalau browser default login ke akun
  lain (biasanya `tobokoding@gmail.com`).

## Google Ads campaign

Campaign: **"TOBO Interior - Renovasi Kantor Jaksel Tangsel"**
- Search only (Display Network & Search Partners dimatikan)
- Budget Rp70.000/hari, bidding "Klik" (Maximize Clicks) + batas CPC maks
  Rp10.000
- Target: Jakarta Selatan & Tangerang Selatan
- 13 keyword, 10 headline + 3 deskripsi RSA, 4 sitelink, negative keyword
  terpasang
- Budget bulanan disepakati Rp1,5–3 juta untuk fase validasi awal (4–6
  minggu), sebelum evaluasi scaling. Target: swasta (B2C/B2B kecil), BUKAN
  B2G — proyek institusi (UIN, Kemenag) datang dari relasi/tender, bukan
  dari iklan ini.

### Temuan performa

- **16 Agu 2026**: 61 klik, 1.481 tayangan, Rp594.407 terpakai, CTR ~4,1% —
  **0 konversi** (signifikan secara statistik untuk sample sebesar ini).
- Root cause: keyword `"desain interior kantor"` (phrase match) menyerap 77%
  klik & budget, berstatus "Skor Kualitas rendah" — kemungkinan menangkap
  traffic pencari inspirasi desain, bukan pencari kontraktor.
- **Sudah di-pause manual oleh user (16 Agu 2026)**.
- User sudah test end-to-end flow di HP asli — form, redirect, WA handoff
  semua lancar, jadi bukan bug teknis.
- GA4 baru terpasang di landing page pada 16 Agu 2026 — data historis
  sebelum tanggal itu TIDAK mencakup landing page.
- **22 Agu 2026**: laporan GA4 (belum difilter per-hostname, masih gabungan
  dengan `tobointerior.com`) untuk pertama kali menunjukkan `/konsultasi/`
  kena kunjungan (2 pengguna, 9 peristiwa) sejak keyword di-pause — sinyal
  awal yang positif, tapi **belum dikonfirmasi murni dari landing page**
  (perlu filter "Nama Host" = `landing.tobointerior.com` dulu) dan belum
  dicek apakah sampai jadi konversi WA beneran atau baru sekadar buka
  halaman form.
- **25 Agu 2026** — filter GA4 "Nama Host sama persis
  `landing.tobointerior.com`" sudah diterapkan (28 Jul–24 Agu 2026): total
  39 tampilan, 30 pengguna aktif murni dari landing page. `/` 36 tampilan
  (29 pengguna), `/konsultasi/` cuma **3 tampilan (3 pengguna)**, rata-rata
  waktu engagement di `/konsultasi/` **7 detik saja** (form ada 5 field —
  7 detik terlalu singkat untuk isi form beneran, indikasi bounce cepat).
  Breakdown harian ternyata: **2 dari 3 kunjungan terjadi Sabtu 15 Agu**,
  yaitu SEBELUM pause 16 Agu — jadi kemungkinan masih dari keyword
  `"desain interior kantor"` yang bermasalah, BUKAN bukti traffic bersih
  pasca-pause. Cuma **1 kunjungan (Senin 24 Agu)** yang benar terjadi
  setelah pause. Kesimpulan: sinyal "traffic naik pasca-pause" di catatan
  22 Agu ternyata lebih lemah dari yang terlihat — revisi urutan #2 di
  bawah paling penting sebelum ambil kesimpulan.
- **25 Agu 2026** — dicek juga Admin > Penautan produk > Penautan Google
  Ads di properti GA4: **0 link di semua kategori** (Selesai/Persetujuan
  diperlukan/Permintaan dikirim/Dibatalkan) — notifikasi "tautan
  dibatalkan" itu akurat, saat ini benar-benar tidak ada link aktif.
- **25 Agu 2026** — user screenshot manual tabel Kampanye di Google Ads
  (`ads.google.com` tetap diblokir untuk navigasi otomatis Claude, sudah
  dicoba ulang & tetap gagal dengan error "Navigation to this domain is
  not allowed"): periode 5–25 Agu 2026 (14 hari sebelum pause + 9 hari
  setelah), **Konversi 0,00 / Rasio konv. 0,00%**, 1.729 tayangan, 71
  klik, CTR 4,11%, Rp692.184 terpakai. **Dikonfirmasi masih 0 konversi
  total sejak pause**, sejalan dengan temuan GA4 di atas (traffic bersih
  pasca-pause minim & engagement rendah).

### Yang masih harus dilanjutkan di sesi berikutnya

1. ~~Terapkan filter GA4 "Nama Host sama persis
   `landing.tobointerior.com`"~~ — **selesai 25 Agu 2026**, lihat temuan di
   atas.
2. ~~Cek rasio konversi terbaru di Google Ads~~ — **selesai 25 Agu 2026**:
   dikonfirmasi masih 0,00 konversi sejak pause. Pertanyaan sekarang
   bergeser ke keputusan lanjutan: budget validasi awal (Rp1,5–3jt,
   4–6 minggu) — apakah lanjut jalan apa adanya, coba optimasi lain
   (keyword/creative/landing page), atau pause total dulu sambil evaluasi
   ulang funnel.
3. ~~Audit funnel/copy landing page~~ — **selesai 25 Agu 2026**: CTR iklan
   4,11% (sehat), tapi 0 konversi + engagement 7 detik di `/konsultasi/`
   menunjukkan kebocoran terjadi **setelah** klik, bukan di iklan. Root
   cause dugaan: 5 dari 6 portofolio di homepage adalah proyek institusi
   (UIN/Kemenag), plus angka Rp100–300 juta disebut eksplisit — kombinasi
   ini bisa membuat calon klien swasta kecil (target campaign: non-goverment,
   B2B & B2C personal renov rumah) mengira skala/harga tidak cocok untuk
   mereka, lalu bounce cepat. **Fix diterapkan 25 Agu 2026** (commit
   `362a7e8`): angka Rp100–300jt di [index.astro](src/pages/index.astro)
   sekarang di-scope eksplisit ke "proyek institusi tersebut" (bagian
   kredibilitas) dan ke "proyek yang mencakup beberapa ruangan sekaligus"
   (bagian cakupan layanan) — bukan lagi kesan patokan minimum umum.
   **User memilih tunggu beberapa hari dulu lihat efeknya** sebelum lanjut
   ke rekomendasi #2 (majukan bukti sosial dari proyek swasta kecil,
   belum dikerjakan) — cek lagi traffic/engagement `/konsultasi/` pasca
   25 Agu 2026 di sesi berikutnya.
4. Kalau masih 0 konversi meski traffic ke `/konsultasi/` sudah ada,
   curigai tahap terakhir funnel: form diisi tapi tombol "Lanjutkan ke
   WhatsApp" tidak diklik, atau WA terbuka tapi pesan tidak jadi dikirim.
5. Pertimbangkan link ulang GA4 property ke akun Google Ads — dikonfirmasi
   25 Agu 2026 statusnya benar-benar 0 link (bukan cuma notifikasi lama),
   jadi data atribusi Ads di GA4 saat ini nihil sampai ditautkan ulang.

## Keterbatasan alat yang sudah ketemu (biar tidak diulang cari tahu)

- **`ads.google.com` navigasi via Claude in Chrome tidak konsisten** — di
  sesi 25 Agu 2026, gagal terus-menerus dengan error "Navigation to this
  domain is not allowed" (sudah dicoba: tab baru, sesi baru, subagent
  terpisah, setelah ubah setting "Situs yang diizinkan" di app Claude,
  setelah restart koneksi ekstensi — semua gagal identik, dan terkonfirmasi
  request-nya tidak pernah sampai ke browser user sama sekali/servernya
  yang menolak). TAPI di sesi lain pada hari yang sama (`umrah-sendiri`,
  window & tab Chrome yang sama persis), akses ke `ads.google.com` **berhasil
  normal**. Penyebab pasti tidak ketemu — kemungkinan pembatasan level
  server yang di-scope per-sesi/percakapan, bukan per-project atau
  per-browser. Jangan asumsikan ini "diblokir total selamanya" — coba dulu
  di sesi baru sebelum langsung minta user screenshot manual.
- `analytics.google.com` bisa diakses normal via Claude in Chrome.
- `landing.tobointerior.com` (situs live) diblokir dari Browser pane bawaan
  (`mcp__Claude_Browser__*`) untuk read-tools — test perubahan lewat local
  dev server (`.claude/launch.json`, nama config `tobo-lp-dev`) dulu, baru
  verifikasi live via `curl`.
- Dev server lokal kadang nyangkut di port 4321 dari sesi sebelumnya — kalau
  perubahan kode tidak ke-reflect di browser tool padahal source sudah benar,
  cek `lsof -i :4321`, `kill -9 <pid>`, restart. Selalu cross-check dengan
  `curl -s http://localhost:4321/` langsung ke server sebelum curiga ada bug
  beneran (browser tool sering cache stale).
- Wizard "Buat kampanye tanpa panduan" di Google Ads bisa **kehilangan data
  sitelink** yang diisi di dalam dialog bersarang kalau user pindah halaman
  via sidebar sebelum konfirmasi penuh — lebih aman tambah sitelink lewat
  halaman **Aset** setelah campaign jadi, bukan di tengah wizard.

## Preferensi & gaya kerja user

- Komunikasi dalam **Bahasa Indonesia**.
- User cukup teknis (paham git, deploy, DNS) tapi mengandalkan panduan
  langkah-persis untuk UI pihak ketiga (Google Ads, GA4, Coolify) — kasih
  instruksi klik per klik, bukan cuma penjelasan konsep.
- Suka pendekatan "coba dulu, sambil jalan diperbaiki" — banyak keputusan
  diambil cepat lewat opsi berjenjang (AskUserQuestion) daripada diskusi
  panjang.
- Selalu commit + push tiap perubahan kode, lalu ingatkan user untuk
  **Redeploy manual di Coolify** — tidak ada auto-deploy.
