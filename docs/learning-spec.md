# Hurahopi — Spesifikasi Pembelajaran dan Konten

[Dokumentasi utama](../README.md).

Tanggal: 26 Oktober 2026. Status: baseline mesin disetujui; kurikulum dan parameter menunggu validasi pendidik.

Bagian dari [rencana pengembangan](plan.md). [Daftar keputusan](decisions.md) mencatat status persetujuan; aturan produk ada di [PRD](prd.md). Parameter uji tetap memerlukan evaluasi pendidik.

## Konten, Analisis, dan Pembelajaran Adaptif

### Struktur Materi

Struktur: **bab → tingkat (level) → keterampilan → aktivitas**. Setiap bab memiliki prasyarat berupa bab atau tingkat lain sehingga sistem dapat menurunkan anak ke tingkat yang dipahami. Aktivitas pendukung (bahasa, visual-spasial, kreasi) dikaitkan dengan keterampilan matematika pada bab yang sama.

Daftar bab berikut adalah rancangan awal (**Rekomendasi**) untuk ditinjau pendidik:

| Kelompok | Contoh bab |
| --- | --- |
| Pra-angka dan bilangan awal | Membandingkan dan mengelompokkan; membilang 1–10; mengenal angka; bilangan sampai 20 |
| Operasi hitung | Penjumlahan; pengurangan; perkalian; pembagian |
| Bilangan lanjutan | Nilai tempat; bilangan sampai 100 dan 1.000; pecahan sederhana |
| Geometri dan pengukuran | Bangun datar dan bangun ruang; panjang, berat, waktu, uang |
| Pola dan penalaran | Pola dan barisan; logika dan pemecahan masalah; data sederhana |
| Jalur olimpiade | Soal penalaran dan pemecahan masalah yang lebih menantang (paket 12 bulan) |

Setiap bab memiliki beberapa tingkat dari yang paling dasar. Usia hanya menentukan titik awal tes penempatan, misalnya anak 4 tahun mulai dari pra-angka dan anak 9 tahun mulai dari bilangan lanjutan.

Setiap aktivitas terbit dilengkapi target keterampilan, rentang usia awal, tingkat tantangan relatif terhadap keterampilan, prasyarat jika diperlukan, jenis interaksi, bidang pendukung (bila ada), rubrik, instruksi, narasi, serta aturan petunjuk dan pengulangan.

### Tes Penempatan

**Ditetapkan:** penempatan berbasis aturan; usia hanya titik acuan, tes boleh belum selesai, respons ketuk-pilih cukup, dan kegagalan teknis tidak dinilai salah. Hasil awal per bab bersifat sementara sampai bukti cukup. Orang tua dapat meminta tes ulang atau mengubah titik latihan dengan alasan tercatat; pengubahan ini tidak menciptakan bukti penguasaan.

**Parameter uji placement-v0.1** untuk bab contoh:

1. Titik awal uji internal: usia 4–5 mulai L1, 6–7 mulai L2, 8–10 mulai L3. Ini hanya fixture pengembangan bab awal, bukan pemetaan kemampuan untuk seluruh usia/materi dan bukan tes penempatan lengkap Fase 1.
2. Kumpulkan tiga respons objektif valid pada instans berbeda di satu tingkat tanpa petunjuk sebelum respons pertama; instruksi audio dapat diulang. Dari tiga respons valid tersebut, dua atau tiga benar mandiri mengarahkan probe ke tingkat berikutnya; nol atau satu mengarahkan ke prasyarat. Lewati/gagal teknis diganti instans lain selama batas penyajian belum habis. Jika belum ada tiga respons valid, hasil probe belum lengkap dan tidak mengarahkan turun tingkat. Jangan mengulang tingkat yang probe validnya sudah selesai dalam satu tes.
3. Tingkat awal sementara adalah tingkat tertinggi dengan minimal dua respons benar dari tiga. Bila L1 belum lolos, gunakan L1 dengan dukungan dan tandai kebutuhan prasyarat pra-angka; jangan menyimpulkan anak gagal atau menampilkan skor.
4. Maksimal sembilan aktivitas per bagian, maksimal dua bagian pada sesi terpisah; anak dapat berhenti lebih awal. Setelah batas/ketidakselesaian, gunakan hasil yang ada dengan status sementara. Jika tidak ada bukti valid, pakai titik awal acuan dengan label data terbatas. Respons setelah petunjuk/bantuan tidak dihitung benar mandiri; aktivitas dilewati/gagal teknis menghabiskan slot penyajian tetapi bukan jawaban salah.
5. Simpan daftar tingkat yang sudah diprobe, respons, dan posisi berikutnya. Berhenti setelah probe pada batas L1/L4 menghasilkan arah keluar dari rentang, atau arah berikutnya kembali ke tingkat yang sudah selesai diuji. Jangan menyatakan kelompok bab yang belum diuji sebagai dikuasai.

Pendidik memvalidasi peta lintas bab, titik usia acuan, prasyarat, dan batas probe sebelum tes ini dipakai sebagai penempatan publik. Bab awal hanya membuktikan mesin penempatan; sampel usia lebih besar memerlukan materi yang sesuai sebelum evaluasi lintas usia.

### Aturan Naik dan Turun Tingkat

**Ditetapkan:** penilaian objektif di server dan perubahan tingkat dengan alasan yang dapat dijelaskan. **Parameter uji mastery-v0.1** disetujui untuk implementasi prototipe, lalu dikalibrasi pendidik dan uji keluarga. Parameter bukan standar perkembangan anak.

| Aturan | Definisi operasional prototipe |
| --- | --- |
| Bukti yang masuk jendela | Sepuluh instans objektif valid terakhir pada anak, kursus, keterampilan, tingkat, dan versi aturan yang sama; hanya latihan, bukan penempatan, contoh, atau karya terbuka |
| Aktivitas berbeda | Satu instans yang disajikan hanya menyumbang satu bukti; muat ulang/retry pengiriman tidak menambah bukti. Templat boleh berulang dengan parameter soal berbeda; duplikat fingerprint latihan tidak dihitung dua kali dalam jendela |
| Benar mandiri | Benar pada respons jawaban pertama, tanpa petunjuk maupun bantuan yang diketahui/dikonfirmasi. Memutar ulang instruksi audio tidak dianggap petunjuk |
| Naik tingkat | Minimal 8 dari 10 benar mandiri, tersebar pada minimal dua sesi, minimal dua templat, dengan minimal dua instans dari masing-masing dua templat tersebut; prasyarat dan konten tingkat berikutnya harus tersedia |
| Kesulitan saat sesi | Respons pertama salah pada tiga instans berturut-turut menawarkan bantuan visual pada instans berjalan dan menandai latihan prasyarat untuk sesi berikutnya; tidak langsung mengubah status penguasaan atau rencana yang dibekukan |
| Turun titik latihan | Parameter tambahan untuk simulasi: enam bukti valid terakhir pada tingkat yang sama, berasal dari minimal dua sesi dengan minimal tiga bukti per sesi, dan paling banyak dua benar mandiri; sesi berikutnya berlatih prasyarat. Di L1, berikan dukungan L1 dan tandai kebutuhan pra-angka |
| Riwayat penguasaan | Pencapaian sebelumnya dipertahankan; catat titik latihan saat ini dan status perlu ulasan secara terpisah. Penurunan bukan penghapusan pencapaian |
| Bukti tidak lengkap | Lewati/gagal teknis/pemrosesan tertunda tidak masuk denominator. Jawaban dibantu tetap tercatat sebagai bukti valid, tetapi bukan benar mandiri |
| Hasil terlambat | Diterapkan pada perhitungan berikutnya menurut waktu penyajian instans, dengan deduplikasi; aktivitas yang sedang ditampilkan tidak diganti |
| Tidak ada konten lanjutan | Pertahankan tingkat latihan dan beri ulasan; jangan menawarkan aktivitas belum terbit |

Latihan membekukan rencana sesi; penempatan memilih probe berikutnya berdasarkan hasil dalam versi aturan yang dibekukan, tanpa mengganti instans yang sudah disajikan.

Evaluasi aturan setelah terminalisasi suatu instans. Naik/turun titik latihan diterapkan pada pemilihan aktivitas sesi berikutnya; sesi berjalan memakai rencana yang telah dibekukan. Satu evaluasi tidak boleh menghasilkan dua perubahan yang berlawanan. Jendela dipisahkan saat versi aturan/rubrik penilaian berubah; migrasi hasil lama memerlukan evaluasi tersendiri. Tantangan loncat tingkat dan pengingat ulasan setelah jeda tetap rekomendasi untuk kalibrasi D4, bukan cabang wajib milestone awal.

Satu penyajian soal beserta hasilnya disimpan sebagai `attempts`; respons pertama dan koreksi berada pada `responses`. Seluruh rujukan bukti/media memakai attempt_id yang sama. `evaluation_status` pending/not_applicable/failed tidak menjadi bukti mastery; kriteria benar mandiri tetap sesuai tabel.

Simpan perubahan pada `mastery_changes`: `mastery_change_id`, `child_id`, `course_id`, `from_level_id`, `to_level_id`, `reason`, `rule_version_id`, dan waktu. Bukti attempt disimpan sebagai relasi `mastery_change_evidence`; `evidence_attempt_ids` pada API merupakan proyeksi relasi tersebut. Target keterampilan ditelusuri melalui `level_skills` dan instans bukti; state dipisahkan per anak, `level_skill_id`, `mastery_rule_version_id`, dan `rubric_version_id`. Lihat [ERD pembelajaran](database/learning.md). Ringkasan bidang dengan minimal tiga aktivitas berbeda pada dua sesi adalah ambang laporan yang terpisah; ambang tersebut tidak menggantikan syarat naik tingkat 8/10.

### Aktivitas dan Sumber Soal

Soal berasal dari tiga sumber, semuanya masuk ke bank konten terkurasi dan memiliki versi:

| Sumber | Kegunaan | Pemeriksaan |
| --- | --- | --- |
| Templat dengan pembuat soal berparameter | Variasi tak terbatas untuk keterampilan berhitung (rentang bilangan, jenis operasi, jumlah benda) | Kunci dihitung di server |
| Draf dari AI | Soal cerita, variasi konteks, soal penalaran dan olimpiade, narasi, serta petunjuk | Alur [alur soal AI](learning-spec.md#pembuatan-soal-oleh-ai) |
| Tulisan editor | Aktivitas khusus dan soal inti | Peninjauan pendidik |

Siklus konten: **draf → peninjauan pendidik → penerbitan → pengarsipan**. Perubahan pada konten terbit menghasilkan versi baru agar riwayat jawaban tetap dapat ditelusuri. Narasi disiapkan sebagai aset yang ditinjau sebelum penerbitan.

Target volume awal (**Rekomendasi**, ditetapkan bersama pendidik pada D4): delapan sampai sepuluh bab dengan lima sampai delapan tingkat per bab pada peluncuran, setiap tingkat memiliki minimal dua templat permainan. Kesiapan juga dinilai melalui matriks bab, keterampilan, usia awal, tingkat tantangan, jenis interaksi, dan bidang pendukung. Setiap jalur yang dapat dipilih sistem harus memiliki konten terbit serta aktivitas cadangan yang memadai.

Referensi worksheet pihak lain hanya menjadi bahan inspirasi; konten produk menggunakan materi orisinal dan aset dengan hak penggunaan yang sesuai. Pemilik produk menyiapkan pendidik secara paralel; keluaran pendidik mencakup peta bab dan tingkat, rubrik berversi, contoh jawaban atau karya, peninjauan instruksi serta aset, dan evaluasi soal buatan AI.

### Pembuatan Soal oleh AI

**Status: Ditetapkan — D13.** AI membantu menyiapkan draf sebelum publikasi. Alur validasi, peninjauan, dan penerbitan berikut menjadi baseline implementasi; akun/model produksi mengikuti D5.

1. **Permintaan.** Editor memilih keterampilan, tingkat, jenis interaksi, bidang pendukung, dan jumlah draf. Permintaan ke AI hanya memuat spesifikasi konten, **tanpa data anak**.
2. **Keluaran terstruktur.** AI mengembalikan soal, opsi jawaban, kunci, langkah penyelesaian, petunjuk bertahap, dan narasi dalam format terstruktur yang divalidasi skemanya.
3. **Validasi otomatis di server**, sebelum draf muncul pada antrean:
   - Kunci jawaban numerik/terstruktur dihitung ulang secara deterministik; draf dengan kunci yang tidak cocok ditolak. Soal penalaran yang belum mempunyai validator solusi memerlukan pemeriksaan kebenaran dan pembahasan secara penuh oleh pendidik; lolos skema tidak dianggap membuktikan kebenaran solusi.
   - Rentang bilangan, panjang kalimat, kosakata, dan kesesuaian dengan tingkat serta usia diperiksa.
   - Pemeriksaan keamanan konten untuk anak dan deteksi duplikasi terhadap bank soal.
4. **Peninjauan pendidik.** Setiap draf ditinjau manusia sebelum terbit. Soal numerik yang kuncinya terverifikasi tetap ditinjau; soal penalaran, soal cerita, dan olimpiade mendapat peninjauan penuh atas kebenaran, kejelasan, dan kelayakan untuk usia.
5. **Penerbitan.** Soal terbit sebagai versi konten dengan catatan asal (dibuat AI, konfigurasi model, peninjau, versi rubrik).
6. **Pemantauan.** Tingkat penolakan draf, alasan penolakan, dan soal yang kemudian dilaporkan keliru dipantau; perubahan model atau instruksi menjalani evaluasi ulang.

Pertimbangan tambahan:

- Soal olimpiade dari AI harus orisinal; tidak boleh menyalin soal kompetisi atau worksheet berhak cipta, dan pemeriksaan keaslian menjadi bagian peninjauan.
- Anak tidak menerima teks mentah dari AI pada saat bermain. Soal selalu berasal dari bank terkurasi.
- **Ditetapkan — D13:** Fase 1 memakai pra-generasi terkurasi. Semua draf AI, termasuk numerik, ditinjau pendidik sebelum terbit. Tidak ada pembuatan soal langsung per anak saat bermain. Anggaran dan batas penggunaan penyedia ditetapkan sebelum integrasi produksi.

### Bukti Kemampuan dan Minat

| Dimensi | Bukti yang digunakan |
| --- | --- |
| Kemampuan | Ketepatan jawaban, penyelesaian tugas, penggunaan petunjuk, dan pengamatan sesuai rubrik |
| Minat | Pola keterlibatan dan respons sederhana anak, seperti "Seru?" atau "Mau mencoba lagi?" |
| Konteks | Bantuan orang tua atau pengasuh, aktivitas dilewati, gangguan koneksi, dan kegagalan pemrosesan media |

Cara penilaian per jenis respons:

- **Jawaban objektif** (pilihan, mencocokkan, menyusun): dinilai di server dengan kunci jawaban.
- **Jawaban suara**: AI mentranskripsikan suara; isi jawaban lalu diperiksa di server (misalnya mengubah "dua puluh tiga" menjadi angka 23 dan membandingkannya dengan kunci). Bila transkripsi meragukan, anak diminta mengonfirmasi atau memilih jawaban. Pelafalan tidak dinilai.
- **Gambar dan mewarnai**: bila tugas memiliki bentuk terstruktur (misalnya mewarnai tiga dari empat bagian, atau menempatkan lima benda), pemeriksaan dilakukan secara deterministik. Karya terbuka memperoleh deskripsi dan pengamatan proses dari AI tanpa nilai benar atau salah.

Ambang awal yang direkomendasikan untuk menampilkan ringkasan suatu bidang (matematika per bab maupun bidang pendukung) adalah **tiga aktivitas berbeda dengan bukti valid pada setidaknya dua sesi**. Sebelum ambang terpenuhi, laporan menampilkan aktivitas yang teramati dengan status data terbatas. Ambang ini merupakan parameter produk untuk diuji bersama pendidik, bukan standar perkembangan anak yang telah tervalidasi.

Kesimpulan minat mempertimbangkan respons ketertarikan bersama perilaku selama aktivitas. Ketepatan, kecepatan, dan waktu layar tidak menjadi dasar tunggal. Bukti yang belum memadai ditampilkan sebagai "indikasi awal" atau "perlu lebih banyak aktivitas". Laporan bersifat deskriptif dan tidak menetapkan diagnosis maupun bakat permanen.

**Peran AI dalam analisis** (**Rekomendasi**):

| Fungsi AI | Data anak dikirim ke penyedia? | Catatan |
| --- | --- | --- |
| Pembuatan draf soal dan narasi | Tidak | Alur [alur soal AI](learning-spec.md#pembuatan-soal-oleh-ai) |
| Transkripsi jawaban suara | Ya (audio anak) | Memerlukan prasyarat [kesiapan penyedia AI](operations.md#kesiapan-penyedia-ai) |
| Pengamatan gambar anak | Ya (gambar anak) | Memerlukan prasyarat [kesiapan penyedia AI](operations.md#kesiapan-penyedia-ai) |
| Penentuan tingkat dan jalur belajar | Tidak digunakan | Aturan [aturan adaptif](learning-spec.md#aturan-naik-dan-turun-tingkat) |
| Penulisan narasi laporan | Hanya ringkasan terstruktur, tanpa media mentah | Opsional; laporan dapat disusun dari templat tanpa AI (D5) |

### Laporan, Ketertelusuran, dan Ekspor PDF

Snapshot laporan menyimpan daftar klaim sebagai data dokumen miliknya; setiap key klaim dipetakan ke row `report_evidence` dengan FK ke attempt/pengamatan yang benar. Klaim tidak memiliki entitas/edit API terpisah.

Laporan memuat perkembangan per bab, pengamatan pendukung, indikasi minat, contoh bukti, alasan perubahan tingkat, dan saran pendampingan. Waktu pembaruan serta status analisis terbaru ditampilkan agar orang tua dapat membedakan ringkasan selesai dari analisis yang masih berlangsung. Kegagalan analisis diperlakukan sebagai kekurangan data.

Hash review mencakup seluruh isi efektif termasuk binding/checksum aset, versi templat/generator/rubrik, dan konfigurasi penilaian. Pergantian aset draf menaikkan revisi dan membatalkan review; aset/binding konten terbit tidak ditimpa. Layanan review/publish minimum sudah diperlukan S04 sebelum konten dipakai pada uji keluarga S08.

Percobaan menyimpan versi aktivitas. Pengamatan dan perubahan tingkat mencatat referensi bukti, versi rubrik, konfigurasi model (bila AI terlibat), dan versi profil yang digunakan. Perubahan model dan rubrik menjalani evaluasi ulang.

**Ekspor PDF** — baseline implementasi; retensi mengikuti D2:

- Orang tua, dan pengasuh yang diizinkan, dapat mengekspor laporan satu profil untuk periode tertentu.
- PDF berisi Peta Belajar, perkembangan, pengamatan pendukung, dan saran pendampingan. Pemilik akun dapat memilih menyertakan contoh karya anak (gambar). Ekspor pengasuh berisi laporan tanpa salinan media anak, termasuk ketika pengasuh boleh melihat portofolio; izin melihat portofolio tidak memberikan izin mengunduh media melalui ekspor.
- PDF dibuat worker River melalui Gotenberg internal dari HTML/CSS dan aset yang disiapkan server. Hasil disimpan sementara dan disajikan melalui endpoint yang memeriksa sesi serta izin; berkas dihapus mengikuti masa simpan D2 dan tidak dianggap arsip permanen.
- Worker merender tanpa menahan transaksi database. Publikasi PDF mengunci child lalu snapshot/ekspor dan memeriksa ulang data_version, izin, serta status belum dibatalkan. Hasil yang selesai setelah penghapusan tidak boleh kembali siap; output sementara dibuang. Unduh hanya dari snapshot valid dan ekspor ready.
- Setiap ekspor mencatat pengguna, profil, dan waktu. Penghapusan profil atau media membatalkan ekspor yang masih tersimpan.
- Ekspor PDF tersedia pada semua paket, selama uji coba, dan setelah masa akses berakhir.
- Gotenberg sudah ditetapkan pada D6; uji tata letak, font, izin, pembatalan, dan penggunaan sumber daya dilakukan pada M3 sebelum fitur dinyatakan siap.

### Video Penjelasan Bab (Opsional)

Setiap bab dapat memiliki satu video penjelasan 1–2 menit. Karena cara produksinya belum ditentukan, rancangan berikut membuat video tidak menjadi syarat peluncuran:

- Model data menyediakan slot video opsional per bab.
- Bila video belum ada, bab menampilkan **pengantar interaktif** singkat (tiga sampai lima layar bergambar dengan narasi audio) yang memakai aset dan komponen yang sama dengan permainan.
- Video ditampilkan saat anak pertama kali memasuki bab dan dapat diputar ulang; anak boleh melewatkannya.

Opsi produksi video (D12):

| Opsi | Kelebihan | Catatan |
| --- | --- | --- |
| Papan tulis digital dengan rekaman suara | Termurah dan tercepat | Tampilan sederhana; perlu pengisi suara yang jelas dan ramah anak |
| Animasi dengan karakter Hopi memakai alat desain/animasi | Identitas merek kuat | Butuh waktu desain per video |
| Animator lepas per video | Kualitas paling tinggi | Biaya per video; perlu panduan gaya dan naskah dari pendidik |
| Pengantar interaktif di aplikasi | Tanpa biaya hosting video; konsisten dengan permainan | Bukan video sungguhan |

Rekomendasi: luncurkan dengan pengantar interaktif, lalu tambahkan video pada bab yang paling sering dianggap sulit berdasarkan data pemakaian. Video disajikan melalui Cloudinary dan tidak melalui YouTube, agar anak tidak terpapar video rekomendasi atau iklan lain. Bandwidth video diukur pada pengujian beban.

### Worksheet Printable dan Latihan Olimpiade

**Worksheet printable** (paket 3 dan 12 bulan):

- Berkas PDF per bab dan tingkat, dengan kunci jawaban untuk orang tua.
- Rekomendasi Fase 1: PDF yang dirancang dan diterbitkan sebagai aset konten (kualitas tata letak terjaga), bukan dibuat otomatis. Pembuatan otomatis dari pembuat soal dapat dievaluasi kemudian.
- Sistem menyarankan worksheet berdasarkan Peta Belajar.
- Cantuman nama profil anak pada kaki halaman dapat dipakai untuk mengurangi penyebaran berkas.
- Berkas yang telah diunduh tetap dimiliki orang tua; unduhan baru memerlukan akses paket yang sesuai.

**Latihan olimpiade** (paket 12 bulan):

- Kumpulan soal penalaran dan pemecahan masalah bertingkat, dengan pembahasan; sebagian dapat berasal dari alur AI ([alur soal AI](learning-spec.md#pembuatan-soal-oleh-ai)) dengan peninjauan penuh.
- Anak yang paketnya belum mencakup olimpiade melihat pratinjau terkunci beserta beberapa soal contoh.
- Rekomendasi: jalur olimpiade terbuka setelah anak mencapai tingkat ambang pada bab terkait, agar anak tidak menghadapi soal di luar kesiapannya. Sebelum ambang, orang tua tetap dapat melihat daftar isinya.
- Aktivitas visual-spasial mendukung jalur olimpiade (penalaran bentuk dan pola).
- Rentang konten olimpiade ditetapkan pendidik pada D4.

## Contoh bab awal

**Ditetapkan untuk prototipe:** bab `counting-1-10` — Mengenal jumlah dan membilang 1–10. Contoh berikut memakai profil sintetis Alya, usia acuan lima tahun, pada kursus `math`, locale `id-ID`. Usia tidak membatasi tingkat latihan. Bab ini menguji fondasi alur; seluruh kurikulum usia 4–10 masih memerlukan materi lain.

| Tingkat | Target keterampilan | Parameter contoh | Prasyarat dan contoh instruksi |
| --- | --- | --- | --- |
| L1 | `count_objects`: jumlah 1–3 | Objek terpisah dan mudah dibedakan, susunan sederhana | Periksa kemampuan mencocokkan satu-satu; "Pilih piring berisi dua apel." |
| L2 | `count_objects`: jumlah 1–5 | Variasi jumlah dan susunan sampai lima, tanpa simbol angka sebagai syarat | L1; "Masukkan empat apel ke keranjang Hopi." |
| L3 | `count_objects`: jumlah 1–10 | Variasi sampai sepuluh, susunan baris dan tersebar tanpa tumpang tindih | L2; "Ada berapa ikan? Pilih kelompok yang sama banyak." |
| L4 | `quantity_to_numeral`: hubungan jumlah dan lambang 1–10 | Lambang angka dan kelompok konkret | L3 serta pengenalan lambang; "Pilih angka yang sesuai dengan jumlah apel." |

Nama tingkat menandai target maksimum, bukan kewajiban semua soal memakai nilai maksimum. Pada L4, `choose_group` dapat meminta memilih kelompok sesuai lambang, sedangkan `build_group` meminta membuat kelompok sesuai lambang. Audio/preview memperkenalkan lambang sebelum menjadi bukti penilaian. Prasyarat pengenalan lambang perlu ditinjau pendidik; konten yang belum tersedia tidak boleh dilewati oleh mesin.

L1–L3 memantau keterampilan `count_objects` pada tingkat berbeda. L4 membuka keterampilan baru, sehingga jendela bukti L3 tidak dipakai sebagai nilai penguasaan L4. Jika anak belum menguasai pra-angka, pendamping mendapat status kebutuhan prasyarat dan konten dukungan; fitur produksi memerlukan jalur pra-angka terbit yang lengkap.

### Dua templat inti

Renderer memakai React/DOM khusus Hurahopi; ketuk/keyboard tersedia sejak S04. Drag memakai dnd-kit ketika diimplementasikan. Kanvas kreasi memakai Konva/react-konva pada M3. Tidak menggunakan renderer LMS atau engine BKT untuk menentukan tingkat; algoritme penilaian dan jendela bukti tetap mengikuti spesifikasi ini.

| Templat | Tampilan dan respons | Penilaian server | Petunjuk dan alternatif |
| --- | --- | --- | --- |
| `choose_group` | Audio dan tiga kelompok benda; anak mengetuk satu `option_id` | Cocokkan ID dengan kunci yang dibekukan pada instans | Sorot benda satu per satu; keyboard/ketuk; ulang audio bebas |
| `build_group` | Anak memindahkan objek ke keranjang dengan ketuk atau drag; kirim daftar ID objek terpilih | Pastikan ID unik dan tersedia pada instans, lalu bandingkan jumlah dengan target | Contoh mencocokkan satu-satu; ketuk untuk memasukkan/mengembalikan; tombol selesai eksplisit |

Editor menyiapkan versi templat, rentang parameter, rubrik, instruksi, aset, dan aturan generator untuk ditinjau. Variasi numerik deterministik hanya dibuat dalam rentang templat terbit yang telah disetujui; tidak perlu memanggil AI saat bermain. Draf soal/narasi AI tertentu tetap harus ditinjau sebelum menjadi versi konten. Perubahan generator/rentang menghasilkan versi baru.

Untuk fixture internal, siapkan minimal 20 variasi per tingkat untuk latihan dan tiga probe penempatan terpisah per tingkat, pada kedua templat yang relevan. Angka volume ini adalah kebutuhan demonstrasi teknis, bukan target konten peluncuran. Generator memakai seed tersimpan dan validator yang memastikan tepat satu opsi benar, distraktor berbeda, jumlah dalam rentang, ID objek unik, dan aset/audio tersedia. Saat bank tidak cukup, gunakan ulang sebagai ulasan tanpa menggandakan bukti yang sama atau menyatakan penguasaan baru.

Fingerprint soal dibuat server dari versi konten dan parameter stimulus yang disajikan, termasuk target, konfigurasi kelompok/objek, serta bentuk penyajian. Mengubah urutan opsi, ID request, timestamp, atau mengulang seed tidak menciptakan bukti baru. Simpan fingerprint pada instans sehingga retry/reload tidak bisa memperbesar jendela mastery. Kecukupan variasi latihan ditinjau pendidik agar keberhasilan tidak hanya menghafal satu tampilan.

### Contoh enam slot misi L2

| Slot | Kegiatan | Bukti yang digunakan |
| --- | --- | --- |
| 1 | Pilih piring berisi tiga apel | Objektif `count_objects`, `choose_group` |
| 2 | Masukkan empat apel ke keranjang | Objektif `count_objects`, `build_group` |
| 3 | Pilih kelompok berisi lima ikan | Objektif `count_objects`, `choose_group` |
| 4 | Buat kelompok dua buah | Objektif `count_objects`, `build_group` |
| 5 | Bahasa: dengarkan "Hopi punya tiga apel", lalu ceritakan/konfirmasi jumlah bersama pendamping | Pengamatan bahasa matematika; jawaban suara opsional setelah fiturnya siap, alternatif ketuk tersedia |
| 6 | Kreasi: buat gambar berisi empat buah dengan susunan pilihan anak | Karya terbuka/observasi; tanpa benar-salah untuk kualitas gambar |

Aktivitas visual-spasial ditawarkan pada sesi berikutnya, misalnya membuat jumlah yang sama dalam susunan berbeda. Ketiga bidang pendukung direncanakan dalam rentang tiga sesi. Pada prototipe sebelum media tersedia, kegiatan pendukung adalah instruksi terpandu dengan status dilakukan/lewat dan bantuan, bukan analisis gambar/suara yang disimulasikan seolah nyata.

### Rubrik contoh

| Bukti | Penyimpanan | Pengaruh terhadap mastery |
| --- | --- | --- |
| Respons pertama benar tanpa bantuan | `first_response_correct=true`, `independent_correct=true` | Satu benar mandiri |
| Respons pertama salah, kemudian benar | Respons pertama tetap salah; hasil akhir completed | Satu bukti valid, bukan benar mandiri |
| Hint sebelum jawaban benar | Hint event tersimpan; `independent_correct=false` | Satu bukti valid, bukan benar mandiri |
| Bantuan pendamping diketahui/dikonfirmasi | `assistance_source=adult` | Bukan benar mandiri; bantuan yang tidak diketahui tidak boleh diklaim terdeteksi sistem |
| Lewati atau gagal teknis | Status dan alasan, tanpa skor salah | Di luar denominator |
| Gambar terbuka | Deskripsi kegiatan dan referensi karya jika ada persetujuan | Di luar penilaian objektif; tidak menyimpulkan bakat |

Petunjuk dan contoh terbimbing dicatat sebagai kejadian server. Sebelum respons awal, mengulang instruksi audio diperbolehkan tanpa kehilangan status mandiri. Pengamatan minat seperti "ingin mencoba lagi" memerlukan respons anak atau observasi yang dicatat; durasi dan ketepatan saja tidak cukup.

## Contoh evaluasi 8 dari 10

Tabel berikut adalah sepuluh instans objektif terakhir pada L2, keterampilan yang sama, dan `mastery-v0.1`. Ini cuplikan dari tiga sesi, bukan seluruh isi sesi; aktivitas pendukung tidak dimasukkan ke tabel. Semua instans memiliki parameter/fingerprint berbeda.

| Bukti | Sesi | Templat | Hasil | Benar mandiri |
| --- | --- | --- | --- | --- |
| E01 | A | choose_group | Benar pertama | Ya |
| E02 | A | build_group | Benar pertama | Ya |
| E03 | A | choose_group | Salah pertama, lalu benar | Tidak |
| E04 | A | build_group | Benar pertama | Ya |
| E05 | B | choose_group | Benar pertama | Ya |
| E06 | B | build_group | Benar pertama setelah petunjuk | Tidak |
| E07 | B | choose_group | Benar pertama | Ya |
| E08 | B | build_group | Benar pertama | Ya |
| E09 | C | choose_group | Benar pertama | Ya |
| E10 | C | build_group | Benar pertama | Ya |

Hasil: **8/10 benar mandiri**, tiga sesi, dua templat dengan lima bukti masing-masing. Jika prasyarat dan konten L3 terbit tersedia, buat perubahan L2 → L3 dengan `reason=MASTERY_THRESHOLD_MET`, versi aturan, dan referensi E01–E10. Terapkan L3 mulai sesi berikutnya. Salah lalu benar dan benar dengan petunjuk tetap termasuk denominator; keduanya tidak menambah numerator mandiri.

| Variasi | Keputusan |
| --- | --- |
| 7/10 benar mandiri dengan distribusi cukup | Tetap L2, lanjut latihan |
| 8/10 tetapi seluruhnya satu sesi | Tetap L2, kumpulkan bukti lintas sesi |
| 8/10 tetapi hanya satu templat | Tetap L2, tawarkan templat lain |
| 8/10 tetapi templat kedua hanya satu bukti | Tetap L2 sampai distribusi minimal terpenuhi |
| 8 bukti mandiri dan 2 aktivitas dilewati | Belum ada jendela sepuluh bukti valid; jangan mengubah lewat menjadi salah |
| E10 dikirim ulang beberapa kali | Tetap satu bukti E10 dan satu apresiasi |
| Syarat terpenuhi tetapi L3 belum terbit | Pertahankan latihan tersedia, catat kesiapan; jangan menampilkan draf |

## Contoh laporan orang tua

Contoh narasi dari tabel di atas:

> Alya menyelesaikan 8 dari 10 aktivitas membilang sampai 5 secara mandiri dalam tiga sesi. Satu aktivitas selesai setelah mencoba kembali, dan satu menggunakan petunjuk. Pada sesi berikutnya Alya akan mencoba menghitung sampai 10. Di rumah, ajak Alya menghitung sendok saat menyiapkan makan.

Klaim hasil mengacu pada E01–E10 dan perubahan tingkat; saran rumah berasal dari templat yang ditinjau pendidik. Tampilan laporan menyediakan tombol melihat contoh aktivitas. Durasi belajar hanya ditampilkan jika ada catatan interaksi yang memadai dan tidak digabungkan sebagai ukuran kemampuan.

Contoh ketika baru ada dua aktivitas pada satu sesi:

> Hari ini Alya mencoba menghitung apel dan membuat kelompok benda. Bukti masih terbatas untuk merangkum penguasaan tingkat. Kami akan mengamati lagi pada sesi berikutnya.

Contoh kegiatan kreasi:

> Alya mengikuti kegiatan membuat gambar empat buah. Kegiatan ini menjadi contoh penerapan jumlah benda. Pengamatan minat masih memerlukan aktivitas tambahan.

Jangan menulis "Alya berbakat visual" dari satu gambar atau "sulit matematika" dari rekaman gagal. Laporan menampilkan apa yang teramati, bantuan, dan kecukupan bukti. Referensi pendek E01–E10 hanya untuk dokumen contoh; implementasi menyimpan ID percobaan asli.

## Keluaran pendidik sebelum publikasi

- Peta seluruh bab/tingkat/keterampilan, prasyarat, dan jalur cadangan untuk rentang usia Fase 1.
- Peninjauan instruksi, audio, gambar, parameter generator, kunci, serta rubrik pada kedua templat dan seluruh jenis respons.
- Evaluasi aturan penempatan, kenaikan, penurunan, dan ambang laporan pada contoh kemampuan berbeda; catat perubahan sebagai versi aturan/rubrik.
- Contoh jawaban mandiri, dibantu, meragukan, karya terbuka, serta data terbatas; setiap contoh memiliki hasil yang diharapkan.
- Persetujuan konten terikat versi dan peninjau; fixture internal tidak otomatis menjadi konten terbit.

Prinsip mengikuti perkembangan kemampuan dan mengamati kemajuan anak mengacu pada [IES: Teaching Math to Young Children](https://ies.ed.gov/ncee/wwc/PracticeGuide/18). Panduan tersebut mendukung pendekatan untuk usia dini, bukan validasi ambang 8/10, pembagian tingkat contoh, atau keseluruhan kurikulum Hurahopi usia 4–10.
