# Clai DS — Design System Aplikasi Musik Clai

**Versi:** 3.0 — 23 Oktober 2026. **Warna dikembalikan ke v1.0 "Volt"** atas vonis pemilik (palet Senja v2.0 "bikin pusing dan sakit mata"); struktur player-first, ikon SVG, dan daftar anti-slop dari v2.0 dipertahankan penuh. Yang dibuang dari v2 hanyalah dunianya (arang cokelat + amber), bukan rancangannya.
**Induk:** `SPESIFIKASI-PRODUK.md` Bab 7. Sumber kebenaran visual; papan visual pendamping: `design-board.html`.
**Metode:** dibangun ulang memakai skill **impeccable** (pbakaus/impeccable, Apache-2.0, terpasang di `~/workspace/custom-skills/impeccable/`) — khususnya `reference/craft-floor.md` (lantai kualitas + daftar tolak AI-slop), `reference/new-work.md`, dan `reference/android.md`. Standar pemilik yang sudah dikunci (spring physics) adalah brief yang menang bila bertabrakan dengan saran umum skill.
**Identitas:** nama kerja **Clai**, dunia visual **"Volt"** (v1, dipulihkan). Logo & ikon digambar sendiri. Dilarang memakai/meniru logo, ikon, atau aset Spotify/Spotube.

---

## 1. Dunia Visual: Volt

Satu kalimat penentu: **layar gelap netral yang tenang di mata, satu percikan volt yang menyala hanya di tempat yang penting.**

- Latar gelap netral (bukan cokelat, bukan biru) — vonis pemilik atas v2: permukaan berwarna di area luas membuat pusing dan sakit mata; warna hanya boleh jadi aksen kecil, bukan suasana seluruh layar.
- Satu aksen: **Volt #B8F135**. Kemiripan keluarga dengan hijau pemutar lain diterima sebagai harga kenyamanan mata pemilik; pembeda Clai diletakkan di struktur (player-first), tipografi, dan perilaku — bukan di hue.
- Tipografi adalah wajahnya: judul memakai huruf display berkarakter (Space Grotesk); sampul album tidak mendominasi layar sendirian.
- Struktur **player-first**: yang dijual aplikasi ini adalah pemutarnya, jadi Beranda menaruh pemutar & antrian di muka — bukan rak rekomendasi meniru toko.

Prinsip turunan:
1. **Netral & tenang, satu aksen.** Aksen muncul hemat — tombol putar, indikator aktif, progres — dan tidak pernah menjadi latar area luas.
2. **Gerak adalah fisika, bukan kurva.** Semua motion memakai spring (Bab 5). Tidak ada gerakan linear; tidak ada easing "benar tapi kaku". (Brief pemilik — menang atas saran motion umum.)
3. **Densitas seperti pemutar sungguhan.** Target sentuh ≥ 48dp meski baris terlihat ramping.
4. **Status selalu jujur.** Sedang diputar, buffering, antrian berikutnya, mode shuffle/repeat — terlihat tanpa membuka menu.
5. **Native Android dihormati** (impeccable `android.md`): komponen & pola Material (tombol, chip, switch, snackbar, bottom sheet), elevasi lewat tingkat tonal permukaan, satu FAB untuk satu aksi primer, snackbar untuk umpan balik sesaat — bukan toast/dialog refleks.
6. **Mockup terlihat seperti mockup.** Contoh konten di papan visual dilabeli "contoh".

## 2. Token Warna (Volt — dipulihkan dari v1.0)

| Token | Nilai | Pakai untuk |
|---|---|---|
| `bg/base` | #0A0A0E | Latar aplikasi |
| `bg/surface-1` | #121218 | Permukaan layar, kartu |
| `bg/surface-2` | #1B1B23 | Elemen di atas surface-1: baris aktif, chip terpilih, lembar menu, kartu Sedang Diputar |
| `bg/surface-3` | #26262F | Elemen tertinggi: dialog, tooltip |
| `ink/primary` | #F4F6F1 | Teks utama |
| `ink/secondary` | #A8AEA4 | Teks pendukung: artis, durasi, keterangan |
| `ink/faint` | #62665F | Teks paling redup: penanda, label nonaktif |
| `accent/main` | #B8F135 | **Volt** — tombol putar, indikator aktif, progres, judul sedang diputar |
| `accent/on-accent` | #10130A | Teks/ikon di atas accent — kontras ≥ 7:1 |
| `accent/pressed` | #9BD62A | Keadaan ditekan |
| `state/liked` | #FF5C7A | Hati tersimpan (satu-satunya warna semantik di luar aksen) |
| `state/error` | #FF6B5E | Gagal — selalu disertai teks, tidak hanya warna |
| `state/warning` | #FFB224 | Peringatan |
| `line/divider` | #FFFFFF pada 8% | Garis pemisah antar baris |
| `scrim` | #000000 pada 56% | Latar redup di belakang lembar/dialog |

Aturan aksen dinamis dari sampul (F2): hanya mewarnai latar tonal di balik sampul pada Player & kartu Sedang Diputar, diturunkan saturasi 20% dan wajib lolos kontras; tombol putar & indikator status selalu `accent/main`.

## 3. Tipografi

Dua wajah: **Space Grotesk** untuk display (judul layar, judul lagu di Player, angka besar) — berkarakter, geometris, hangat-teknis; **Inter** hanya untuk teks isi & label (sederhana, terbaca). Angka waktu tabular selalu.

| Gaya | Font | Ukuran/Garis | Berat | Pakai |
|---|---|---|---|---|
| `type/display` | Space Grotesk | 28/34 | 600 | Sapaan Beranda, judul lagu Player |
| `type/title` | Space Grotesk | 20/26 | 600 | Judul layar |
| `type/headline` | Inter | 17/23 | 600 | Judul seksi, judul lagu kartu |
| `type/body` | Inter | 15/21 | 400–500 | Judul baris lagu, isi |
| `type/caption` | Inter | 13/18 | 400 | Artis, keterangan |
| `type/micro` | Inter | 11/14 | 600 kapital | Label seksi fungsional di layar Antrian saja |
| `type/time` | Inter tabular | 12/16 | 500 | Waktu berjalan/sisa, durasi |

Judul panjang: elipsis satu baris di daftar; di Player boleh marquee setelah 3 detik diam.

## 4. Spasi, Radius, Elevasi

- **Grid 4dp:** token 4, 8, 12, 16, 20, 24, 32, 40, 48. Margin layar 16. Ruang di atas judul seksi lebih besar daripada di bawahnya (32 atas / 12 bawah) — ritme, bukan simetri malas.
- **Radius:** 8 (sampul mini, baris), 12–16 (kartu & sampul besar — rentang impeccable untuk kartu), 20 (lembar bawah), 999 hanya untuk kontrol kecil (chip, tombol putar, sakelar). Tidak ada pil raksasa dekoratif.
- **Elevasi = tonal, satu mekanisme:** kedalaman dinyatakan tingkat permukaan + paling banyak SATU bayangan lembut (offset nyata + blur lembut) pada elemen melayang (kartu Sedang Diputar, Mini Player, lembar). Dilarang menggabung garis tepi 1px DAN bayangan lebar pada elemen yang sama ("ghost card"). Dilarang bayangan offset keras tanpa blur dan halo warna tanpa offset.
- **Sampul:** 1:1; tidak tersedia → placeholder pola nada di atas surface-2 (bukan kotak kosong).

## 5. Motion — Parameter Spring (standar pemilik, tidak dapat ditawar)

Semua animasi spring fisik (massa 1); yang dianimasikan hanya `transform` dan `opacity` (+ blur/mask hanya untuk momen pahlawan bila tetap mulus). Spring berhenti saat setel (ambang kecepatan 0,001):

| Gerakan | Stiffness | Damping | Catatan |
|---|---|---|---|
| Kemunculan elemen daftar (berjenjang ≤ 30 ms/item) | 260 | 26 | Overshoot sangat tipis |
| Mini Player → Player (hero) | 180 | 24 | Momen pahlawan satu-satunya; sampul bergerak kontinu mengikuti drag |
| Lembar menu bawah | 300 | 28 | Cepat berhenti, tanpa pantulan |
| Tekan tombol (skala 0,96) | 400 | 22 | Umpan balik seketika |
| Ekualiser baris lagu | gelombang 3 batang, 900 ms | — | Berhenti saat jeda & di luar layar |
| Pergantian lagu | 200 | 26 | Sampul lama 0,94 + pudar, baru dari 1,04 |

Pengaturan sistem "hapus animasi" dihormati: spring → crossfade 80 ms, marquee mati.

## 6. Komponen Baku

### 6.1 Baris Lagu (SongRow)
[sampul 48] [judul body/500 + artis caption secondary] [durasi tabular] [⋯ SVG 48dp]. Keadaan: normal; **sedang diputar** (judul accent + ekualiser 3 batang di sudut sampul); ditekan (surface-2); tidak tersedia (faint + keterangan). Menu ⋯ = ContextSheet (6.6), isinya sama di semua layar.

### 6.2 Kartu Sedang Diputar (NowCard) — komponen tanda tangan Beranda
Blok surface-2 radius 16, TANPA garis tepi: sampul 96 kiri; kanan: judul headline, artis caption, garis progres 3dp accent + waktu tabular, deret kontrol ringkas (prev, PlayButton 48, next, hati). Mengetuk badan kartu membuka Player. Inilah hal pertama yang dilihat pengguna membuka aplikasi — bukan rak.

### 6.3 Baris Antrian Berikutnya (UpNextRow)
SongRow + nomor urut kecil tabular faint di kiri (urutan antrian memang informasi — pengecualian sah dari larangan nomor seksi).

### 6.4 Baris Koleksi (CollectionRow)
Baris lebar asimetris (bukan grid kartu seragam): mosaik/sampul 56, nama title-ish body 600, meta caption (jumlah lagu · terakhir diputar), chevron SVG. Playlist, album tersimpan, file lokal, dan unduhan semuanya baris ini dengan meta berbeda.

### 6.5 Chip Saring (FilterChip)
Pil 32, surface-2; terpilih accent + on-accent. Satu baris gulir.

### 6.6 Tombol Putar (PlayButton)
Lingkaran accent, ikon SVG on-accent: 40 mini / 48 kartu & halaman / 64 Player. Loading = cincin progres menggantikan ikon pada tombol yang sama.

### 6.7 Mini Player (MiniPlayer)
Tinggi 64 mengambang di atas nav (jarak 8), radius 12, surface-2 + bayangan tunggal. Isi: sampul 48, judul+artis, play/pause 40, next 40; progres 2dp accent di tepi atas. Geser atas = Player; geser samping = next/prev (ambang 64dp).

### 6.8 Lembar Menu Konteks (ContextSheet)
Material bottom sheet: radius 20 atas, pegangan seret, kepala konteks (sampul+judul+artis), aksi berikon SVG 24 + label, baris 52. Urutan baku: Putar Berikutnya · Tambah ke Antrian · Mulai Radio Lagu · Simpan ke Koleksi · Tambah ke Playlist · Unduh (F2) · Buka Album · Buka Artis · Bagikan.

### 6.9 Player (NowPlaying) — tipografi memimpin
Komposisi rata kiri (bukan simetris tengah): baris atas (tutup SVG, nama konteks sebagai teks caption biasa, menu) · sampul 176 rata kiri radius 16 · judul lagu **display 28 Space Grotesk** hingga 2 baris · artis headline secondary · seek + waktu tabular · deret kontrol: shuffle SVG · prev · **PlayButton 64** · next · repeat (aktif = accent + titik 4dp; repeat-satu + lencana "1") · deret bawah: Lirik · Antrian · Perangkat (F3) sebagai tombol teks berikon SVG.

### 6.10 Baris Antrian (QueueRow)
SongRow + pegangan seret SVG (6 titik) di kanan; seksi dibedakan label micro fungsional: "SEDANG DIPUTAR" / "BERIKUTNYA" / "LANJUTAN — [konteks]" / "PUTAR OTOMATIS — KARENA KAMU MEMUTAR [judul]". Seksi Putar Otomatis: kepala seksi + sakelar Material (matikan untuk sesi ini); barisnya berjudul `ink/secondary` + lencana mikro "AUTO" accent; tanpa pegangan seret, geser-hapus tetap bisa.

### 6.11 Navigasi Bawah (BottomNav)
4 tujuan: Beranda · Cari · Koleksi · Statistik (F3). Ikon SVG garis 1,8 (digambar sendiri, satu gaya), label micro; aktif = ink/primary + titik accent 4dp; latar bg/base 94%.

### 6.12 Keadaan Sistemik
- **Skeleton:** blok surface-2 berdenyut (opacity 0,6↔1, 1200 ms) seukuran konten akhir — bukan spinner layar penuh; animasi berhenti di luar layar.
- **Kosong:** ikon besar redup + satu kalimat + **satu** tombol aksi jelas.
- **Gagal:** pesan di tempat aksi terjadi + "Coba lagi" di posisi sama; lagu gagal dilewati dengan pemberitahuan menyebut judulnya (spec produk 5.4); umpan balik sesaat memakai **snackbar** Material.
- **FAB:** maksimal satu per layar, hanya aksi primer layar itu (mis. "+" buat playlist di Koleksi).

### 6.13 Ikon
Set vektor SVG digambar sendiri, kotak 24, garis 1,8, ujung membulat: putar, jeda, next, prev, acak, ulang, ulang-1, hati, hati-penuh, antrian, lirik, unduh, selesai-unduh, cari, beranda, koleksi, statistik, tambah, centang, titik-tiga vertikal, pegangan-seret, perangkat, timer-tidur, ekualiser, tutup, chevron-kanan, bagikan, daftar-putar, album, artis, berkas-lokal, peringatan. **Dilarang memakai glyph Unicode/emoji sebagai ikon** (temuan v1: ⌂ ⌕ ♥ ⠿ — dibuang).

## 7. Pola Layar

1. **Beranda (player-first):** sapaan display ringkas → **NowCard** → "Antrian Berikutnya" (3 UpNextRow + tautan Buka Antrian) → "Koleksi Kamu" (CollectionRow) → "Terakhir Diputar" sebagai baris lagu. Tidak ada rak geser rekomendasi; penemuan ada di Cari.
2. **Cari:** kolom cari lengket + chip kategori; hasil bertahap; rak penemuan sumber tinggal di sini.
3. **Koleksi:** chip tab + cari lokal + CollectionRow; satu FAB "+".
4. **Album/Artis/Playlist:** kepala sampul + PlayButton 48 menimpa tepi bawah kepala; daftar lagu.
5. **Player:** sesuai 6.9. **Antrian:** sesuai 6.10 dengan kepala lagu berjalan tetap.
6. **Lirik (F2):** layar penuh surface-1; baris aktif headline ink/primary, lain faint; ketuk baris = seek.

## 8. Daftar Tolak AI-Slop (dari impeccable craft-floor, difilter untuk aplikasi ini)

- Teks gradasi; kaca/blur dekoratif (glassmorphism); halo cahaya tanpa offset.
- Grid kartu seragam ikon+judul+teks sebagai struktur halaman; kartu di dalam kartu.
- Label "eyebrow/kicker" kecil di atas judul-judul seksi pada permukaan promosi/dokumen (judul berdiri sendiri); nomor seksi 01/02/03 kecuali urutan sungguh informasi (antrian boleh).
- Garis aksen tepi (border-left/right > 1px) pada kartu/baris sebagai dekorasi.
- Bayangan offset keras tanpa blur di luar dunia neobrutalist (bukan dunia kita).
- Ikon glyph Unicode/emoji; ikon dari dua set berbeda dalam satu layar.
- Font display bawaan sistem sebagai suara display; semua teks satu font tanpa hierarki wajah.
- Teks sekunder abu-abu netral di atas permukaan berwarna (warnai dari hue-nya).
- Animasi masuk yang sama di semua seksi; animasi properti layout.
- **Do:** satu aksen per layar; status putar selalu terlihat; target ≥ 48dp; angka waktu tabular; label semantik TalkBack untuk semua tombol ikon; keadaan nonaktif tidak hanya lewat warna; kontras teks isi ≥ 4,5:1 & teks besar ≥ 3:1 (lantai impeccable); placeholder sampul berpola; ubahan token lewat dokumen ini dulu (Bab 9).

## 9. Keterlacakan

Token = konstanta kode (satu berkas token). Mengubah token/identitas = ubah dokumen ini → kode → papan visual. Riwayat: v1.0 "Volt" (#B8F135, rak ala Spotify) dinilai mirip Spotube oleh pemilik 23 Okt 2026; v2.0 "Senja" (arang cokelat + amber, player-first) DITOLAK pemilik di hari yang sama — "warna ini bikin pusing dan sakit mata", v1 dinilai lebih bagus; v3.0 ini memulihkan palet Volt v1.0 dan mempertahankan seluruh rancangan struktural v2.0 (player-first, NowCard, ikon SVG, anti-slop). Pelajaran tetap: kenyamanan mata pemilik mengalahkan keunikan hue.
