# SPESIFIKASI PRODUK — Aplikasi Musik "Clai"

**Versi:** 2.0 — 23 Oktober 2026
**Status:** Dokumen induk produk — **Phase 1 (PRD)** dari proses 5 fase proyek (PRD → User Flow & Logic Mapping → Technical Blueprint & Architectural Critique → Design System & Component Guidelines → Implementation Plan). Menggantikan v1.0 yang masih berorientasi pengguna tunggal; v2.0 menyelaraskan seluruh dokumen dengan arah produk publik open source.
**Basis evidence:** Bedah Spotify Web Player, 22–23 Okt 2026 (`~/workspace/research_notes/bedah-spotify-20261022/report.md` + `EVIDENCE.md` — graf REA PC: 5.467 node, 4.323 modul, 976 endpoint).
**Pemilik produk:** Riyan Zakaria Zulkarnain. **Distribusi:** open source (GitHub), untuk pengguna umum.

---

## 0. Prinsip Mengikat — Universal & Tidak Naif

Dua prinsip ini mengikat setiap bab sesudahnya; bila ada rancangan yang bertabrakan dengannya, rancangan itu yang kalah.

**Universal.** Clai dirancang untuk pengguna umum, bukan untuk satu orang. Konsekuensinya: tidak ada fitur yang mengandaikan kebiasaan, perangkat, bahasa, atau riwayat milik satu pengguna tertentu. Setiap fitur harus masuk akal bagi (a) pengguna baru tanpa riwayat apa pun, (b) pengguna di perangkat kelas bawah, (c) pengguna di luar Indonesia, dan (d) pengguna dengan kebutuhan aksesibilitas. Personalisasi adalah lapisan tambahan di atas dasar yang sudah utuh tanpanya — bukan fondasinya.

**Tidak naif.** Setiap fitur dirancang melawan kenyataan, bukan melawan kondisi ideal: sensor bisa bohong (GPS meloncat), jaringan bisa membusuk di tengah lagu, sumber bisa berubah bentuk, perangkat bisa membunuh proses latar, pengguna bisa salah perintah. Rancangan wajib menyebut perilaku saat hal-hal itu terjadi — mode gagal yang aman dan jujur dinyatakan lebih bernilai daripada mode sukses yang rapuh. Klaim kemampuan apa pun harus punya cara mengukurnya (Bab 3).

Turunannya yang berlaku global: (1) teks antarmuka selalu siap diterjemahkan, tidak ada kalimat yang tertanam di kode; (2) data pribadi tetap di perangkat — tanpa akun wajib, tanpa telemetri; (3) tidak ada perilaku yang berubah diam-diam terhadap pengguna.

---

## 1. Visi & Posisi

Clai adalah pemutar musik streaming Android kelas produk komersial: pengalaman setara Spotify pada semua hal yang berada di sisi client, dengan katalog bersumber terbuka yang tidak mengunci pengguna, dan seluruh data pribadi (koleksi, riwayat, statistik) dimiliki pengguna di perangkatnya sendiri.

Yang membedakan dari aplikasi sejenis yang sudah ada (Spotube, InnerTune, dan fork-nya):

1. **Player sebagai produk utama, bukan tempelan.** Queue, transisi antar lagu, dan keandalan putar diperlakukan sebagai fitur kelas satu — di Spotify, queue adalah komponen paling dominan di kodenya (442 referensi, halaman tersendiri); di aplikasi gratisan justru di situlah bug menumpuk.
2. **Arsitektur sumber yang dapat diganti.** Pelajaran utama dari bedah: kekuatan Spotify terkunci di servernya. Clai tidak menikah dengan satu sumber — lapisan sumber (provider) diabstraksi sejak baris pertama, sehingga sumber audio/metadata bisa ditambah atau diganti tanpa membongkar player.
3. **Tidak ada iklan, tidak ada akun wajib, tidak ada pelacakan.** Gratis bukan karena ditukar data pengguna.
4. **Kualitas terukur, bukan kesan.** Setiap klaim "bagus" di dokumen ini punya angka dan cara mengujinya (Bab 3). Fitur yang tidak lulus gerbang kualitasnya dianggap belum ada.

## 2. Pengguna & Masalah

Clai adalah produk publik. Persona-persona berikut adalah jangkar rancangan — setiap fitur di Bab 5 harus dapat ditunjuk melayani minimal satu persona tanpa merugikan persona lain:

1. **Pendengar harian arus utama.** Referensi rasanya adalah pemutar besar; kesabarannya tipis terhadap kegagalan putar dan UI yang terasa proyek hobi. Kebutuhan: putar andal, queue bebas, pencarian cepat.
2. **Pengguna perangkat kelas bawah.** HP RAM 2–4 GB, penyimpanan sempit, kuota terbatas, jaringan tidak stabil. Kebutuhan: aplikasi ringan, hemat data, perilaku jaringan buruk yang dirancang (Bab 3), unduhan terkelola.
3. **Pengguna dalam perjalanan (termasuk pengendara motor).** Tangan & mata tidak bebas; HP mungkin di saku, di holder, atau di dashboard. Kebutuhan: kendali tanpa layar (tombol perangkat audio & suara) — dilayani fitur tanda tangan Bab 5.14.
4. **Pengguna baru (cold start).** Baru memasang, riwayat kosong, koleksi kosong. Kebutuhan: nilai sejak lagu pertama — penemuan & Putar Otomatis tidak boleh bergantung pada riwayat personal (Bab 5.4).
5. **Pengguna lama berkoleksi besar.** Ribuan lagu tersimpan, playlist bertahun-tahun. Kebutuhan: koleksi cepat dicari, data tidak pernah hilang melewati update (Bab 8), ekspor/impor.
6. **Pengguna internasional & aksesibilitas.** Bahasa antarmuka selain Indonesia; pengguna TalkBack dan penskalaan huruf. Kebutuhan: i18n sejak F1 (Bab 4.3), gerbang aksesibilitas Bab 3.

**Masalah yang diselesaikan (berlaku umum, diamati dari kegagalan aplikasi sejenis):**
- Playback putus/error di tengah lagu tanpa pemulihan.
- Queue tidak bisa dikelola bebas (tambah/urutkan/hapus).
- Pencocokan lagu dari dua sumber (model Spotube) menghasilkan lagu yang salah.
- Musik berhenti total saat konteks habis, memaksa pengguna menyusun antrean sendiri.
- UI terasa seperti proyek hobi: tidak konsisten, animasi kaku, status putar tidak jelas.
- Koleksi dan riwayat hilang saat ganti aplikasi/perangkat (tidak ada ekspor).
- Aplikasi musik tidak dapat dipakai dengan aman saat berkendara.

## 3. Gerbang Kualitas (Quality Gates) — bagian yang membuat dokumen ini "tingkat tinggi"

Angka di bawah adalah syarat rilis tiap fase. Diukur pada **dua kelas perangkat bernama** — Kelas Menengah (RAM 6 GB, kelas Redmi Note) dan **Kelas Bawah (RAM 3 GB, kelas Redmi 9 / setara, Android 10)** — dan gerbang dinyatakan lulus hanya bila lulus di *kedua* kelas. Jaringan acuan: 4G Indonesia **dan** profil memburuk (3G/throttle ±400 kbps dengan putus 5 detik berkala); perilaku pada profil memburuk dirancang di Bab 5.4 & 5.9 dan diuji, bukan diharapkan. Kumpulan uji: 100 lagu populer + 100 lagu niche + 25 lagu berdurasi > 7 menit.

| Aspek | Gerbang | Cara mengukur |
|---|---|---|
| Tekan play → bunyi | ≤ 2 detik (median), ≤ 4 detik (p95) | Instrumentasi di aplikasi, mode uji |
| Kegagalan putar | < 1% dari upaya putar; 0 kegagalan tanpa pesan & pemulihan otomatis | Log sesi lokal selama uji 2 minggu |
| Transisi antar lagu | Gapless ≤ 50 ms celah; crossfade halus tanpa klik/potongan | Uji dengar + pengukuran waveform rekaman output |
| Animasi | 60 fps pada scroll daftar 1.000 item & animasi player; motion memakai fisika spring (overshoot halus), dilarang easing linear/kaku | Profiling frame di perangkat uji |
| Cold start | ≤ 1,5 detik sampai beranda interaktif | Pengukuran rilis build |
| Stabilitas | Crash-free ≥ 99,5% sesi | Laporan crash lokal |
| Konsumsi data | Mode hemat data tersedia; kualitas audio bisa dipilih eksplisit (rendah/normal/tinggi) dengan label bitrate jujur | Inspeksi lalu lintas saat uji |
| Baterai | Putar latar 1 jam ≤ 5% baterai perangkat uji (layar mati) | Uji baterai terkontrol |
| Aksesibilitas | Kontrol utama terjangkau satu tangan; target sentuh ≥ 48dp; kontras teks ≥ WCAG AA; semua kontrol berikon punya label semantik TalkBack; antarmuka utuh pada penskalaan huruf 130% tanpa teks terpotong; mode gerak dikurangi (reduced motion) menonaktifkan overshoot spring | Audit manual + TalkBack walkthrough per rilis |
| Bahasa | Seluruh teks antarmuka tersedia Bahasa Indonesia & Inggris; mengganti bahasa tidak memerlukan mulai ulang | Uji string coverage (tidak ada teks tertanam) per rilis |

**Aturan main:** fase tidak dinyatakan selesai karena "fiturnya jalan", melainkan karena gerbangnya lulus. Kekurangan yang ditemukan saat uji dikembalikan ke fase yang sama, bukan ditumpuk sebagai utang.

## 4. Arsitektur Tingkat Tinggi

### 4.1 Prinsip
- **Clean-room total.** Tidak ada kode, ikon, logo, gambar, atau aset Spotify yang dipakai. Yang diambil dari hasil bedah hanyalah perilaku & struktur fitur (spec), sesuai metode clean-room.
- **Player tidak tahu sumber.** Semua sumber (streaming, file lokal) menyajikan kontrak data yang sama ke player.
- **Offline-first untuk data pengguna.** Koleksi, playlist, riwayat, statistik: lokal dulu, tanpa server, tanpa akun.
- **Satu sumber di awal, tanpa jalan buntu.** Sumber pertama: YouTube Music (jalur InnerTube — metadata & audio dari sumber yang sama, menghindari bug pencocokan dua sumber). Lapisan provider memastikan sumber kedua (mis. metadata Spotify resmi untuk impor playlist) dapat ditambah sebagai modul, bukan operasi bedah.

### 4.2 Lapisan sistem
1. **Design System "Clai DS"** — token warna/tipografi/spasi/motion (Bab 7). Semua layar dibangun dari komponen ini; tidak ada gaya ad-hoc per layar.
2. **Presentasi** — Beranda, Pencarian, Koleksi, Halaman Konten, Player, Queue, Lirik, Unduhan, Statistik, Pengaturan.
3. **Mesin Player (Playback Engine)** — state machine antrian & pemutaran (Bab 5.4), independen dari UI; berjalan sebagai foreground service Android agar putar latar kebal dari penutupan sistem.
4. **Lapisan Provider (sumber)** — adaptor YouTube Music; adaptor File Lokal; (modul masa depan) adaptor impor Spotify. Tiap adaptor wajib lulus kontrak uji yang sama.
5. **Data Lokal** — basis data relasional di perangkat: koleksi, playlist, riwayat putar (untuk statistik & "terakhir diputar"), cache metadata, antrean unduhan. Skema bermigrasi ter versi — data pengguna tidak boleh hilang saat update.
6. **Mesin Unduhan** — antrean, jeda/lanjut, manajemen kuota penyimpanan, verifikasi integritas.
7. **Layanan Sinkron Perangkat (Connect versi Clai)** — dirancang sejak sekarang (protokol & model datanya), dibangun pada fase lanjut: WebSocket antar perangkat milik pengguna sendiri, satu perangkat jadi remote, satu jadi pemutar.
8. **Observabilitas lokal** — pencatatan kegagalan putar & crash di perangkat (untuk gerbang kualitas), tanpa telemetri keluar.

### 4.3 Keputusan teknologi (awal, dapat diganti dengan alasan tertulis)
- **Platform:** Android native-first (target pertama Android 10+). Bukan web/PWA — putar latar, kontrol media sistem, dan unduhan adalah warga kelas satu di Android, warga kelas dua di web.
- **Kerangka UI:** Flutter (satu basis kode, animasi spring 60fps dapat dikendalikan penuh, ekosistem audio `just_audio` + `audio_service` matang untuk foreground service & kontrol notifikasi/lock screen).
- **Audio:** ExoPlayer (via just_audio) — mendukung gapless, crossfade melalui pengelolaan dua player, loudness, dan format hasil sumber.
- **Bahasa data:** semua entitas bertipe kuat (Bab 6); tidak ada peta string liar antar lapisan.
- **Lokalisasi:** semua teks antarmuka hidup di berkas string (ARB Flutter), tidak ada kalimat tertanam di kode; bahasa awal Bahasa Indonesia + Inggris, dipilih dari pengaturan sistem dengan pengganti manual di Pengaturan. Bahasa lain dapat ditambah komunitas tanpa menyentuh kode (konsekuensi open source).

## 5. Peta Fitur Lengkap

Setiap fitur ditulis lengkap sejak dokumen ini — pembangunan bertahap (Bab 9) tidak mengurangi spec; fase hanya menentukan urutan, bukan kedalaman. Penanda fase: **F1** fondasi & inti, **F2** pendalaman, **F3** lanjutan, **F4** ekspansi.

### 5.1 Beranda (F1, diperdalam F2)
- Rak horizontal bertema dengan "lihat semua": Terakhir Diputar, Putar Lagi, Album & Playlist Tersimpan Terbaru.
- F2: rak penemuan dari sumber (rilis baru, tangga lagu, campuran per suasana) + rak personal lokal: "Favoritmu Akhir-akhir Ini", "Yang Sering Kamu Putar" — dihitung dari riwayat di perangkat, jujur dilabeli sebagai hitungan lokal, bukan menyamar sebagai algoritma server.
- Beranda tidak pernah kosong: status kosong punya rancangan (ajakan mencari lagu pertama), bukan layar blank.

### 5.2 Pencarian (F1)
- Kategori hasil: Lagu, Album, Artis, Playlist, (F4: Podcast/Episode).
- Pencarian bertahap (hasil muncul saat mengetik dengan debounce), riwayat pencarian tersimpan lokal dan bisa dihapus per item / seluruhnya (di Spotify ini ada sebagai `fusionSearchHistory` — pola yang sama, implementasi sendiri).
- Filter per kategori di halaman hasil; status kosong & status gagal jaringan dirancang terpisah (gagal ≠ tidak ditemukan).

### 5.3 Halaman Konten (F1)
- **Album:** sampul besar, daftar lagu bernomor, durasi total, tahun, artis; aksi: putar, acak-putar, simpan ke koleksi, unduh (F2), bagikan sebagai teks/tautan sumber.
- **Artis:** lagu populer, diskografi (album/single/kompilasi), artis terkait (bila sumber menyediakan).
- **Playlist (sumber):** seperti album + keterangan pemilik.
- Semua daftar lagu memakai baris lagu standar Clai DS: sampul mini, judul, artis, durasi, indikator sedang-diputar (animasi ekualiser kecil), menu konteks konsisten: Putar Berikutnya, Tambah ke Antrian, Mulai Radio Lagu, Tambah ke Playlist, Simpan/Unduh, Buka Album/Artis.

### 5.4 Mesin Player & Queue — jantung produk (F1, diperdalam F2)
Spec ini ditulis paling rinci karena di sinilah standar "tingkat tinggi" dibuktikan.

**State machine putar:** status eksplisit — `idle → loading → playing ⇄ paused → buffering → ended/error`. UI selalu mencerminkan status mesin secara jujur (tidak ada spinner abadi; buffering > 5 detik menampilkan status + opsi lewati).

**Queue:**
- Bagian-bagiannya: "Sedang Diputar", "Berikutnya" (antrian manual pengguna), "Lanjutan" (sisa konteks: album/playlist asal), dan bila konteks hampir habis: "Putar Otomatis" (lihat sub-bab Putar Otomatis Cerdas di bawah).
- Tambah ke antrian menyisip **setelah** lagu berjalan + antrian manual sebelumnya (perilaku Spotify), bukan menimpa konteks.
- Geser urut (drag) dengan pegangan jelas, hapus geser-samping, kosongkan antrian, urungkan penghapusan (snackbar undo 5 detik).
- Ketuk lagu di daftar mana pun memulai konteks baru (album/playlist/hasil cari) — perilaku dapat diprediksi dan didokumentasikan di bantuan dalam aplikasi.

**Kontrol putar:**
- Play/pause, next/prev, seek dengan pratinjau waktu; prev dalam 3 detik pertama kembali ke lagu sebelumnya, setelah itu mengulang dari awal (konvensi Spotify).
- **Shuffle** dua mode (F2 untuk mode kedua): acak murni (Fisher–Yates atas salinan antrian; posisi lagu berjalan tetap) dan **Smart Shuffle** lokal (mengutamakan lagu tersimpan & sering diputar disisipkan berkala — dilabeli jelas saat aktif).
- **Repeat:** mati / semua / satu lagu, status selalu terlihat di tombol.
- **Kecepatan putar** 0.5–2.0× (F4, terutama untuk podcast).

**Audio lanjutan (F2):**
- **Crossfade** 0–12 detik dapat diatur; **gapless** untuk album yang direkam menyambung; keduanya diuji terhadap gerbang Bab 3.
- **Normalisasi kenyaringan** antar lagu (target −14 LUFS, dapat dimatikan) agar volume tidak melompat antar sumber.
- **Equalizer** 5 pita + preset (datar, bass, vokal, akustik, kustom tersimpan).
- **Sleep timer:** berbasis menit atau "akhir lagu ini"; versi pendalamannya yang mengingat posisi tidur adalah **Lanjut Besok** (Bab 5.14 D).

**Putar Otomatis Cerdas (Autoplay) (F1 — inti, default AKTIF):**
- Masalah yang diselesaikan (dilaporkan langsung oleh pemilik): memutar satu lagu dari pencarian lalu musik berhenti total saat lagu selesai, memaksa pengguna menyusun antrian sendiri. Di Clai perilaku itu dilarang: **konteks yang habis tidak pernah berarti musik berhenti.**
- Saat konteks putar (album/playlist/pencarian) mencapai lagu terakhirnya, mesin otomatis masuk mode Putar Otomatis dan menyambung dengan lagu-lagu terkait tanpa henti dan tanpa jeda antarmuka, dipilih dengan **Tangga Kemiripan** di bawah.
- Sambungan dibuat sebagai daftar tak berujung yang diisi bertahap (ambil ±15 lagu, isi ulang saat tersisa 5) — bukan daftar mati yang berhenti lagi setelah belasan lagu.

**Tangga Kemiripan (aturan pemilihan sambungan, dari terdekat):**
1. Lagu lain dari **artis yang sama**.
2. **Lagu serupa** menurut data keterkaitan sumber (radio/related) — dihitung dari pola dengar global sumber, bukan dari riwayat pengguna.
3. Lagu dari **artis serupa** (graf artis terkait dari sumber).
4. **Genre & niche yang sama** dari metadata, sebagai jaring pengaman bila tangga 1–3 tipis.
5. **Lapisan personal (penguat, bukan syarat):** lagu yang pengguna simpan/sering putar *dan* lolos tangga 1–4 boleh dinaikkan peringkatnya. Tanpa lapisan ini pun tangga 1–4 bekerja penuh.

**Cold start (pengguna baru):** karena tangga 1–4 bergantung pada *lagu yang sedang diputar* dan data global sumber — bukan pada riwayat personal — pengguna hari pertama dengan riwayat kosong mendapat sambungan yang sama relevannya dengan pengguna lama. Riwayat hanya memengaruhi tangga 5. Ini disengaja dan diuji (Bab 8: uji antrean otomatis dari akun uji ber-riwayat kosong).

**Anti-drift (penjaga relevansi):** kemiripan selalu ditambatkan ke **benih sesi** (lagu-lagu konteks asal + lagu yang pengguna biarkan selesai dalam sesi ini), tidak pernah ke sambungan sebelumnya — penambatan berantai dilarang karena membuat aliran hanyut ke arus utama generik dalam belasan lagu. Aturan pendamping: satu artis maksimal 2 lagu berturut-turut; lagu yang **di-skip pengguna dalam sesi ini adalah sinyal negatif** — lagu itu dan lagu yang sangat mirip dengannya diturunkan keras peringkatnya untuk sisa sesi; lagu yang dibiarkan selesai adalah sinyal positif ringan untuk benih sesi.
- Di layar Antrian, sambungan ini tampil sebagai seksinya sendiri: **"Putar Otomatis — karena kamu memutar [judul lagu]"**, terpisah dari antrian manual. Lagu di seksi ini bisa dihapus per lagu; seksi ini selalu kalah prioritas: antrian manual pengguna dan konteks baru selalu mendahului.
- Prioritas mode: Repeat-Satu > antrian manual > konteks (dengan Repeat-Semua mengulang konteks dulu) > Putar Otomatis. Konteks satu lagu dengan Repeat-Semua diperlakukan sebagai Putar Otomatis (mengulang satu lagu selamanya bukan perilaku yang diinginkan siapa pun).
- Pengguna bisa mematikan Putar Otomatis di Pengaturan → Perilaku Putar, dan mematikannya untuk sesi ini saja dari layar Antrian (sakelar di kepala seksi Putar Otomatis). Bila mati, perilaku kembali eksplisit: konteks habis = berhenti, dan UI menyatakannya (tidak diam-diam).
- Pintu masuk manual: aksi **"Mulai Radio Lagu"** di menu konteks lagu apa pun — memulai stasiun tak berujung dari lagu itu, memakai mesin yang sama.

**Pemulihan (F1):**
- URL aliran sumber berumur terbatas: mesin memperbarui URL secara proaktif sebelum kedaluwarsa dan reaktif saat gagal (maks 2 percobaan), tanpa campur tangan pengguna.
- Lagu yang gagal total dilewati otomatis dengan pemberitahuan yang menyebut judulnya — tidak pernah berhenti diam-diam.

### 5.5 Mini Player & Player Penuh (F1)
- Mini player persisten di atas navigasi: sampul, judul/artis berjalan, play/pause, next; geser atas membuka player penuh.
- Player penuh: sampul besar, seek bar dengan waktu berjalan/sisa, kontrol lengkap, akses cepat ke Queue & Lirik (tab/geser), menu lagu aktif.
- Gerakan membuka/menutup memakai spring fisik (parameter di Bab 7), sampul mengecil-membesar kontinu mengikuti geseran (bukan potongan dua layar).
- Kontrol media sistem: notifikasi media & lock screen (play/pause/next/prev/seek), headset/bluetooth, jeda otomatis saat telepon masuk / audio lain mengambil fokus; lanjut-otomatis dapat diatur.

### 5.6 Koleksi / Library (F1)
- Tab: Playlist, Album, Artis, Lagu Tersimpan (Liked), File Lokal, Unduhan (F2).
- Urutkan (terbaru disimpan, A–Z, terakhir diputar) & saring dengan kotak cari lokal.
- Liked Songs adalah playlist sistem yang tidak bisa dihapus; like bisa dari mana pun (baris lagu, player, notifikasi).
- **File Lokal (F1):** memindai folder musik perangkat; file lokal masuk antrean & playlist campuran dengan lagu streaming secara setara (di Spotify, local-files adalah bagian koleksi resmi — pola ini kita adopsi sebagai warga kelas satu, bukan bonus).
- Ekspor/impor koleksi (F3): satu berkas JSON — pindah perangkat tanpa kehilangan apa pun.

### 5.7 Playlist Milik Pengguna (F1)
- Buat/ganti nama/hapus; tambah lagu dari menu konteks mana pun; susun ulang drag; hapus geser + undo.
- Sampul otomatis: mosaik 2×2 dari 4 lagu pertama (pola mosaic CDN Spotify, dibuat lokal dari sampul lagu).
- Duplikat saat menambah diberi tahu dengan pilihan "tetap tambahkan / lewati" (bukan diam-diam dobel).

### 5.8 Lirik (F2)
- Lirik sinkron baris-per-baris (sumber terbuka lirik: LRClib; cadangan: lirik dari sumber lagu), baris aktif tersorot & ikut bergeser; ketuk baris = lompat ke detik itu.
- Tanpa lirik ≠ error: tampilkan status rapi + (bila ada) lirik tak-sinkron polos.

### 5.9 Unduhan & Offline (F2)
- Unduh album/playlist/lagu; antrean dengan jeda/lanjut/batal; indikator progres per item & global.
- Putar offline penuh dari unduhan; mode "hanya unduhan" agar tidak menyedot kuota tanpa sadar.
- Manajemen penyimpanan: lihat pemakaian, tetapkan batas (mis. 2/4/8 GB), hapus per album/playlist/semua; unduhan terverifikasi integritasnya (berkas korup diunduh ulang, bukan diputar setengah).

### 5.10 Statistik & Riwayat Lokal (F3)
- "Clai Wrapped" lokal: lagu/artis teratas per bulan/semester/tahun, total menit mendengar, pola jam dengar — semua dihitung di perangkat dari riwayat putar, dapat dihapus pengguna kapan pun (riwayat adalah miliknya).
- Halaman riwayat: daftar putar terakhir dengan waktu; putar ulang dari sana.

### 5.11 Connect Versi Clai (F3)
- Perangkat milik pengguna (HP, PC, tablet) saling menemukan di jaringan lokal / lewat kode pasangan; satu jadi remote penuh (lihat antrean, ubah volume, pindah lagu) untuk pemutar lain — meniru peran `dealer` WebSocket Spotify dengan protokol rancangan sendiri.
- Tanpa server pusat wajib: mode LAN murni harus jalan; mode internet (relay) adalah tambahan, bukan syarat.

### 5.12 Podcast & Audiobook (F4)
- Jenis konten episode dengan posisi-dengar tersimpan per episode, lanjutkan-otomatis, kecepatan putar, lewati hening (bila sumber mendukung datanya).
- Dirancang datanya sejak Bab 6 agar tidak perlu migrasi menyakitkan nanti.

### 5.13 Pengaturan (F1, lengkap F2)
- Kualitas streaming (rendah/normal/tinggi + hanya WiFi untuk kualitas tinggi), mode hemat data, kualitas unduhan.
- Perilaku putar: crossfade, gapless, normalisasi, lanjut-otomatis, fokus audio.
- Tampilan: tema gelap bawaan; (F2) aksen dinamis dari sampul lagu yang sedang diputar.
- Data: kelola unduhan, hapus cache, hapus riwayat, ekspor/impor koleksi.
- Tentang: versi, lisensi sumber terbuka yang dipakai, catatan clean-room.

### 5.14 Fitur Tanda Tangan (yang hanya ada di Clai)

Fitur-fitur ini adalah pembeda produk; semuanya dirancang universal & tidak naif sesuai Bab 0.

**A. Mode Berkendara — kendali tanpa layar (F2).**
- Untuk pengendara motor & mobil. Pemicu (pengguna memilih, tidak ditebak diam-diam): tersambung ke perangkat Bluetooth yang ditandai pengguna sebagai perangkat kendaraan, deteksi aktivitas berkendara sistem, atau sakelar manual.
- **Kendali berlapis sesuai kemampuan perangkat audio yang tersambung** — Clai mendeteksi dan memakai yang tersedia, dari terkaya ke termiskin: (i) tombol media standar (putar/jeda/berikutnya/sebelumnya), (ii) pola tekan pada satu tombol (1x putar/jeda, 2x berikutnya, 3x sebelumnya, tahan = perintah suara), (iii) perintah suara penuh. Skema aktif ditampilkan & dapat diganti di Pengaturan; tidak ada asumsi semua pengguna memakai perangkat yang sama.
- **Berlaku untuk HP di mana pun:** di saku, holder, atau dashboard. Bila layar terlihat (holder/dashboard), Mode Berkendara juga menampilkan permukaan sentuh raksasa (tombol ≥ 72dp, usap di mana saja untuk berikutnya/sebelumnya); bila tidak, mode ini murni audio.
- **Pengumuman suara (opsional, bawaan mati):** judul lagu berikutnya dibacakan TTS sistem sebelum diputar; dapat dimatikan per sesi.
- **Volume Ikut Kecepatan (opsional, bawaan mati, di dalam Mode Berkendara):** versi tidak naif dari penyesuaian volume otomatis — pengguna mengkalibrasi sendiri dua titik (volume saat berhenti & volume tercepatnya) dan aplikasi **hanya boleh bergerak di antara dua titik itu**; perubahan diramp maksimal 1 langkah per 2 detik mengikuti **median kecepatan 10 detik**; sinyal GPS berakurasi buruk diabaikan (volume membeku, tidak menebak); membeku total saat panggilan/notifikasi; kembali ke titik berhenti setelah berhenti > 15 detik. Plafon yang ditetapkan pengguna tidak pernah dilampaui dalam keadaan apa pun — fitur ini gagal-aman: keraguan apa pun = volume tetap.

**B. Antrean Suara (F2).**
- Menambah/memutar lagu dengan suara, tanpa menyentuh HP. Rantai kerjanya (tanpa LLM, semuanya di perangkat/sistem): pengenal suara bawaan Android (Bahasa Indonesia & Inggris) → **pengurai perintah berbasis aturan** (pola terbatas: "putar X", "tambah X setelah ini", "lewati", "jeda", "lanjut", beserta padanannya dalam Inggris) → kata kunci lagu dicari ke sumber → kandidat diberi skor (kemiripan judul/artis, versi resmi, riwayat pengguna sebagai penguat) → pemenang masuk antrean manual (di atas Putar Otomatis) → konfirmasi suara pendek + earcon.
- **Ketidakpastian dinyatakan, tidak ditebak:** dua kandidat berskor dekat → aplikasi menanyakan lewat suara dan mendengarkan jawaban; keyakinan pengenal suara rendah → minta ulangi (maks 2x) lalu menyerah dengan sopan; lagu tidak ditemukan → dikatakan apa adanya. Tidak ada lagu yang masuk antrean tanpa konfirmasi suara dari aplikasi yang menyebut judulnya.

**C. Tangkap dari Mana Saja (F2).**
- Clai mendaftar sebagai tujuan berbagi Android: teks atau tautan yang dibagikan dari aplikasi mana pun (chat, media sosial, browser) diterjemahkan menjadi lagu yang diputar/diantrekan — tautan sumber dikenali langsung; teks bebas diperlakukan sebagai kata kunci pencarian; bila yang dibagikan adalah gambar daftar lagu, teksnya dibaca dengan OCR di perangkat lalu tiap baris dicocokkan.
- Kegagalan pencocokan dilaporkan per item (ditemukan/tidak), tidak diam-diam: pengguna melihat daftar hasil tangkapan sebelum diputar massal.

**D. Lanjut Besok — tidur yang mengingat (F2, pendalaman Sleep Timer 5.4).**
- Sleep timer standar menghilangkan konteks; Lanjut Besok menyimpan **posisi tidur**: lagu & detik saat pengguna tertidur (perkiraan dari timer + tidak ada interaksi lanjutan), beserta konteksnya.
- Esoknya (atau kapan pun sesudahnya) satu kartu di Beranda menawarkan "Lanjutkan dari semalam" — memulihkan konteks persis dari titik itu. Bila pengguna ternyata masih terjaga dan berinteraksi setelah timer, posisi tidur dibatalkan otomatis.

---

## 6. Model Data (entitas inti)

Semua entitas punya `id` stabil dari sumbernya + `providerId` asal — tidak ada entitas "tanpa sumber".

- **Track**: id, judul, artis[], albumId, durasiMs, sampulUrl, isExplicit, providerId, playableRef (cara provider mengambil aliran), liked, lastPlayedAt, playCount.
- **Album**: id, judul, artis[], tahun, sampulUrl, trackIds (urut), providerId.
- **Artist**: id, nama, sampulUrl, providerId.
- **Playlist**: id, nama, pemilik (sumber/pengguna), deskripsi, trackIds urut, isSystem (Liked), sampulMosaik (diturunkan), updatedAt.
- **QueueState**: konteks asal (jenis+id), daftar urutan efektif, indeks berjalan, mode shuffle/repeat, riwayat-balik (untuk prev yang benar).
- **PlaybackEvent**: trackId, dimulai, durasiDidengarMs, selesai (bool), sumber konteks — bahan statistik & "terakhir diputar". Disimpan lokal, retensi dapat diatur pengguna.
- **DownloadItem**: trackId, status, byteTotal/byteTurun, path lokal, checksum, kualitas.
- **UserPrefs**: semua pengaturan Bab 5.13 dalam satu skema berversi.

## 7. Design System "Clai DS" (ringkas, dokumen visual menyusul sebagai artefak tersendiri)

- **Grid & spasi:** kelipatan 4dp; margin layar 16dp; ritme vertikal konsisten antar layar.
- **Tipografi:** satu keluarga font; skala tetap (display 28, judul 20, isi 15, keterangan 13, mikro 11) — tidak ada ukuran liar per layar.
- **Warna:** [DIPUTUSKAN di DESIGN-SYSTEM.md v3.0] tema gelap netral bertingkat 3 level permukaan; satu aksen utama Volt #B8F135 (palet v1.0 yang dipulihkan setelah "Senja" v2.0 ditolak pemilik karena membuat pusing); struktur player-first v2 dipertahankan; aksen dinamis opsional dari sampul (diatur DS Bab 2).
- **Motion (standar pemilik, tidak bisa ditawar):** semua transisi memakai **fisika spring** (kekakuan/peredaman terdefinisi per jenis gerakan), overshoot halus pada kemunculan elemen, tidak ada gerakan linear dan tidak ada easing "benar tapi kaku". Daftar putar & player harus terasa hidup: sampul diskala mengikuti drag, elemen masuk berjenjang ≤ 30 ms antar item.
- **Komponen baku:** Baris Lagu, Kartu Rak, Chip Filter, Tombol Putar Mengambang, Mini Player, Lembar Menu Konteks, Indikator Buffer — dirancang sekali, dipakai di semua layar.
- **Ikon & logo:** digambar sendiri (vektor), nama & identitas visual sendiri. Dilarang menyerupai logo/ikon Spotify.

## 8. Standar Teknis & Pengujian

- **Cakupan uji:** logika mesin player (state machine, shuffle, pemulihan URL) wajib uji unit ≥ 90% baris; kontrak provider diuji dengan rekaman respons (fixture) agar perubahan sumber terdeteksi sebagai kegagalan uji, bukan kegagalan di tangan pengguna.
- **Uji UI emas (golden):** layar-layar kunci dibandingkan tangkapannya pada setiap perubahan DS.
- **Uji lapangan:** daftar periksa rilis di perangkat uji kedua kelas Bab 3 + perangkat pemilik (termasuk perilaku agresif penghemat baterai Xiaomi/MIUI terhadap foreground service).
- **Uji cold start:** antrean Putar Otomatis diuji dari profil pengguna ber-riwayat kosong pada kumpulan niche Bab 3 — relevansi sambungan dinilai terhadap Tangga Kemiripan (Bab 5.4), bukan terhadap riwayat.
- **CI:** setiap perubahan menjalankan uji + lint; build rilis hanya dari CI yang hijau.
- **Versi & migrasi:** basis data dan preferensi bermigrasi teruji (uji migrasi dari tiap versi sebelumnya); kegagalan migrasi tidak boleh menghapus data — jalur aman: cadangan otomatis sebelum migrasi.
- **Rilis bertahap per fase** ke panel uji Bab 3 dulu, lalu publik; fase dinyatakan selesai oleh gerbang Bab 3, bukan oleh tanggal.

## 9. Roadmap Fase (urutan pembangunan — semua sudah dirancang di atas)

- **F1 — Fondasi & Inti:** Clai DS v1, arsitektur provider + YouTube Music, beranda dasar, pencarian, halaman konten, mesin player & queue lengkap (tanpa audio-lanjutan), mini/player penuh, koleksi, playlist pengguna, file lokal, pengaturan dasar, observabilitas lokal. *Keluar F1 bila:* semua gerbang Bab 3 yang relevan lulus pada fitur F1.
- **F2 — Pendalaman Audio & Offline + Fitur Tanda Tangan:** crossfade, gapless, normalisasi, EQ, sleep timer, Smart Shuffle, lirik sinkron, unduhan & mode offline, beranda personal lokal, aksen dinamis, **Mode Berkendara & Volume Ikut Kecepatan berpagaman, Antrean Suara, Tangkap dari Mana Saja, Lanjut Besok (Bab 5.14)**.
- **F3 — Kepemilikan Penuh:** statistik/Wrapped lokal, ekspor-impor koleksi, Connect versi Clai (LAN).
- **F4 — Ekspansi Konten:** podcast/audiobook, kecepatan putar, Connect via internet (relay opsional), platform kedua (desktop/iOS) bila pemilik memutuskan.

## 10. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Sumber (InnerTube) berubah/diblokir | Putar gagal massal | Lapisan provider + uji kontrak ber-fixture; kegagalan terdeteksi dini; provider cadangan dapat ditambah tanpa menyentuh player |
| URL aliran kedaluwarsa di tengah antrean panjang | Lagu berhenti | Pembaruan URL proaktif (5.4 Pemulihan) + uji antrean 100 lagu |
| Penghemat baterai agresif (MIUI dsb.) membunuh putar latar | Putar berhenti saat layar mati | Foreground service + panduan izin per perangkat di dalam aplikasi; uji khusus perangkat pemilik |
| Kemiripan berlebihan dengan Spotify (produk kini publik & open source) | Masalah identitas/hukum di distribusi umum | Clean-room (4.1) tanpa pengecualian "penggunaan pribadi": tanpa aset/kode/nama/ikon mereka; nama "Clai" adalah kata ciptaan sendiri yang dapat dimiliki (bukan nama artis/merek); identitas visual sendiri (Bab 7); catatan metode clean-room disimpan di repo sebagai bukti proses |
| Jalur InnerTube adalah API tidak resmi (di luar ToS layanan sumber) | Sumber dapat ditutup/diblokir; distribusi publik memperbesar permukaan risiko | Lapisan provider (pemisahan total dari player) + provider alternatif dapat ditambah komunitas; risiko ini dinyatakan terbuka di README repo — pengguna & kontributor tahu persis wilayahnya; tidak ada aset/kode sumber yang didistribusikan Clai |
| Scope membengkak menunda rilis inti | Produk tidak pernah jadi | Fase mengunci urutan (Bab 9); fitur baru masuk fase berikutnya, bukan menyusup ke fase berjalan |

## 11. Metrik Keberhasilan Produk

Produk publik diukur dari penggunanya, bukan dari pemiliknya:

- **Metrik utama — aktivasi pengguna baru:** pengguna yang baru memasang berhasil memutar lagu pertamanya ≤ 2 menit sejak aplikasi pertama dibuka, dan Putar Otomatis menyambung tanpa campur tangan setelah konteks pertamanya habis. (Diukur pada panel uji pengguna baru tiap rilis mayor; aplikasi sendiri tidak bertelemetri — Bab 0.)
- **Keandalan lapangan:** rasio sesi tanpa kegagalan putar ≥ 99% pada panel uji lintas kedua kelas perangkat Bab 3, dipertahankan setiap rilis.
- **Integritas data:** 0 kehilangan data koleksi/playlist melewati update versi apa pun, dibuktikan uji migrasi Bab 8 — bukan oleh laporan anekdot.
- **Adopsi open source:** rilis dianggap hidup bila ada pemasangan & isu/laporan dari pengguna di luar pemilik pada 30 hari pertama sejak rilis publik (terlihat dari kanal distribusi & repo), dan waktu tanggap isu kritis playback ≤ 7 hari.
