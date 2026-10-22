# Clai

**Pemutar musik streaming Android, open source.** Player-first, tanpa iklan, tanpa akun wajib — seluruh data pribadi (koleksi, riwayat, statistik) tinggal di perangkat pengguna.

Clai dibangun dengan metode *clean-room*: perilakunya dirancang dari spesifikasi terbuka dan hasil riset produk, tanpa memakai kode, ikon, logo, atau aset milik pemutar mana pun.

## Kenapa Clai

- **Player adalah produknya.** Antrean (queue) yang bebas dan jujur, transisi antar lagu yang andal, dan pemulihan otomatis saat aliran gagal — diperlakukan sebagai fitur kelas satu, bukan tempelan.
- **Musik tidak berhenti.** Saat konteks (album/playlist/hasil cari) habis, **Putar Otomatis Cerdas** menyambung dengan lagu serupa — artis yang sama, lagu terkait, artis serupa, genre & niche yang sama — dengan penjaga anti-drift agar aliran tidak hanyut ke lagu generik.
- **Sumber dapat diganti.** Katalog & audio datang lewat lapisan *provider* yang terabstraksi; sumber pertama YouTube Music (satu sumber untuk metadata + audio, menghindari lagu salah-cocok), sumber lain dapat ditambah tanpa membongkar player.
- **Fitur tanda tangan:**
  - **Mode Berkendara** — kendali penuh tanpa layar: pola tombol perangkat audio + perintah suara, untuk HP di saku, holder, atau dashboard; **Volume Ikut Kecepatan** berkalibrasi & berplafon yang ditetapkan pengguna sendiri.
  - **Antrean Suara** — "tambahin lagu X setelah ini" cukup diucapkan; tanpa LLM, memakai pengenal suara bawaan perangkat.
  - **Tangkap dari Mana Saja** — bagikan teks/tautan/tangkapan layar daftar lagu dari aplikasi apa pun, Clai mencocokkannya.
  - **Lanjut Besok** — pengatur waktu tidur yang mengingat posisi; lanjutkan persis dari semalam dengan satu ketuk.

## Prinsip yang mengikat

1. **Universal** — dirancang untuk pengguna umum: pengguna baru tanpa riwayat, perangkat kelas bawah, jaringan buruk, bahasa Indonesia & Inggris, kebutuhan aksesibilitas.
2. **Tidak naif** — setiap fitur dirancang melawan kenyataan: sensor bisa bohong, jaringan bisa membusuk, sumber bisa berubah; mode gagal selalu aman dan dinyatakan jujur.
3. **Data milik pengguna** — tanpa akun wajib, tanpa telemetri, ekspor/impor koleksi dalam berkas terbuka.

## Status — dibangun per fase, satu fase sampai tuntas

Proyek ini dikerjakan dengan proses blueprint 5 fase (doktrin *Boil the Ocean*: spesifikasi lengkap & kedap sebelum kode ditulis). Lihat [`docs/CARA-KERJA.md`](docs/CARA-KERJA.md).

| Fase | Artefak | Status |
|---|---|---|
| 0 — Riset & Bukti | Bedah perilaku pemutar referensi | ✅ Selesai |
| 1 — PRD | [`docs/SPESIFIKASI-PRODUK.md`](docs/SPESIFIKASI-PRODUK.md) | ✅ Tertutup (v2.0) |
| 2 — User Flow & Logic Mapping | [`docs/USER-FLOW-DAN-LOGIKA.md`](docs/USER-FLOW-DAN-LOGIKA.md) | ✅ Tertutup (v1.0) |
| — Design System (artefak Fase 4, mendahului) | [`docs/DESIGN-SYSTEM.md`](docs/DESIGN-SYSTEM.md) | 🟡 v3.0 — diverifikasi ulang pada Fase 4 |
| 3 — Technical Blueprint & Architectural Critique | — | ⏳ Berikutnya |
| 4 — Design System & Component Guidelines | — | ⏳ |
| 5 — Implementation Plan → Implementasi | — | ⏳ |

Kode aplikasi belum ditulis — dengan sengaja. Blueprint diselesaikan dulu sampai standar, baru implementasi berjalan per tugas atomik dari Fase 5.

## Tumpukan (keputusan awal, dibuktikan ulang di Fase 3)

Flutter + ExoPlayer (via `just_audio`/`audio_service`), Android 10+, arsitektur provider-agnostik, basis data lokal di perangkat.

## Lisensi

Lisensi open source akan ditetapkan sebelum rilis publik pertama dan dicatat di berkas ini.
