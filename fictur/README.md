# FicTur

Asisten AI percakapan yang mengubah cerita/deskripsi/ide fiksi menjadi gambar,
dilengkapi puluhan tool produktivitas & visualisasi (flowchart, mindmap, quiz,
flashcard, deploy tool, dsb).

## Status project ini

Project ini adalah hasil **pemisahan struktural murni** dari file asli
`fictur-272-fixed.html` (1 file HTML ~347KB berisi seluruh CSS + JS inline)
menjadi struktur folder modular:

```
fictur/
├── public/
│   ├── index.html        # markup + <link>/<script> ke semua modul
│   └── assets/           # kosong — source asli tidak referensi asset lokal
├── src/
│   ├── css/               # 15 file, dipecah per fungsi (lihat di bawah)
│   └── js/                 # 27 file, dipecah per tanggung jawab
├── package.json
├── vercel.json
├── .gitignore
└── README.md
```

**Yang SUDAH dilakukan:** pemisahan file. Setiap baris CSS dan JS telah
diverifikasi identik (`diff` byte-per-byte = 0 perbedaan) dengan source
asli — tidak ada logika, nilai, endpoint, atau perilaku yang diubah sedikit
pun. Semua file JS digabungkan sebagai `<script>` tag biasa (bukan ES
module) dalam urutan persis seperti definisi aslinya, supaya seluruh
variabel scope global (`history`, `chatEl`, `pendingImageDataUrl`, dst.)
tetap saling terhubung tanpa risiko putus.

**Yang BELUM dilakukan (di luar cakupan sesi ini):**
- Backend Vercel Functions (auth, API key, admin, IP whitelist, rate limit)
- Migrasi ke Supabase
- Konversi ke ES Modules asli (`import`/`export`)
- Perubahan terhadap endpoint AI pihak ketiga yang dipakai source asli

## Menjalankan secara lokal

Karena `index.html` memuat CSS/JS lewat path absolut (`/src/css/...`,
`/src/js/...`), buka lewat static server — jangan buka file langsung
lewat `file://`.

```bash
npm run dev
# atau
npx serve public
```

Lalu buka `http://localhost:3000` (atau port yang ditampilkan).

## Deploy ke Vercel

1. Push folder ini ke GitHub repo.
2. Import repo di Vercel.
3. Vercel akan otomatis mendeteksi `public/` sebagai output directory
   (lihat `vercel.json`) — tidak ada build step, tidak ada environment
   variable yang wajib diisi untuk versi frontend-only ini.
4. Deploy.

## Struktur CSS (`src/css/`)

| File | Isi |
|---|---|
| `variables.css` | Theme tokens (warna, dsb) untuk dark/light |
| `base.css` | Reset dasar, ambient background, scrollbar |
| `theme.css` | Tombol toggle dark/light |
| `layout.css` | Header + input dock |
| `components.css` | Model picker (dropdown) |
| `chat.css` | Area chat utama |
| `messages.css` | Image placeholder, footer action row pesan |
| `markdown.css` | Rendering teks/tabel markdown |
| `code.css` | Code block, modal preview kode |
| `documentation.css` | Layar dokumentasi/onboarding |
| `tools.css` | Seluruh 34 tool eksternal + quiz + flashcard + soal essay dst. |
| `sidebar.css` | Sidebar riwayat chat |
| `responsive.css` | Media query mobile |
| `auth.css`, `admin.css` | **Kosong** — source asli tidak punya UI auth/admin |

## Struktur JS (`src/js/`)

Diberi nomor urut (`01-` s/d `27-`) karena harus dimuat sebagai
`<script>` tag berurutan (non-module) — urutan ini **tidak boleh
diubah** tanpa memverifikasi ulang dependensi antar file, karena banyak
fungsi bergantung pada variabel global yang dideklarasikan di file
bernomor lebih kecil.

Ringkasan isi tiap file ada sebagai komentar di bagian atas masing-masing
bila relevan; nama file mengikuti taxonomy yang diminta (`config`,
`state`, `history`, `theme`, `ui`, `models`, `attachments`, `chat`,
`streaming`, `markdown`, `quiz`, `tools`, `visual-tools`, `deployment`,
`flashcard`, `code`) — beberapa tanggung jawab besar (mis. "tools")
terpecah jadi beberapa file bernomor (`14-tools.js`, `16-tools-2.js`, dst.)
karena volumenya besar dan saling terkait lewat dispatcher `runTool()`
di `14-tools.js`.

## Catatan penting soal endpoint pihak ketiga

Source asli memanggil sejumlah endpoint pihak ketiga langsung dari
browser (proxy CORS worker, `notrack.ai`, layanan downloader/upscaler/
QRIS tak resmi, dsb.) — **semuanya dipertahankan apa adanya** dalam
proses pemisahan ini, sesuai instruksi "jangan ubah apa pun". Tidak ada
audit keamanan atau keputusan produksi yang diambil di tahap ini;
migrasi endpoint tersebut ke backend (kalau diperlukan) adalah pekerjaan
terpisah yang belum dikerjakan.

⚠️ **Peringatan sebelum push ke GitHub publik:** `src/js/24-chat-media.js`
berisi API key Pexels dalam bentuk plaintext (`PEXELS_API_KEY`), persis
seperti di source asli. File ini belum diubah sesuai instruksi, tapi kalau
repo ini akan publik, sebaiknya key tersebut dicabut/rotate dan dipindah
ke environment variable + backend proxy sebelum di-push — bukan sesuatu
yang aman dibiarkan di client-side kode publik.
