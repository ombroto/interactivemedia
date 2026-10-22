# PinKuis H5P — Dokumen Desain

**Tanggal:** 2026-10-22
**Status:** menunggu persetujuan pemilik produk — sudah lewat tinjauan builder
**Repo:** `c:\laragon\www\h5p\intermed` (Next.js 16.4.2, React 19.3, TypeScript strict, Tailwind v4, App Router)
**Design:** canvas "PinKuis H5P" — <https://claude.ai/code/artifact/1b1a7739-acbb-4a72-bd66-0926b09a8bf7> (13 artboard)

Dokumen ini ditulis untuk builder yang akan menyusun rencana implementasi. Setiap keputusan ditandai sumbernya: **[PEMILIK]** = diputuskan pemilik produk dalam sesi brainstorming, **[SAYA]** = keputusan penulis dokumen yang tidak dikonfirmasi ke pemilik, boleh dibantah dengan alasan teknis.

---

## 1. Maksud dan alasannya

Guru/penyelenggara sudah punya konten H5P (biasanya dibuat di [Lumi](https://lumi.education/) lalu diekspor sebagai satu file `.h5p`). Hari ini tidak ada cara mudah membagikannya ke sekelompok peserta dan mengumpulkan skornya: Lumi hanya mengekspor file, dan memasang Moodle/WordPress hanya untuk satu kuis terlalu berat.

PinKuis mengisi celah itu dengan meniru hal yang paling dikenal orang dari Kahoot: **satu PIN 6 angka yang dibagikan ke ruangan**. Pemilik mengunggah `.h5p`, menyetel batas waktu, publish, lalu membacakan PIN. Peserta cukup membuka link, menulis nama, mengerjakan, dan langsung melihat posisinya di antara yang lain.

Yang membuatnya berbeda dari Kahoot: kontennya H5P penuh (Interactive Video, Course Presentation, Question Set — bukan hanya pilihan ganda), dan peserta tidak perlu akun sama sekali.

### Tolok ukur keberhasilan

Seorang guru bisa mengambil file `.h5p` dari Lumi, mengunggahnya, dan membacakan PIN ke kelas dalam waktu di bawah dua menit, tanpa membaca dokumentasi. Peserta masuk hanya dengan link dan nama. Setelah kelas bubar, guru punya tabel skor semua peserta yang bisa diunduh.

---

## 2. Goals

1. Pemilik konten login (akun disiapkan admin), unggah file `.h5p`, dan konten itu bisa diputar utuh dengan runtime H5P asli — semua tipe konten H5P, bukan hanya kuis pilihan ganda. **[PEMILIK]**
2. Satu konten dapat satu sesi: satu PIN 6 angka, satu papan peringkat. **[PEMILIK]**
3. Pemilik menyetel timer pengerjaan per peserta, lalu publish, lalu mendapat PIN dan link siap bagi. **[PEMILIK]**
4. Peserta masuk lewat PIN manual atau link yang PIN-nya sudah terisi, menulis nama, langsung mengerjakan. Tanpa akun. **[PEMILIK]**
5. Saat timer habis, pengerjaan peserta otomatis diselesaikan dengan skor yang sudah terkumpul. **[PEMILIK]**
6. Selesai mengerjakan, peserta melihat 10 nama dengan skor tertinggi, plus posisi dan skornya sendiri walau ia di luar 10 besar. **[PEMILIK]**
7. Pemilik melihat skor seluruh peserta untuk sesinya, dan bisa mengunduhnya sebagai CSV. **[PEMILIK]** (CSV dari design)
8. Peserta mengerjakan kapan saja selama sesi masih dipublish — tidak serentak, tidak ada lobi. **[PEMILIK]**
9. Dalam satu sesi nama peserta harus unik; yang tampil di papan peringkat adalah nama yang ditulis peserta sendiri. **[PEMILIK]**
10. Nama konten yang kembar otomatis diberi angka di belakangnya. **[PEMILIK]**
11. Perangkat utama web desktop; seluruh layar tetap bekerja di lebar ponsel. **[PEMILIK]**
12. Tampilan mengikuti design canvas, memakai konvensi codebase sendiri. **[PEMILIK]**

---

## 3. Non-goals

Hal-hal berikut sengaja **tidak** dibangun di versi ini:

1. **Halaman registrasi.** Tidak ada cara membuat akun dari dalam aplikasi. Akun dibuat admin lewat perintah seed di server. **[PEMILIK]**
2. **Editor H5P di dalam aplikasi.** Design punya tab "Buat di editor H5P" di layar Unggah; pemilik memutuskan versi pertama cukup unggah file `.h5p`. Tab itu tetap digambar tapi berstatus nonaktif (lihat §7.2). **[PEMILIK]**
3. **Permainan serentak gaya Kahoot.** Tidak ada lobi, tidak ada tombol "mulai" host, tidak ada papan skor yang bergerak hidup, tidak ada WebSocket. **[PEMILIK]**
4. **Beberapa sesi dari satu konten.** Satu konten = satu sesi selamanya. Tidak ada "buka sesi baru dengan papan skor bersih". **[PEMILIK]**
5. **Akun peserta.** Peserta hanya punya nama dan cookie. **[PEMILIK]**
6. **Pemasangan library dari H5P Hub.** Library hanya datang dari dalam file `.h5p` yang diunggah. **[SAYA]**
7. **Panel admin.** Tidak ada UI untuk mengelola akun. **[PEMILIK]**
8. **Dua bahasa.** Antarmuka hanya bahasa Indonesia, sesuai design (`<html lang="id">`). **[SAYA]**
9. **Aplikasi mobile native.** Responsive saja. **[PEMILIK]**

---

## 4. Anti-goals

Keadaan yang membuat proyek ini gagal walau fiturnya lengkap:

1. **Peserta tersangkut di layar teknis.** Kalau peserta harus memilih tipe konten, menunggu loading panjang tanpa penjelasan, atau melihat pesan error H5P mentah, produknya gagal. Jalur PIN → nama → konten harus mulus.
2. **Skor hilang.** Peserta mengerjakan 15 menit lalu skornya tidak tercatat karena tab tertutup, jaringan terputus sebentar, atau timer habis — ini kegagalan paling mahal. Skor harus tercatat progresif di server, bukan hanya dikirim sekali di akhir.
3. **Kebocoran kunci jawaban.** Peserta tidak boleh bisa membaca `content.json` konten lain, melihat daftar konten pemilik, atau membuka dasbor skor.
4. **Timer yang bisa diakali.** Batas waktu yang dihitung di browser saja (lalu peserta tinggal mengubah jam komputernya atau menutup tab) membuat fitur ini tidak ada artinya. Waktu dihitung di server.
5. **Papan peringkat yang diisi hantu.** Satu orang mengirim banyak nama palsu lewat API tanpa mengerjakan apa pun akan merusak kepercayaan pada papan skor.
6. **Lokal jalan, Vercel mati.** Apa pun yang bekerja hanya karena ada folder disk di Laragon adalah utang yang harus dibayar saat deploy. Jalur penyimpanan harus sama di kedua tempat sejak hari pertama.

---

## 5. Kendala

### 5.1 Lingkungan

- Pengembangan di Windows + Laragon, `c:\laragon\www\h5p\intermed`. **[PEMILIK]**
- Target produksi **Vercel**. **[PEMILIK]** Konsekuensi yang mengikat: filesystem fungsi serverless bersifat sementara, jadi tidak ada direktori upload permanen; body request route handler dibatasi ~4,5 MB, jadi file `.h5p` tidak boleh melewati server aplikasi; fungsi punya batas durasi (60 detik di plan Hobby).
- Codebase hasil `create-next-app` yang masih kosong: hanya `app/layout.tsx`, `app/page.tsx`, `app/globals.css`. Belum ada komponen, library, atau database. Jadi "konvensi codebase" yang harus diikuti sebatas: App Router, TypeScript strict, Tailwind v4 lewat loader turbopack di `next.config.ts`, path alias `@/*`.
- **Akun layanan belum ada.** Pemilik produk belum punya akun Postgres terkelola maupun Vercel Blob, dan meminta dipandu. **[PEMILIK]** Konsekuensinya langkah pertama rencana — sebelum kode apa pun — adalah instruksi membuat keduanya (tier gratis cukup) dan mengisi `DATABASE_URL` serta `BLOB_READ_WRITE_TOKEN` di `.env.local`, lengkap dengan cara memastikan keduanya hidup. Tanpa itu §6.1 dan §6.2 tidak bisa diuji sama sekali.
- **Skala sasaran: sekitar 100 peserta mengerjakan serentak dalam satu sesi.** **[PEMILIK]** Konsekuensinya: koneksi Postgres dari fungsi serverless harus lewat pooler (connection string pooled Neon, atau driver HTTP-nya) karena seratus fungsi yang membuka koneksi langsung akan menghabiskan kuota; dasbor memakai polling berinterval tetap (10 detik) alih-alih muat-ulang manual; dan papan peringkat dibaca lewat kueri terindeks, bukan menghitung ulang seluruh tabel. Angka ini belum menuntut layanan cache terpisah.
- `AGENTS.md` memperingatkan versi Next.js ini punya breaking change dibanding pengetahuan model; dokumentasi resminya ada di `node_modules/next/dist/docs/`. Builder **harus** membaca panduan yang relevan di sana sebelum menulis route handler, server action, atau konfigurasi cache, karena `next.config.ts` sudah menyalakan `cacheComponents` dan `partialPrefetching`.

### 5.2 H5P

- File `.h5p` adalah arsip ZIP berisi `h5p.json`, `content/content.json`, aset konten, dan folder library yang dipakai. Paket ekspor Lumi ("One .h5p file") menyertakan library di dalamnya, jadi konten bisa diputar tanpa registry library di server.
- Skor H5P tidak bisa dihitung sendiri oleh aplikasi; skor datang dari event **xAPI** yang dipancarkan konten lewat `H5P.externalDispatcher`. Konsekuensinya: tipe konten yang tidak memancarkan skor (Accordion, Timeline, Image Hotspots, dan sejenisnya) tidak bisa dipakai untuk papan peringkat. Design sudah mengantisipasi ini dengan badge "Tidak ada skor" dan tombol "Tidak bisa jadi sesi".
- Konten H5P memuat JS/CSS/font dari jalur relatif terhadap basis yang kita berikan. Basis itu harus URL HTTP yang stabil.

---

## 6. Keputusan teknis

Semua di bagian ini **[SAYA]** kecuali ditandai lain.

### 6.1 Penyimpanan file: Vercel Blob, sejak lokal

Saat pemilik memilih file:

1. Browser meminta token upload ke `POST /api/upload/token` (route handler memakai `handleUpload` dari `@vercel/blob/client`), lalu **mengunggah file langsung dari browser ke Vercel Blob**. File tidak pernah lewat server aplikasi, sehingga batas 4,5 MB tidak berlaku.
2. Setelah upload selesai, browser memanggil `POST /api/contents` dengan URL blob mentah itu.
3. Route handler itu mengunduh blob, membuka ZIP-nya di memori (`fflate`), memvalidasi, lalu mengunggah setiap berkas hasil ekstraksi ke Blob dengan prefiks `h5p/<contentId>/…`, dan menyimpan barisnya di database.
4. Blob `.h5p` mentah dihapus setelah ekstraksi berhasil.

Jalur ini identik di lokal dan di Vercel; yang dibutuhkan di lokal hanya `BLOB_READ_WRITE_TOKEN` di `.env.local`.

**Batas ukuran file: 20 MB.** Angka ini yang mengisi placeholder "[BATAS UKURAN] MB" di design. Alasannya ekstraksi-dan-unggah-ulang harus selesai dalam batas durasi fungsi; 20 MB memberi ruang aman. Lihat §11 risiko 1.

### 6.2 Database: Postgres

Satu Postgres dipakai untuk lokal maupun produksi — **[SAYA]** pilih layanan terkelola (Neon, atau Vercel Postgres) alih-alih memasang Postgres di Laragon, supaya tidak ada perbedaan dialek antara dua lingkungan dan tidak ada langkah pemasangan tambahan di mesin Windows. Satu `DATABASE_URL` di `.env.local` sudah cukup untuk mulai bekerja.

Akses database lewat **Drizzle ORM** dengan migrasi berkas SQL (`drizzle-kit`). Alasan memilihnya di atas Prisma: tidak ada langkah generate biner, jejak bundle kecil, dan jalan mulus di runtime serverless.

Karena sasarannya seratus peserta serentak (§5.1), sambungan ke database **wajib** lewat connection string pooled, bukan sambungan langsung. Skrip seed dan migrasi boleh memakai sambungan langsung karena jalannya di mesin pengembang, bukan di fungsi.

### 6.3 Pemutar H5P: `h5p-standalone`

`h5p-standalone` (v3.x) memutar paket `.h5p` yang sudah diekstrak sepenuhnya di sisi klien. Ia tidak butuh server H5P, tidak butuh registry library, dan tidak butuh PHP.

- Aset inti H5P (`frame.bundle.js`, `styles/h5p.css`, dan kawan-kawan) dari paket npm `h5p-standalone` disalin ke `public/h5p-core/` dan dilayani sebagai berkas statis.
- `h5pJsonPath`, `librariesPath`, `contentPath` **tidak** diarahkan ke URL Blob, melainkan ke route satu-origin `GET /h5p/[contentId]/[...path]`.
- Skor diambil dengan menyimak `H5P.externalDispatcher.on('xAPI', …)`.

**Berkas konten dilayani lewat proxy satu-origin, bukan URL Blob langsung.** Dua alasan bergabung di sini. Pertama, `content/content.json` memuat kunci jawaban, dan URL Vercel Blob bersifat publik permanen: sekali URL-nya terlihat di tab Network, ia bisa diteruskan ke siapa pun dan tetap hidup setelah sesi ditutup — bertabrakan langsung dengan anti-goal §4.3. Kedua, memuat konten dari origin lain adalah risiko terbesar di sisi pemutar (§11.3), karena sebagian library H5P memuat font dan worker dari jalur relatif.

Route proxy itu memeriksa kewenangan lebih dulu — percobaan aktif di sesi yang dipublish, atau sesi login pemilik konten itu (untuk pratinjau) — lalu meneruskan isi berkas dari Blob. Prefiks Blob memakai UUID acak dan tidak pernah muncul di HTML atau respons API. Karena berkas konten tidak pernah berubah setelah diunggah, respons boleh dibubuhi `Cache-Control: private, max-age=31536000, immutable`, sehingga pemuatan kedua dan seterusnya tidak menyentuh fungsi lagi. Batas paket 20 MB (§6.1) membuat biaya meneruskan berkas ini tetap kecil.

### 6.4 Autentikasi

- Email + password. Password di-hash dengan **bcrypt** (`bcryptjs`, implementasi JS murni) — dipilih di atas argon2 karena tidak ada binding native yang perlu cocok dengan runtime Vercel. Cost factor 12.
- Sesi disimpan di tabel `auth_sessions` (token acak 32 byte, hash-nya yang disimpan), dibawa di cookie `httpOnly`, `secure` di produksi, `sameSite=lax`, umur 30 hari. Sesi berbasis tabel dipilih di atas JWT supaya bisa dicabut.
- Tidak ada route registrasi. Pembuatan akun lewat `npm run seed:user -- --email <email> --name <nama> --password <sandi>`, sebuah skrip `tsx` yang menulis langsung ke database. **[PEMILIK]** memutuskan jalur seed; bentuk perintahnya **[SAYA]**.
- Seluruh route di bawah `/app/*` dilindungi; pengunjung tanpa sesi dialihkan ke `/login`.

### 6.5 Identitas dan timer peserta

- Saat peserta mengirim nama, server membuat baris `attempts` dan menaruh token percobaan di cookie `httpOnly` dengan nama yang mengandung id sesi (`pk_attempt_<sessionId>`). Memuat ulang halaman melanjutkan percobaan yang sama; cookie ini satu-satunya hal yang membuat halaman hasil milik dia.
- `attempts.started_at` dicatat server saat peserta menekan **Mulai**. Batas waktu = `started_at + time_limit_seconds`, dihitung **di server**. Angka di layar hanya hitungan mundur kosmetik yang disinkronkan dari tenggat yang dikirim server.
- Skor dicatat progresif: setiap event xAPI berskor dikirim ke `POST /api/attempts/progress` (dibatasi maksimum satu permintaan per 3 detik, plus satu kiriman terakhir lewat `navigator.sendBeacon` saat halaman ditutup). Inilah yang membuat "otomatis selesai dengan skor yang ada" benar-benar punya skor saat waktu habis.
- Penyelesaian terjadi di tiga keadaan: peserta menekan "Selesai dan kirim"; konten memancarkan xAPI `completed` di tingkat akar; atau permintaan apa pun masuk setelah tenggat terlampaui, yang menutup percobaan dengan status `timeout`. Tidak ada cron: penutupan karena waktu habis dievaluasi secara lazy saat percobaan itu disentuh, dan juga saat dasbor atau papan peringkat dibaca.

### 6.6 Aturan skor

- Simpan `score_raw` dan `score_max` per percobaan, diambil dari `result.score.raw` dan `result.score.max` pada statement xAPI.
- Statement dengan verb **`completed`** untuk objek akar (id-nya sama dengan `contentId` akar) adalah sumber paling tepercaya; nilainya menimpa apa pun yang terkumpul sebelumnya.
- Selama itu belum datang, kumpulkan statement **`answered`** per objek anak: simpan nilai terakhir per `object.id`, lalu jumlahkan raw dan max-nya. Ini yang membuat Question Set dan Course Presentation punya skor masuk akal saat waktu habis di tengah jalan.
- Persen = `round(raw / max * 100)`, dihitung saat dibaca, bukan disimpan. `max = 0` → persen nol.
- **Peringkat: `score_raw` menurun, lalu `duration_seconds` menaik.** Percobaan yang belum selesai tidak diberi posisi. Diambil apa adanya dari catatan di design: *"Urutan: skor tertinggi, lalu durasi tercepat. Peserta yang belum selesai belum diberi posisi."*
- **Daftar putih tipe konten berskor.** `h5p.json` → `mainLibrary` dicocokkan ke daftar di `lib/h5p/scorable.ts`. Konten di luar daftar tetap tersimpan dan bisa dipratinjau, tapi tombol "Buat sesi" diganti tombol nonaktif "Tidak bisa jadi sesi", persis seperti di design.

  Daftar itu punya dua tingkat. **Wajib terbukti benar** sebelum pekerjaan dianggap selesai, dengan tes atas statement xAPI nyata: `H5P.QuestionSet`, `H5P.InteractiveVideo`, `H5P.CoursePresentation`. **[PEMILIK]** **Diizinkan tapi belum diverifikasi**, ditandai begitu di kode dan diuji sejauh ada waktu: `H5P.MultiChoice`, `H5P.Blanks`, `H5P.TrueFalse`, `H5P.DragQuestion`, `H5P.DragText`, `H5P.MarkTheWords`, `H5P.Summary`, `H5P.SingleChoiceSet`, `H5P.InteractiveBook`, `H5P.Column`, `H5P.MemoryGame`, `H5P.SortParagraphs`. Tiga tipe pertama itulah yang menentukan apakah aturan skor di atas benar; kalau salah satu dari sembilan sisanya ternyata berperilaku lain, itu perbaikan setelahnya, bukan penghalang rilis.

### 6.7 PIN dan link

- PIN 6 angka desimal, dibuat acak kriptografis, dijamin unik di antara sesi yang belum ditutup. Ditampilkan berkelompok tiga-tiga (`482 913`), tapi disimpan dan diterima tanpa spasi.
- Link peserta: `https://<domain>/join?pin=482913`, sesuai design. Membukanya mengisi kolom PIN otomatis dan memunculkan baris "PIN terisi otomatis dari link".
- PIN sesi yang sudah ditutup tidak dipakai ulang (percobaan lama tetap bisa dirujuk).

### 6.8 Daur hidup sesi dan konten

Semua **[SAYA]**, diturunkan dari aturan "satu konten satu sesi" milik pemilik produk.

- **Menutup sesi bersifat permanen.** Tidak ada "buka lagi": status hanya berjalan `draft` → `published` → `closed`. Karena itu tombol "Tutup sesi" di dasbor wajib memakai dialog konfirmasi yang menyebut jumlah peserta yang sedang mengerjakan dan menyatakan tindakan ini tidak bisa dibatalkan. Peserta yang sedang mengerjakan saat sesi ditutup tetap boleh menyelesaikan percobaannya.
- **Memakai kuis yang sama untuk kelas kedua = unggah ulang berkasnya.** Konsekuensinya penomoran nama kembar (Goal 10) adalah fitur alur utama, bukan kasus pinggiran: pemilik akan sering punya "Kuis Bab 3" dan "Kuis Bab 3 2". Setiap unggahan adalah konten baru dengan sesi, PIN, dan papan peringkatnya sendiri.
- **Menghapus konten yang punya sesi berjalan ditolak.** Tombol hapus di kartu `Upload` baru aktif untuk konten tanpa sesi, atau yang sesinya sudah `closed`. Untuk yang `closed` dan punya skor, dialog konfirmasi menyebut berapa peserta yang datanya akan ikut terhapus; penghapusan meng-cascade ke sesi, percobaan, event, dan berkas Blob. Tidak ada soft-delete — pemilik yang ingin menyimpan hasil mengunduh CSV lebih dulu, dan dialog itu mengatakannya.

---

## 7. Layar dan alur

Tiga belas artboard di canvas memetakan seluruh aplikasi. Artboard `Desk*` adalah acuan utama (desktop); artboard ponsel (`JoinPin`, `JoinName`, `Play`, `Result`, lebar 390) adalah bentuk yang sama di layar sempit, bukan halaman terpisah — satu halaman responsive, bukan dua implementasi. **[PEMILIK]** memilih desktop sebagai perangkat utama.

### 7.1 Peta route

| Route | Artboard | Keterangan |
|---|---|---|
| `/` | `Main` | Kolom PIN besar + tombol Login di header |
| `/login` | — | Tidak ada di design; gaya mengikuti `DeskJoinName` (lihat §8) |
| `/join?pin=NNNNNN` | `DeskJoinPin` / `JoinPin` | PIN terisi dari query, lalu alihkan ke `/p/<pin>` |
| `/p/[pin]` | `DeskJoinName` / `JoinName` | Info sesi + kolom nama + tombol Mulai |
| `/p/[pin]/kerjakan` | `DeskPlay` / `Play` | Pemutar H5P, timer, tombol "Selesai dan kirim" |
| `/p/[pin]/hasil` | `DeskResult` / `Result` | Skor sendiri, posisi, 10 besar |
| `/app/konten` | `Upload` | Unggah `.h5p` + daftar konten |
| `/app/sesi/[id]/atur` | `Settings` | Judul, timer, jendela akses, opsi peringkat |
| `/app/sesi/[id]/bagikan` | `Share` | PIN besar, link peserta, QR |
| `/app/sesi/[id]` | `Dashboard` | Statistik, tabel semua peserta, CSV, tutup sesi |
| `/app/konten/[id]/pratinjau` | — | Tombol "Pratinjau" di kartu `Upload`; memutar konten tanpa percobaan, tanpa timer, tanpa skor tercatat |
| `/app/pengaturan` | — | Butir "Pengaturan" di menu profil; hanya ganti password (§8) |

Setelah login, pemilik mendarat di `/app/konten`.

### 7.2 Penyimpangan dari design, dan alasannya

Design dibuat dengan asumsi **tanpa akun** (catatan kanvas: "Alur pemilik konten (tanpa login)"), sementara pemilik kemudian memutuskan ada login sungguhan. Tiga penyesuaian berikut adalah **[SAYA]**:

1. **Blok "Link pengelola, simpan sekarang" di `Share` dihapus**, beserta tombol "Kirim ke email saya". Teksnya berbunyi *"Karena tidak ada akun, link ini satu-satunya cara membuka dasbor skor"* — alasan keberadaannya hilang begitu dasbor dilindungi sesi login. Membiarkannya berarti membangun jalur akses kedua yang lebih lemah dari password.
2. **Tab "Buat di editor H5P" di `Upload` tetap digambar, tapi nonaktif** dengan `aria-disabled` dan satu baris penjelasan "Belum tersedia". Menghapusnya membuat tab tunggal yang aneh; membuatnya berfungsi adalah non-goal §3.2. Ini meninggalkan jejak yang jelas untuk versi berikutnya.
3. **Baris "Tersimpan di perangkat ini lewat link pengelola (data contoh)"** di `Upload` diganti "Konten milik akun Anda", karena konten sekarang milik akun, bukan browser.

Dua hal lain yang perlu diketahui builder:

4. Teks `(data contoh)` di `Upload` dan `Dashboard` adalah penanda data palsu di mockup, bukan bagian antarmuka. Hilangkan.
5. `Main` di design adalah halaman PIN dengan tombol Login di header, bukan halaman pemasaran. Pertahankan: pengunjung yang datang dari link peserta tidak boleh disuguhi halaman promosi.

### 7.3 Alur pemilik

1. Login di `/login`.
2. `/app/konten`: tarik file `.h5p` ke area unggah. Setelah unggah dan ekstraksi, kartu konten muncul dengan nama berkas, ukuran, tanggal, tipe konten terbaca dari `h5p.json`, badge berskor/tidak, dan status sesi. Panel "Pemeriksaan otomatis file terakhir" melaporkan hasil validasi (struktur paket, library lengkap di dalam paket, tipe konten berskor) dengan baris peringatan bila ada.
3. Tombol **Buat sesi** pada konten berskor membawa ke `/app/sesi/[id]/atur`: judul sesi (tampil ke peserta setelah memasukkan PIN), timer (tanpa batas / batas per peserta dalam menit), jendela akses opsional (dibuka/ditutup), dan satu opsi peringkat (tampilkan 10 besar). Panel ringkasan gelap di kanan memuat tombol **Publish dan buat PIN**.
4. Publish memunculkan `/app/sesi/[id]/bagikan`: PIN raksasa, Salin PIN, Tampilkan layar penuh, link peserta siap salin, dan QR code yang bisa diunduh PNG.
5. `/app/sesi/[id]` adalah dasbor: empat kartu statistik (peserta masuk, selesai, sedang mengerjakan, rata-rata skor), tabel semua peserta (posisi, nama, skor, %, durasi, status) dengan pencarian nama, Unduh CSV, Tampilkan PIN, dan Tutup sesi.

Setelah publish, konten tidak bisa diganti; timer dan jendela akses masih bisa diubah. Jalan masuk untuk mengubahnya adalah tombol **Edit sesi** di kartu konten pada `Upload` (yang di design memang menunjuk ke `Settings`), yang membuka kembali `/app/sesi/[id]/atur` dalam mode terbit — judul, timer, jendela akses, dan opsi peringkat masih bisa disunting, pilihan kontennya tidak. Timer yang diubah **tidak** memengaruhi percobaan yang sedang berjalan, karena tenggatnya sudah disalin ke `attempts.deadline_at` saat peserta menekan Mulai (§6.5).

### 7.4 Alur peserta

1. Buka link (PIN terisi) atau masukkan PIN di `/`. PIN salah, sesi belum dipublish, sudah ditutup, atau di luar jendela akses → pesan yang menyebut sebabnya, tetap di halaman PIN.
2. `/p/[pin]`: kartu biru berisi PIN, judul sesi, badge tipe konten dan batas waktu, serta peringatan *"Waktu mulai berjalan saat Anda menekan Mulai dan tidak berhenti meski tab ditutup."* Di sebelahnya kolom nama (maksimum 40 karakter) dan tombol **Mulai**.
3. Nama sudah dipakai di sesi itu → pesan di kolom nama, peserta memilih nama lain. Pembandingan nama: dipangkas spasi tepi, spasi dalam dirapatkan, lalu case-insensitive. **[SAYA]** Keunikan nama **wajib, tidak bisa dimatikan** — pemilik produk menyatakannya sebagai aturan, bukan pilihan, jadi centang "Tolak nama yang sama dalam satu sesi" di artboard `Settings` dihapus dan keunikan ditegakkan dengan indeks unik di database, bukan pemeriksaan di aplikasi yang rawan balapan. **[SAYA]**
4. `/p/[pin]/kerjakan`: header gelap berisi judul sesi, nama peserta, timer oranye besar, tombol "Selesai dan kirim", dan bar kemajuan. Di bawahnya kartu putih berisi pemutar H5P. Navigasi dan tombol "Periksa" di dalam kartu itu milik H5P.
   **Bar kemajuan di header itu kemajuan waktu, bukan kemajuan soal** — porsi waktu yang sudah terpakai dari batas waktu, dan bar itu tidak muncul sama sekali pada sesi tanpa batas waktu. **[SAYA]** Teks "Pertanyaan 3 dari 10" di artboard `DeskPlay` hanya ditampilkan bila jumlahnya bisa dibaca dari statement xAPI yang sudah masuk, dan disembunyikan bila tidak; aplikasi **tidak** mengurai `content.json` untuk menghitung soal, karena polanya berbeda di tiap tipe konten dan sering tidak diketahui sebelum konten berjalan.
5. Selesai (atau waktu habis) → `/p/[pin]/hasil`: kartu biru dengan skor `8/10` dan posisi `#14`, keterangan "dari N peserta yang sudah selesai", dan catatan bahwa peringkat masih bisa berubah selama sesi dibuka. Di sebelahnya daftar **10 besar**, dan bila peserta di luar sepuluh, barisnya sendiri ditempel di bawah dengan latar oranye muda dan label "(Anda)".

Peserta yang membuka ulang `/p/[pin]` setelah selesai (cookie percobaan masih ada) langsung dibawa ke halaman hasil, bukan disuruh mengerjakan lagi. **[SAYA]**

**Peserta yang kehilangan cookie di tengah jalan** (mode pribadi, ganti perangkat, cookie dibersihkan) tidak bisa melanjutkan percobaannya, dan mengirim nama yang sama akan ditolak karena nama sudah terpakai. Melanjutkan hanya dengan nama tidak boleh: itu membuka pintu siapa pun mengambil alih skor orang lain hanya dengan mengetik namanya. Jadi jalan keluarnya lewat pemilik: **tabel peserta di dasbor mendapat aksi hapus baris**, yang membebaskan nama itu sehingga peserta bisa masuk lagi dari awal. Aksi ini tidak ada di design dan ditambahkan sebagai kolom aksi di ujung baris tabel `Dashboard`, dengan konfirmasi. Pesan "nama sudah dipakai" di layar nama menyebutkan jalan keluar ini agar peserta tahu harus meminta apa. **[SAYA]**

---

## 8. Sistem visual

Diambil langsung dari artboard. Semuanya neo-brutalis: garis tegas 2px `#17181C`, sudut membulat 10–20px, bayangan padat tanpa blur (`6px 6px 0 #17181C`) pada kartu penting.

| Token | Nilai | Pemakaian |
|---|---|---|
| `--color-ink` | `#17181C` | Teks utama, semua garis, tombol gelap |
| `--color-cream` | `#F3F0E8` | Latar halaman pemilik |
| `--color-blue` | `#2446C8` | Latar halaman PIN, kartu penting, aksi sekunder |
| `--color-blue-soft` | `#E6EBFB` | Latar badge, teks di atas biru |
| `--color-orange` | `#F08A3C` | Tombol aksi utama peserta, timer |
| `--color-orange-soft` | `#FFF4E8` / `#FFF0E2` | Panel sorotan, baris "(Anda)" |
| `--color-green` | `#1F7A4D` / `#145536` / `#E3F1E9` | Status terbit, badge selesai |
| `--color-red` | `#A1281C` | Hapus, logout |
| `--color-muted` | `#45474E` | Teks penjelas |
| `--color-rule` | `#D9D4C7` / `#E4E0D5` | Garis pemisah halus |

Tipografi: **Bricolage Grotesque** 700/800 untuk judul dan angka besar (`letter-spacing` negatif −0.02 s.d. −0.035em), **IBM Plex Sans** 400/500/600 untuk teks, **JetBrains Mono** 500/700 untuk PIN, skor, durasi, dan angka tabel. Dimuat lewat `next/font/google`, bukan `<link>` ke Google Fonts, supaya tidak ada permintaan pihak ketiga saat render.

Token didefinisikan sebagai `@theme` di `app/globals.css` (Tailwind v4 berbasis CSS, bukan `tailwind.config.js`), lalu dipakai sebagai utility Tailwind biasa. Artboard memakai inline style karena itu format design component; di implementasi semuanya jadi class Tailwind.

Dua halaman pemilik tidak ada di design dan dibangun dari potongan yang sudah ada: `/app/pengaturan` dan `/login`.

`/app/pengaturan` **[SAYA]** hanya memuat satu hal: ganti password, dengan kolom password lama dan password baru, dan setiap sesi login lain dicabut setelah berhasil. Butir "Pengaturan" di menu profil pada tiga artboard pemilik dipertahankan dan menunjuk ke sini. Alasan memilih membangunnya alih-alih menghapus butir menunya: karena tidak ada registrasi dan tidak ada "lupa password", admin akan membagikan password awal lewat jalur apa pun yang dipakainya, dan pemilik harus punya cara mengubahnya sendiri. Pemilik yang betul-betul lupa passwordnya hanya bisa dibantu admin lewat skrip seed — itu konsekuensi yang diterima, dan halaman login tidak menjanjikan hal lain.

Halaman `/login` **[SAYA]**: bangun dari potongan yang sudah ada — latar krem, kartu putih bergaris 2px dengan bayangan padat, judul Bricolage, dua kolom bergaya `#judul` di `Settings`, tombol gelap bergaya "Mulai" di `DeskJoinName`. Tanpa tautan "lupa password" dan tanpa "daftar", karena keduanya tidak ada.

### 8.1 Aksesibilitas

Design sudah dikerjakan dengan benar di sisi ini dan standarnya harus dipertahankan: elemen `<button>`, `<a href>`, `<input>` + `<label>` yang nyata (tidak ada `div` yang diberi `onClick`), label tersembunyi untuk kolom PIN dan durasi, `aria-label` pada tombol beriikon saja, `role="img"` + label pada SVG bermakna dan `aria-hidden` pada yang dekoratif, tinggi sasaran sentuh minimum 44px, `role="timer"` dengan `aria-label` terbaca pada hitungan mundur, dan `<table>` dengan `<th scope="col">` di dasbor.

---

## 9. Model data

Delapan tabel. Nama kolom `snake_case`.

**`users`** — `id`, `email` (unik), `name`, `password_hash`, `created_at`.

**`auth_sessions`** — `id`, `user_id`, `token_hash` (unik), `expires_at`, `created_at`.

**`contents`** — `id`, `user_id`, `name` (nama tampil, sudah lolos penomoran anti-kembar), `original_filename`, `size_bytes`, `main_library` (mis. `H5P.QuestionSet`), `library_title` (mis. `Question Set`), `is_scorable`, `status` (`processing` | `ready` | `failed`, dipakai §11.1 agar UI tidak menggantung selama ekstraksi), `blob_prefix` (UUID acak, tidak pernah dikirim ke klien), `checks` (JSONB: hasil pemeriksaan otomatis yang ditampilkan di panel `Upload`), `created_at`. Unik `(user_id, name)`.

**Sumber `name`** — judul dari `h5p.json` (`title`) bila ada, kalau tidak nama berkas tanpa ekstensi `.h5p`. **Penomoran anti-kembar**: nama yang sudah dipakai akun itu diberi spasi dan angka urut dari 2 — `Kuis Bab 3`, lalu `Kuis Bab 3 2`, lalu `Kuis Bab 3 3`. Angka dicari dengan mencoba naik sampai menemukan yang kosong, di dalam transaksi yang sama dengan insert, sehingga dua unggahan serentak tidak bisa memenangkan nama yang sama. Tidak ada UI ganti nama konten di versi ini. **[SAYA]**

**`quiz_sessions`** — `id`, `content_id` (unik — satu konten satu sesi, **[PEMILIK]**), `user_id`, `title`, `pin` (6 karakter, unik di antara yang belum ditutup), `time_limit_seconds` (nullable = tanpa batas), `opens_at`, `closes_at` (keduanya nullable), `show_leaderboard`, `status` (`draft` | `published` | `closed`), `published_at`, `closed_at`, `created_at`.

**`attempts`** — `id`, `session_id`, `display_name`, `name_key` (bentuk ternormalisasi untuk uji kembar), `token_hash`, `status` (`active` | `finished` | `timeout`), `started_at`, `finished_at`, `deadline_at` (disalin saat mulai supaya perubahan timer tidak mengubah tenggat yang sedang berjalan), `score_raw`, `score_max`, `created_at`. Unik `(session_id, name_key)` — tanpa syarat, karena keunikan nama wajib (§7.4).

**`rate_limits`** — `key`, `window_started_at`, `count`. Penghitung pembatasan laju §10, dibersihkan secara lazy saat kunci yang sama disentuh.

**`attempt_events`** — `id`, `attempt_id`, `object_id`, `verb`, `raw`, `max`, `received_at`. Menyimpan statement xAPI berskor terakhir per objek; inilah bahan perhitungan skor dan jejak audit kalau ada sengketa nilai.

Indeks yang penting: `attempts(session_id, status, score_raw DESC, duration)` untuk papan peringkat, `quiz_sessions(pin)` untuk pencarian PIN, `attempt_events(attempt_id, object_id)` unik.

`duration_seconds` adalah kolom terhitung dari `finished_at - started_at`, atau dihitung saat dibaca.

---

## 10. Permukaan API

Route handler di `app/api/*`; mutasi dari form pemilik boleh memakai Server Action. Yang penting adalah batas kewenangannya, bukan bentuknya.

**Pemilik (butuh sesi login):**
`POST /api/upload/token` (token upload blob) · `POST /api/contents` (ekstrak + daftarkan) · `DELETE /api/contents/[id]` · `POST /api/sessions` (buat sesi draft) · `PATCH /api/sessions/[id]` (judul, timer, jendela, opsi) · `POST /api/sessions/[id]/publish` (buat PIN) · `POST /api/sessions/[id]/close` · `GET /api/sessions/[id]/attempts` (tabel dasbor, polling) · `GET /api/sessions/[id]/attempts.csv`.

**Peserta (tanpa login):**
`GET /api/public/sessions/[pin]` (judul, tipe konten, batas waktu, status — **tidak** membocorkan apa pun tentang konten selain itu) · `POST /api/public/sessions/[pin]/join` (nama → cookie percobaan) · `POST /api/public/attempts/start` · `POST /api/public/attempts/progress` (statement xAPI) · `POST /api/public/attempts/finish` · `GET /api/public/attempts/result` (skor sendiri, posisi, 10 besar).

Aturan yang tidak boleh dilanggar: peserta hanya boleh menyentuh percobaan yang cocok dengan cookie-nya; berkas konten hanya dilayani lewat route proxy §6.3 yang memeriksa percobaan aktif atau sesi pemilik, tidak pernah sebagai URL Blob; dan setiap route peserta memeriksa tenggat lebih dulu, sehingga permintaan yang datang setelah waktu habis menutup percobaan alih-alih memperbarui skor.

Pembatasan laju diperlukan di `join` (per IP) dan `progress` (per percobaan) untuk melawan anti-goal §4.5. Pada skala seratus peserta serentak, penghitungnya cukup disimpan di Postgres (satu tabel `rate_limits` berisi kunci, jendela waktu, dan hitungan) — **[SAYA]** memilih itu di atas menambah Redis/Upstash, karena satu layanan tambahan berarti satu akun, satu kredensial, dan satu titik gagal lagi untuk keuntungan yang belum dibutuhkan di skala ini.

---

## 11. Risiko

1. **Ekstraksi paket besar bisa melewati batas durasi fungsi.** Membongkar ZIP dan mengunggah ratusan berkas satu per satu ke Blob adalah bagian paling lambat. Penanggulangan: batas 20 MB, unggah paralel berkelompok (mis. 10 sekaligus), `maxDuration` dieksplisitkan di route handler, dan status konten `processing` → `ready` supaya UI tidak menggantung. Kalau ternyata masih mepet, pindahkan ke pekerjaan latar; jangan naikkan batas ukuran file dulu.
2. **Beragamnya statement xAPI antar tipe konten.** Interactive Video dan Course Presentation memancarkan pola yang berbeda dari Question Set. Aturan §6.6 masuk akal secara umum tapi harus diuji dengan paket `.h5p` nyata untuk ketiga tipe wajib sebelum dianggap benar. Ini bagian yang paling pantas dibuktikan dengan tes lebih dulu.
   **Pemilik produk akan membuat konten ujinya sendiri di Lumi dan mengekspornya sebagai file `.h5p`.** **[PEMILIK]** Ini ketergantungan pada orang lain, bukan pada kode, jadi rencana harus menempatkannya lebih awal: minta ketiga berkas itu di awal pekerjaan dan taruh di `test/fixtures/h5p/`. Sementara menunggu, pekerjaan yang tidak bergantung padanya (autentikasi, skema database, layar pemilik) boleh jalan lebih dulu. Statement xAPI yang terekam dari berkas-berkas itu disimpan sebagai fixture JSON supaya tes skor bisa jalan tanpa browser.
3. **Pemutar H5P memuat berkas lewat proxy.** Keputusan §6.3 menghilangkan masalah lintas-origin, tapi memunculkan yang lain: satu konten bisa berisi ratusan berkas library, dan tiap berkas kini satu permintaan ke route proxy. Penanggulangannya header cache `immutable`, tapi pemuatan pertama tetap harus dibuktikan dengan satu paket `.h5p` nyata sebelum apa pun yang lain dibangun. **Ini uji kelayakan hari pertama**; kalau terlalu lambat, jalan keluarnya meneruskan hanya `content/*` lewat proxy dan membiarkan berkas library dilayani dari Blob (library bukan rahasia — kunci jawaban ada di `content/`).
4. **Jendela akses dan timer memakai zona waktu.** Kolom `datetime-local` tidak membawa zona. Simpan semuanya UTC, tampilkan di zona browser, dan tuliskan zonanya di layar `Settings` agar pemilik tidak keliru satu hari.

---

## 12. Yang diserahkan ke builder

- Struktur folder dan pemecahan komponen (`components/`, `lib/`, kolokasi dalam route) — codebase masih kosong, jadi tidak ada konvensi yang harus diikuti selain App Router + alias `@/*`.
- Pilihan kerangka tes dan kedalamannya, dengan satu syarat: aturan skor §6.6, perhitungan tenggat §6.5, dan penomoran nama kembar §6.6/§9 dikembangkan dengan tes lebih dulu, karena ketiganya mudah salah dan mahal kalau ketahuan belakangan.
- Bentuk komponen pemutar H5P (`useEffect` + ref, atau wrapper tersendiri) dan cara membersihkan instansnya.
- Teks pesan kesalahan yang tidak muncul di design (PIN salah, nama terpakai, sesi ditutup, file bukan `.h5p`, ukuran lewat batas) — tulis dalam bahasa Indonesia, nada sama dengan design: kalimat pendek, tanpa jargon, menyebut sebab dan langkah berikutnya.
- Pustaka QR code dan cara menghasilkan PNG untuk diunduh.
- Bentuk polling dasbor (interval 10 detik sudah ditetapkan §5.1; cara mengimplementasikannya bebas).
- Urutan pengerjaan dan pemecahan tugas — itu isi dokumen rencana, bukan dokumen ini.
