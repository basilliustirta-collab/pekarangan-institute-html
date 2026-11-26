# Pekarangan (versi statis)

Hasil konversi total dari PHP + SQLite menjadi situs statis: **HTML + CSS + JavaScript + JSON**.
Tidak butuh PHP, database, atau server khusus.

## Cara menjalankan

**Metode utama: hosting statis (disarankan untuk publik)**
- *Netlify*: buka app.netlify.com/drop, seret folder ini. Selesai. (`_headers` dan `_redirects` ikut terbaca.)
- *GitHub Pages*: unggah isi folder ini ke repositori, lalu Settings → Pages → Deploy from branch → `main` / root.
- *Uji lokal lewat server*: `python -m http.server 8080` di folder ini, lalu buka http://localhost:8080

**Metode alternatif (paling mudah): klik dua kali `index.html`.**
Semua halaman sudah berisi konten jadi (tidak memakai `fetch`), jadi bisa dibuka langsung dari folder.
Syarat: internet untuk font Google (tanpa internet tampil dengan font sistem), dan aplikasi email untuk mengirim pesanan/donasi.

## Mengubah isi situs (pengganti panel admin)

Semua isi ada di `data/*.json`. Setelah mengedit:

    python _tools/build.py

Skrip (Python 3, diuji di 3.12, tanpa paket tambahan) menulis ulang seluruh file `.html`. Python hanya dibutuhkan untuk langkah ini, bukan untuk membuka/hosting situs.

| File | Isi |
|---|---|
| `data/site.json` | Semua teks halaman (`teks`) + `whatsapp` (opsional, mis. `"6281234567890"`) |
| `data/products.json` | Produk. Foto: taruh file di `assets/img/` (nama 24 hex + `.jpg`, plus thumbnail `t_...`), isi nama file di `foto`. `aktif: 0` untuk menyembunyikan |
| `data/programs.json` | Program donasi. **Saat ini kosong**, lihat contoh di bawah |
| `data/faq.json`, `data/team.json` | FAQ (halaman Program), tim (halaman Tentang) |
| `data/orders.json`, `data/donations.json` | Catatan untuk halaman Transparansi. **Hanya data publik** (nama sudah disamarkan) |

Contoh isi `programs.json` (membuka kembali form donasi):

    [{"id":1,"fokus":"Belajar","nama":"Ruang belajar bersama","deskripsi":"Perpustakaan dan kelas keterampilan.",
      "target":30000000,"awal":12000000,"sisa_hari":40,"aktif":1,"foto":""}]

Dana terkumpul = `awal` + donasi berstatus `Diproses`/`Tersalurkan` pada `donations.json` yang `program_id`-nya sama.

## Yang berubah dari versi PHP

| Dulu (PHP) | Sekarang (statis) |
|---|---|
| `include header/footer`, loop database | Dirender saat build menjadi HTML jadi |
| `produk.php?id=6` | `produk-6.html` (13 halaman produk) |
| Form pesanan/donasi disimpan ke SQLite | Divalidasi di browser (aturan sama), lalu membuka **email** berisi rincian (atau WhatsApp bila diisi). **Tidak tersimpan otomatis**, pengelola mencatat manual di JSON |
| Filter tanggal transparansi (`?dari=&sampai=`) diproses server | Diproses JavaScript di browser; ringkasan angka dihitung ulang |
| Panel `/admin` (login, CRUD, unggah foto) | Dihapus (tidak mungkin statis). Ganti dengan edit JSON + `build.py` |
| CSRF, sesi, rate-limit, header keamanan PHP | Tidak relevan tanpa server. CSP dipasang lewat `<meta>`, header lain lewat `_headers` |
| Database `.sqlite` | Dibuang dari paket; kolom pribadi (email donatur, kontak pembeli, pesan, hash admin, log IP) **tidak diekspor** |

Folder `data/` ikut publik saat dideploy. Jangan menaruh email, nomor telepon, atau nama lengkap donatur/pembeli di dalamnya.
