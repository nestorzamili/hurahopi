# Hurahopi — Daftar Keputusan

[Dokumentasi utama](../README.md).

Tanggal baseline: 25 Oktober 2026. Pemilik produk menyetujui keputusan prioritas, lalu mendelegasikan pemilihan modul berdasarkan riset open source dan perapian dokumen. Pilihan D1/D6 di bawah menjadi ketetapan implementasi; validasi kurikulum, kebijakan bisnis/data, serta kesiapan akun produksi tetap mempunyai status masing-masing.

## Status D1–D15

| ID | Status | Keputusan atau pekerjaan tersisa | Batas dan penanggung jawab |
| --- | --- | --- | --- |
| D1 | Ditetapkan | shadcn/ui varian Base UI, Tailwind, token bersama; layout orang tua diadaptasi selektif dari Satnaing; admin memakai Shadcn Admin Kit; aktivitas anak khusus | Tim pengembang; S01/S04/S09 |
| D2 | Terbuka | Jadwal retensi transkrip, analisis, portofolio, PDF, akun tidak aktif, backup, catatan transaksi; penarikan persetujuan dan tinjauan privasi | Pemilik produk dan penasihat hukum; sebelum siklus hidup media/data anak diaktifkan |
| D3 | Sebagian ditetapkan | Tujuh hari tanpa kartu ditetapkan sejak rencana awal. Maksimal tiga penerima per keluarga, titik mulai trial, kuota media AI, dan perpanjangan trial masih rekomendasi | Pemilik produk; sebelum trial produksi |
| D4 | Sebagian ditetapkan / Parameter uji | Bab contoh, dua templat, penilaian server, aturan yang dapat dijelaskan, dan parameter 8/10 disetujui untuk prototipe. Peta lengkap, prasyarat, rubrik, parameter penempatan/penurunan, dan kalibrasi tetap perlu pendidik | Pendidik bersama pengembang; sebelum publikasi konten dan evaluasi belajar |
| D5 | Terbuka | Evaluasi penyedia/model untuk draf soal, transkripsi, gambar; akses akun, dukungan data anak, retensi, biaya; AI untuk narasi laporan opsional | Tim pengembang menyiapkan evaluasi; pemilik produk memilih sebelum data anak dikirim |
| D6 | Ditetapkan; verifikasi integrasi tersisa | Stack/modul pada tabel arsitektur: Go Blueprint, TypeScript, pgx, Goose, TanStack Router/Query, OpenAPI TypeScript/Fetch, OAuth2/OIDC, dnd-kit, Konva, River, Gotenberg, Resend, Sentry, Compose/Caddy. Metode/biaya/status iPaymu dikonfirmasi pada akun merchant; akun/domain layanan disiapkan saat integrasi | Tim pengembang; implementasi sesuai milestone, verifikasi akun sebelum penggunaan produksi |
| D7 | Terbuka | Harga 1/3/12 bulan, refund, profil berbayar yang dihapus, kanal dukungan | Pemilik produk; sebelum penjualan publik |
| D8 | Terbuka | RPO, RTO, ketersediaan, kapasitas dan skenario beban bersamaan | Pengembang mengukur, pemilik produk menetapkan; sebelum peluncuran komersial |
| D9 | Ditetapkan | Langganan per anak/kursus; satu sesi dan lease belajar aktif per anak; login dewasa boleh beberapa perangkat; pemindahan belajar dikonfirmasi | Implementasi sejak fondasi |
| D10 | Terbuka | Usulan segmen berurutan tanpa pro rata; perilaku peningkatan paket langsung dan penggabungan belum final | Pemilik produk; sebelum penjualan publik |
| D11 | Terbuka / Fase 2 | Paket gabungan atau per mata pelajaran dan harga Fase 2 | Setelah Fase 1 berhasil |
| D12 | Terbuka | Pengantar interaktif direkomendasikan sebelum video; produksi video, jumlah/format worksheet, jumlah soal olimpiade dan biaya belum final | Pemilik produk dan pendidik; sebelum kelengkapan konten Tahap 5 |
| D13 | Ditetapkan | AI membuat draf sebelum anak bermain; semua draf divalidasi dan ditinjau pendidik sebelum terbit; tidak ada generasi langsung per anak pada Fase 1 | Pengembang dan editor/peninjau; sejak alur konten |
| D14 | Ditetapkan | Maksimal dua pengasuh per keluarga, izin per profil; awalnya belajar dan laporan; portofolio/PDF diaktifkan terpisah. Ketersediaan selama trial direkomendasikan dan difinalkan bersama D3 | Pengembang; sebelum fitur pengasuh |
| D15 | Ditetapkan | Tabel dan kolom `snake_case`; PK nama entitas tunggal + `_id`, misalnya `users.user_id`, `children.child_id`; FK mengikuti PK tujuan dengan awalan peran jika diperlukan | Instruksi pemilik produk; seluruh model/API domain dan migrasi sejak S02 |

## Alasan keputusan prioritas

### D1 — Fondasi UI

Satu fondasi shadcn/ui varian Base UI dan Tailwind dipakai di semua area. Dashboard orang tua mengambil layout/tabel terpilih dari Satnaing; seluruh primitive interaktif diselaraskan dengan fondasi bersama. Admin memakai Shadcn Admin Kit untuk CRUD dan form. Aktivitas anak memakai renderer React/DOM khusus agar audio, ukuran objek, dan interaksinya sesuai kebutuhan anak. Token merek, fokus keyboard, serta status interaksi tetap satu sistem.

Penjelasan layout: dunia Hopi adalah pengalaman utama anak, dengan beranda satu misi, layar aktivitas fokus, peta ringkas, dan koleksi. Beranda orang tua adalah ringkasan pendamping per anak; struktur visual Satnaing diadaptasi untuk kebutuhan tersebut. Contoh desktop/ponsel serta handoff pendamping ada di [UX](ux-spec.md#tata-letak-orang-tua). Sketsa menunjukkan rancangan yang akan diuji, bukan desain final yang telah tervalidasi.

Trade-off yang diterima: komponen sumber tidak menjadi fork aplikasi penuh, sehingga integrasi domain tetap dikerjakan Hurahopi. Admin Kit menambah konsep ra-core/data provider, tetapi menghemat CRUD berulang. Penggunaan dibatasi pada area admin; halaman anak dan laporan orang tua mengikuti alur produk sendiri.

### D6 — Stack pendukung dan pemakaian ulang

Pilihan lengkap dan sumber resminya ada pada [tabel stack](architecture.md#stack-inti-dan-layanan-pendukung). Keputusan utama: Go Blueprint hanya untuk scaffold backend; frontend dari template resmi Vite; Goose untuk migrasi; TanStack Router/Query; tipe OpenAPI dihasilkan dengan openapi-typescript dan dipakai openapi-fetch; protokol Google memakai OAuth2/OIDC. Sesi/PIN/izin/lease tetap dikelola Go/PostgreSQL.

River ditambahkan saat job tahan restart pertama diperlukan. dnd-kit dipasang saat drag diperlukan; Konva/react-konva untuk kanvas M3; Gotenberg untuk PDF worker M3. Resend dipilih untuk email transaksional dan Sentry untuk error teknis tersanitasi. Biaya/fitur akun merchant dan kredensial layanan diverifikasi ketika integrasi; hal tersebut tidak menjadikan pilihan teknologi kembali terbuka.

Trade-off yang diterima: Gotenberg menambah service internal dan kebutuhan sumber daya; queue River memakai database yang sama dengan aplikasi; komponen yang diadaptasi perlu pencatatan asal serta pembaruan terkontrol. Tidak ada fork LMS, engine BKT, auth demo Clerk/JWT aplikasi, atau layanan Redis/S3/MinIO pada baseline. Alasan dan batas pemakaian ulang ada di [arsitektur](architecture.md#pemakaian-ulang-dan-batas-modul).

Versi paket dan image dikunci ketika mulai dipakai setelah build/tes kompatibilitas. Bila hasil scaffold membutuhkan penyesuaian, selaraskan kode hasilnya dengan spesifikasi; jangan mengubah kebutuhan keluarga, versi stack inti, atau model akses demi mengikuti template. Status dipilih tidak berarti modul sudah terpasang atau PoC integrasinya sudah lulus.

### D9 — Unit pembelian dan penggunaan

Hak belajar terikat `child_id` dan `course_id`. Ini memisahkan progres, trial, pembelian, dan paket kakak/adik. Checkout wajib menunjukkan anak penerima sebelum pembayaran. Hak akses satu anak tidak membuka fitur anak lain.

Pemindahan perangkat memindahkan lease belajar, mempertahankan sesi/percobaan, dan tidak mengeluarkan orang tua dari laporan. Aturan ini **menggantikan** pembatasan satu perangkat untuk seluruh akun dalam draf 24 Oktober. Satu auth_session browser hanya memegang satu lease aktif; dua tab berbagi konteks. Backend mengunci auth_session sebelum anak dan memerlukan pause eksplisit untuk pindah profil. Penolakan jawaban perangkat lama diperiksa server; dua anak berlangganan boleh belajar bersamaan.

### D13 — AI di alur editorial

AI membantu menyediakan variasi konten tanpa menjadi jalur penerbitan langsung. Draf numerik tetap ditinjau. Anak menerima versi konten terbit dengan kunci/rubrik yang bisa ditelusuri. Penyedia AI boleh diganti melalui adapter tanpa mengubah format konten terbit.

Layanan review/publish dan penyimpanan peran/review tersedia S04; UI editorial lengkap S09. Review terikat definisi efektif beserta versi/checksum aset. Pergantian aset draf menggugurkan review dan versi terbit tidak ditimpa. Ini memastikan D13 tetap berlaku sebelum uji keluarga S08.

### D14 — Pengasuh

Dua pengasuh adalah batas awal produk yang disetujui, bukan angka standar industri. Undangan menunggu ikut memakai kuota untuk mencegah undangan tak terbatas; cek kuota dilakukan secara atomik saat mengundang/menerima. Izin diberikan per anak, dan izin portofolio/PDF awalnya nonaktif. Pengasuh tidak mendapat kewenangan pembelian, perubahan profil, pengunduhan/penghapusan media, atau mengundang orang lain.

### D4 — Parameter pembelajaran

Aturan naik tingkat: minimal 8 dari 10 aktivitas objektif berbeda benar pada respons pertama tanpa petunjuk/bantuan, minimal dua sesi dan dua templat. [Spesifikasi pembelajaran](learning-spec.md#aturan-naik-dan-turun-tingkat) mendefinisikan denominator dan deduplikasinya. Parameter ini disetujui untuk prototipe, bukan klaim validitas pedagogis.

Rincian probe penempatan, ambang penurunan, jumlah contoh konten, dan tugas uji dalam spesifikasi adalah default uji yang dibuat agar mesin dapat diprogram dan disimulasikan. Pendidik dapat merevisinya dengan versi aturan baru sebelum pemakaian publik.

### D15 — Penamaan dan rancangan database

Seluruh tabel domain Hurahopi mengikuti [konvensi ERD](database/design.md#konvensi-nama-dan-tipe). UUID menjadi tipe PK opaque. Tabel penghubung tetap memiliki PK sendiri dan UNIQUE pasangan bisnis. Nama dibuat singkat tetapi spesifik: `rule_versions` untuk konfigurasi aturan, `child_permissions` untuk izin, dan `content_generations` untuk permintaan draf editorial. Pertahankan pembeda bermakna seperti `auth_sessions` dan `study_sessions`; jangan menambah awalan proyek atau singkatan yang sulit dipahami. Enam ERD serta kamus data mengacu pada `schema.json` dan diperbarui manual bersama agar nama dan relasinya konsisten. Nama JSON/OpenAPI untuk identitas domain mengikuti PK; pemetaan `record.id` hanya di adapter admin. Metadata internal Goose/River mengikuti migrasi resmi library dan dikecualikan dari model domain.

Nama field menggunakan konteks: `avatar`, `role`, `provider`, `locale`, `price`, `currency`; kode lokal cukup `code`. PK/FK tetap memakai nama entitas + `_id`, sedangkan hash, timestamp dan unit tetap jelas.

Model Fase 1 disederhanakan menjadi 59 tabel; S02–S03 memakai 11 tabel. Penyajian/hasil soal menyatu pada `attempts`, jawaban pada `responses`; klaim/varian media dimiliki snapshot/media, bukti tetap FK. Roles admin tetap pada pengguna, dan izin undangan/aktif memakai satu child_permissions. Seri konten/paket memakai versi tanpa katalog identitas kosong.

Personalisasi sederhana disimpan pada `children`, tanpa entitas satu-ke-satu dengan siklus hidup yang sama. Rancangan mencakup seluruh Fase 1; migrasi dibuat per cerita/milestone. Riwayat lease mempertahankan baris lama dengan satu baris aktif per anak, mendukung recovery dan penolakan perangkat tercabut pada D9. ERD tidak menetapkan harga, retensi, aturan upgrade, atau parameter pedagogis yang masih terbuka.

## Hal yang dapat dimulai tanpa keputusan tambahan

Bootstrap, OpenAPI alur pertama, migrasi inti, fake provider, identitas keluarga, PIN/mode anak, lease belajar, fixture konten, penilaian deterministik, dan laporan berbasis templat dapat dikerjakan dengan data sintetis. Trial produksi, harga, pengolahan media anak, AI data anak, dan penggunaan akun layanan nyata mengikuti prasyarat masing-masing. Pilihan modul sudah ditetapkan; fake provider lokal/test tetap dipakai sampai akun dan konfigurasi siap.

Pengembang tidak menandai kebijakan layanan, retensi, atau rubrik sebagai disetujui hanya karena implementasi fixture sudah berfungsi. Catat tanggal, alasan, pihak yang menetapkan, dokumen terdampak, dan kebutuhan migrasi saat keputusan berikutnya berubah.
