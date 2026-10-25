# Hurahopi

Platform belajar matematika berbahasa Indonesia untuk anak 4–10 tahun. Pengalaman utama adalah dunia Hopi untuk anak; orang tua/pengasuh menyiapkan belajar dan membaca laporan, sedangkan admin mengelola konten serta operasi.

**Status:** perencanaan dan rancangan pengembangan tersedia. Aplikasi, kontrak OpenAPI executable, migrasi SQL, dan pengujian aplikasi belum dibuat. Dokumen bukan pernyataan kesiapan produksi.

README ini adalah **satu pintu masuk dokumentasi**. Spesifikasi dipisah menurut tanggung jawab agar aturan tidak disalin ke banyak indeks. Jangan membuat README tambahan di subdirektori; tambahkan tautan ke dokumen topik dari sini. Saat aplikasi tersedia, cara menjalankan dan perintahnya ditambahkan ke README ini.

## Mulai dari sini

1. Baca [PRD](docs/prd.md) untuk kebutuhan dan cakupan, lalu [keputusan](docs/decisions.md) untuk status persetujuannya.
2. Pelajari [UX orang tua/anak](docs/ux-spec.md), [aturan pembelajaran](docs/learning-spec.md), serta [activity diagram](#activity-diagram).
3. Gunakan [arsitektur](docs/architecture.md), [desain database](docs/database/design.md), dan [spesifikasi teknis](docs/technical-spec.md) sebagai acuan implementasi.
4. Ikuti [rencana](docs/plan.md) dan [backlog S01–S09](docs/delivery.md#backlog-implementasi-pertama); jalankan pemeriksaan dokumentasi sebelum mulai coding.

Mulai implementasi dari **S01 → S02 → S03** dengan data sintetis. Retensi, kebijakan trial terperinci, penyedia AI, harga/refund, target operasi, upgrade, dan produksi konten tetap mengikuti keputusan yang masih terbuka.

## Dokumen pengembangan

| Dokumen | Otoritas dan isi |
| --- | --- |
| [PRD](docs/prd.md) | Kebutuhan produk, peran, paket, cakupan, kriteria penerimaan FR-01–FR-14 |
| [Keputusan D1–D15](docs/decisions.md) | Status persetujuan dan alasan; keputusan terbuka tidak otomatis final karena ada rancangan tabel |
| [Plan](docs/plan.md) | Prioritas, tahapan, progres perencanaan, dan urutan mulai coding |
| [Spesifikasi pembelajaran](docs/learning-spec.md) | Penempatan, mastery, rubrik, konten AI, contoh bab, dan laporan berbukti |
| [Spesifikasi UX](docs/ux-spec.md) | Dunia anak, area pendamping/admin, handoff PIN/sesi, wireframe, dan tugas uji |
| [Desain antarmuka](docs/desain.md) | Arah Hopi, dials per area, token warna/font/ruang, komponen/state, responsif, serta batas verifikasi |
| [Arsitektur](docs/architecture.md) | Stack/modul terpilih, sumber/lisensi, akses, layanan, dan siklus hidup data |
| [Spesifikasi teknis](docs/technical-spec.md) | Struktur target repo, transaksi, status, rancangan API, dan contoh payload |
| [Delivery](docs/delivery.md) | Milestone M0–M6, backlog S01–S09, dependensi, dan skenario pengujian |
| [Pengujian dan operasional](docs/operations.md) | Privasi, integrasi, deployment, backup/restore, kesiapan layanan, dan gate peluncuran |

## Skill proyek

[AGENTS.md](AGENTS.md) mengarahkan agen ke skill lokal dan dokumen yang relevan. Folder `skills/` adalah pemasangan lokal proyek yang dibaca melalui pointer tersebut; pemasangan ini tidak mendaftarkan plugin global. Pointer tersedia pada giliran/sesi berikutnya; pada sesi pemasangan, berkas sudah dibaca langsung.

| Skill | Pemakaian |
| --- | --- |
| [hurahopi](skills/hurahopi/SKILL.md) | Mengerjakan perubahan produk/UX/schema/API sesuai keputusan dan milestone Hurahopi |
| [antislop](skills/antislop/SKILL.md) | Core filter; penyelesaian mode, alasan desain, dan Delivery Gate |
| [antislop-ui](skills/antislop-ui/SKILL.md) | Komposisi dan keputusan visual |
| [antislop-copywriting](skills/antislop-copywriting/SKILL.md) | Bahasa produk dan teks |
| [antislop-human](skills/antislop-human/SKILL.md) | Kontras, keyboard, fokus, dan state yang dapat dipahami |
| [antislop-layoutmobile](skills/antislop-layoutmobile/SKILL.md) | Reflow, ukuran sentuh, overflow, dan tata letak lintas lebar |
| [antislop-code](skills/antislop-code/SKILL.md) | Komentar kode yang memberi informasi tanpa mengubah logika saat hanya membersihkan komentar |

Anti-slop berasal dari [repo yang diminta](https://github.com/miqdadbadjuber/anti-slop); [notice/lisensi](THIRD_PARTY_NOTICES.md) dan [manifest hash sumber](skills/antislop/source.json) mencatat snapshot terpasang. UI memakai [docs/desain.md](docs/desain.md) sebagai arah; dokumen itu juga memenuhi fungsi `DESIGN.md` yang disebut upstream, tanpa salinan kedua.

Pilihan During berlaku pada percakapan pemasangan ini. Preferensi global belum disimpan; sesi lain mengikuti resolusi mode pada core. Skill tidak memperbarui dirinya sendiri. Untuk pembaruan, ambil snapshot sumber yang dipilih, tinjau perubahan, lalu perbarui berkas/vendor manifest dan jalankan pemeriksaan sebelum mengganti salinan lokal.

Jika ada konflik, daftar keputusan menentukan status persetujuan, PRD menentukan perilaku produk, dan spesifikasi menerjemahkannya ke implementasi. Selaraskan dokumen terkait sebelum coding. Perubahan model data harus memperbarui model sumber, ERD/kamus data, kontrak, dan backlog yang terdampak.

## Database

Field memakai konteks yang jelas: `avatar`, `email`, `role`, `provider`, `locale`, `price`, dan `currency`; kode lokal cukup `code`. Sufiks PK/FK, hash, waktu, dan unit tetap dipertahankan.

Semua tabel/kolom domain memakai `snake_case`; PK menggunakan nama entitas tunggal + `_id`, misalnya `users.user_id`, `children.child_id`, dan `study_sessions.study_session_id`. FK mengikuti PK tujuan dengan awalan peran bila diperlukan. Gunakan nama yang menjelaskan entitas; jangan menambahkan awalan proyek, `tbl_`, `master_`, atau `transaction_` tanpa makna domain.

Model mencakup **59 tabel untuk seluruh Fase 1**, dengan **11 tabel untuk S02–S03**. S04–S07 menambah 26 tabel, S09 membuktikan 1 tabel media sintetis, dan 21 lainnya menunggu M3–M5. Jumlah tabel mengikuti kebutuhan relasi/riwayat; bukan kuota minimum untuk memulai. Metadata internal Goose/River tetap mengikuti library dan berada di luar tabel domain.

Penyajian soal dan hasil memakai satu `attempt_id`, dengan respons berurutan di `responses`. Detail alasan penggabungan dan data yang tetap relasional ada di [batas model database](docs/database/design.md#batas-model-dan-alasan-tabel).

Review isi 26 Oktober 2026 menyelaraskan satu konteks browser antar-tab, persetujuan konten beserta aset, layanan review sebelum uji keluarga, serta publikasi PDF terhadap penghapusan data. Aturan rinci ada pada desain database dan skenario T14–T17 di delivery; pembuktian SQL dilakukan saat implementasi.

| Dokumen/model | Isi |
| --- | --- |
| [Desain database](docs/database/design.md) | Konvensi, ERD ringkas, integritas, protokol transaksi/data, dan urutan migrasi |
| [ERD akun dan akses](docs/database/identity-access.md) | Keluarga, profil, sesi, persetujuan, undangan, dan izin |
| [ERD kurikulum dan konten](docs/database/content.md) | Materi, keterampilan, versi/rubrik/aturan, aset, aktivitas, dan review |
| [ERD pembelajaran](docs/database/learning.md) | Placement, sesi/lease, respons, attempt, mastery, progres, dan apresiasi |
| [ERD media dan laporan](docs/database/media-reports.md) | Karya/rekaman, observasi, klaim/bukti, snapshot, dan PDF |
| [ERD pembayaran](docs/database/billing.md) | Paket/harga, pesanan, gateway, trial, refund, dan akses |
| [ERD operasional](docs/database/operations.md) | Generasi konten, worker, idempotensi, audit, dan penghapusan |
| [Kamus data](docs/database/data-dictionary.md) | Kolom, tipe, nullable, PK/FK, UNIQUE, CHECK, dan aturan setiap tabel |
| [Model sumber](docs/database/schema.json) | Sumber terstruktur untuk enam ERD dan kamus data; bukan migrasi SQL |

## Wireframe dan referensi

| Berkas | Kegunaan |
| --- | --- |
| [Beranda orang tua](docs/wireframes/parent-home.svg) | Sketsa desktop/ponsel untuk pendamping |
| [Pengalaman anak](docs/wireframes/child-experience.svg) | Sketsa dunia Hopi, aktivitas fokus, dan apresiasi |
| Situs referensi lokal: `references/hurahopi-site/index.html` | Arah visual merek; perilaku produk tetap mengikuti PRD terbaru |
| Copywriting lokal: `references/hurahopi_copywriting_revised.html` | Bahan pesan/merek yang diselaraskan dengan kebutuhan terbaru |
| Folder lokal `references/` | Materi sumber pengguna untuk kajian; keberadaannya tidak memberi hak publikasi ulang |

Wireframe adalah sketsa, belum hasil validasi pengguna. Folder `references/` diabaikan oleh Git dan tetap disimpan secara lokal; clone baru tidak menyertakannya. Path referensi di atas bersifat opsional dan relatif terhadap root proyek. Dokumentasi, skill, dan pemeriksaan dapat dipakai tanpa folder tersebut; arah visual yang diperlukan sudah dicatat pada desain antarmuka. Aset/konten aplikasi mengikuti pemeriksaan hak penggunaan pada spesifikasi pembelajaran.

## Activity diagram

Tiga puluh diagram mencakup FR-01–FR-14. Buka [akun/keluarga](docs/diagrams/account-family.md), [pembelajaran anak](docs/diagrams/learning.md), [laporan/pembayaran](docs/diagrams/parent-billing.md), atau [admin/operasi](docs/diagrams/admin-operations.md). Diagram memakai flowchart Mermaid untuk urutan proses; ERD menjelaskan struktur penyimpanannya.

### Cara membaca diagram

- Bentuk membulat: mulai/selesai; kotak: aktivitas; belah ketupat: keputusan; panah berlabel: kondisi perpindahan.
- Pelaku disebut pada judul/kotak: pendamping, anak, sistem, worker, atau penyedia. ID diagram lain berarti subalur yang dirinci pada diagram tersebut.
- Pemeriksaan identitas, izin keluarga/profil, mode/PIN, lease, dan hak fitur berlaku pada seluruh operasi terkait walaupun tidak diulang pada setiap kotak.
- Label D2/D3/D4/D5/D7/D8/D10/D12 mengacu ke keputusan atau parameter yang belum sepenuhnya final; lihat status aslinya sebelum mengimplementasikan fitur produksi.
- Error jaringan, retry, dan pencabutan mengikuti A06/L09/O05; hasil yang sudah commit tidak boleh digandakan.

### Daftar fitur

| ID | Fitur | Diagram | Kebutuhan |
| --- | --- | --- | --- |
| A01 | Masuk dengan Google atau tautan email | [Buka](docs/diagrams/account-family.md#a01--masuk-dengan-google-atau-tautan-email) | FR-01 |
| A02 | Membuat dan mengubah profil anak | [Buka](docs/diagrams/account-family.md#a02--membuat-dan-mengubah-profil-anak) | FR-02 |
| A03 | Memberikan atau menarik persetujuan | [Buka](docs/diagrams/account-family.md#a03--memberikan-atau-menarik-persetujuan) | FR-02/09/14 |
| A04 | Menyiapkan mode anak dan kembali ke orang tua | [Buka](docs/diagrams/account-family.md#a04--menyiapkan-mode-anak-dan-kembali-ke-orang-tua) | FR-03/05 |
| A05 | Undangan, izin, dan pencabutan pengasuh | [Buka](docs/diagrams/account-family.md#a05--undangan-izin-dan-pencabutan-pengasuh) | FR-10 |
| A06 | Memindahkan sesi belajar ke perangkat lain | [Buka](docs/diagrams/account-family.md#a06--memindahkan-sesi-belajar-ke-perangkat-lain) | FR-05/06 |
| A07 | Menghapus profil atau akun dan data turunannya | [Buka](docs/diagrams/account-family.md#a07--menghapus-profil-atau-akun-dan-data-turunannya) | FR-14 |
| L01 | Tes penempatan dan hasil sementara | [Buka](docs/diagrams/learning.md#l01--tes-penempatan-dan-hasil-sementara) | FR-04 |
| L02 | Beranda anak, misi, dan pengantar bab | [Buka](docs/diagrams/learning.md#l02--beranda-anak-misi-dan-pengantar-bab) | FR-04/05/06 |
| L03 | Pilihan visual dan soal cerita objektif | [Buka](docs/diagrams/learning.md#l03--pilihan-visual-dan-soal-cerita-objektif) | FR-06 |
| L04 | Membuat kelompok, mencocokkan, dan menyusun | [Buka](docs/diagrams/learning.md#l04--membuat-kelompok-mencocokkan-dan-menyusun) | FR-06 |
| L05 | Jawaban suara dan alternatif ketuk | [Buka](docs/diagrams/learning.md#l05--jawaban-suara-dan-alternatif-ketuk) | FR-06/09 |
| L06 | Menggambar, mewarnai, dan karya terbuka | [Buka](docs/diagrams/learning.md#l06--menggambar-mewarnai-dan-karya-terbuka) | FR-06/09 |
| L07 | Evaluasi penguasaan dan titik latihan berikutnya | [Buka](docs/diagrams/learning.md#l07--evaluasi-penguasaan-dan-titik-latihan-berikutnya) | FR-07 |
| L08 | Apresiasi, koleksi, dan akhir sesi | [Buka](docs/diagrams/learning.md#l08--apresiasi-koleksi-dan-akhir-sesi) | FR-06/07 |
| L09 | Pause, muat ulang, dan koneksi terputus | [Buka](docs/diagrams/learning.md#l09--pause-muat-ulang-dan-koneksi-terputus) | FR-05/06 |
| P01 | Beranda orang tua, laporan, dan bukti | [Buka](docs/diagrams/parent-billing.md#p01--beranda-orang-tua-laporan-dan-bukti) | FR-07/10/12 |
| P02 | Ekspor laporan PDF | [Buka](docs/diagrams/parent-billing.md#p02--ekspor-laporan-pdf) | FR-11 |
| P03 | Melihat, mengunduh, atau menghapus portofolio | [Buka](docs/diagrams/parent-billing.md#p03--melihat-mengunduh-atau-menghapus-portofolio) | FR-09/10/14 |
| P04 | Uji coba tujuh hari dan masa akses habis | [Buka](docs/diagrams/parent-billing.md#p04--uji-coba-tujuh-hari-dan-masa-akses-habis) | FR-12 |
| P05 | Membeli dan memperpanjang paket | [Buka](docs/diagrams/parent-billing.md#p05--membeli-dan-memperpanjang-paket) | FR-12/13 |
| P06 | Callback pembayaran dan rekonsiliasi | [Buka](docs/diagrams/parent-billing.md#p06--callback-pembayaran-dan-rekonsiliasi) | FR-13 |
| P07 | Worksheet printable | [Buka](docs/diagrams/parent-billing.md#p07--worksheet-printable) | FR-12 |
| P08 | Latihan olimpiade | [Buka](docs/diagrams/parent-billing.md#p08--latihan-olimpiade) | FR-06/12 |
| O01 | Menyiapkan konten, aset, dan materi paket | [Buka](docs/diagrams/admin-operations.md#o01--menyiapkan-konten-aset-dan-materi-paket) | FR-08/12 |
| O02 | Pembuatan draf soal AI | [Buka](docs/diagrams/admin-operations.md#o02--pembuatan-draf-soal-ai) | FR-08 |
| O03 | Peninjauan, publikasi versi, dan pengarsipan | [Buka](docs/diagrams/admin-operations.md#o03--peninjauan-publikasi-versi-dan-pengarsipan) | FR-08 |
| O04 | Harga, dukungan, penyesuaian akses, dan keluar paksa | [Buka](docs/diagrams/admin-operations.md#o04--harga-dukungan-penyesuaian-akses-dan-keluar-paksa) | FR-12/13/14 |
| O05 | Worker, kegagalan layanan, biaya, dan pemantauan | [Buka](docs/diagrams/admin-operations.md#o05--worker-kegagalan-layanan-biaya-dan-pemantauan) | FR-08/09/11/13/14 |
| O06 | Cadangan, pemulihan, dan pencegahan data terhapus muncul kembali | [Buka](docs/diagrams/admin-operations.md#o06--cadangan-pemulihan-dan-pencegahan-data-terhapus-muncul-kembali) | FR-14 |

### Pemetaan ke PRD

| Kebutuhan | Diagram utama |
| --- | --- |
| FR-01 — Identitas/keluarga | A01 |
| FR-02 — Profil/persetujuan | A02, A03 |
| FR-03 — Mode anak | A04 |
| FR-04 — Penempatan | L01, L02, P01 |
| FR-05 — Sesi/perangkat | A04, A06, L02, L09 |
| FR-06 — Aktivitas/progres/apresiasi | L03, L04, L05, L06, L08, L09 |
| FR-07 — Adaptif/laporan | L07, P01 |
| FR-08 — Konten/AI | O01, O02, O03, O05 |
| FR-09 — Media/portofolio | A03, L05, L06, P03 |
| FR-10 — Pengasuh | A05, P01, P02, P03 |
| FR-11 — PDF | P02, O05 |
| FR-12 — Akses/paket/worksheet/olimpiade | P04, P05, P07, P08, O04 |
| FR-13 — Pembayaran | P05, P06, O04, O05 |
| FR-14 — Penghapusan/operasi | A07, P03, O05, O06 |

### Batas implementasi dan konsistensi

Seluruh Fase 1 dipetakan; milestone awal tetap S01–S09 dalam [delivery](docs/delivery.md). L03/L04 merinci dua templat awal dan pola respons objektif yang dipakai ulang; variasi materi tidak memerlukan alur akun/pembayaran terpisah. Audio soal cerita dan video bab mengikuti L02/L03, sedangkan jawaban suara mempunyai alur L05 sendiri. Personalisasi koleksi dirinci sebagai bagian L08 dan kontraknya ditambahkan pada milestone fitur sesuai konteks anak.

Perubahan alur harus memperbarui PRD, [UX](docs/ux-spec.md), [spesifikasi teknis](docs/technical-spec.md), diagram yang terdampak, serta pengujian. Versi/rubrik, otorisasi server, dan referensi bukti tetap sumber kebenaran; diagram bukan pengganti kontrak API.

Rujukan notasi: [dokumentasi flowchart Mermaid](https://mermaid.js.org/syntax/flowchart.html).

## Pemeriksaan dokumentasi

Perubahan diperiksa melalui penelaahan tautan, konsistensi istilah/PK/FK, contoh payload, dan kesesuaian kebutuhan dengan alur fitur. Pertahankan satu README dan pastikan dokumen terkait diperbarui pada perubahan yang sama.

[Model data](docs/database/schema.json), enam ERD, dan kamus data dipelihara bersama secara manual. Model data menjadi acuan metadata sebelum migrasi SQL dibuat; perubahan tabel/field/relasi juga mengubah diagram dan uraian kolom terkait. Setelah implementasi dimulai, migrasi serta pengujian PostgreSQL membuktikan constraint/transaksi yang dirancang.

Kontras memakai rumus dan tabel pada [skill aksesibilitas](skills/antislop-human/SKILL.md); pasangan warna yang sudah dihitung dicatat di desain antarmuka. Pemeriksaan dokumen tidak membuktikan bahwa UI sudah dirender, navigasi keyboard sudah bekerja, atau transaksi SQL sudah lulus.
