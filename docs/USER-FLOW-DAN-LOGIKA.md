# USER FLOW & LOGIC MAPPING — Clai

**Dokumen Phase 2** dari proses 5 fase (lihat `CARA-KERJA.md`). **Versi:** 1.0 — 23 Oktober 2026.
**Induk:** `SPESIFIKASI-PRODUK.md` v2.0 (Phase 1, tertutup). Dokumen ini tidak mengubah keputusan PRD; ia memetakan alur, logika, dan keadaan tepi yang PRD tetapkan sebagai fitur. Konflik antara dokumen ini dan PRD diselesaikan untuk PRD, dan dicatat sebagai temuan.
**Konvensi penomoran:** layar `S-xx`, alur `F-xx`, aturan logika `L-xx`, keadaan `MS` (missing states). Penanda fase (F1–F4) mengikuti PRD Bab 9.

---

## 1. Arsitektur Informasi

### 1.1 Struktur navigasi

Navigasi utama: **4 tab bawah** — Beranda, Cari, Koleksi, Statistik. Di atas semua tab, **Mini Player persisten** (S-20) menempel di atas bilah tab; Player Penuh (S-21), Antrian (S-22), dan Lirik (S-23) adalah lapisan di atas segalanya (bukan tab). Pengaturan (S-30) dijangkau dari Beranda & Koleksi. Mode Berkendara (S-40) adalah permukaan pengganti penuh saat aktif.

Aturan struktur:
- Tidak ada layar yang terkubur lebih dari **3 ketuk** dari tab asalnya (kecuali Pengaturan turunan).
- Halaman Konten (album/artis/playlist) selalu berupa tumpukan (stack) di atas tab asal; tombol kembali mengembalikan ke tab asal dengan posisi gulir utuh.
- Konteks putar **tidak** terikat navigasi: berpindah tab/layar tidak pernah mengubah apa yang sedang diputar (L-01).

### 1.2 Inventaris layar

| ID | Layar | Isi inti | Masuk dari | Fase |
|---|---|---|---|---|
| S-01 | Beranda | NowCard (sedang diputar), Antrian Berikutnya (pratinjau), kartu Lanjut Besok (bila ada), baris koleksi, rak Terakhir Diputar/Putar Lagi | Tab; mulai dingin | F1 (personal F2) |
| S-02 | Cari | Kotak cari, riwayat cari, jelajah kategori/rak penemuan sumber | Tab | F1 |
| S-03 | Hasil Cari | Hasil per kategori (Lagu/Album/Artis/Playlist) + chip saring | S-02 | F1 |
| S-04 | Koleksi | Tab internal: Playlist, Album, Artis, Lagu Tersimpan, File Lokal, Unduhan | Tab | F1 (Unduhan F2) |
| S-05 | Album | Sampul, daftar lagu, aksi konten | S-01/02/03/04, menu konteks | F1 |
| S-06 | Artis | Lagu populer, diskografi, artis terkait | S-01/02/03/04, menu konteks | F1 |
| S-07 | Playlist Sumber | Daftar lagu + keterangan pemilik | S-02/03/04 | F1 |
| S-08 | Playlist Pengguna | Daftar lagu dapat disusun, mosaik sampul | S-04 | F1 |
| S-09 | Lagu Tersimpan | Daftar liked | S-04 | F1 |
| S-10 | File Lokal | Hasil pindai folder musik perangkat | S-04 | F1 |
| S-11 | Unduhan | Daftar unduhan + pemakaian penyimpanan | S-04 | F2 |
| S-12 | Statistik | Ringkasan dengar lokal, Wrapped lokal, riwayat | Tab | F3 |
| S-13 | Hasil Tangkap | Daftar hasil pencocokan dari berbagi masuk (per item: ditemukan/tidak) | Intent berbagi sistem | F2 |
| S-20 | Mini Player | Sampul, judul/artis, putar/jeda, berikutnya | Persisten | F1 |
| S-21 | Player Penuh | Sampul, seek, kontrol lengkap, akses S-22/S-23 | S-20, S-01 (NowCard) | F1 |
| S-22 | Antrian | Sedang Diputar, Berikutnya, Lanjutan konteks, seksi Putar Otomatis | S-21, S-01 | F1 |
| S-23 | Lirik | Lirik sinkron / polos / status tanpa lirik | S-21 | F2 |
| S-30 | Pengaturan | Kualitas, Perilaku Putar, Mode Berkendara, Data & Penyimpanan, Bahasa, Tentang | S-01, S-04 | F1–F2 |
| S-31 | Pengaturan Mode Berkendara | Pemicu, skema kendali, pengumuman, kalibrasi Volume Ikut Kecepatan | S-30 | F2 |
| S-40 | Mode Berkendara (permukaan) | Kontrol raksasa / status audio-first | Pemicu Bab 5.14 PRD | F2 |
| S-50 | Lembar Menu Konteks | Aksi per lagu/konten (dipakai ulang di semua daftar) | Tekan lama / tombol ⋯ | F1 |
| S-51 | Lembar Bagikan/Kelola | Aksi konten (simpan, unduh, ekspor) | S-05/06/07/08 | F1–F3 |

### 1.3 Titik masuk eksternal (bukan dari dalam aplikasi)

| Titik masuk | Tujuan | Aturan |
|---|---|---|
| Notifikasi media & lock screen | S-21 (ketuk) / kontrol langsung | Ketuk notifikasi membuka S-21 di atas layar kunci sesuai izin sistem; kontrol tidak membuka UI |
| Tombol perangkat audio (BT/kabel) | Mesin putar | Diproses global, termasuk saat aplikasi di latar & layar mati (L-14) |
| Intent berbagi (teks/tautan/gambar) | S-13 | Aplikasi tidak perlu sedang berjalan; hasil tangkap tidak pernah memutar massal tanpa tampil di S-13 dulu (PRD 5.14 C) |
| Tautan sumber (YouTube Music dsb.) dibuka dengan Clai | Konten setara di S-05/06/07 atau putar langsung | Tautan yang tidak dikenali jatuh ke S-13 sebagai teks |
| Perintah suara sistem di Mode Berkendara | Antrean Suara (F-08) | Hanya aktif penuh di Mode Berkendara; di luar itu lewat tombol mikrofon S-21/S-22 |


---

## 2. Mesin Status Putar Global

Satu-satunya sumber kebenaran status putar. UI (S-20/21/22, notifikasi, Mode Berkendara) adalah cermin; tidak ada status lokal per layar.

### 2.1 Status & transisi

| Dari | Kejadian | Ke | Catatan logika |
|---|---|---|---|
| idle | Putar konteks/lagu | loading | Konteks & benih sesi ditetapkan atomik sebelum loading (L-03) |
| loading | Aliran siap | playing | Gerbang "tekan→bunyi" PRD Bab 3 diukur di transisi ini |
| loading | Gagal (URL mati/jaringan) | error → pemulihan | Masuk prosedur pemulihan L-10; UI tidak menampilkan error mentah |
| playing | Jeda (pengguna/fokus hilang) | paused | Posisi disimpan persisten setiap 5 detik & saat transisi (L-16) |
| paused | Putar | playing | Tanpa memuat ulang aliran bila URL belum kedaluwarsa |
| playing | Data kurang (jaringan memburuk) | buffering | > 5 detik: status jujur + opsi lewati tampil (PRD 5.4) |
| buffering | Data cukup | playing | Otomatis, tanpa intervensi |
| buffering | > 20 detik tanpa kemajuan | error → pemulihan | Lewati otomatis per L-10 setelah upaya pemulihan habis |
| playing | Lagu selesai | loading (lagu berikut) | Urutan ditentukan mesin antrean Bab 3 dokumen ini |
| playing/paused | Konteks habis & Putar Otomatis mati | ended | UI menyatakan berhenti karena konteks habis (tidak diam-diam) |
| status apa pun | Proses dibunuh sistem | — | Saat hidup lagi: pulihkan antrean, konteks, indeks, posisi terakhir (L-16) |

### 2.2 Matriks interupsi (audio focus & perangkat)

| Kejadian | Perilaku wajib |
|---|---|
| Panggilan masuk | Jeda; posisi disimpan. Lanjut otomatis setelah panggilan **hanya bila** pengaturan lanjut-otomatis aktif (bawaan: aktif) dan jeda < 30 menit |
| Audio lain mengambil fokus permanen (aplikasi lain memutar media) | Jeda; tidak merebut kembali fokus tanpa aksi pengguna |
| Audio lain mengambil fokus sementara (navigasi suara, asisten) | Turunkan volume (duck) atau jeda singkat sesuai pengaturan fokus; pulihkan volume sesudahnya tanpa lonjakan (ramp ≥ 300 ms) |
| Headset/BT terputus saat playing | Jeda seketika (tidak pernah beralih meledak ke speaker); status paused dengan alasan "perangkat terputus" |
| Headset/BT tersambung saat paused | Tidak auto-play dari sambungan saja (kecuali di Mode Berkendara dengan pemicu Bluetooth kendaraan yang pengguna tetapkan — PRD 5.14 A) |
| Alarm/timer sistem berbunyi | Duck selama bunyi sistem, pulih sesudahnya |
| Notifikasi biasa | Tidak menginterupsi audio (kecuali pengumuman Mode Berkendara yang pengguna aktifkan sendiri) |

---

## 3. Logika Antrean (aturan mengikat semua alur)

- **L-01** Antrean dimiliki mesin player, bukan layar. Navigasi tidak mengubahnya.
- **L-02** Struktur antrean selalu empat bagian berurutan: Sedang Diputar → Berikutnya (manual) → Lanjutan (sisa konteks) → Putar Otomatis (bila aktif & konteks menipis). S-22 menampilkan persis struktur ini.
- **L-03** Memutar konteks baru mengganti Lanjutan & benih sesi secara atomik; Berikutnya (manual) **dipertahankan** kecuali pengguna memilih "putar & ganti antrean" dari menu konteks. (Keputusan tertutup: manual adalah milik pengguna, tidak dibuang diam-diam.)
- **L-04** "Tambah ke Antrian" menyisip di akhir bagian Berikutnya. "Putar Berikutnya" menyisip di awal bagian Berikutnya.
- **L-05** Isi ulang Putar Otomatis dipicu saat total sisa (Berikutnya+Lanjutan+Otomatis) < 5, mengambil ±15 kandidat dari Tangga Kemiripan (PRD 5.4) terhadap benih sesi + sinyal sesi. Isi ulang berjalan di latar dan **tidak pernah mengubah 3 lagu terdekat berikutnya** yang sudah tampil (mencegah daftar melompat di depan mata pengguna).
- **L-06** Menghapus lagu dari seksi Putar Otomatis = sinyal negatif sesi untuk lagu itu (tidak ditawarkan lagi sesi ini). Menghapus dari Berikutnya/Lanjutan tidak berpengaruh ke mesin kemiripan.
- **L-07** Skip oleh pengguna dicatat sebagai sinyal negatif sesi setelah lagu diputar < 30 detik atau di-skip eksplisit; lagu yang selesai ≥ 80% adalah sinyal positif untuk benih sesi (PRD 5.4).
- **L-08** Race antrean: semua mutasi antrean (pengguna, isi ulang otomatis, Connect remote, perintah suara) masuk **satu antrean operasi serial** pada mesin player. Tidak ada dua penulis bersamaan; UI optimistis diperbolehkan hanya untuk geser-urut visual dengan indeks versi antrean — bila versi berubah di tengah geser, geser dibatalkan dan daftar dirender ulang (tidak ada data hilang).
- **L-09** Repeat-Satu mengalahkan segalanya; Repeat-Semua mengulang Lanjutan sebelum Putar Otomatis; konteks satu lagu + Repeat-Semua = diperlakukan sebagai Putar Otomatis (PRD 5.4).
- **L-10** Pemulihan lagu gagal: segarkan URL (maks 2 upaya: proaktif sebelum kedaluwarsa, reaktif saat gagal) → bila tetap gagal, lewati otomatis + pemberitahuan menyebut judul → 3 kegagalan berurutan menghentikan pemulihan otomatis dan menampilkan status jujur dengan tombol coba lagi (mencegah loop lewati tanpa akhir saat sumber mati total).
- **L-11** Putar Otomatis **offline**: Tangga Kemiripan dijalankan atas katalog lokal (unduhan + file lokal + metadata tersimpan). Bila kandidat lokal < 5, seksi Putar Otomatis ditandai "terbatas — offline" di S-22 dan musik berhenti secara eksplisit di ujungnya; tidak ada upaya jaringan diam-diam.


---

## 4. Alur Inti

Format tiap alur: langkah bernomor; `◆` = titik keputusan; `✖` = cabang gagal dengan perilaku wajibnya.

### F-01 — Mulai dingin & putar pertama (cold start, pengguna baru)

1. Aplikasi dibuka → S-01. Mesin putar idle; antrean kosong dipulihkan bila ada sisa sesi (L-16), bila tidak S-01 menampilkan status kosong yang dirancang (ajakan mencari lagu pertama), bukan rak kosong.
2. Pengguna mengetuk Cari (S-02) atau rak penemuan → mengetik → S-03.
3. Ketuk lagu → konteks = hasil cari (satu lagu) → loading → playing. **Benih sesi = lagu itu.**
4. Lagu selesai → konteks habis → Putar Otomatis menyambung dari Tangga Kemiripan (tanpa riwayat personal — tangga 1–4, PRD 5.4). Pengguna baru menerima nilai penuh sejak lagu pertama.
✖ Jaringan mati saat langkah 2–3: S-03 menampilkan status gagal-jaringan (dibedakan dari "tidak ditemukan"), S-01 menawarkan File Lokal & Unduhan sebagai jalur alternatif yang tetap berfungsi.
✖ Lagu pertama gagal diputar: L-10 berlaku; bila sumber mati total, status jujur + saran periksa koneksi/sumber, bukan spinner abadi.

### F-02 — Pencarian → putar → sambungan otomatis (alur harian utama)

1. S-02: ketik dengan debounce 300 ms; hasil bertahap. Riwayat cari lokal tampil saat kotak kosong & fokus.
2. Saring per kategori (chip). Ketuk lagu memulai konteks pencarian; ketuk album/artis/playlist membuka S-05/06/07.
3. Selama putar: S-22 selalu dapat dibuka untuk melihat Berikutnya/Lanjutan/Putar Otomatis & asal sambungan ("karena kamu memutar X").
4. ◆ Pengguna menambah manual (L-04) → manual selalu mendahului sambungan otomatis. ◆ Pengguna skip → L-07.
✖ Hasil kosong untuk kueri: status "tidak ditemukan" + saran ejaan/kueri lain; berbeda visual & teks dari gagal jaringan.

### F-03 — Kelola antrean

1. S-22: geser urut di Berikutnya & Lanjutan (pegangan seret); geser-samping untuk hapus; snackbar urung 5 detik.
2. "Kosongkan antrean" mengosongkan Berikutnya+Lanjutan+Otomatis, mempertahankan Sedang Diputar; konfirmasi sekali (aksi massal).
3. Sakelar seksi Putar Otomatis: mati untuk sesi ini → seksi hilang, perilaku berhenti-di-ujung dinyatakan di S-22.
✖ Race: isi ulang otomatis tiba saat pengguna menyeret → L-08 (geser dibatalkan aman, render ulang).
✖ Pengguna menghapus lagu yang sedang loading sebagai berikutnya → mesin membatalkan muat itu dan mengambil kandidat sesudahnya; tidak ada lagu hantu.

### F-04 — Jelajah konten → putar konteks

1. S-05/06/07: Putar = mulai dari lagu pertama; Acak = konteks yang sama dengan shuffle aktif; ketuk lagu = mulai konteks dari lagu itu (yang di atasnya menjadi "sudah lewat", dapat di-prev).
2. Simpan ke koleksi / Unduh (F2) dari lembar S-51 tanpa meninggalkan halaman.
✖ Konten sumber tidak lagi tersedia (dihapus/diblokir wilayah): halaman menampilkan status "tidak tersedia" dengan alasannya bila sumber memberi tahu; lagu individual yang mati di dalam daftar ditandai, tidak memblokir putar lagu lain (putar melompati yang mati per L-10).

### F-05 — Suka, simpan & playlist pengguna

1. Like dari S-50, S-21, notifikasi, atau baris lagu mana pun → masuk S-09 (Liked, playlist sistem tak terhapus).
2. Tambah ke Playlist → pemilih playlist (atau buat baru di tempat) → duplikat ditanya eksplisit: "tetap tambahkan / lewati" (PRD 5.7).
3. S-08: susun ulang seret, hapus geser + urung.
✖ Like saat offline: tetap tercatat lokal seketika (data pengguna offline-first, PRD 4.1); tidak ada operasi suka yang menunggu jaringan sama sekali.

### F-06 — Unduhan & putar offline (F2)

1. Unduh dari S-51 (lagu/album/playlist) → antrean unduhan S-11: jeda/lanjut/batal per item & global; progres jujur (byte, bukan persen palsu).
2. Selesai → item tersedia di S-11 & ditandai di daftar lagu mana pun; putar memilih berkas lokal lebih dulu daripada aliran (L-12 di Bab 6).
3. Mode "hanya unduhan" (S-30): pencarian & rak dibatasi ke yang tersedia offline, ditandai jelas agar pengguna tidak mengira katalog menyusut.
✖ Penyimpanan penuh di tengah unduhan: unduhan terjeda dengan alasan; item selesai tidak ikut terhapus; tawaran kelola penyimpanan (S-11) dengan batas kuota (PRD 5.9).
✖ Berkas unduhan korup: verifikasi integritas gagal → unduh ulang otomatis sekali → gagal lagi: item ditandai rusak dengan aksi unduh ulang manual; tidak pernah diputar setengah.

### F-07 — Mode Berkendara (F2)

1. ◆ Pemicu: Bluetooth perangkat kendaraan yang ditandai tersambung / deteksi aktivitas / sakelar manual (S-31). Masuk mode selalu diumumkan visual (dan suara bila pengumuman aktif) — tidak pernah diam-diam.
2. S-40 aktif: permukaan raksasa bila layar terlihat; audio-first bila tidak. Skema kendali dipilih dari kemampuan perangkat audio yang tersambung (PRD 5.14 A) dan ditampilkan ringkas saat mode mulai ("Ketuk 2x = berikutnya, tahan = suara").
3. Keluar mode: sakelar manual, Bluetooth kendaraan terputus > 60 detik, atau tiba & berhenti > 5 menit dengan konfirmasi. Keluar tidak menghentikan musik.
✖ Perangkat audio berganti di tengah mode (BT helm mati, beralih speaker HP): jeda seketika (matriks 2.2), skema kendali dievaluasi ulang saat perangkat baru tersambung; mode tetap aktif.
✖ GPS tidak tersedia/akurasi buruk saat Volume Ikut Kecepatan aktif: volume membeku pada nilai terakhir (gagal-aman PRD 5.14 A); tidak ada tebakan.

### F-08 — Antrean Suara (F2; di dalam & di luar Mode Berkendara)

1. Pemicu: tahan tombol perangkat audio (di Mode Berkendara) atau tombol mikrofon S-21/S-22 → nada siap → pengguna berbicara (id/en).
2. Pengenal suara → teks + skor keyakinan → pengurai aturan mengekstrak {perintah, kata kunci}.
◆ Keyakinan rendah / perintah tak dikenali: minta ulangi (maks 2x) → menyerah sopan dengan earcon gagal; antrean tidak berubah.
3. Kata kunci dicari → kandidat diskor → ◆ pemenang jelas: masuk antrean (L-04 sebagai Putar Berikutnya, atau Putar untuk perintah "putar") + konfirmasi suara menyebut judul. ◆ Dua kandidat dekat: pertanyaan suara ("Yang A atau B?") → jawaban didengar sekali → ◆ tidak dijawab/ambigu lagi: batalkan, jangan menebak.
✖ Offline: pengenal suara perangkat tidak tersedia → dinyatakan ("perintah suara butuh koneksi di perangkat ini"), antrean tidak berubah.
✖ Musik sedang keras: volume di-duck selama mendengarkan agar pengenal mendengar pengguna, dipulihkan sesudahnya (ramp matriks 2.2).

### F-09 — Tangkap dari berbagi (F2)

1. Pengguna membagikan teks/tautan/gambar ke Clai (dari aplikasi lain) → S-13.
2. Tautan sumber dikenali → item langsung terpetakan. Teks bebas → diperlakukan kata kunci per baris/item. Gambar → OCR di perangkat → baris-baris teks.
3. S-13 menampilkan per item: cocok (dengan lagu terpilihnya) / tidak ditemukan. Pengguna memilih "Putar semua yang cocok" / "Antrekan" / per item.
✖ Item tidak cocok tidak menghalangi item lain; tidak ada putar massal tanpa layar S-13 (PRD 5.14 C).
✖ Gambar tidak terbaca OCR: dinyatakan sebagai gagal-baca dengan saran kirim sebagai teks; bukan daftar kosong.

### F-10 — Tidur & Lanjut Besok (F2)

1. Sleep timer diatur (menit / akhir lagu) dari S-21/S-30 → volume memudar di 30 detik terakhir → berhenti; posisi tidur direkam (lagu, detik, konteks, antrean).
2. ◆ Pengguna berinteraksi lagi sebelum "tertidur" (memutar/mengubah apa pun setelah timer habis dalam 10 menit): posisi tidur dibatalkan (ia ternyata terjaga).
3. Sesi berikutnya: kartu Lanjut Besok di S-01 → ketuk: konteks, antrean, dan posisi dipulihkan persis; kartu hilang setelah dipakai atau 48 jam.
✖ Kartu kedaluwarsa / antrean asal sudah tidak tersedia sebagian (lagu sumber mati): pulihkan yang tersedia, lewati yang mati per L-10, nyatakan rinciannya di S-22.

### F-11 — File lokal (F1)

1. Izin media diberikan → pindai folder musik → S-10; metadata dari berkas (tag) dengan sampul tertanam bila ada.
2. File lokal masuk antrean/playlist campuran setara streaming (PRD 5.6); di Tangga Kemiripan, file lokal adalah kandidat sah (tangga berdasarkan metadata artis/judul lokal).
✖ Izin ditolak: S-10 menjelaskan kebutuhan izin + tombol minta lagi; tidak ada pindai diam-diam. Berkas terhapus dari perangkat: entri ditandai hilang saat diputar & ditawarkan dibersihkan dari daftar, tidak error mentah.

### F-12 — Data milik pengguna: ekspor/impor (F3) & pengaturan

1. Ekspor koleksi (S-30 → Data): satu berkas JSON (skema berversi — dirinci Phase 3) berisi playlist, liked, riwayat opsional.
2. Impor: berkas dibaca → ringkasan perubahan ditampilkan sebelum diterapkan (tambah/gabung/ganti) → pengguna mengonfirmasi → diterapkan atomik (gagal di tengah = tidak ada perubahan sebagian).
✖ Berkas versi lebih baru dari aplikasi: impor ditolak dengan penjelasan versi; tidak ada impor paksa.

### F-13 — Connect sebagai remote (F3)

1. Pasangan perangkat (kode di jaringan lokal) → perangkat remote melihat antrean & status yang sama (cermin mesin yang sama — Bab 2).
2. Aksi remote (putar/jeda/lewati/ubah antrean/volume) masuk antrean operasi serial yang sama (L-08) — remote dan lokal tidak pernah bertabrakan.
✖ Koneksi putus: remote menunjukkan status terputus; pemutar tetap memutar mandiri; tidak ada aksi yang menggantung.

### F-14 — Podcast & episode (F4)

1. Episode diputar dari halaman acara; posisi per episode tersimpan; selesai menandai episode & menawarkan episode berikut acara yang sama (bukan Putar Otomatis musik).
2. Kecepatan putar & lewati hening berlaku per jenis konten episode, tidak mengubah pengaturan musik.


---

## 5. Matriks Missing-States per Layar

Setiap layar wajib mengimplementasikan semua baris yang berlaku untuknya. "—" berarti keadaan tidak mungkin terjadi di layar itu (dengan alasan). Teks status memakai kunci string i18n (Bab 4.3 PRD), tidak tertanam.

| Layar | Memuat | Kosong | Gagal jaringan | Gagal sumber | Offline | Sebagian | Race / konflik |
|---|---|---|---|---|---|---|---|
| S-01 Beranda | Rangka kartu (skeleton) berbentuk NowCard & baris, bukan spinner tengah | Status kosong terancang + ajakan cari (pengguna baru) | Rak sumber gagal → rak lokal tetap tampil + penanda rak gagal dengan coba-lagi | Sama seperti jaringan, teks dibedakan ("sumber bermasalah") | Mode offline nyata: hanya NowCard, unduhan, lokal, Lanjut Besok | Rak tampil bertahap per rak; rak lambat tidak menahan rak lain | Sesi dipulihkan (L-16) menimpa status kosong — urutan: pulihkan dulu, render kemudian |
| S-02 Cari | Riwayat tampil seketika (lokal) | Riwayat kosong: tampilkan rak penemuan | Rak penemuan gagal → riwayat tetap ada + penanda | — | Penemuan disembunyikan; riwayat & koleksi lokal tetap dapat dicari | — | — |
| S-03 Hasil Cari | Hasil bertahap per kategori; kategori selesai duluan tampil duluan | "Tidak ditemukan" ≠ gagal (teks & ikon berbeda) | Status gagal + coba lagi; kueri dipertahankan | Status sumber + coba lagi | Pencarian dibatasi katalog lokal & dinyatakan | Sebagian kategori gagal: yang berhasil tampil, yang gagal berpenanda per bagian | Kueri berubah saat hasil lama mengalir: hasil kueri lama dibuang (token kueri, L-13) |
| S-04 Koleksi | Lokal seketika (tidak ada muat jaringan untuk data sendiri) | Per tab: status kosong per jenis + cara mengisi | — (data lokal) | — | Normal penuh (offline-first) | File lokal masih memindai: penanda progres pindai | — |
| S-05/06/07 Konten | Skeleton daftar lagu | Daftar kosong dari sumber: status "belum ada lagu" | Gagal + coba lagi; metadata tersimpan (cache) ditampilkan dengan penanda basi bila ada | "Tidak tersedia" + alasan sumber bila diberi | Hanya bila metadata & lagu ter-cache/unduh; selain itu status offline jujur | Sebagian lagu mati ditandai per baris (F-04) | Putar dimulai saat daftar berubah dari sumber: konteks memakai snapshot daftar saat ketuk (L-15) |
| S-08 Playlist Pengguna | Lokal seketika | Playlist kosong: ajakan tambah dari menu konteks | — | — | Normal penuh | — | Diedit dari Connect remote bersamaan: operasi serial L-08, versi daftar |
| S-11 Unduhan | Daftar + progres nyata per item | Belum ada unduhan: penjelasan + cara unduh | Unduhan aktif terjeda-otomatis, penanda menunggu jaringan | Item sumber mati: ditandai, antrean lanjut ke item lain | Normal (justru habitatnya) | Penyimpanan penuh (F-06) | Item diunduh sedang diputar dari aliran: unduhan selesai mengalihkan sumber ke lokal tanpa jeda dengar (L-12) |
| S-12 Statistik | Dihitung lokal; progres hitung untuk riwayat besar | Riwayat kosong: "belum ada data" + mulai terbentuk setelah putar pertama | — | — | Normal penuh | Periode tanpa data ditampilkan sebagai nol jujur, bukan disembunyikan | — |
| S-13 Hasil Tangkap | OCR/pencarian per item berprogres per baris | Semua item tidak cocok: daftar kegagalan jujur per item (F-09) | Pencocokan terjeda; yang cocok sudah tampil | Per item | OCR tetap jalan (di perangkat); pencocokan menunggu jaringan & dinyatakan | Hasil sebagian dapat langsung dipakai | Berbagi kedua tiba saat pertama diproses: antrean tangkap serial, tidak menimpa |
| S-21 Player | Status mesin jujur (Bab 2): loading/buffering dengan waktu tunggu | — | Ikut prosedur L-10; tidak ada error mentah | Ikut L-10 | Memutar lokal/unduhan normal; sampul dari cache/berkas | Lirik/aksi sumber menyusul terpisah dari audio | Seek saat buffering: seek diterapkan ke target setelah buffer, tidak membatalkan muat |
| S-22 Antrian | Render dari mesin (seketika) | Antrean kosong: status + ajakan | — (struktur lokal; isi ulang otomatis berpenanda) | Isi ulang gagal: seksi Otomatis berpenanda "sambungan tertunda" | L-11 (terbatas-offline) | — | L-08 (versi antrean) |
| S-23 Lirik | Penanda "mencari lirik" ≤ 2 dtk lalu status | Tanpa lirik: status rapi + lirik polos bila ada (PRD 5.8) | Gagal: status + coba lagi; audio tidak terpengaruh | Sumber lirik gagal ≠ audio gagal | Lirik ter-cache tampil; belum ter-cache: status offline | Sinkron tidak tersedia untuk lagu ini: tampil polos | Lagu berganti saat lirik dimuat: lirik lagu lama dibuang (token lagu, L-13) |
| S-30/31 Pengaturan | Seketika (lokal) | — | — | — | Normal penuh | Kalibrasi Volume Ikut Kecepatan butuh GPS: status menunggu sinyal dinyatakan | Perubahan pengaturan saat fitur berjalan berlaku seketika & dicatat (mis. mematikan Putar Otomatis membuang seksi Otomatis kecuali 1 lagu berikutnya yang sudah dimuat) |
| S-40 Mode Berkendara | Masuk ≤ 1 dtk dari pemicu | — | Audio mengikuti L-10; permukaan tetap responsif | Sama jaringan | Musik lokal/unduhan; perintah suara dinyatakan butuh jaringan (F-08) | GPS buruk: Volume membeku (F-07) | Perangkat audio berganti: F-07 cabang ✖ |

**Aturan token (L-13):** setiap permintaan asinkron yang terikat kueri/lagu/layar membawa token identitas; hasil yang tokennya sudah tidak berlaku dibuang tanpa menyentuh UI. Ini menutup kelas bug "hasil lama menimpa hasil baru" di seluruh aplikasi.

**Perintah perangkat global (L-14):** tombol perangkat audio (Bluetooth/kabel), kontrol notifikasi & lock screen, dan perintah Mode Berkendara diproses langsung oleh mesin player tanpa membuka atau mengubah UI; dari keadaan aplikasi apa pun (latar, layar mati, belum pernah dibuka sesi ini) hasilnya identik: perintah berlaku pada antrean yang sama (L-01).

**Snapshot konteks (L-15):** memulai putar selalu membekukan daftar konteks pada saat ketuk ke dalam antrean mesin; perubahan daftar di sumber sesudahnya tidak mengubah antrean yang sedang berjalan.

**Pemilihan sumber putar (L-12):** untuk lagu yang sama, urutan prioritas sumber putar: berkas unduhan lokal → file lokal → aliran sumber. Pengalihan antar sumber untuk lagu yang sedang diputar tidak boleh menimbulkan jeda dengar atau pengulangan segmen.

**Pemulihan sesi (L-16):** posisi, antrean, konteks, mode shuffle/repeat, dan volume disimpan persisten (posisi: tiap 5 detik & pada setiap transisi status). Aplikasi hidup kembali selalu menawarkan keadaan terakhir secara diam-diam sudah terpasang (bukan dialog), dengan status paused — tidak pernah auto-bunyi setelah dibunuh sistem, kecuali pemicu Mode Berkendara yang pengguna tetapkan.

## 6. Aturan Offline Menyeluruh

| Kemampuan | Offline | Catatan |
|---|---|---|
| Putar unduhan & file lokal | ✅ penuh | Termasuk antrean campuran & playlist |
| Koleksi, liked, playlist pengguna | ✅ penuh | Data lokal, tidak pernah menunggu jaringan |
| Putar Otomatis | ⚠️ terbatas | L-11: tangga atas katalog lokal; ditandai terbatas |
| Pencarian | ⚠️ lokal saja | Dibatasi koleksi/unduhan/lokal & dinyatakan |
| Penemuan/rak sumber, halaman artis baru | ❌ | Status offline jujur + jalan pintas ke Unduhan |
| Lirik | ⚠️ cache saja | Yang belum ter-cache: status, bukan error |
| Antrean Suara | ⚠️ tergantung perangkat | Pengenal suara di perangkat: jalan; berbasis layanan: dinyatakan butuh jaringan |
| Tangkap (OCR) | ⚠️ sebagian | OCR jalan; pencocokan lagu menunggu jaringan |
| Statistik & Lanjut Besok | ✅ penuh | Seluruhnya dihitung/direkam lokal |

Prinsip penutup: **offline tidak pernah ditampilkan sebagai kesalahan.** Ia keadaan biasa dengan kemampuan yang dinyatakan terus terang per layar (matriks Bab 5), dan semua yang bersifat data pengguna bekerja penuh di dalamnya.

---

## 7. Penutupan Phase 2 — cakupan & penyerahan

- Tercakup: arsitektur informasi & inventaris 21 layar/titik masuk; mesin status putar + matriks interupsi; 16 aturan logika antrean/sistem (L-01…L-16); 14 alur inti F-01…F-14 termasuk seluruh fitur tanda tangan PRD 5.14; matriks missing-states 17 kelompok layar × 7 kelas keadaan; aturan offline menyeluruh.
- Ditugaskan eksplisit ke fase berikut (bukan menggantung): format berkas ekspor JSON & skema DB fisik → **Phase 3**; rupa visual tiap status di matriks Bab 5 (skeleton, ikon status) → **Phase 4** verifikasi DS; pemecahan alur menjadi tugas atomik → **Phase 5**.
- Temuan untuk PRD: tidak ada konflik; satu penegasan baru (L-03: antrean manual dipertahankan saat konteks berganti) diusulkan naik ke PRD 5.4 pada revisi berikutnya.
