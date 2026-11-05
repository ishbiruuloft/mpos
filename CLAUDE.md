# Pedigree Shareable

Versi umum dari sistem pedigree di folder induk. **Tujuannya berbeda:** menghasilkan satu
berkas HTML mandiri yang bisa dikirim ke orang lain, dan bisa dipakai peternak mana pun —
bukan hanya satu pemilik.

Beda pokok dari proyek induk:

| | Proyek induk | Proyek ini |
|---|---|---|
| Sumber data | `test pedigree.xlsx` + skrip generator | Formulir di peramban |
| Identitas pemilik | Ditulis tetap di kode | Diisi lewat `profil.html` |
| Logo | Berkas di `Gambar Mpos/` | Diunggah pengguna |
| Hasil akhir | Halaman lokal + JPG | **Berkas HTML mandiri** + JPG |
| Perlu Terminal | Ya (`update-pedigree.sh`) | Tidak sama sekali |

## Berkas

| Berkas | Peran |
|---|---|
| `profil.html` | Identitas pemilik: nama, alamat kandang, no. telepon, logo. Langkah pertama. |
| `input.html` | Formulir data merpati + unggah foto. Bisa ekspor `.xlsx` bila ingin arsip. |
| `pedigree.html` | Tampilan silsilah, unduh JPG A4, dan **Bagikan HTML**. |

Tidak ada skrip build. Semua berjalan di peramban, dari `file://`, tanpa internet.

## Penyimpanan

Dua kunci `localStorage`:

- `pedigree.profile` — `{ name, address, phone, logo }`
- `pedigree.data` — `{ records: [...], photos: { "<ID>": dataURI } }`

Ketiga halaman membaca kunci yang sama, jadi perubahan langsung terlihat tanpa langkah ekspor.

## Cara "Bagikan HTML" bekerja

`pedigree.html` menyalin dirinya sendiri lewat `document.documentElement.outerHTML`, lalu
mengganti isi blok `/* SHARED:BEGIN */ … /* SHARED:END */` dengan data saat itu:

```js
var SHARED = true;
var SHARED_DATA = { records, photos, profile, dibuat };
```

Pada salinan itu `SHARED === true`, sehingga halaman **tidak membaca localStorage penerima** —
isinya tetap pedigree pengirim, dan data penerima sendiri tidak tertimpa. Tombol bagikan serta
tautan ke `profil.html`/`input.html` ikut dibuang dari salinan karena tidak berguna bagi penerima.

`outerHTML` dipakai karena `fetch()` ke berkas sendiri diblokir pada `file://`. Kotak hasil
render dibersihkan lebih dulu supaya yang tersalin hanya kerangkanya.

## Jebakan

1. **`</script>` harfiah di dalam blok skrip mengakhiri skrip lebih awal** — berlaku juga di
   dalam komentar dan string, karena pemisahnya dilakukan parser HTML, bukan JS. Ini sempat
   mematikan seluruh halaman. Data JSON yang ditanam karena itu dilucuti dengan
   `.replace(/<\//g, "<\\/")`, dan komentar tidak boleh menuliskan urutan itu apa adanya.

2. **Menguji sintaks tanpa Node.** macOS punya JXA:
   ```
   osascript -l JavaScript -e "... new Function(src) ..."
   ```
   `new Function()` memparsing tanpa menjalankan. Penghitung kurung buatan sendiri **tidak bisa
   dipercaya** di berkas ini — regex seperti `/<\//g` dan `\*\//` mengandung `//` yang disangka
   awal komentar, sehingga melaporkan ketidakseimbangan palsu.

3. **Kuota localStorage** 5–10 MB. Foto beresolusi tinggi cepat memenuhinya; `profil.html`
   memperkecil logo ke sisi 600 px sebelum menyimpan. Kegagalan kuota ditangkap dan dilaporkan.

4. Jebakan proyek induk soal **quarantine macOS** tidak berlaku di sini — semua gambar masuk
   lewat unggahan dan langsung jadi data URI, tidak pernah dibaca dari path berkas.

## Aturan tampilan

Sama persis dengan proyek induk — lihat `../CLAUDE.md`: 4 level silsilah, foto hanya sampai
level 2, pohon selalu penuh 15 kotak, kotak kosong bergaris putus-putus, semua kotak putih,
tinggi level 3–4 separuh dan memanjang otomatis bila prestasinya banyak.
