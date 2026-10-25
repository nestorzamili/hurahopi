# Hurahopi — Pengujian, Privasi, dan Operasional

[Dokumentasi utama](../README.md).

Tanggal: 26 Oktober 2026. Status: rencana verifikasi; bukan pernyataan kesiapan produksi.

Bagian dari [rencana pengembangan](plan.md). [Daftar keputusan](decisions.md) mencatat status persetujuan; aturan produk ada di [PRD](prd.md). Parameter uji tetap memerlukan evaluasi pendidik.

## Strategi Pengujian

Arahan visual mengikuti [desain antarmuka](desain.md). Penelaahan dokumen mencakup instruksi skill, sumber vendor, serta hasil perhitungan kontras; hasil ini tidak menyatakan halaman sudah lulus aksesibilitas. Pada prototipe, verifikasi keyboard, fokus, zoom, seluruh state, dan kontras pada permukaan aktual sesuai gate anti-slop.

Alat yang ditetapkan pada D6 adalah Go testing untuk backend/worker, Vitest untuk frontend, integrasi dengan PostgreSQL, dan Playwright untuk alur pengguna. Verifikasi modul mengikuti milestone; pemilihan modul tidak menggantikan pengujian integrasi.

| Area | Skenario Utama |
| --- | --- |
| Akun dan akses | Isolasi keluarga, pencabutan sesi, **pemindahan lease belajar dengan konfirmasi (jawaban perangkat lama ditolak; login orang tua tetap aktif)**, satu sesi belajar aktif per profil dan satu lease aktif per auth_session antar-tab, cakupan mode anak, verifikasi PIN, callback login, tautan sekali pakai, dan CSRF |
| Pengasuh | Penawaran izin undangan tidak memberi akses, penerimaan memindahkan/upsert izin ke membership tanpa mewarisi izin lama, undangan dan penerimaan, izin per profil, larangan pembelian dan penghapusan, pencabutan akses seketika, serta pencatatan perubahan izin |
| Penempatan dan adaptif | Anak jauh di bawah atau di atas usia acuan, penurunan ke prasyarat, kenaikan tingkat, tes yang tidak selesai, tes ulang, dan kestabilan aturan |
| Aktivitas dan sesi | Pemulihan setelah muat ulang atau koneksi terputus, petunjuk, pengiriman ulang, dan penghargaan satu kali |
| Soal AI | Draf dengan kunci keliru ditolak validasi, konten tidak sesuai anak, duplikasi, soal tanpa peninjauan tidak dapat terbit, dan versi konten lama tetap terbaca |
| Analisis dan laporan | Penilaian objektif, pengubahan jawaban suara menjadi angka, transkripsi meragukan, penilaian gambar terstruktur vs karya terbuka, data terbatas, referensi bukti, serta hasil tertunda atau gagal |
| Ekspor PDF | Isolasi data, izin pengasuh, larangan menyertakan media anak pada ekspor pengasuh, kedaluwarsa berkas, pembatalan saat penghapusan, dan publikasi hasil atomik terhadap penghapusan/perubahan data_version, cancelled tidak kembali ready, hasil konsisten pada profil dengan data terbatas |
| Media | Izin mikrofon ditolak, unggah gagal, proteksi file asli/turunan, audio dengan rentang byte, penghapusan, dan invalidasi CDN tertunda |
| Pembayaran | Ledger trial tetap terkait keluarga setelah penghapusan profil sesuai kebijakan, kuota tidak kembali karena profil dibuat ulang, uji coba, status tertunda atau berhasil, callback berulang/terlambat/palsu, verifikasi ke iPaymu, rekonsiliasi, pembayaran kedaluwarsa, perpanjangan, dan segmen akses |
| Hak akses fitur | Worksheet dan olimpiade terkunci pada paket 1 bulan, worksheet terbuka pada 3 bulan, keduanya pada 12 bulan, dan perilaku saat pergantian segmen |
| Worker | Restart, pengulangan aman, hasil logis tunggal, idempotensi penyedia, dan rekonsiliasi respons ambigu |
| Kompatibilitas | Seluruh interaksi, termasuk perekaman suara dan menggambar, pada Android Chrome dan iOS Safari, tablet, serta laptop |
| Review konten | Perubahan binding/checksum aset menggugurkan persetujuan; versi/binding terbit immutable; review S04 tersedia sebelum uji keluarga S08 |
| Integritas dan operasi | Kesesuaian ERD/PK/FK/kontrak API; FK komposit menolak keluarga/kursus yang salah; partial unique menolak sesi/lease ganda; respons pertama milik instans yang benar; satu attempt per slot dengan respons pertama pada sequence 1, penawaran izin tidak efektif, roles admin default kosong, pemulihan database/media dan catatan penghapusan setelah restore |
| Integrasi modul | Regenerasi tipe/client, migrasi Goose terpisah, satu router/primitive UI, adapter admin, sanitasi Sentry/Resend, dan cache yang dibersihkan saat konteks berubah |

Rancangan database dan prosedur pemeriksaannya ada pada [ERD/database](database/design.md#memelihara-rancangan). Ikuti penelaahan pada [README utama](../README.md#pemeriksaan-dokumentasi); pengujian migrasi dan transaksi PostgreSQL tetap dilakukan per milestone. Tambahkan uji email asinkron untuk retry payload yang sama, penghapusan ciphertext, serta secret yang tidak masuk log/argumen River saat fitur worker diaktifkan.

## Kriteria Penerimaan

Kriteria minimum yang diajukan untuk peninjauan:

- Seluruh kasus wajib isolasi keluarga, izin pengasuh, penilaian jawaban di server, pengiriman ulang, hak akses paket, dan satu sesi belajar aktif per anak lulus.
- Tidak ada soal buatan AI yang dapat sampai kepada anak tanpa lolos validasi otomatis dan peninjauan pendidik.
- Setiap kesimpulan dalam kumpulan evaluasi laporan memiliki referensi bukti yang valid; kasus tanpa bukti menghasilkan status data terbatas.
- Callback pembayaran tidak dapat mengaktifkan akses tanpa verifikasi; pembayaran yang callback-nya terlewat berhasil direkonsiliasi.
- Pendidik meninjau peta bab, tingkat, contoh soal, rekomendasi, dan pengamatan pada setiap jenis interaksi, serta hasil tes penempatan pada sampel anak dengan kemampuan berbeda. Rubrik serta ambang kualitas disepakati sebelum penilaian kelayakan peluncuran.
- Uji keluarga menunjukkan bahwa anak usia 4–5 tahun dapat menggunakan alur tanpa membaca, dan sesi harian berada dalam rentang durasi yang diusulkan.
- Pengujian perangkat memverifikasi interaksi, pemulihan sesi, perekaman, dan pemutaran audio. Pengujian beban mengukur latensi API, antrean, bandwidth proxy media, dan unduhan PDF.
- Pemulihan cadangan berhasil pada lingkungan terpisah, termasuk pemeriksaan catatan penghapusan. Target kehilangan data dan waktu pemulihan mengikuti D8.

## Operasional dan Prasyarat Produksi

### Rancangan Operasional

Produksi awal memakai VM aplikasi terpisah dari VM PostgreSQL melalui jaringan privat. VM aplikasi menjalankan Caddy, API, worker River, dan Gotenberg saat fitur PDF tersedia, dengan batas sumber daya masing-masing. Gotenberg dan port database tidak diekspos publik. Pengembangan/pengujian internal dapat memakai satu host Compose. Susunan ini belum mencakup failover otomatis; kapasitas, kebutuhan replika, dan target ketersediaan ditetapkan melalui pengujian serta D8.

Rencana operasional mencakup:

- Image berversi, pemeriksaan kesehatan, penghentian proses secara terkendali, dan restart otomatis.
- Lingkungan pengembangan, staging, dan produksi terpisah, termasuk database, media, kredensial, dan akun iPaymu sandbox/produksi.
- Migrasi SQL Goose dijalankan sebagai langkah deploy terkontrol dengan kredensial migrasi terpisah; API/worker tidak menjalankannya otomatis saat startup. Skema tetap kompatibel dengan aplikasi yang masih berjalan saat rilis.
- Cadangan PostgreSQL di luar VM asal, base backup, arsip WAL, dan uji pemulihan sesuai [dokumentasi PostgreSQL](https://www.postgresql.org/docs/current/backup.html).
- Prosedur cadangan, ekspor, pemulihan, dan penghapusan media Cloudinary sesuai fitur akun serta kebijakan retensi. Lihat [backup dan pengelolaan versi Cloudinary](https://cloudinary.com/documentation/backups_and_version_management).
- Pemantauan sesi, kegagalan media, kegagalan pembayaran dan callback, pergantian perangkat yang tidak wajar, penolakan draf soal AI, kegagalan pembuatan PDF, latensi analisis, **biaya layanan AI per anak aktif**, kapasitas disk, koneksi database, antrean, kegagalan worker, dan hasil backup.
- Pengelolaan pembaruan OS, PostgreSQL, HTTPS, deployment, dan pemulihan.

### Kesiapan Modul dan Layanan Terpilih

Pilihan teknologi sudah ditetapkan pada [arsitektur](architecture.md#stack-inti-dan-layanan-pendukung). Pemeriksaan berikut adalah gate integrasi, bukan daftar kandidat baru:

| Modul/layanan | Bukti sebelum diaktifkan |
| --- | --- |
| Scaffold dan komponen UI | Versi/commit sumber, notice/lisensi, hasil build, satu TanStack Router dan fondasi Base UI; halaman/auth demo tidak terbawa |
| Goose dan OpenAPI TypeScript/Fetch | Migrasi DB kosong/upgrade, hak kredensial migrasi, regenerasi tipe, client cookie/CSRF/error, serta schema runtime sesuai |
| Admin Kit | Data/auth provider Go, filter/sort/paginasi berizin, aksi review/publish tidak dapat dilewati generic CRUD, pencabutan akses berlaku pada request berikutnya |
| dnd-kit dan Konva | Kesesuaian versi React, ketuk/keyboard, Android Chrome/iOS Safari, pemulihan state, batas ukuran karya; tidak memuat kanvas pada halaman yang tidak memerlukan |
| River | Restart/retry, hasil logis tunggal, enqueue transaksional, batas konkurensi, observasi antrean, dan tidak ada token/media mentah dalam job args |
| Gotenberg | Jaringan internal, timeout/konkurensi/batas sumber daya, font/gambar/page break Indonesia, penolakan URL arbitrer, pembatalan/hasil tidak disajikan setelah izin atau data dicabut |
| Resend | Domain pengirim terverifikasi, API key server, penerima dewasa yang benar, retry/idempotensi pengiriman sesuai alur, dan token email tidak masuk log/event |
| Sentry dan slog | DSN/environment terpisah, event hanya memuat error teknis yang disanitasi; session replay/screenshot/PII otomatis nonaktif; body, cookie, PIN, token, jawaban/media/nama anak tidak terkirim |
| iPaymu | Kredensial sandbox/produksi, metode/biaya merchant, validasi callback, pengecekan status server dan rekonsiliasi; konfigurasi merchant dicatat sebagai bukti D6 |

Sanitasi telemetry memakai daftar field yang diizinkan, seperti kode error, nama operasi/route template, release, environment, dan stack teknis yang sudah disaring. Uji dengan data sintetis yang memuat token/nama/jawaban sebagai penanda untuk membuktikan penanda tidak keluar pada event/log. Cache query tidak dipersistenkan untuk data keluarga; logout, mode anak, perubahan profil/izin membatalkan request dan menghapus cache yang tidak lagi berizin. Sanitasi dan pembatasan berlaku juga pada payload galat SDK serta breadcrumb.

### Kesiapan Penyedia AI

Pengembangan dan evaluasi awal menggunakan data sintetis. **Pembuatan soal** tidak mengirim data anak sehingga dapat berjalan lebih dahulu. **Transkripsi suara dan pengamatan gambar** memproses data anak; penggunaannya dengan data anak sungguhan memerlukan persetujuan orang tua serta konfirmasi dukungan, retensi, akses akun, dan kelayakan penyedia AI produksi.

Jika OpenAI digunakan, panduan resminya mensyaratkan **Zero Data Retention** sebelum memproses data pribadi anak di bawah 13 tahun atau usia persetujuan digital yang berlaku. Aktivasi kontrol ini memerlukan persetujuan penyedia; kelayakan endpoint, model, serta pengecualiannya harus diperiksa. `store: false` saja tidak membuktikan pemenuhan prasyarat. Rujukan: [panduan pengguna anak](https://developers.openai.com/api/docs/guides/safety-checks/under-18-api-guidance) dan [kontrol retensi](https://developers.openai.com/api/docs/guides/your-data).

Selama prasyarat ini belum terpenuhi, jawaban suara dan analisis gambar anak sungguhan tidak diaktifkan; anak memakai ketuk-pilih dan kanvas terstruktur yang diperiksa secara deterministik. Retensi pada penyedia AI dan retensi portofolio di Hurahopi diatur terpisah. Kesiapan akun produksi tidak menghalangi penyusunan spesifikasi atau pengembangan dengan data sintetis.

### Kesiapan Peluncuran

Sebelum peluncuran komersial, siapkan VM dan jaringan privat, database serta cadangan di luar VM asal, konfigurasi Cloudinary authenticated, akun layanan produksi (termasuk penyedia AI sesuai [kesiapan penyedia AI](operations.md#kesiapan-penyedia-ai)), **aktivasi merchant iPaymu dan pengujian di sandbox**, domain dan pengirim Resend, konfigurasi Sentry tersanitasi, service Gotenberg internal, peninjau konten, serta dokumen privasi, persetujuan, dan syarat layanan yang sesuai alur produk. Harga, ketentuan penjualan, dan target pemulihan mengikuti [daftar keputusan](decisions.md).

## Daftar pemeriksaan per lingkungan

| Lingkungan | Data dan integrasi | Syarat |
| --- | --- | --- |
| Lokal/internal | Profil, jawaban, gambar, dan audio sintetis; fake provider; fixture akses | Tidak mengirim data anak sungguhan; fixture/dev-login gagal aktif bila environment produksi |
| Staging | Database/media dan akun konfigurasi terpisah; Resend dengan penerima uji dewasa, Sentry terpisah, iPaymu sandbox, Gotenberg saat PDF diuji | Rahasia server terpisah; kontrak, penolakan akses, dan sanitasi diuji; tidak memakai salinan data produksi tanpa proses yang disetujui |
| Uji keluarga awal | Aktivitas deterministik dan laporan dasar | Konten/instruksi ditinjau pendidik, persetujuan sesuai penggunaan; perekaman/media tidak diaktifkan sampai D2 siap; pengiriman data anak ke AI menunggu D5 |
| Produksi komersial | Layanan produksi terpilih dan harga nyata | D2/D3/D5 bila AI data anak aktif/D7/D8/D10/D12 selesai sesuai cakupan; pemeriksaan integrasi D6 serta restore/pembayaran/isolasi/konten lulus |

D8 harus menetapkan angka RPO, RTO, ketersediaan, dan skenario beban bersamaan sebelum klaim kesiapan produksi. Sasaran 500 anak aktif per bulan tidak sama dengan 500 sesi serentak. Jangan membuat target latensi atau biaya produksi seolah telah disepakati; ukur baseline lalu ajukan target.

## Bukti verifikasi yang disimpan

Catat commit/build yang diuji, lingkungan, tanggal, hasil pengujian, versi aturan/rubrik, dan referensi konten. Laporan uji keluarga menyertakan tugas, jumlah peserta per kelompok usia, kesulitan yang diamati, serta perubahan yang diperlukan; jangan menyimpulkan efektivitas belajar dari satu sesi penggunaan.

Verifikasi dokumen ini memakai referensi resmi yang diperiksa pada 25 Oktober 2026. Kebijakan penyedia, fitur akun, dan biaya diverifikasi lagi sebelum integrasi; tautan bukan bukti bahwa akun Hurahopi telah memenuhi prasyarat. [Panduan OpenAI untuk pengguna anak](https://developers.openai.com/api/docs/guides/safety-checks/under-18-api-guidance) dan [kontrol data OpenAI](https://developers.openai.com/api/docs/guides/your-data) mendasari batas pengaktifan AI di atas.
