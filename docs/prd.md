# Hurahopi — Kebutuhan Produk (PRD)

[Dokumentasi utama](../README.md).

Tanggal: 25 Oktober 2026. Status: baseline produk disetujui; rincian terbuka mengikuti decisions.md.

Bagian dari [rencana pengembangan](plan.md). [Daftar keputusan](decisions.md) mencatat status persetujuan; aturan produk ada di [PRD](prd.md). Parameter uji tetap memerlukan evaluasi pendidik.

## Ringkasan Produk dan Cakupan

### Tujuan

Hurahopi adalah platform belajar berbayar untuk anak usia **4–10 tahun**. Pada **Fase 1**, bahasa pengantar sepenuhnya bahasa Indonesia dan mata pelajaran hanya **matematika**.

Tujuan Fase 1:

- Membantu orang tua mengenalkan matematika kepada anak sejak dini.
- Membuat belajar matematika menyenangkan melalui permainan dan aktivitas kreatif.
- Mendukung anak yang ingin melangkah ke jalur olimpiade matematika.
- Menyediakan perjalanan belajar yang berbeda untuk setiap anak berdasarkan hasil tes awal; usia hanya menjadi acuan awal, bukan penentu mutlak.
- Membantu orang tua dan pengasuh memahami kemampuan yang teramati, indikasi minat, dan perkembangan anak.

Target pemakaian adalah **5–15 menit per hari**, disesuaikan dengan usia anak.

### Peta Fase

| Fase | Bahasa pengantar | Materi | Status |
| --- | --- | --- | --- |
| Fase 1 | Bahasa Indonesia | Matematika, didukung aktivitas bahasa, visual-spasial, dan kreasi | Ditetapkan |
| Fase 2 (setelah Fase 1 dinilai berhasil) | Bahasa Inggris ditambahkan | Matematika dalam bahasa Inggris, serta English, coding untuk anak, dan Mandarin | Ditetapkan sebagai arah; rincian Terbuka |

Pengenalan bahasa Inggris dan seluruh mata pelajaran selain matematika berada pada Fase 2. Fase 1 hanya menyiapkan fondasi data dan teks antarmuka agar penambahan tersebut tidak memerlukan penulisan ulang ([kesiapan Fase 2](architecture.md#kesiapan-fase-2)).

### Dasar Produk yang Ditetapkan

| Aspek | Ketentuan |
| --- | --- |
| Usia sasaran | 4–10 tahun |
| Mata pelajaran Fase 1 | Matematika saja |
| Bidang pendukung | Bahasa, visual-spasial, dan kreasi, untuk membuat pembelajaran matematika lebih menarik ([peran bidang pendukung](prd.md#peran-bidang-pendukung)) |
| Bahasa Fase 1 | Bahasa Indonesia penuh (teks dan audio) |
| Durasi belajar harian | 5–15 menit, disesuaikan usia |
| Bentuk pembelajaran | Permainan edukatif, tantangan bertingkat, petunjuk, percobaan, dan umpan balik |
| Bentuk jawaban | Pilihan visual, mencocokkan, menyusun, menggambar atau mewarnai, dan jawaban suara pendek |
| Penempatan | Tes pertama menentukan titik mulai tiap anak; tingkat soal diturunkan sampai tingkat yang dipahami anak |
| Struktur materi | Bab dan tingkat (level); setiap bab dapat memiliki satu video penjelasan 1–2 menit (opsional) |
| Pembuatan soal | Dengan bantuan AI, melalui alur terkurasi ([alur soal AI](learning-spec.md#pembuatan-soal-oleh-ai)) |
| Portofolio | Gambar dan rekaman suara anak |
| Laporan | Ringkasan perkembangan, bukti aktivitas, saran pendampingan, dan ekspor PDF |
| Akun | Satu akun orang tua dengan beberapa profil anak; pengasuh dapat diundang dengan akses terpisah |
| Langganan | Per profil anak; satu akun keluarga mengelola beberapa profil dan masa akses masing-masing (D9) |
| Perangkat | Satu sesi belajar aktif per anak; pindah perangkat memerlukan konfirmasi. Login orang tua dapat aktif pada beberapa perangkat (D9) |
| Autentikasi | Google atau tautan masuk melalui email |
| Identitas visual | Mengembangkan Hopi, teman-temannya, dan referensi visual yang tersedia |
| Penghargaan | Petualangan dan koleksi |
| Model komersial | Uji coba tujuh hari tanpa kartu; paket 1, 3, dan 12 bulan; harga dikelola admin; perpanjangan manual |
| Payment gateway | iPaymu (my.ipaymu.com) |
| Stack inti | React + Vite, Go + Fiber, PostgreSQL dikelola sendiri, hosting VM, Cloudinary |
| Sasaran awal | Hingga 500 anak aktif per bulan |
| Pelaksanaan | Bertahap tanpa tenggat tetap; anggaran disediakan sesuai kebutuhan dengan pencatatan pemakaian layanan |

Perbedaan paket (Ditetapkan):

| Paket | Isi |
| --- | --- |
| 1 bulan | Seluruh pembelajaran adaptif, aktivitas pendukung, portofolio, akses pengasuh, dan laporan orang tua |
| 3 bulan | Paket 1 bulan + worksheet printable |
| 12 bulan | Paket 3 bulan + latihan soal olimpiade |

Harga dikelola admin dan belum ditetapkan.

### Peran Bidang Pendukung

Matematika adalah satu-satunya mata pelajaran pada Fase 1. Bahasa, visual-spasial, dan kreasi bukan mata pelajaran terpisah; ketiganya dikemas sebagai aktivitas yang **selalu terkait konsep matematika** yang sedang dipelajari. Pengaturan berikut berstatus **Rekomendasi** dan perlu ditinjau pendidik (D4).

| Bidang pendukung | Peran dalam belajar matematika | Contoh aktivitas |
| --- | --- | --- |
| Bahasa | Memahami istilah matematika dan soal cerita; menyampaikan jawaban dan cara berpikir | Memahami "lebih banyak", "kurang", "jumlah"; mendengar soal cerita Hopi; menyebutkan jawaban atau menjelaskan secara singkat dengan suara |
| Visual-spasial | Memahami bentuk, posisi, pola, dan ruang yang menopang geometri dan penalaran olimpiade | Mencocokkan dan menyusun bangun datar; melanjutkan pola; menyusun potongan; menentukan posisi pada kisi |
| Kreasi | Menerapkan konsep dengan cara bebas sehingga belajar terasa menyenangkan | Menggambar lima benda; mewarnai bagian pecahan; membuat pola sendiri; membuat bentuk dari potongan |

Ketentuan:

- Tingkat dan Peta Belajar utama tetap ditentukan dari penguasaan bab matematika.
- Aktivitas pendukung memberikan bukti tambahan dan indikasi minat, ditampilkan pada laporan sebagai pengamatan, bukan penilaian benar atau salah untuk karya terbuka.
- Porsi aktivitas pendukung dalam sesi disarankan sekitar sepertiga, dikalibrasi melalui uji keluarga.

### Batas Versi Pertama

Asumsi cakupan yang diajukan untuk peninjauan (**Rekomendasi**):

- Pasar awal Indonesia, mata uang rupiah, zona waktu tampilan Asia/Jakarta.
- Pemilik akun dapat mengundang pengasuh dengan akses terpisah ([autentikasi dan otorisasi](architecture.md#autentikasi-dan-otorisasi)).
- Penggunaan daring, dengan penyimpanan progres dan pemulihan sesi ketika koneksi terputus.
- Laporan tersedia di aplikasi dan dapat diekspor sebagai PDF.
- Penilaian gambar mengikuti proses serta rubrik tugas; analisis suara berfokus pada isi jawaban.
- Penilaian pelafalan dan percakapan bebas belum termasuk.
- AI membantu pembuatan soal, transkripsi jawaban suara, dan pengamatan gambar. Aturan naik dan turun tingkat memakai aturan yang dapat dijelaskan ([aturan adaptif](learning-spec.md#aturan-naik-dan-turun-tingkat)); AI belum menentukan tingkat anak.

## Pengguna dan Kebutuhan Fungsional

### Pengalaman Anak

Alur utama: **pendamping memilih profil dan menyiapkan sesi → beranda dunia Hopi → anak mulai/lanjut misi → pengantar bab bila tersedia → aktivitas satu per satu → apresiasi → selesai untuk hari ini**. Beranda anak memiliki satu tindakan utama, peta perjalanan ringkas, dan koleksi. Aktivitas memakai layar khusus dengan audio, objek konkret, petunjuk, serta pilihan berhenti; penawaran paket berada di area dewasa. Lihat [tata letak anak](ux-spec.md#tata-letak-utama-anak) dan [diagram pembelajaran](diagrams/learning.md).

| Jenis interaksi | Contoh aktivitas |
| --- | --- |
| Pilihan visual | Memilih jumlah, bentuk, atau gambar yang tepat |
| Mencocokkan | Memasangkan angka dengan jumlah, bentuk dengan nama |
| Menyusun | Mengurutkan angka, pola, atau potongan bangun |
| Menggambar dan mewarnai | Menggambar jumlah benda, mewarnai bagian pecahan, berkarya bebas dengan instruksi sederhana |
| Jawaban suara | Menyebut angka, istilah, atau jawaban pendek; menjelaskan singkat |
| Soal cerita bergambar | Mendengar cerita Hopi lalu menjawab |

Prinsip yang direkomendasikan:

- Satu fokus per layar, tombol sentuh besar, dan audio instruksi berbahasa Indonesia yang dapat diputar ulang, karena anak 4–6 tahun belum mandiri membaca.
- Aktivitas menyusun menyediakan alternatif ketuk-pilih untuk layar sentuh. Jawaban suara selalu memiliki alternatif ketuk-pilih.
- Angka dan jumlah ditampilkan dengan benda bergambar sebelum simbol pada tingkat awal.
- Petunjuk dan percobaan ulang tersedia; jawaban salah tidak diberi suara atau tampilan yang menghukum.
- Anak dapat memilih personalisasi, seperti dekorasi atau koleksi, sementara sistem menentukan aktivitas belajar.
- Penghargaan berupa stiker, lencana, dan koleksi Hopi mengikuti partisipasi dan kemajuan. Rangkaian hari belajar (streak) ditampilkan secara ramah dan tidak menampilkan kehilangan sebagai hukuman.
- Tidak ada penghitung waktu yang menekan anak.

Templat permainan yang dapat dipakai ulang oleh banyak bab dan tingkat:

| Templat | Contoh penggunaan |
| --- | --- |
| Menghitung benda | Membilang 1–10, membandingkan jumlah |
| Seret dan pasangkan | Mencocokkan angka dengan jumlah, bentuk dengan nama |
| Garis bilangan | Penjumlahan dan pengurangan, urutan angka |
| Bingkai sepuluh dan pasangan bilangan | Pemahaman bilangan sampai 20 |
| Menyusun dan mengurutkan | Pola, urutan, potongan bangun |
| Kanvas gambar dan warna | Menggambar jumlah, mewarnai bagian, membuat pola |
| Jawab dengan suara | Menyebut angka atau istilah |
| Memori dan mencocokkan | Pengenalan angka dan bentuk |
| Cepat tepat (usia lebih besar) | Kefasihan fakta penjumlahan dan perkalian |
| Soal cerita bergambar | Penerapan operasi hitung |

### Rancangan Sesi

Parameter durasi, komposisi, dan pengenalan berikut berstatus **Rekomendasi** untuk diuji bersama keluarga serta ditinjau pendidik. Aturan perangkat pada tabel sudah **Ditetapkan — D9**.

| Parameter | Rancangan awal |
| --- | --- |
| Pengenalan | Tiga sesi awal memperkenalkan seluruh jenis interaksi (pilihan, mencocokkan, menyusun, menggambar, suara) agar bukti tersedia untuk setiap bentuk respons; penyelesaian mengikuti bukti valid yang terkumpul |
| Sasaran durasi | Usia 4–5 tahun sekitar 5 menit; usia 6–7 tahun sekitar 8–10 menit; usia 8–10 tahun sekitar 10–15 menit. Durasi mengikuti usia acuan, lalu disesuaikan oleh perilaku sesi anak |
| Isi sesi | Sekitar 4–8 aktivitas pendek; sesi dapat diakhiri lebih awal kapan saja |
| Komposisi | Sekitar dua pertiga aktivitas matematika inti pada tingkat yang sedang dilatih (termasuk ulasan ringan dan sesekali tantangan satu tingkat di atas) dan sekitar sepertiga aktivitas pendukung yang terkait bab yang sama |
| Cakupan bidang pendukung | Ketiga bidang pendukung ditawarkan dalam rentang tiga sesi; sesi lebih pendek diperhitungkan melalui riwayat |
| Akhir sesi | Sistem menutup sesi setelah sasaran tercapai dengan ajakan kembali besok; tambahan satu putaran bersifat pilihan dan tidak terus-menerus |
| Perangkat | Satu sesi belajar aktif per anak, termasuk saat didampingi pengasuh; laporan dan pembayaran dapat dibuka di perangkat lain ([sesi belajar dan perangkat](architecture.md#sesi-belajar-dan-perangkat)) |

Aturan pemulihan dan penyelesaian yang diusulkan:

- Simpan percobaan bersama versi aktivitas, aset, rubrik, dan parameter soal yang ditampilkan. Memuat ulang halaman mempertahankan percobaan yang sama.
- Sediakan petunjuk, percobaan ulang, pilihan melewati aktivitas, dan penghentian sesi. Bedakan penyelesaian mandiri dari penyelesaian dengan bantuan.
- Catat aktivitas yang dilewati atau gagal secara teknis sebagai konteks, tanpa mengubahnya menjadi jawaban salah.
- Tawarkan aktivitas lain atau alternatif ketuk-pilih ketika mikrofon tidak diizinkan atau media gagal diproses. Kemampuan pada bentuk respons yang tidak teramati tetap berstatus data terbatas.
- Berikan penghargaan satu kali untuk setiap penyelesaian. Pengiriman ulang jawaban atau analisis tidak menambah penghargaan yang sama.
- Terapkan hasil analisis terlambat (suara atau gambar) pada pembaruan profil berikutnya. Aktivitas yang telah ditampilkan dan jawaban yang telah dikirim tetap dipertahankan.

### Pengalaman Orang Tua dan Pengasuh

**Orang tua (pemilik akun)** dapat membuat serta mengelola profil anak, meninjau laporan dan mengekspornya sebagai PDF, mengelola langganan, mengakses portofolio, mengundang dan mencabut pengasuh, mengelola perangkat aktif, mengunduh worksheet (paket 3 dan 12 bulan), serta mengakses latihan olimpiade (paket 12 bulan). Portofolio mendukung tampilan gambar, pemutaran suara, pengunduhan, dan penghapusan media.

Beranda orang tua berfokus pada anak yang dipilih: status misi dan tombol Mulai belajar/Lanjutkan, ringkasan kegiatan, perkembangan dengan bukti, saran rumah, serta karya terbaru yang diizinkan. Navigasi desktop memuat Beranda, Perkembangan, Karya, Keluarga, dan Langganan; ponsel memakai Beranda, Perkembangan, Karya, dan Menu. Komponen sumber disesuaikan menjadi pengalaman pendamping keluarga. Lihat [tata letak orang tua](ux-spec.md#tata-letak-orang-tua).

**Pengasuh** adalah pengguna terpisah dengan login sendiri yang diundang oleh pemilik akun. Hak pengasuh dibatasi dan diatur per profil anak oleh pemilik akun:

| Kemampuan | Pemilik akun | Pengasuh |
| --- | --- | --- |
| Mendampingi dan memulai sesi belajar pada profil yang diizinkan | Ya | Ya |
| Melihat laporan dan Peta Belajar | Ya | Ya, pada profil yang diizinkan |
| Mengekspor laporan PDF | Ya | Ya bila diizinkan pemilik |
| Melihat portofolio | Ya | Ya bila diizinkan pemilik |
| Mengunduh atau menghapus media portofolio | Ya | Tidak |
| Membeli atau mengelola langganan | Ya | Tidak |
| Mengundang pengguna lain, mengubah profil, atau menghapus akun | Ya | Tidak |

Pemilik akun dapat mencabut akses pengasuh kapan saja, dan setiap undangan, pencabutan, serta perubahan izin dicatat. D14 menetapkan maksimal dua pengasuh per keluarga. Izin awal per profil adalah mendampingi belajar dan melihat laporan; akses portofolio serta ekspor PDF memerlukan aktivasi terpisah oleh orang tua. Undangan menunggu dan pengasuh aktif bersama-sama memakai kuota dua; pencabutan atau kedaluwarsa undangan membebaskan kuota. Ketersediaan pengasuh selama uji coba direkomendasikan; finalisasi mengikuti D3.

Izin portofolio membatasi tampilan/pemutaran pada profil yang dipilih. Laporan dan halaman bukti tidak menyisipkan gambar/audio portofolio bagi pengasuh tanpa izin portofolio. Ekspor PDF pengasuh memuat laporan tanpa salinan media anak agar izin ekspor tidak melewati larangan mengunduh media; pemilik akun dapat memilih menyertakan gambar pada ekspornya sendiri.

**Laporan orang tua** memuat:

- **Peta Belajar**: tingkat anak pada setiap bab matematika dan perubahannya.
- **Pengamatan pendukung**: indikasi pada bahasa, visual-spasial, dan kreasi, beserta contoh bukti.
- Indikasi minat, waktu belajar harian dan mingguan, serta jumlah hari belajar.
- Penjelasan bila tingkat soal diturunkan atau dinaikkan, supaya orang tua memahami alasannya.
- Saran pendampingan singkat (misalnya aktivitas menghitung benda di rumah).

Orang tua dapat meminta tes ulang atau mengatur titik mulai bab tertentu bila merasa hasil penempatan kurang tepat. Profil awal memuat nama panggilan, usia dalam tahun, dan avatar. Informasi bantuan orang tua ditampilkan ketika diketahui dari interaksi atau konfirmasi.

Pengaturan awal mencakup persetujuan penggunaan AI, pemrosesan media, dan penyimpanan portofolio. Area orang tua dilindungi PIN ketika perangkat berada dalam mode anak, dengan pemeriksaan akses pada backend; pengasuh memakai PIN sendiri.

### Pengelolaan Konten dan Operasional

Panel admin mencakup pengelolaan bab, tingkat, keterampilan, templat aktivitas dan parameter pembuat soal, **antrean draf soal AI dan peninjauan**, aset dan narasi, rubrik, video bab, berkas worksheet, soal olimpiade, paket harga, akses pelanggan (termasuk penyesuaian manual dan keluar paksa), serta pemantauan proses AI, pesanan, dan pembayaran.

Peran yang direkomendasikan adalah orang tua, pengasuh, editor konten, peninjau konten, dan pengelola operasional. Editor menyiapkan draf (termasuk draf dari AI); peninjau menyetujui konten sebelum terbit. Pemberian peran admin dicatat secara eksplisit. Hak editor dan peninjau dibatasi pada pengelolaan konten, dengan akses portofolio anak diatur terpisah.

## Desain Antarmuka dan Komponen UI

### Arah Desain

Rancangan yang direkomendasikan menggunakan satu sistem desain Hurahopi dengan penyesuaian pengalaman untuk setiap kelompok pengguna.

| Area | Karakter pengalaman |
| --- | --- |
| Anak | Karakter Hopi dan teman-temannya, tombol sentuh besar, objek bergambar, audio, satu fokus aktivitas per layar, dan animasi sebagai umpan balik. Antarmuka tenang untuk usia 4–5 tahun dan lebih padat untuk usia 8–10 tahun |
| Orang tua dan pengasuh | Navigasi sederhana, tipografi jelas, Peta Belajar dan ringkasan berbukti yang mudah dipahami, grafik berlabel |
| Admin | Formulir, tabel, filter, antrean peninjauan draf soal, dialog konfirmasi, dan status konten yang konsisten |

Fondasi bersama mencakup warna semantik, tipografi, jarak antarelemen, radius, status interaksi, fokus keyboard, dan dukungan pengurangan animasi. Tata letak responsif untuk ponsel, tablet, dan PC. Identitas visual dikembangkan dari referensi Hurahopi yang tersedia. Sebelum implementasi, tinjau contoh layar tes penempatan, sesi anak, laporan orang tua, halaman paket, dan editor admin sebagai dasar penyepakatan komponen inti.

### Komponen yang Digunakan

**Ditetapkan — D1/D6.** Gunakan satu sistem shadcn/ui varian Base UI dan Tailwind CSS. Dashboard orang tua mengadaptasi layout/tabel terpilih dari Satnaing Shadcn Admin. Panel editor/operasional memakai Shadcn Admin Kit; menu dan data yang terlihat tetap mengikuti peran backend. Aktivitas anak memakai renderer khusus React/DOM dengan alternatif ketuk/keyboard; dnd-kit mendukung drag ketika diperlukan dan Konva/react-konva dipakai untuk kanvas pada tahap media.

Komponen sumber diselaraskan dengan identitas Hurahopi, aksesibilitas, dan kontrak API. Auth/fixture demo, branding sumber, atau generic CRUD tidak boleh menggantikan PIN, izin keluarga, dan peninjauan konten. Laporan PDF dibuat worker melalui Gotenberg dan tetap mengikuti izin, bukti, serta retensi laporan. Pemilihan library tidak mengubah cakupan fitur maupun aturan pedagogis.

Daftar paket, tahap pemasangan, alasan, dan sumber lisensi berada pada [arsitektur](architecture.md#stack-inti-dan-layanan-pendukung). Semua komponen tetap diuji pada alur pengguna; status dipilih tidak sama dengan fitur sudah selesai diimplementasikan.

## Langganan dan Pembayaran

### Ketentuan yang Ditetapkan

- Uji coba tujuh hari tanpa kartu.
- Paket 1, 3, dan 12 bulan; harga dapat diatur admin; pembayaran per periode dengan perpanjangan manual.
- Paket 3 bulan menambahkan worksheet printable; paket 12 bulan menambahkan worksheet printable dan latihan olimpiade.
- Payment gateway: iPaymu.

Harga peluncuran belum ditetapkan.

Ringkasan fitur menurut status akses (**Rekomendasi** untuk baris yang belum disebut pada ketentuan di atas):

| Fitur | Uji coba | 1 bulan | 3 bulan | 12 bulan | Setelah akses berakhir |
| --- | --- | --- | --- | --- | --- |
| Pembelajaran adaptif dan permainan | Ya | Ya | Ya | Ya | Tidak |
| Aktivitas pendukung, suara, dan menggambar | Ya, dengan batas wajar pemrosesan media | Ya | Ya | Ya | Tidak |
| Portofolio | Ya | Ya | Ya | Ya | Lihat, unduh, hapus |
| Laporan dan ekspor PDF | Ya | Ya | Ya | Ya | Ya |
| Akses pengasuh | Ya | Ya | Ya | Ya | Laporan dan portofolio sesuai izin |
| Worksheet printable | 1 contoh | Tidak | Ya | Ya | Hanya yang telah diunduh |
| Latihan olimpiade | Pratinjau terkunci | Tidak | Tidak | Ya | Tidak |

### Rancangan Uji Coba Tujuh Hari

Tujuan uji coba adalah membuat orang tua **melihat bukti bahwa anaknya belajar dan platform memahami anaknya**, bukan sekadar mencoba permainan. Dengan 5–15 menit per hari, tujuh hari menghasilkan paling banyak sekitar 35–105 menit pemakaian. Karena itu seluruh konten pada jalur anak sebaiknya terbuka, sementara fitur yang menjadi pembeda paket diperkenalkan melalui pratinjau.

**Rekomendasi isi uji coba:**

| Fitur | Uji coba | Alasan |
| --- | --- | --- |
| Tes penempatan | Penuh | Momen keunggulan utama: perjalanan anak terasa dipersonalisasi sejak hari pertama |
| Latihan adaptif, permainan, naik dan turun tingkat | Penuh, tanpa batas tingkat atau bab pada jalur anak | Pembatasan tingkat menyembunyikan kecerdasan sistem; biaya tambahan per pengguna kecil |
| Aktivitas bahasa, visual-spasial, dan kreasi | Penuh | Membuat pengalaman lebih menarik dan memberi bukti tambahan untuk laporan |
| Jawaban suara dan menggambar | Penuh, dengan batas wajar pemrosesan media AI per hari | Setiap pemrosesan memiliki biaya; uji coba tanpa kartu rentan disalahgunakan |
| Portofolio | Ya; tetap dapat dilihat dan diunduh setelah uji coba | Karya anak yang tersimpan menjadi alasan emosional untuk kembali |
| Pengantar bab dan video | Penuh | Bagian dari pengalaman belajar inti |
| Laporan, Peta Belajar, dan ekspor PDF | Penuh, ditambah ringkasan khusus hari ke-3 dan hari ke-7 | Bukti kemajuan adalah pendorong utama pembelian; PDF dapat dibagikan kepada pasangan atau guru |
| Akses pengasuh | Ya | Pengasuh yang ikut mendampingi meningkatkan penggunaan harian |
| Worksheet printable | Satu worksheet contoh sesuai tingkat anak | Memberi sesuatu yang nyata untuk dicetak; penuh hanya pada paket 3 dan 12 bulan |
| Latihan olimpiade | Pratinjau terkunci dengan beberapa soal contoh | Memperkenalkan nilai paket 12 bulan tanpa membuka isinya |
| Satu sesi belajar aktif per anak | Berlaku sejak uji coba | Pemindahan perangkat dikonfirmasi; laporan orang tua tetap dapat diakses |

**Titik komunikasi yang direkomendasikan:**

1. **Setelah tes penempatan:** tampilkan Peta Belajar awal kepada orang tua dengan bahasa positif ("Hopi menemukan titik mulai terbaik untuk [nama]").
2. **Hari ke-3:** ringkasan awal berisi tingkat yang sudah dilewati, waktu belajar, contoh karya anak, dan saran pendampingan.
3. **Hari ke-5:** pengingat paket dan perbedaan 1, 3, serta 12 bulan.
4. **Hari ke-7:** ringkasan satu minggu (hari belajar, jumlah aktivitas, tingkat naik, bagian yang dikuasai, karya terbaik) beserta PDF, dengan ajakan memilih paket.
5. **Setelah uji coba berakhir:** progres anak, laporan, dan portofolio tetap dapat dibuka; anak tidak dapat memulai sesi baru sampai berlangganan.
6. **Penawaran khusus yang jujur** (misalnya diskon pembelian dalam 48 jam setelah uji coba) dapat dipertimbangkan, selama batas waktunya benar-benar berlaku dan tidak diperbarui otomatis.

Hal yang dihindari: penghitung waktu palsu, pesan yang membuat orang tua merasa bersalah, serta tekanan kepada anak untuk meminta orang tua membeli.

**Parameter yang diukur** untuk menyesuaikan rancangan: persentase orang tua yang menyelesaikan tes penempatan, jumlah hari belajar pada tujuh hari, penggunaan suara dan menggambar, biaya pemrosesan AI per anak uji coba, konversi uji coba ke pembelian per paket, dan titik anak berhenti belajar.

### Rancangan Aturan Komersial

Aturan waktu trial, perhitungan durasi, dan penyusunan segmen berikut masih **Rekomendasi** sesuai D3/D10. Unit pesanan per anak/kursus sudah **Ditetapkan — D9**; verifikasi pembayaran dan akses server menjadi syarat teknis implementasi.

- Uji coba dimulai pada sesi pertama (tes penempatan), dengan batas penerima mengikuti D3. Penggantian profil tidak mereset riwayat uji coba pada akun.
- Durasi paket dihitung dalam bulan kalender. Waktu disimpan dalam UTC dan ditampilkan dalam Asia/Jakarta.
- Pembelian selama akses aktif menambah masa akses dari akhir hak akses yang tersedia, termasuk sisa uji coba. Pembelian setelah akses berakhir dimulai dari konfirmasi pembayaran berhasil.
- Setiap pesanan terkait tepat satu profil anak, satu kursus, dan satu versi harga yang dibekukan saat checkout dibuat (D9). Checkout menampilkan nama profil penerima. Pembelian profil lain tidak memperpanjang akses profil ini.
- **Hak akses tersimpan sebagai segmen waktu**: setiap pembelian membuat segmen dengan tanggal mulai, tanggal akhir, dan tingkat paket. Fitur yang berlaku pada suatu saat mengikuti segmen yang sedang aktif. Aturan peningkatan paket di tengah masa aktif ditetapkan pada D10; versi pertama dapat memakai segmen berurutan tanpa perhitungan pro rata.
- Aktivasi akses mengikuti verifikasi server. Notifikasi berulang tidak menambah masa akses dua kali; notifikasi terlambat tidak mengembalikan transaksi berhasil menjadi menunggu.
- Sesi yang sedang berlangsung dapat diselesaikan ketika akses habis. Sesi baru dan kelanjutan sesi yang telah ditinggalkan memeriksa ulang hak akses.
- Orang tua tetap dapat membuka laporan dan portofolio setelah akses belajar berakhir.
- Refund versi pertama ditangani melalui dukungan dan penyedia pembayaran, dengan penyesuaian akses tercatat. Ketentuan refund, profil berbayar yang dihapus, dan kanal dukungan ditetapkan sebelum penjualan (D7).
- Tampilan paket: paket 3 bulan dapat disorot sebagai pilihan utama dan paket 12 bulan sebagai nilai terbaik per bulan; perbedaan fitur ditampilkan berdampingan.

### Integrasi iPaymu

Integrasi memakai **Redirect Payment** (**Ditetapkan — D6**; iPaymu tetap gateway yang ditetapkan). Pembeli diarahkan ke halaman pembayaran iPaymu; metode pembayaran seperti virtual account, QRIS, gerai retail, dan kartu mengikuti yang aktif pada akun merchant (dikonfirmasi pada D6). Perpanjangan manual sesuai kebutuhan karena rancangan tidak memerlukan tagihan berulang otomatis.

Alur:

1. Orang tua memilih paket; backend membuat pesanan dengan harga yang dibekukan dan identitas referensi unik (`referenceId`). Pembelian hanya dapat dilakukan oleh pemilik akun, bukan pengasuh.
2. Backend membuat transaksi Redirect Payment dengan `notifyUrl`, `returnUrl`, dan `cancelUrl`, lalu mengarahkan orang tua ke halaman pembayaran iPaymu. Batas waktu pembayaran ditetapkan (misalnya 24 jam untuk virtual account).
3. Setelah pembayaran, iPaymu mengirim **callback** ke `notifyUrl`. Halaman `returnUrl` hanya menampilkan status kepada pengguna dan tidak mengaktifkan akses.
4. Backend memvalidasi tanda tangan callback sesuai [dokumentasi callback iPaymu](https://docs.ipaymu.com/en/docs/callback) dan **memverifikasi status melalui API pengecekan transaksi iPaymu** sebelum mengaktifkan akses.
5. Aktivasi bersifat idempoten berdasarkan ID transaksi iPaymu dan pesanan; callback yang sama dapat tiba berkali-kali.
6. Worker menjalankan rekonsiliasi berkala untuk pesanan yang masih menunggu atau berstatus ambigu, sehingga pembayaran yang callback-nya terlewat tetap teraktivasi.
7. Pemetaan status transaksi (misalnya status selesai langsung dan status diterima dengan penyelesaian terpisah) dikonfirmasi pada lingkungan sandbox sebelum integrasi dianggap selesai.

Akun iPaymu memiliki lingkungan sandbox dan produksi dengan kredensial berbeda. Kunci API dan kode VA hanya berada di server. Dokumentasi resmi dan aktivasi merchant diperiksa sebelum implementasi, karena rincian API dapat berubah.

## Kriteria penerimaan produk

ID kebutuhan berikut dipakai untuk menautkan implementasi dan pengujian dalam [delivery](delivery.md). Cakupan penuh Fase 1 tetap berlaku; milestone awal hanya memilih urutan implementasi.

Seluruh kebutuhan di bawah dipetakan ke [activity diagram per fitur](../README.md#pemetaan-ke-prd); diagram mempertahankan status kebijakan dan parameter pada daftar keputusan.

| ID | Kebutuhan | Bukti penerimaan |
| --- | --- | --- |
| FR-01 | Identitas dan keluarga | Google atau tautan email sekali pakai; data keluarga lain tidak dapat diakses; sesi orang tua pada perangkat lain tetap aktif |
| FR-02 | Profil dan persetujuan | Orang tua mengelola nama panggilan, usia, avatar, dan persetujuan per anak; setiap perubahan tercatat |
| FR-03 | Mode anak | PIN diverifikasi server; konteks anak tidak dapat membaca laporan, membeli, mengundang pengasuh, atau mengubah profil |
| FR-04 | Penempatan | Tes singkat, adaptif secara aturan, bisa dilanjutkan; hasil yang belum cukup diberi status sementara; kegagalan teknis tidak menjadi salah |
| FR-05 | Sesi belajar | Satu sesi aktif per anak; konfirmasi pemindahan tidak kehilangan progres; anak berbeda dapat belajar bersamaan |
| FR-06 | Aktivitas dan progres | Penilaian objektif di server; petunjuk/bantuan tercatat; muat ulang mempertahankan percobaan; pengiriman ulang tidak menggandakan progres atau hadiah |
| FR-07 | Adaptif dan laporan | Setiap perubahan tingkat mempunyai versi aturan serta referensi bukti; laporan membedakan data terbatas dan penguasaan |
| FR-08 | Konten dan AI | Setiap versi terbit telah ditinjau; AI hanya menghasilkan draf; versi lama tetap dapat dirujuk percobaan |
| FR-09 | Media dan portofolio | Alternatif ketuk tersedia; akses media diperiksa server; persetujuan serta kebijakan retensi diberlakukan sebelum penggunaan data anak |
| FR-10 | Pengasuh | Maksimal dua per keluarga; izin per profil; portofolio/PDF awalnya nonaktif; pencabutan berlaku pada permintaan berikutnya |
| FR-11 | PDF | Ekspor diperiksa per profil dan izin; bukti konsisten; berkas sementara dapat dibatalkan/dihapus |
| FR-12 | Akses komersial | Hak belajar per anak/kursus; paket membatasi worksheet/olimpiade; laporan dan portofolio tetap tersedia setelah masa belajar habis |
| FR-13 | Pembayaran | Hanya pemilik akun; callback diverifikasi dan status diperiksa ke penyedia; satu pembayaran mengaktifkan akses satu kali; rekonsiliasi tersedia |
| FR-14 | Privasi dan operasi | Penghapusan mencakup turunan dan ekspor; data terhapus tidak muncul kembali setelah restore; pemulihan diuji |

## Batas keputusan komersial yang belum final

Tujuh hari tanpa kartu dan paket 1/3/12 bulan sudah ditetapkan. Usulan maksimal tiga penerima uji coba per keluarga, waktu mulai uji coba, kuota AI harian, serta perpanjangan uji coba belum final (D3). Gunakan fixture akses sintetis untuk pengembangan internal sampai D3 diputuskan; fixture tidak tersedia pada produksi. Harga, refund, dan penggabungan paket mengikuti D7/D10. Status rekomendasi pada rincian komersial tidak boleh ditampilkan sebagai ketentuan penjualan final.
