# Hurahopi — Milestone dan Backlog Pengembangan

[Dokumentasi utama](../README.md).

Tanggal: 26 Oktober 2026. Status seluruh pekerjaan implementasi: **Belum dimulai**. ID di bawah adalah backlog lokal, bukan tiket yang telah dibuat pada tracker eksternal.

Acuan: [rencana](plan.md), [PRD](prd.md#kriteria-penerimaan-produk), [keputusan](decisions.md), [spesifikasi teknis](technical-spec.md), dan [pengujian](operations.md). Tidak ada tenggat tetap; kemajuan dinilai dari hasil yang dapat dijalankan dan bukti pengujian.

## Milestone

| ID | Hasil yang dapat ditinjau | Tahap rencana | Dependensi/gate |
| --- | --- | --- | --- |
| M0 | Scaffold Go Blueprint/Vite, perintah lokal, CI, kontrak awal, tipe/client bertipe, migrasi Goose | Fondasi Tahap 2 paralel validasi Tahap 1 | D1/D6 sudah ditetapkan; catatan asal/lisensi dan kompatibilitas versi tersedia |
| M1 | Satu bab, dua templat, penempatan, misi, laporan dari frontend sampai database | Tahap 1–2 | Data sintetis, fixture akses, draft konten internal; prototipe lalu ditinjau |
| M2 | Editor minimum Shadcn Admin Kit serta pembuktian media sintetis lengkap | Tahap 2 | Adapter API/admin dan review/publikasi konten teruji; proxy media dan penghapusan dibuktikan |
| M3 | Suara, kanvas Konva, portofolio, draf AI, pengasuh, PDF River/Gotenberg | Tahap 3 | D2/D5 untuk data anak sesuai fitur; Resend dan service Gotenberg siap; D14 ditetapkan |
| M4 | Trial, pembayaran, paket, worksheet/olimpiade, deployment dan restore | Tahap 4 | D3/D6 merchant/D7/D8/D10 dan keputusan konten terkait |
| M5 | Konten seluruh usia/materi Fase 1 dan uji 10–20 keluarga | Tahap 5 | D4 lengkap, D12, prasyarat layanan dan data anak terpenuhi |
| M6 | Peluncuran bertahap dan pemantauan | Tahap 6 | Seluruh gate peluncuran lulus dan dukungan tersedia |

M1 bukan versi komersial dan satu bab belum mewakili seluruh target usia 4–10. Fixture konten/akses tidak dipublikasikan sebagai layanan belajar berbayar. Pengujian keluarga pada M1 hanya memakai konten yang sudah ditinjau dan alur deterministik yang memenuhi persetujuan/privasi terkait.

## Backlog implementasi pertama

Persiapan agen dan desain tersedia pada [AGENTS.md](../AGENTS.md), [skill Hurahopi](../skills/hurahopi/SKILL.md), serta [desain antarmuka](desain.md). S01 memetakan token ke fondasi UI; S04 menerapkannya pada layar anak/pendamping. Gate dokumen dan kontras tidak menggantikan build serta uji interaksi prototipe.

Urutan utama **S01 → S02 → S03 → S04 → S05 → S06 → S07 → S08 → S09**. UI dapat dikerjakan bersamaan dengan domain setelah kontrak S02 tersedia; nomor urut bukan kewajiban membuat semua backend sebelum melihat layar.

| ID | Pekerjaan dan dependensi | Kriteria selesai | Kebutuhan |
| --- | --- | --- | --- |
| S01 | Scaffold backend Go Blueprint, frontend Vite React/TS, fondasi Base UI/Tailwind, TanStack Router, Compose PostgreSQL, konfigurasi dan CI | Versi/asal/lisensi dicatat, hasil generator diselaraskan dengan struktur repo, perintah lokal/health jelas, frontend build dan Go compile/vet lulus; tidak membawa auth/layanan demo atau secret | D1/D6 |
| S02 | OpenAPI executable, tipe openapi-typescript/client openapi-fetch, migrasi Goose; setelah S01 | Path/schema/auth/error sesuai spesifikasi; regenerasi tipe konsisten; client cookie/CSRF bertipe; migrasi awal DB kosong berhasil melalui proses terpisah; konvensi PK/FK/ERD divalidasi; skenario constraint disiapkan dan diuji saat tabel terkait dibuat | FR-01/05/06 |
| S03 | Identitas, keluarga, profil, persetujuan, PIN dan mode anak; setelah S02 | OAuth2/OIDC Google dan adapter Resend dengan fake lokal/test; CSRF, token sekali pakai, isolasi keluarga/mode anak lulus; API key/domain nyata diperiksa sebelum pengiriman nyata; River ditambahkan bila email menjadi job asinkron | FR-01/02/03 |
| S04 | Seed bab L1–L4, beranda Hopi, layar aktivitas `choose_group`/`build_group`, peta/koleksi ringkas, dan beranda pendamping dari komponen Satnaing; setelah S02, integrasi setelah S03 | Susunan layar mengikuti UX/sketsa; versi/rubrik/seed tersimpan; layanan/roles/review/publish minimum tersedia dan pergantian aset menggugurkan review; DTO anak terbatas tanpa kunci/data dewasa, audio fixture dan ketuk/keyboard; cache terisolasi; dnd-kit saat drag dikerjakan | FR-03/06/08 |
| S05 | Mesin penempatan dan sesi/lease, pause/resume/transfer, tes ulang dan penyesuaian titik latihan; setelah S03/S04 | Profil dan izin diperiksa; tes bisa sementara; dua start tetap satu sesi; satu lease aktif per auth_session, dua tab tidak dapat memulai anak berbeda tanpa pause eksplisit; transfer perlu konfirmasi dan PIN; generasi stale ditolak; instans/progres sama setelah pindah/reload; generasi lease berasal dari counter anak yang tidak direset saat riwayat dibersihkan; penyesuaian orang tua tidak menciptakan bukti mastery | FR-04/05 |
| S06 | Respons, hint, bantuan, penilaian server dan apresiasi; setelah S05 | Retry logis tidak menggandakan percobaan/hadiah; respons pertama immutable; salah/bantuan/lewat/gagal teknis dibedakan; hint vs jawaban dan transfer vs jawaban diserialisasi | FR-06 |
| S07 | Adaptif, alasan, laporan dan halaman bukti; setelah S06 | Parameter versi disimpan; matriks mastery lulus; narasi memakai bukti; data terbatas jelas; laporan tetap dapat dibaca ketika fixture hak belajar berakhir | FR-07/12 |
| S08 | Integrasi prototipe U01–U11, peninjauan pendidik, putaran uji keluarga yang memenuhi prasyarat; setelah alur S03–S07 | Konten keluarga memiliki review pendidik tercatat dari layanan S04; beranda pendamping dan dunia anak jelas berbeda sesuai UX; handoff PIN/sesi bekerja; frontend–API–DB dapat didemokan; masalah penggunaan diperbaiki, uji tidak mengklaim validitas belajar | FR-01–07 |
| S09 | Editor/review Shadcn Admin Kit, data/auth provider Go, adapter TanStack Router, pembuktian Cloudinary sintetis; setelah S06, dapat paralel S07/S08 | UI memakai layanan review/publish S04; daftar/form/filter/paginasi terhubung API; review/publish melalui aksi domain; revisi menggugurkan review; original/turunan/audio-range terlindungi; upload/play/delete sintetis dibuktikan; media anak menunggu D2 | FR-08/09/14 |

Migrasi mengikuti [ERD, kamus data, dan urutan tabel](database/design.md#urutan-implementasi). S02 membuat kerangka/tabel kursus; setiap cerita berikutnya menambah tabel/constraint yang dipakainya, termasuk FK komposit scope keluarga/kursus. S02–S03 memakai 11 tabel; S04–S07 menambah 26, S09 membuktikan 1 media sintetis, dan 21 lainnya menunggu M3–M5. Seluruh 59 tabel Fase 1 tidak dibuat sekaligus. Model, ERD, kamus data dan kontrak ditinjau bersama; CI menjalankan pemeriksaan aplikasi/migrasi PostgreSQL ketika implementasinya tersedia.

S03 memakai adapter Resend yang sudah dipilih dan fake sender/dev identity lokal untuk pengembangan tanpa akun nyata. Startup produksi menolak bypass/dev identity/fixture akses. Fake sender hanya boleh dipakai di lokal/test; produksi memerlukan pengirim Resend, domain, API key dan konfigurasi email yang sah. Login Google dan tautan email yang berfungsi dengan layanan produksi tetap kewajiban Fase 1.

S04 membuat layanan revisi/review/publish dan kolom roles pengguna dan tabel review minimum. Review dapat dicatat melalui alat internal yang memakai layanan domain yang sama sebelum UI S09 tersedia. Fixture sintetis lokal tetap terpisah dari konten keluarga yang harus disetujui pendidik pada revisi/hash/aset sebenarnya.

S04 menggunakan fixture audio dan aset sintetis/orisinal yang aman dipakai. Pengantar interaktif prototipe tidak mengunci keputusan produksi video D12. Layar editor minimum dibangun dengan form/daftar Shadcn Admin Kit dan aksi domain review/publish; pemeriksaan publikasi tetap berada di backend. Sentry staging diaktifkan sebelum uji eksternal setelah sanitasi event diuji; sebelumnya gunakan log lokal yang sudah disanitasi.

## Kumpulan pengujian per milestone

M1 memerlukan T01–T15 dan T17. T16 masuk M3 saat worker PDF tersedia; nomor skenario tidak berarti seluruh fitur harus dibangun pada M1. T17 menjadi gate sebelum uji keluarga S08, meskipun UI editorial S09 belum tersedia.

| ID | Skenario | Hasil wajib | Jenis verifikasi |
| --- | --- | --- | --- |
| T01 | Orang tua mencoba ID anak keluarga lain | Tidak mengakses data atau mengubah progres | Integrasi API/PostgreSQL |
| T02 | Mode anak memanggil laporan, profil, atau checkout kelak | Ditolak sebelum data dewasa dimuat | Integrasi otorisasi dan e2e |
| T03 | Dua start untuk anak yang sama bersamaan | Tepat satu sesi terbuka dan lease pemilik yang konsisten; baris riwayat lease lama tetap tersedia | Integrasi transaksi |
| T04 | Dua browser belajar pada anak yang sama | Browser kedua perlu konfirmasi; setelah transfer, jawaban lama ditolak; login laporan tetap aktif | Integrasi dan Playwright dua context |
| T05 | Jawaban bersamaan dengan transfer/hint | Urutan commit menentukan hasil tunggal yang konsisten | Integrasi transaksi |
| T06 | Reload/koneksi putus setelah server menyimpan jawaban, termasuk jawaban terakhir dan penerbitan lease | Instans/hasil sama; tidak ada hadiah/progres ganda; recovery tidak mengambil alih lease browser lain atau merotasi lease pengganti setelah transfer keluar-lalu-kembali | Integrasi idempotensi dan e2e |
| T07 | 7/10, 8/10 satu sesi, satu templat, dan 8/10 valid | Hanya 8/10 yang memenuhi distribusi dan prasyarat menaikkan tingkat | Unit domain |
| T08 | Salah lalu benar, hint, bantuan, lewat, gagal teknis | Respons pertama tetap; bantuan bukan mandiri; lewat/teknis bukan salah | Unit dan integrasi |
| T09 | Penempatan terputus dan usia jauh dari kemampuan | Hasil sementara dapat dilanjutkan; tidak mengklaim bab tak diuji | Unit dan e2e |
| T10 | Laporan dengan bukti sedikit, cukup, atau perubahan tingkat | Setiap klaim punya referensi; ambang laporan berbeda dari mastery | Unit dan integrasi |
| T11 | Dua anak dengan akses masing-masing belajar bersamaan | Tidak saling mencabut sesi atau memakai hak akses saudaranya | Integrasi dan e2e |
| T12 | Dev identity/fixture diaktifkan pada environment produksi | Startup menolak; tidak ada bypass hak belajar | Pemeriksaan konfigurasi |
| T13 | Logout, ganti profil, pindah ke mode anak, atau izin dicabut ketika query masih berjalan | Request dibatalkan, cache konteks lama dibersihkan, data dewasa/anak lain tidak muncul | Integrasi TanStack Query dan e2e |
| T14 | Dua tab satu auth_session memulai anak berbeda bersamaan | Maksimal satu lease browser; request kedua ditolak tanpa mengganti konteks; pergantian sah memerlukan pause eksplisit | Integrasi PostgreSQL dan Playwright dua tab |
| T15 | Gambar/audio/binding draf diganti setelah review, atau aset terbit ditimpa | Hash/revisi berubah dan review gugur; storage key/versi terbit serta binding immutable | Integrasi domain/editorial S04 |
| T16 | PDF sedang dirender ketika bukti/media dihapus atau izin dicabut | Finalisasi gagal saat versi/status/izin tidak cocok; hasil dibuang, cancelled tidak kembali ready, unduh ditolak | Integrasi worker/PostgreSQL M3 |
| T17 | Konten akan dipakai keluarga S08 sebelum UI editor S09 tersedia | Review pendidik tercatat melalui layanan S04 pada revisi/hash/aset yang dipakai; fixture lokal tidak dianggap persetujuan keluarga | Gate konten S04/S08 |

Unit backend memakai Go testing, UI logic memakai Vitest bila ada logika yang perlu diverifikasi, integrasi memakai PostgreSQL, dan alur pengguna memakai Playwright. Fokus pada perilaku domain, otorisasi, kontrak, dan transaksi; hindari tes yang sekadar menyalin implementasi tampilan. Perubahan dokumen diverifikasi dengan pemeriksaan tautan dan konsistensi, tanpa membuat tes aplikasi yang belum ada.

## Kriteria selesai setiap cerita

- Perilaku dan skenario penolakan memenuhi PRD serta cerita; contoh dapat didemokan.
- Alur sesuai [activity diagram fitur](../README.md#activity-diagram) dan tata letak UX; perubahan pada diagram/kontrak/kode diselaraskan dalam pekerjaan yang sama.
- ERD/model sumber/kamus data, OpenAPI, tipe hasil openapi-typescript, client openapi-fetch, migrasi Goose, dan kode sesuai; perubahan kontrak diperiksa sebelum digabung.
- Pemeriksaan lint/typecheck/build serta pengujian relevan lulus; kegagalan yang diketahui dicatat secara spesifik.
- Tidak ada kunci jawaban dalam DTO anak, secret dalam repo/log, atau pemeriksaan izin yang hanya berada di UI.
- Catat cara menjalankan dan hasil verifikasi; perbarui dokumen/keputusan yang berubah.
- Fitur yang akun/konfigurasinya belum siap menggunakan fake adapter lokal/test; pilihan provider tetap sesuai arsitektur dan integrasi produksi belum dihitung selesai.
- Versi/asal/lisensi komponen sumber tercatat; tidak ada router, primitive UI, atau layanan template tambahan yang bertentangan dengan stack terpilih.

## Pekerjaan setelah alur pertama

| Kelompok | Hasil yang harus dibangun | Keputusan/prasyarat |
| --- | --- | --- |
| Media/AI | Rekam suara, transkripsi, konfirmasi jawaban, kanvas Konva/react-konva, portofolio Cloudinary, worker River, monitoring biaya | D2, D5, persetujuan orang tua, evaluasi model dan batas pemrosesan; modul media sudah dipilih |
| Editorial AI | Spesifikasi draf, keluaran terstruktur, validator, duplikasi, review, provenance, versi terbit | D13 berlaku; penyedia/anggaran D5 |
| Pengasuh/PDF | Undangan Resend dua slot, izin per anak, pencabutan, ekspor worker River ke Gotenberg internal dan file sementara | D14 berlaku; konfigurasi akun/domain dan service siap, retensi D2 ditetapkan |
| Komersial | Trial, ledger penerima, harga berversi, order per anak/kursus, verifikasi iPaymu, callback/reconcile, segmen akses | D3, D6 merchant, D7, D10 |
| Materi paket | Worksheet, soal/pembahasan olimpiade, akses fitur, pengantar/video | D4, D12; Fase 1 lengkap |
| Operasi | Compose/Caddy, VM app/DB terpisah, backup WAL/off-VM, restore, penghapusan, Sentry/slog dan kapasitas | D2, D8; akun/domain Resend, DSN Sentry, merchant dan layanan produksi siap |

Tentukan detail cerita lanjutan sebelum implementasi milestone pemiliknya. Status D11 tetap Fase 2; menyiapkan course/locale pada skema tidak berarti membangun mata pelajaran tambahan sekarang.

## Bukti penutupan milestone

M0 ditutup dengan startup lokal, kontrak/tipe/client tervalidasi, migrasi Goose, catatan asal/lisensi, dan hasil CI. M1 ditutup dengan demo alur, matriks T01–T15 dan T17, serta catatan peninjauan/uji prototipe; jika uji keluarga belum memenuhi prasyarat, tandai bagian tersebut belum selesai. M2 menambahkan bukti adapter Admin Kit, aksi editorial, dan media sintetis. M3 menyertakan uji kompatibilitas kanvas, retry River, PDF Gotenberg, serta email Resend nyata pada lingkungan yang disiapkan. M3–M6 mengikuti seluruh gate dalam [operasional](operations.md), bukan hanya keberhasilan happy path.
