# Hurahopi: arah desain antarmuka

[Dokumentasi utama](../README.md).

Tanggal: 26 Oktober 2026. Acuan visual untuk prototipe dan implementasi S04. Token berikut merapikan arah yang sudah terlihat pada referensi Hopi dan wireframe; kegunaannya tetap diuji pada prototipe. Perilaku layar mengikuti [UX](ux-spec.md), kebutuhan mengikuti [PRD](prd.md), dan status persetujuan mengikuti [keputusan](decisions.md).

## Sumber dan batas tanggung jawab

| Acuan | Yang diambil |
| --- | --- |
| Papan karakter Hopi lokal: `references/hurahopi-site/index.html` dan `references/hurahopi-site/style.css` | Karakter, suasana petualangan, keluarga warna hijau/krim, Nunito dan Baloo 2 |
| [Wireframe pendamping](wireframes/parent-home.svg) | Hierarki ringkasan per anak, misi, bukti, dan tindakan pendamping |
| [Wireframe anak](wireframes/child-experience.svg) | Satu misi utama, panggung ilustrasi, objek konkret, aktivitas fokus |
| [Arsitektur UI](architecture.md#pemakaian-ulang-dan-batas-modul) | Primitive Base UI/Tailwind bersama, adaptasi Satnaing untuk pendamping, Admin Kit untuk admin |

Dokumen ini menetapkan bahasa visual dan alasan pemilihannya. Path `references/` relatif terhadap root proyek, bersifat lokal opsional, dan tidak disertakan di Git. Token serta keputusan visual di bawah dapat dipakai tanpa folder tersebut. Urutan layar, izin, dan transisi tetap berada di UX/activity diagram. Sketsa lama memperlihatkan struktur; ilustrasi placeholder dan bentuk semua komponennya belum menjadi desain final. Aset sumber dipakai setelah hak penggunaannya dikonfirmasi; pembuatan logo, karakter, atau foto baru merupakan pekerjaan tersendiri.

## Identitas dan Design Read

Hopi mengajak anak melakukan kegiatan kecil yang konkret: menyiapkan bekal, mengelompokkan buah, atau menyusun benda. Kehangatan datang dari karakter, ekspresi, benda yang bisa dihitung, dan suara instruksi. Area dewasa memberi ruang untuk membaca bukti serta menentukan pendampingan.

| Area | Design Read | ENERGY | RHYTHM | MOTION |
| --- | --- | --- | --- | --- |
| Anak | Dunia Hopi yang hangat, satu ajakan utama, lalu satu tugas konkret | 2 | 2 | 1 |
| Orang tua/pengasuh | Ringkasan per anak yang tenang, dengan bukti dan tindakan belajar yang jelas | 1 | 2 | 1 |
| Admin | Ruang kerja editorial yang padat secukupnya, dengan hierarki tindakan/revisi | 1 | 1 | 1 |

ENERGY menunjukkan intensitas sambutan; RHYTHM menunjukkan variasi komposisi; MOTION menunjukkan banyaknya gerak. Nilai 1–3 memakai istilah [anti-slop](../skills/antislop/SKILL.md). Nilai diterapkan per area, konsisten di dalam layar. Motion 1 berarti perubahan state dan hover/focus secukupnya; tidak ada loop karakter, parallax, scroll-reveal, atau confetti sebagai default.

Motif identitas adalah perjalanan kecil Hopi: adegan dan satu tujuan per misi. Pada layar aktivitas, benda dan instruksi menjadi pusat perhatian; karakter mendampingi dari tepi. Pada laporan, motif cukup berupa penanda bab dan ilustrasi kecil yang terkait aktivitas. Satu layar mempunyai satu fokus utama; bagian lain mendukungnya.

## Warna dan pemakaian

| Token | Nilai | Pemakaian dan alasan |
| --- | --- | --- |
| `color.canvas` | `#F6F5EF` | Latar dewasa yang terang dan hangat, berasal dari wireframe |
| `color.scene` | `#FFF8E8` | Panggung anak dan ruang ilustrasi; membedakan dunia belajar dari area dewasa |
| `color.surface` | `#FFFFFF` | Bidang baca, form, pilihan, dan dialog dengan latar solid |
| `color.ink` | `#173F35` | Teks utama dan angka; mempertahankan keluarga hijau dengan kontras kuat |
| `color.muted` | `#526C63` | Teks pendukung dan batas kontrol yang perlu terlihat |
| `color.primary` | `#126E60` | Tindakan utama, tautan, pilihan aktif dan focus ring |
| `color.accent` | `#FFE19A` | Satu sorotan hangat pada momen misi/apresiasi, dengan teks ink |
| `color.separator` | `#DCE4DA` | Pemisah dekoratif; bukan satu-satunya batas input atau status |
| `color.error` | `#B34735` | Galat form/operasi dewasa, selalu disertai penjelasan teks |

Hijau, bidang netral terang, dan satu aksen hangat membentuk bahasa UI. Warna benda/karakter mengikuti kebutuhan materi, seperti apel yang harus bisa dibedakan; warna ilustrasi tidak dipakai sebagai status jawaban. Teks ditempatkan pada bidang solid, bukan langsung di atas ilustrasi yang berubah-ubah. Tema terang dipilih untuk panggung ilustrasi dan pembacaan laporan; Fase 1 tidak menambahkan toggle tema tanpa rancangan serta pengujian tema kedua.

Pasangan referensi hasil perhitungan dengan [rumus kontras](../skills/antislop-human/SKILL.md):

| Teks | Latar | Rasio | Hasil target 4,5:1 |
| --- | --- | --- | --- |
| `#FFFFFF` | `#126E60` | 6,13:1 | PASS |
| `#173F35` | `#FFF8E8` | 11,03:1 | PASS |
| `#526C63` | `#FFFFFF` | 5,70:1 | PASS |
| `#173F35` | `#FFE19A` | 9,17:1 | PASS |
| `#B34735` | `#FFFFFF` | 5,43:1 | PASS |
| `#173F35` | `#FFFFFF` | 11,67:1 | PASS |
| `#173F35` | `#F6F5EF` | 10,69:1 | PASS |
| `#526C63` | `#F6F5EF` | 5,22:1 | PASS |
| `#126E60` | `#FFFFFF` | 6,13:1 | PASS |
| `#126E60` | `#F6F5EF` | 5,61:1 | PASS |
| `#126E60` | `#FFF8E8` | 5,79:1 | PASS |
| `#526C63` | `#FFF8E8` | 5,39:1 | PASS |

Target Hurahopi adalah 4,5:1 untuk seluruh teks produk, termasuk label berukuran 18px. [WCAG 1.4.3](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) membedakan teks besar berdasarkan 18pt atau 14pt tebal, bukan sembarang teks 18px. Shorthand pada skill upstream tidak dipakai untuk melonggarkan target. Kontrol, selected state, dan fokus tetap memerlukan batas yang terlihat; separator dekoratif tidak menggantikan focus ring atau label status. Pasangan hover/disabled/gambar yang belum dirender belum dianggap teruji oleh tabel ini.

## Tipografi, ukuran, dan ruang

Nunito menjadi font teks, label dan angka: bentuknya mengikuti papan merek dan dapat dipakai konsisten pada ketiga area. Baloo 2 digunakan terbatas pada sapaan/judul anak untuk suara Hopi yang bermain; instruksi tugas, form, dan tabel memakai Nunito agar mudah dipindai. Font sistem menjadi fallback sampai berkas/font berlisensi siap dipasang; proses build tidak bergantung pada Google Fonts jarak jauh.

| Elemen | Rentang awal | Pemakaian |
| --- | --- | --- |
| Sapaan/judul dunia anak | 28–40px, tebal | Satu judul pendek, menyesuaikan lebar |
| Instruksi anak | 22–28px, tebal | Satu tujuan, ditambah audio yang dapat diulang |
| Label tindakan anak | 18–22px | Kata kerja yang spesifik, bukan label seluruhnya kapital |
| Teks dewasa | 16–18px | Paragraf/form/laporan dengan line-height sekitar 1,5 |
| Judul dewasa | 24–32px | Menunjukkan anak, periode, atau tugas editorial |
| Metadata dewasa | 14–16px | Tanggal/status sekunder; tidak untuk instruksi penting |

Angka di tabel adalah referensi ukuran pada zoom normal; implementasi memakai rem serta clamp atau perubahan ukuran menurut kebutuhan konten. Angka latihan dan jumlah objek lebih menonjol daripada metadata. Baris teks dewasa dibatasi sekitar 60–70 karakter; teks dan kontrol tidak dipotong saat zoom 200%.

Skala ruang: 4, 8, 12, 16, 24, 32, 48. Jarak kecil mengikat label dengan kontrol; jarak lebih besar memisahkan tugas/kelompok informasi. Radius awal: 8px untuk input, 12px untuk tombol, 16px untuk panel/dialog, 24px untuk panggung ilustrasi anak. Bayangan hanya pada lapisan di atas konten, seperti dialog/menu; panel biasa cukup memakai bidang dan jarak.

Target sentuh dewasa minimal 48 × 48 CSS px dan anak minimal 56 × 56, dengan jarak antar-target anak sekitar 12px. Ini pilihan prototipe Hurahopi yang diuji pada perangkat sasaran; [WCAG 2.5.8](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html) menetapkan batas AA 24 × 24 dengan pengecualian tertentu. Ukuran target tidak mengecil hanya untuk mempertahankan jumlah kolom.

## Komposisi layar

### Beranda anak

Satu adegan Hopi dan satu ajakan misi menjadi fokus. Nama panggilan/avatar menunjukkan siapa yang belajar. Mulai misi atau Lanjutkan membuka sesi yang sudah disiapkan pendamping. Petualanganku dan Koleksiku adalah tindakan pendukung. Area orang tua tetap terlihat tetapi memerlukan PIN sebelum data dewasa dimuat.

Panggung ilustrasi memperoleh ruang paling besar; tombol utama berada dekat ajakan. Tidak ada daftar menu dashboard, kartu metrik, skor kemampuan, atau penawaran paket di dunia anak. Apresiasi menunjukkan kegiatan yang selesai tanpa peringkat atau ancaman kehilangan streak.

### Aktivitas anak

Urutannya adalah berhenti/progres langkah, instruksi dan Dengar lagi, objek/pilihan, lalu tindakan menjawab/bantuan. Panggung benda menjadi fokus. Ketuk memilih, tombol konfirmasi mengirim; pemilihan lokal tidak dianggap respons server. Pilihan tetap sama saat reload. Hint muncul setelah bantuan dicatat; pengulangan audio tidak dihitung sebagai hint.

Satu kolom dipakai bila tiga pilihan tidak muat tanpa memperkecil benda atau target. Saat ruang cukup, pilihan menjadi dua atau tiga kolom dengan lebar nyaman. Tidak mengunci orientasi atau memerlukan layar penuh. Latar dekoratif tidak boleh membuat benda yang dihitung sulit dibedakan.

### Orang tua dan pengasuh

Beranda mempunyai pemilih profil yang jelas, ringkasan periode, satu tindakan menyiapkan belajar, lalu bukti dan saran pendampingan. Pada layar lebar, ringkasan dan tindakan dapat berdampingan; pada layar sempit, konteks anak dan tindakan utama muncul lebih dahulu. Laporan menggunakan judul, paragraf singkat, baris status dan bukti; tidak semua informasi dijadikan kartu seragam.

Pengasuh melihat profil/fitur yang diizinkan. Kontrol pembelian, pengubahan profil atau unduhan media tidak dimunculkan sebagai tindakan yang dapat dipakai pengasuh. Seluruh tampilan tetap mengikuti izin terbaru server. Keadaan data terbatas ditampilkan sebagai keterbatasan bukti, bukan persentase kemampuan yang dibuat-buat.

### Admin

Daftar dan editor mengambil struktur Admin Kit, memakai token yang sama. Fokus mengikuti tugas: antrean yang perlu ditinjau, detail revisi, preview aktual, bukti validator, lalu keputusan. Publikasi terpisah dari edit status generik; versi/hash dan perubahan aset tetap terlihat bagi peninjau. Grafik, badge, dan kartu hanya muncul untuk informasi operasional yang benar-benar ada.

## Komponen dan keadaan

| Komponen | State serta keputusan desain |
| --- | --- |
| Tombol utama | Satu tindakan utama per kelompok; default, hover/focus, pending, disabled. Pending menjelaskan proses dan mencegah kiriman ganda |
| Pilihan jawaban | Default/selected memakai batas, label atau penanda pilihan; tidak hanya warna. Tidak membocorkan pilihan benar sebelum penilaian |
| Pemutar instruksi | Label Dengar lagi selalu tersedia, tombol mulai eksplisit; kegagalan audio memberi jalan melanjutkan ketuk/teks pendamping |
| Umpan balik | Benar: konfirmasi konsep. Salah: ajakan mencoba bantuan. Tanpa alarm, penilaian bakat, atau animasi menghukum |
| Field/form | Label terlihat, petunjuk bila perlu, galat dekat field. Fokus dipindahkan ke galat relevan setelah submit gagal |
| PIN/konfirmasi | Dialog berjudul jelas; fokus terkelola, Escape/batal aman, pemicu mendapat fokus kembali; data dewasa tidak tampil di balik gate |
| Bukti laporan | Aktivitas, waktu, bantuan, dan alasan klaim dapat ditelusuri; placeholder tidak disamarkan sebagai data anak |
| Notifikasi | Galat yang perlu tindakan tetap terbaca di tempatnya; toast hanya untuk pemberitahuan sekunder, bukan satu-satunya jalur pemulihan |

| Keadaan layar | Bahasa dan pemulihan |
| --- | --- |
| Belum ada data | Jelaskan langkah pertama yang tersedia, seperti menyiapkan belajar; jangan membuat grafik contoh seolah hasil anak |
| Sedang memuat | Sebut yang dimuat; indikator status diumumkan secara aksesibel dan tidak menutupi konteks anak |
| Koneksi gagal | Pertahankan attempt/posisi; kirim ulang operasi yang sama sesuai aturan idempotensi |
| Data terbatas | Tampilkan bukti yang ada dan kebutuhan belajar berikutnya, tanpa klaim penguasaan |
| Sesi dipindahkan | Pesan netral, hentikan kiriman, kembali ke konteks yang sah; tidak mengambil alih otomatis |
| Akses belajar habis | Informasi dan tindakan paket di area dewasa; mode anak menerima pesan netral sesuai UX |
| Media/PDF sedang diproses | Status nyata dari server, pembatalan/pemulihan sesuai izin dan penghapusan; tidak membuat angka progres palsu |

## Responsif dan aksesibilitas

Layar sempit, menengah, dan lebar masing-masing mempunyai susunan yang sengaja dirancang. Breakpoint ditentukan ketika konten tidak lagi muat dengan ukuran baca/sentuh di atas. Pengujian meliputi kisaran 320, 390, 768, 1024, dan 1440 CSS px serta lebar di antaranya; angka ini sampel pengujian, bukan batas perangkat wajib.

Padding halaman mulai sekitar 16px pada layar sempit, bertambah saat ruang tersedia. Ringkasan dewasa dibatasi sekitar 1120px dan panggung aktivitas sekitar 960px agar mata tidak mengikuti konten terlalu jauh. Sidebar dewasa berubah menjadi navigasi yang dapat dibuka/tutup di layar sempit; admin table memakai kolom prioritas atau container scroll bernama. Halaman tidak menutupi overflow untuk menyembunyikan kontrol yang terpotong.

Gunakan elemen HTML semantik dan perilaku keyboard native. Tombol dengan Enter/Space, tautan dengan Enter, pilihan dengan perilaku grup yang sesuai, dialog dengan Escape serta pengelolaan fokus. Focus ring awal 3px memakai primary dengan offset 2px; pada tombol primary gunakan lapisan offset terang agar ring tetap terlihat. Ikon yang bermakna memiliki label; ikon dekoratif tidak ikut dibaca. Status penting tidak bergantung pada warna/audio saja. Instruksi audio tidak autoplay sebagai syarat melanjutkan.

Motion memakai transisi pendek sekitar 120–180ms untuk state, tidak untuk menunda jawaban atau menyampaikan informasi wajib. Reduced motion menonaktifkan gerak nonesensial. Pengujian keyboard ponsel, zoom 200%, safe area, dan layar pendek memastikan input/tindakan tetap dapat dicapai.

## Bahasa produk

Anak menerima kalimat pendek, konkret, dan mengajak melakukan kegiatan. Pendamping menerima observasi serta langkah yang bisa dilakukan. Admin menerima nama aksi dan konsekuensinya.

| Konteks | Contoh bahasa |
| --- | --- |
| Misi anak | Ayo bantu Hopi menyiapkan bekal. |
| Tindakan anak | Mulai misi; Dengar lagi; Bantu aku; Ini jawabanku; Berhenti |
| Jawaban benar | Pas! Ada tiga apel. |
| Perlu bantuan | Ayo kita hitung bersama. |
| Persiapan dewasa | Siapkan belajar Alya |
| Bukti terbatas | Bukti belajar masih sedikit. Coba satu misi lagi pada hari lain. |
| Pemindahan | Belajar sedang terbuka di perangkat lain. Pindahkan ke perangkat ini? |

Contoh nama dan jawaban adalah fixture yang sudah dipakai pada UX, bukan data pelanggan atau testimoni. Harga, jumlah pengguna, manfaat terukur, rating, dan klaim penguasaan hanya ditampilkan dari sumber yang sah. Paket 1/3/12 bulan mengikuti produk; tampilannya membandingkan fitur yang benar-benar berbeda dan tidak memberi label populer tanpa data.

## Penerapan dan verifikasi

Token dipetakan ke CSS variables/theme Tailwind dan primitive Base UI pada S01/S04. Komponen pendamping/admin yang diadaptasi memakai token tersebut; renderer anak memakai komponen domain pada UX. S04 menerapkan beranda anak, dua templat, ringkasan pendamping serta state-nya. Pengasuh/media/PDF dan billing mengikuti milestone masing-masing; fitur yang belum ada tidak diberi tautan palsu.

| Gate anti-slop | Hasil untuk dokumen ini | Bukti dan batas |
| --- | --- | --- |
| Hard Gate | PASS | Arah dari referensi lokal; tanpa aset/testimoni/metrik baru; pasangan token teks telah dihitung |
| Purpose-Gate | PASS | Tujuan warna, tipografi, ruang, karakter, radius, dan motion dicatat pada tiap bagian |
| Liveliness | PASS untuk spesifikasi | Dials per area, satu fokus per layar, motif perjalanan Hopi dan penggunaan aksen dinyatakan; tampilan aktual menunggu prototipe |
| Craftsmanship | PASS untuk dokumen | Aturan komponen, state, responsif, serta pemulihan terhubung ke UX/teknis; build/klik/keyboard aplikasi belum dijalankan |

Tabel ini menilai artefak skill/dokumen yang diminta. Gate UI runtime R-35 belum dinilai: saat prototipe tersedia, simpan bukti build, console, klik setiap kontrol, keyboard, ukuran layar, semua state, serta kontras state aktual. Rasio token tidak membuktikan seluruh halaman aksesibel. Cara memeriksa dokumen dan skill terpusat di [README](../README.md#pemeriksaan-dokumentasi).
