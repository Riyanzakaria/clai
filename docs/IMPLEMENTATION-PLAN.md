# IMPLEMENTATION PLAN — Clai F1

**Fase 5** dari proses resmi (`CARA-KERJA.md`). Induk: PRD v2.0 Bab 9 (cakupan F1), `USER-FLOW-DAN-LOGIKA.md` v1.0 (aturan L-01…L-16), `TECHNICAL-BLUEPRINT.md` (skema, kontrak, ADR-001/002), `DESIGN-SYSTEM.md` v3.1.

**Mesin bangun:** PC DESKTOP-O97OFQO (mandat pemilik 03:45 WIB) — Flutter 3.47.7 + JDK 17 + Android SDK (platform 35/34, build-tools 35.0.0) terpasang di `C:\Users\riyan\ayu-workspace\tools\`. Kode sumber hidup lokal di `~/workspace/aplikasi-musik/app/` (VM), disalin ke PC untuk build; **kode tidak di-push ke repo pada lari ini** — hanya dokumen fase.

**Penyimpangan tercatat (ADR-003):** Blueprint D-01 menyebut drift. Implementasi F1 memakai **sqflite** dengan SQL persis skema Blueprint Bab 2 dan migrasi berversi manual + cadangan pra-migrasi (aturan Bab 2.5 tidak berubah). Alasan: tanpa codegen (build_runner) rantai bangun lebih sedikit titik gagalnya dan skema tetap 1:1 dengan dokumen; menukar ke drift kelak adalah perubahan lapisan data saja karena semua akses lewat `LibraryRepository`. Dicatat terbuka di sini, bukan diam-diam.

---

## 1. Tugas Atomik F1

| ID | Tugas | Bergantung pada | Gerbang verifikasi |
|---|---|---|---|
| T-01 | Scaffold proyek Flutter (`com.riyanzakaria.clai`, nama "Clai"), struktur folder per lapisan Blueprint Bab 1, aset logo disalin ke `assets/logo/` | — | `flutter pub get` bersih; folder & pubspec sesuai rencana |
| T-02 | Token DS v3.1 sebagai kode (`core/tokens.dart`: warna, tipografi Space Grotesk + Inter via google_fonts, spasi, radius) + helper spring (`core/spring.dart`, SpringDescription fisik) | T-01 | analyze bersih; token cocok tabel DS Bab 2–4 |
| T-03 | Model domain (`domain/models.dart`): Track, Album, Artist, PlaylistPage, SearchResults, StreamInfo — semua berkunci `(providerId, id)` | T-01 | analyze bersih |
| T-04 | Basis data sqflite (`data/app_db.dart`): skema v1 persis Blueprint 2.1–2.3 (subset F1: tracks, albums, artists, liked_tracks, playlists, playlist_entries, playback_events, queue_snapshots, stream_cache, user_prefs, search_history), migrasi berversi + salinan cadangan pra-migrasi | T-03 | uji unit migrasi v1 terbuka bersih; skema dibandingkan ke Blueprint |
| T-05 | Kontrak provider (`providers/music_provider.dart`) + klien HTTP dio terkonfigurasi (timeout, prioritas, tanpa retry 4xx) | T-03 | analyze bersih |
| T-06 | Provider YouTube Music (`providers/ytmusic/`): konteks klien WEB_REMIX, search (bertahap per kategori), saran pencarian, browse album/artis/playlist sumber, related (endpoint next), artis serupa | T-05 | uji fixture parser (T-16) lulus |
| T-07 | Resolusi aliran: rantai 6 klien Blueprint 5.4, pemilihan format audio per kualitas, `stream_cache` + kedaluwarsa, probe kegagalan → turun rantai | T-05, T-04 | uji unit pemilihan format & urutan rantai dengan fixture |
| T-08 | Mesin antrean murni (`player/queue_engine.dart`): 4 bagian, snapshot konteks (L-15), mutasi serial (L-08), sisip Berikutnya/Later (L-04), konteks bertahan dari sisipan manual (L-03), prioritas repeat (L-09), mode acak Fisher–Yates berjangkar indeks berjalan | T-03 | **uji unit wajib lulus** (T-16) |
| T-09 | Tangga Kemiripan (`player/similarity.dart`): kandidat dari tangga 1–4, penguat tangga 5, saring sinyal sesi (L-06/L-07), anti-drift berjangkar benih, maks 2 artis sama berurutan, batch ±15/isi-ulang-di-5 (L-05 melindungi 3 terdekat) | T-08 | **uji unit wajib lulus** (T-16) |
| T-10 | AudioHandler (`player/audio_handler.dart`): audio_service + just_audio, satu ConcatenatingAudioSource cermin antrean efektif, jendela materi ±60, pemulihan L-10 (segarkan ≤2×, lewati + pemberitahuan bernama, berhenti sesudah 3 gagal berurutan), simpan posisi tiap 5 dtk & snapshot (L-16), matriks interupsi via audio_session | T-07, T-08, T-09 | analyze bersih; uji asap logika pemulihan (penghitung gagal) |
| T-11 | Layanan perpustakaan (`services/library_service.dart`): suka dari mana pun, playlist pengguna (CRUD, susun ulang, duplikat diberi pilihan), riwayat putar & pencarian, statistik lokal dasar | T-04 | uji unit operasi playlist & suka |
| T-12 | File lokal (`services/local_files.dart` + provider `local`): pindai MediaStore via on_audio_query, izin ditunda sampai fitur dibuka, berkas lokal setara di antrean/playlist (F-11 Phase 2) | T-05, T-08 | analyze bersih; izin di manifest sesuai permintaan-saat-dibuka |
| T-13 | Kerangka UI + navigasi 4 tab (Beranda, Cari, Koleksi, Pengaturan dasar sebagai tab/akses dari Koleksi sesuai IA Phase 2 — Pengaturan lewat ikon di Beranda/Koleksi), Mini Player persisten, lembar Player, layar Antrian penuh (seksi + ringkasan + saring > 50) | T-02, T-10 | analyze bersih; struktur cocok DS 6.7/6.9/6.10 |
| T-14 | Layar-layar: Beranda player-first (NowCard, Antrian Berikutnya, rak Terakhir Diputar/Putar Lagi/koleksi terbaru, keadaan kosong), Cari (debounce 300 ms, token L-13, kategori, riwayat pencarian), Halaman Konten (album/artis/playlist sumber), Koleksi (tab + saring + urut), Playlist pengguna, File Lokal, Pengaturan dasar (kualitas, perilaku putar: autoplay on/off + mode transisi terbaca ADR-002, data: hapus cache/riwayat, tentang + catatan clean-room), Observabilitas lokal (log insiden di perangkat + halaman diagnosis) | T-11, T-12, T-13 | analyze bersih; tiap layar punya keadaan muat/kosong/gagal/sebagian per matriks Phase 2 |
| T-15 | Android: manifest ADR-001 (izin FGS mediaPlayback, notifikasi, wake lock, minta abaikan optimasi baterai), `androidStopForegroundOnPause=false`, ikon adaptif dari logo resmi (latar Ink, foreground mark Volt berpadded), label "Clai" | T-01 | manifest tertinjau baris-per-baris vs ADR-001 |
| T-16 | Uji: unit antrean (L-03, L-04, L-05, L-09, konteks-satu-lagu+Repeat-Semua→Autoplay), unit tangga kemiripan (urutan tangga, pagar artis, sinyal skip, dedupe), fixture parser InnerTube (search & browse terpotong teranonimkan), unit migrasi DB | T-04, T-06, T-08, T-09 | `flutter test` semua lulus di mesin bangun |
| T-17 | Gerbang akhir: `flutter analyze` nol error, `flutter test` lulus, `flutter build apk --debug` sukses → APK ke `~/workspace/aplikasi-musik/builds/clai-f1-debug.apk` | semua | **TITIK BERHENTI** — APK menunggu pemeriksaan pemilik; tidak ada klaim "selesai" sebelum itu |

## 2. Yang secara eksplisit BUKAN F1 (agar tidak menyusup)

Lirik (F2), unduhan & mode offline penuh (F2 — skema `download_items` sudah disiapkan Blueprint), crossfade/gapless terukur & EQ & normalisasi (F2, ADR-002), Smart Shuffle (F2), Fitur Tanda Tangan 5.14 (F2), statistik penuh & ekspor/impor & Connect (F3 — skema ekspor sudah dikunci Blueprint Bab 7), podcast (F4). Tombol/lembar untuk fitur F2+ tidak ditampilkan setengah jadi; slotnya ada di rancangan (DS 6.9 Timer, dsb.) tetapi tidak dirender sebelum fasenya.

## 3. Urutan eksekusi

T-01 → T-02/T-03 → T-04/T-05 → T-06/T-07 → T-08/T-09 → T-10 → T-11/T-12 → T-13 → T-14 → T-15 → T-16 → T-17. Uji (T-16) ditulis bersama tugasnya masing-masing, dijalankan penuh di gerbang akhir.

---

## 4. Catatan Eksekusi (ditulis sesudah lari pertama, 23 Okt 2026)

- **Mesin bangun:** PC DESKTOP-O97OFQO (mandat 03:45). Instalasi toolchain awal diam-diam gagal sebagian (jebakan spasi-ekor `set` pada cmd merusak JAVA_HOME → sdkmanager tidak memasang paket); diperbaiki manual & terverifikasi di disk: JDK 17 di `tools\jdk`, Flutter 3.47.7, SDK platform 34/35 (+36 & CMake terpasang otomatis oleh AGP), build-tools 35.0.0.
- **NDK tidak dipakai:** baris `ndkVersion = flutter.ndkVersion` di `android/app/build.gradle.kts` dilepas — tidak ada kode native yang dibangun dari sumber di F1; dengan baris itu AGP mengunduh NDK r28c (±1 GB) tanpa kebutuhan nyata. Build lulus tanpanya.
- **on_audio_query dibuang (deviasi dari T-12):** plugin versi 2.9.0 tidak memiliki `namespace` yang diwajibkan AGP 9 → konfigurasi Gradle gagal total. Diganti MethodChannel Kotlin sendiri ke MediaStore (`MainActivity.kt`, kanal `clai/local_audio`) — perilaku F-11 sama, dependensi tak-terawat hilang.
- **Gerbang T-17 (lari pertama):** `flutter analyze` 0 error/0 warning (11 info); `flutter test` 16/16 lulus; `flutter build apk --debug` EXIT 0 — APK 169.864.706 byte, paket `com.riyanzakaria.clai`, label "Clai" (aapt2 terverifikasi). APK menunggu pemeriksaan pemilik (titik berhenti resmi).
