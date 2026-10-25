# Hurahopi — Rencana Pengembangan

[Dokumentasi utama](../README.md).

| Informasi | Keterangan |
| --- | --- |
| Revisi | 26 Oktober 2026 |
| Status | Keputusan prioritas dan pilihan modul open source ditetapkan; siap memulai fondasi dan prototipe |
| Kondisi saat ini | Dokumen dan referensi tersedia; aplikasi, kontrak OpenAPI executable, migrasi, dan pengujian belum dibuat |
| Sasaran Fase 1 | Matematika berbahasa Indonesia untuk anak 4–10 tahun, dengan aktivitas pendukung bahasa, visual-spasial, dan kreasi |
| Sasaran implementasi pertama | Pilih profil → penempatan singkat → misi sekitar lima menit → apresiasi → laporan berbukti |

## Cara membaca rencana

Panduan seluruh dokumen, otoritas, ERD, dan indeks alur fitur terpusat di [README utama](../README.md). Plan ini berisi prioritas, tahapan, dan progres; pekerjaan implementasi dirinci pada [delivery](delivery.md).

Status **Ditetapkan** berarti telah disetujui pemilik produk. **Parameter uji** dapat diprogram untuk prototipe, tetapi memerlukan kalibrasi pendidik/uji keluarga. **Rekomendasi** belum menjadi keputusan produk; **Terbuka** mencatat hal yang belum ditetapkan. Default teknis tidak mengubah ketentuan produk.

## Keputusan yang berlaku sekarang

- **D1:** shadcn/ui varian Base UI dan Tailwind; layout orang tua diadaptasi dari Satnaing; panel admin memakai Shadcn Admin Kit; aktivitas anak dibuat khusus.
- **D6:** modul pada [tabel arsitektur](architecture.md#stack-inti-dan-layanan-pendukung) ditetapkan: scaffold Go Blueprint, Goose, TanStack, OpenAPI TypeScript/Fetch, OAuth2/OIDC, dnd-kit, Konva, River, Gotenberg, Resend, dan Sentry. React/Vite/TypeScript, Go/Fiber, PostgreSQL/pgx, Cloudinary, Compose/Caddy, VM, serta iPaymu tetap fondasinya. Verifikasi akun/integrasi dilakukan pada milestone masing-masing.
- **D9:** langganan per anak; satu sesi belajar aktif per anak; orang tua dapat membuka laporan/pembayaran dari perangkat lain. Pemindahan belajar dikonfirmasi di area dewasa dan mencabut lease perangkat lama.
- **D13:** pra-generasi soal AI, validasi otomatis, dan peninjauan pendidik untuk semua soal sebelum terbit.
- **D14:** maksimal dua pengasuh per keluarga; izin per profil; mendampingi dan melihat laporan aktif pada izin awal; portofolio dan ekspor PDF memerlukan aktivasi terpisah.
- **D15:** tabel/kolom `snake_case`; PK nama entitas tunggal + `_id`, misalnya `users.user_id`. FK mengikuti PK tujuan dengan awalan peran bila perlu; model aplikasi mengikuti [ERD](database/design.md).
- **D4 sebagian:** satu bab contoh dengan dua templat inti; parameter naik tingkat 8/10 pada minimal dua sesi dan dua templat menjadi baseline prototipe. Validasi kurikulum dan kelayakan pedagogis tetap pekerjaan pendidik.

Aturan satu perangkat untuk seluruh login pada draf lama **digantikan oleh D9**. Tidak ada pencabutan seluruh login ketika orang tua membuka akun pada perangkat baru.

## Tahapan pelaksanaan

| Tahap | Hasil | Batas selesai |
| --- | --- | --- |
| 1. Validasi rancangan | Keputusan prioritas, contoh bab, rancangan layar, prototipe, dan hasil uji putaran kecil | Dokumen baseline tersedia; prototipe dan peninjauan pendidik belum dilakukan |
| 2. Fondasi dan alur inti | Akun, persetujuan, mode anak, lease belajar, OpenAPI, migrasi, editor minimum, penempatan, dua templat, progres, laporan dasar | Alur frontend–API–database bekerja dengan data sintetis; skenario penolakan akses dan retry lulus |
| 3. Media, AI, dan kolaborasi | Suara, kanvas, portofolio, draf soal AI/antrean peninjauan, pengasuh, PDF | Izin dan siklus hidup media teruji; data anak ke AI hanya setelah D2/D5 dan persetujuan siap |
| 4. Kelengkapan komersial | Trial, checkout iPaymu, segmen akses, worksheet, olimpiade, biaya layanan, penghapusan, deployment, restore | D3/D6 terkait/D7/D8/D10 dan verifikasi pembayaran/restore selesai sesuai cakupan |
| 5. Konten dan uji keluarga | Seluruh bab Fase 1, aktivitas pendukung, pengantar bab, serta uji 10–20 keluarga | Konten ditinjau pendidik, cakupan usia terpenuhi, hasil uji dan perbaikan terdokumentasi |
| 6. Peluncuran bertahap | Akses publik setelah gate teknis, konten, dan operasional | Seluruh kriteria [operasional](operations.md) terpenuhi |

Fondasi teknis Tahap 2 dapat dimulai paralel dengan validasi Tahap 1 menggunakan data sintetis dan konten fixture. Ini tidak berarti prototipe sudah diuji atau konten sudah disetujui untuk anak. Tahap 2 juga membuktikan unggah, pemutaran, dan penghapusan media sintetis sebelum media anak digunakan.

## Mulai coding dari sini

Kerjakan **S01 → S02 → S03** dalam [backlog delivery](delivery.md#backlog-implementasi-pertama): bootstrap repo dan pemeriksaan dasar, kontrak OpenAPI, tipe/client bertipe, serta migrasi Goose bertahap mengikuti [ERD dan urutan implementasi](database/design.md#urutan-implementasi), lalu identitas keluarga dan mode anak. Setelah itu bangun konten contoh dan alur belajar. Rancangan layar bisa diprototipekan bersamaan menggunakan fake API yang mengikuti kontrak.

Pilihan modul D1/D6 sudah selesai. D2, D3, D5, D7, D8, D10, dan produksi konten D12 tetap memiliki batas penetapan tersendiri; kesiapan akun merchant/email/monitoring diperiksa saat integrasi. Coding fondasi memakai fake provider/fixture internal sampai prasyarat penggunaan layanan/data nyata terpenuhi.

## Ukuran keberhasilan dan progres

- [x] Keputusan prioritas dan penggantian aturan perangkat dicatat.
- [x] Kebutuhan produk, arsitektur, contoh pembelajaran, rancangan API, dan backlog dipisah dengan rujukan silang.
- [x] Hasil riset modul ditetapkan pada arsitektur/D1/D6; dokumen daftar kandidat terpisah dihapus setelah sumber dan trade-off penting diringkas.
- [x] Tata letak utama orang tua/anak dan sketsa diperjelas; activity diagram setiap kelompok fitur ditautkan ke kebutuhan produk.
- [x] ERD Fase 1, kamus data, konvensi snake_case/PK, serta sumber model tersedia dan dipelihara bersama; belum menjadi migrasi SQL.
- [x] Review isi 26 Oktober menutup empat celah rancangan: konteks browser bersamaan, revisi aset setelah review, layanan review sebelum S08, dan publikasi PDF terhadap penghapusan; skenario T14–T17 ditambahkan. Implementasi/pembuktian SQL tetap belum dilakukan.
- [x] Model disederhanakan menjadi 59 tabel Fase 1 dengan 11 tabel untuk S02–S03; field kontekstual seperti avatar/role/provider/locale dan satu attempt untuk penyajian/hasil diselaraskan ke API/ERD.
- [x] Skill lokal Hurahopi dan enam modul anti-slop tersedia melalui AGENTS; desain antarmuka, token, serta hasil pemeriksaan kontras terpusat dan tertaut dari README. UI runtime belum dibangun/diuji.
- [ ] Repo aplikasi dan dependensi dikunci.
- [ ] OpenAPI executable, migrasi, tipe yang dihasilkan, dan client bertipe dibuat.
- [ ] Prototipe dijalankan dan ditinjau bersama pendidik/keluarga.
- [ ] Alur inti bekerja dan pengujian yang relevan lulus.
- [ ] Seluruh konten, fitur, dan gate produksi Fase 1 selesai.

Indikator keberhasilan Fase 1 mencakup konversi trial, retensi mingguan, hari belajar, perpanjangan, kepuasan orang tua, penggunaan suara/gambar, biaya AI per anak aktif, serta kesesuaian penempatan menurut keluarga dan pendidik. Target angka ditetapkan pemilik produk setelah ada data awal, sebelum memutuskan memulai Fase 2.
