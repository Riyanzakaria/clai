# Logo Resmi Clai

Ditetapkan 23 Oktober 2026. Mark digambar sebagai vektor (SVG) — bukan hasil generator gambar.

## Konstruksi

- **Mark** (`clai-mark.svg`): huruf **C** geometris berupa busur berpenampang tebal (grid 96, pusat 48, radius 30, tebal garis 13, ujung membulat, celah ±76° di sisi kanan) yang **memegang segitiga putar solid** di tengahnya (titik 42,35 → 64,48 → 42,61). Satu tanda memuat dua bacaan sekaligus: huruf pertama nama, dan tindakan memutar musik.
- **Wordmark:** "Clai" diset dengan **Space Grotesk SemiBold (600)**, huruf kecil selain C kapital, letter-spacing −0,01em. Lockup horizontal: mark di kiri, wordmark sejajar garis dasar mark (lihat `clai-lockup.svg`).

## Warna resmi

| Pemakaian | Mark | Latar |
|---|---|---|
| Utama (ikon aplikasi, splash) | Volt `#B8F135` | Ink `#0A0A0E` |
| Balik (tombol/lencana terang) | Ink `#0A0A0E` | Volt `#B8F135` |
| Mono gelap | Putih `#FFFFFF` | Hitam / transparan |
| Mono terang | Hitam `#000000` | Putih / transparan |

Tidak ada versi gradasi, bayangan, atau garis tepi (outline). Mark tidak pernah diwarnai di luar empat pemakaian di atas.

## Aturan pakai

- **Ruang aman:** sekeliling mark ≥ 13 satuan grid (satu tebal garis) di semua sisi; pada ikon adaptif Android, mark menempati zona aman tengah 66%.
- **Ukuran minimum:** mark 20dp; lockup tinggi 24dp. Di bawah itu pakai mark saja tanpa wordmark.
- **Dilarang:** memutar/mencerminkan mark; mengubah proporsi busur atau segitiga; menaruh mark Volt di atas latar Volt atau latar berwarna cerah lain di luar tabel; menambahkan efek (glow, bayangan, bevel); mengganti wordmark dengan font lain.
- Segitiga selalu menunjuk kanan dan selalu solid — tidak ada versi garis.

## Berkas

| Berkas | Isi |
|---|---|
| `clai-mark.svg` | Mark master (Volt, latar transparan) |
| `clai-lockup.svg` | Lockup mark + wordmark (referensi proporsi) |
| `clai-mark-512.png` / `clai-mark-192.png` | Ekspor raster mark untuk repo/dokumen |
| `logo-sheet.png` | Lembar presentasi: ikon, varian balik, lockup, rampa ukuran, mono |
