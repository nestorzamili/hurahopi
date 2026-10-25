# Hurahopi — Activity Diagram: Orang Tua, Laporan, dan Pembayaran

Tanggal: 25 Oktober 2026. Status: spesifikasi alur untuk implementasi; belum merupakan fitur yang telah berjalan.

[Indeks diagram dan pemetaan fitur](../../README.md#activity-diagram). Aturan produk/parameter yang masih terbuka mengikuti [keputusan](../decisions.md).

## P01 — Beranda orang tua, laporan, dan bukti

Pelaku: Orang tua atau pengasuh berizin, sistem. Cakupan: FR-07/10/12; S07.

Ringkasan berfokus pada anak yang dipilih. Laporan tetap dapat dibaca setelah akses belajar berakhir. Pengasuh tidak mendapat media privat tanpa izin portofolio.

```mermaid
flowchart TD
    S(["Mulai: pilih anak/periode di area dewasa"]) --> C["Server periksa membership, PIN/mode dan izin laporan"]
    C --> Q{"Diizinkan?"}
    Q -->|Tidak| DENY["Tolak tanpa membocorkan data"]
    Q -->|Ya| E["Ambil progres, percobaan dan status analisis"]
    E --> EN{"Bukti ringkasan sudah cukup?"}
    EN -->|Belum| LIMIT["Tampilkan kegiatan teramati dan data terbatas"]
    EN -->|Sudah| CLAIM["Tampilkan tingkat/perubahan dengan alasan dan referensi bukti"]
    LIMIT --> H["Beranda: saran pendampingan, karya berizin, status misi"]
    CLAIM --> H
    H --> CLICK{"Pengguna membuka detail?"}
    CLICK -->|Bukti| EV["Periksa ulang izin; tampilkan aktivitas dan bantuan yang relevan"]
    CLICK -->|Tes ulang atau titik mulai| RET["Pemilik verifikasi PIN/alasan; jadwalkan perubahan tanpa bukti mastery palsu"]
    CLICK -->|Tidak| Z(["Selesai"])
    EV --> Z
    RET --> Z
    DENY --> Z
```

## P02 — Ekspor laporan PDF

Pelaku: Orang tua/pengasuh berizin, sistem, worker/Gotenberg. Cakupan: FR-11; M3.

Orang tua boleh memilih gambar; PDF pengasuh tanpa salinan media anak. Worker merender tanpa menahan transaksi. Publikasi hasil mengunci child lalu snapshot/ekspor dan memeriksa data_version, status, serta izin; penghapusan memakai lock child yang sama. File sementara dan izin unduh diperiksa ulang, termasuk setelah pencabutan atau penghapusan.

```mermaid
flowchart TD
    S(["Mulai: pilih Ekspor PDF, anak dan periode"]) --> C["Server periksa izin laporan dan ekspor"]
    C --> OK{"Boleh mengekspor?"}
    OK -->|Tidak| DENY["Tolak ekspor"]
    OK -->|Ya| ROLE{"Pemilik akun?"}
    ROLE -->|Ya| OPT["Pilih menyertakan gambar yang diizinkan atau tanpa gambar"]
    ROLE -->|Tidak| TEXT["Paksa laporan tanpa salinan media anak"]
    OPT --> Q["Buat job River dengan referensi laporan"]
    TEXT --> Q
    Q --> W["Worker cek ulang izin/penghapusan; siapkan HTML dan aset"]
    W --> PDF["Gotenberg internal menghasilkan PDF"]
    PDF --> VALID{"Di bawah lock: versi data, izin dan status masih sah; hasil berhasil?"}
    VALID -->|Tidak| FAIL["Tandai gagal/dibatalkan; jangan sajikan file"]
    VALID -->|Ya| STORE["Publikasikan output sementara secara atomik; cancelled tidak dapat kembali siap"]
    STORE --> DL["Saat unduh: periksa sesi, izin, snapshot valid, status siap dan expiry lagi"]
    DL --> Z(["Selesai; hapus file sesuai D2"])
    FAIL --> Z
    DENY --> Z
```

## P03 — Melihat, mengunduh, atau menghapus portofolio

Pelaku: Orang tua/pengasuh berizin, sistem/Cloudinary. Cakupan: FR-09/10/14; M3/M4.

Pengasuh hanya melihat/memutar bila diizinkan; tidak mengunduh/menghapus media. Media, turunan, dan PDF yang terdampak mengikuti penghapusan D2.

```mermaid
flowchart TD
    S(["Mulai: buka karya anak"]) --> C["Server periksa kepemilikan/izin portofolio"]
    C --> Q{"Akses diberikan?"}
    Q -->|Tidak| DENY["Tolak akses"]
    Q -->|Ya| VIEW["Proxy media terlindungi; dukung audio range tanpa cache publik"]
    VIEW --> ACT{"Tindakan pengguna"}
    ACT -->|Lihat atau putar| Z(["Selesai"])
    ACT -->|Unduh| OWN{"Pemilik akun?"}
    ACT -->|Hapus| DEL{"Pemilik dan PIN/konfirmasi valid?"}
    OWN -->|Tidak| DENY
    OWN -->|Ya| DL["Kirim file melalui endpoint berizin"]
    DEL -->|Tidak| DENY
    DEL -->|Ya| MARK["Tutup akses karya; batalkan ekspor/job terkait"]
    MARK --> W["Hapus original/turunan; invalidasi CDN; retry bila gagal"]
    W --> R["Hapus bukti turunan yang relevan dan bangun ulang ringkasan"]
    DL --> Z
    R --> Z
    DENY --> Z
```

## P04 — Uji coba tujuh hari dan masa akses habis

Pelaku: Orang tua, sistem. Cakupan: FR-12; M4.

Tujuh hari tanpa kartu ditetapkan; penerima, titik mulai, kuota, dan perpanjangan masih mengikuti D3. Diagram tidak menetapkan angka kebijakan yang belum disetujui.

```mermaid
flowchart TD
    S(["Mulai: keluarga memenuhi alur trial yang ditetapkan D3"]) --> C["Server periksa ledger penerima dan kebijakan trial"]
    C --> E{"Layak mendapat trial?"}
    E -->|Tidak| PK["Area orang tua: tampilkan paket dan akses yang tersedia"]
    E -->|Ya| A["Aktifkan trial per anak/kursus sekali; catat waktu mulai/akhir"]
    A --> LEARN["Belajar dan laporan tersedia sesuai fitur/kuota trial"]
    LEARN --> TIME{"Masa trial/akses berakhir?"}
    TIME -->|Belum| LEARN
    TIME -->|Ya| RUN{"Sesi sedang berjalan tanpa pause/idle?"}
    RUN -->|Ya| FIN["Izinkan menyelesaikan rencana sesi yang sudah dibekukan"]
    RUN -->|Tidak| STOP["Blokir sesi baru/resume; laporan dan portofolio tetap sesuai izin"]
    FIN --> STOP
    STOP --> P["Tawarkan berlangganan hanya kepada orang dewasa"]
    PK --> Z(["Selesai"])
    P --> Z
```

## P05 — Membeli dan memperpanjang paket

Pelaku: Pemilik akun, sistem, iPaymu. Cakupan: FR-12/13; M4.

Pesanan tepat satu anak/kursus dan versi harga. Jadwal segmen/perubahan paket mengikuti D10; harga/refund D7. Halaman kembali tidak mengaktifkan akses.

```mermaid
flowchart TD
    S(["Mulai: orang tua memilih Langganan"]) --> C["Server periksa pemilik akun, unlock dan profil penerima"]
    C --> F["Pilih anak dan paket 1/3/12 bulan; tampilkan harga dan fitur"]
    F --> OK{"Konfirmasi penerima dan harga?"}
    OK -->|Tidak| Z(["Selesai"])
    OK -->|Ya| O["Backend membuat pesanan idempoten dengan harga dibekukan"]
    O --> T["Buat transaksi Redirect Payment iPaymu"]
    T --> GOOD{"Transaksi/redirect tersedia?"}
    GOOD -->|Tidak| FAIL["Tampilkan gagal/tertunda; jangan menggandakan order logis"]
    GOOD -->|Ya| PAY["Orang tua menyelesaikan pembayaran pada iPaymu"]
    PAY --> BACK["Kembali ke status pesanan; tunggu verifikasi P06"]
    BACK --> ST{"Status server"}
    ST -->|Tertunda| PEND["Tampilkan instruksi/masa berlaku dan status terbaru"]
    ST -->|Berhasil| ACTIVE["Tampilkan akses anak dan jadwal paket yang telah diverifikasi"]
    ST -->|Batal atau kedaluwarsa| END["Tawarkan mencoba lagi tanpa mengaktifkan akses"]
    FAIL --> Z
    PEND --> Z
    ACTIVE --> Z
    END --> Z
```

## P06 — Callback pembayaran dan rekonsiliasi

Pelaku: iPaymu, API, worker, sistem akses. Cakupan: FR-13; M4.

Nominal/mata uang/referensi/penerima dicocokkan dengan pesanan. Respons berulang atau datang terlambat tidak menambah segmen akses dua kali atau memundurkan status berhasil.

```mermaid
flowchart TD
    S(["Mulai: callback masuk atau rekonsiliasi pesanan jatuh tempo"]) --> SRC{"Sumber"}
    SRC -->|Callback| SIG{"Validasi callback sesuai dokumentasi lulus?"}
    SIG -->|Tidak| REJECT["Tolak dan catat audit aman"]
    SIG -->|Ya| FETCH["Backend periksa status melalui API iPaymu"]
    SRC -->|Worker rekonsiliasi| FETCH
    FETCH --> MATCH{"Identitas/nominal/mata uang cocok dengan pesanan?"}
    MATCH -->|Tidak atau ambigu| REVIEW["Jangan aktivasi; jadwalkan rekonsiliasi/peninjauan"]
    MATCH -->|Ya| ST{"Status penyedia yang diverifikasi"}
    ST -->|Berhasil| TX["Transaksi DB: lock order; cek pembayaran sudah diterapkan"]
    TX --> AP["Aktifkan segmen anak/kursus sekali sesuai D10; catat audit"]
    ST -->|Menunggu| WAIT["Pertahankan pending; rekonsiliasi lagi sesuai batas"]
    ST -->|Batal atau kedaluwarsa| CL["Update yang diizinkan; jangan mundurkan status sukses"]
    AP --> Z(["Selesai"])
    REVIEW --> Z
    WAIT --> Z
    CL --> Z
    REJECT --> Z
```

## P07 — Worksheet printable

Pelaku: Pemilik akun, sistem. Cakupan: FR-12; M4.

Akses penuh paket 3/12 bulan; contoh trial mengikuti D3. File yang telah diunduh tetap dimiliki orang tua, tetapi unduhan baru memerlukan hak yang sesuai.

```mermaid
flowchart TD
    S(["Mulai: orang tua memilih worksheet anak"]) --> C["Server periksa pemilik, profil dan hak fitur pada segmen aktif"]
    C --> Q{"Paket 3/12 bulan atau contoh trial berizin?"}
    Q -->|Tidak| LOCK["Tampilkan fitur paket di area orang tua"]
    Q -->|Ya| L["Sarankan worksheet terbit sesuai bab/tingkat anak"]
    L --> SEL["Orang tua memilih berkas"]
    SEL --> V["Endpoint unduh memeriksa ulang izin dan versi aset"]
    V --> DL["Kirim PDF yang diterbitkan; kunci untuk pendamping tersedia"]
    DL --> Z(["Selesai"])
    LOCK --> Z
```

## P08 — Latihan olimpiade

Pelaku: Orang tua/anak, sistem. Cakupan: FR-06/12; M4/M5.

Paket 12 bulan membuka fitur; cakupan dan prasyarat pedagogis mengikuti D4. Pratinjau contoh tidak membuka keseluruhan bank soal atau menjadi jalur mastery utama tanpa aturan D4.

```mermaid
flowchart TD
    S(["Mulai: orang tua membuka jalur olimpiade anak"]) --> C["Server periksa profil dan hak paket"]
    C --> PK{"Paket 12 bulan aktif?"}
    PK -->|Tidak| PRE["Area dewasa: daftar/pratinjau contoh sesuai kebijakan paket"]
    PK -->|Ya| SK{"Prasyarat materi D4 terpenuhi?"}
    SK -->|Belum| GUIDE["Tampilkan latihan dasar yang disarankan dan daftar isi kepada orang tua"]
    SK -->|Sudah| PREP["Pendamping siapkan sesi sesuai A04; gunakan soal terkurasi"]
    PREP --> CHILD["Anak mengerjakan soal dan mendapat petunjuk/pembahasan"]
    CHILD --> SAVE["Server simpan bukti dan hasil sesuai aturan jalur"]
    PRE --> Z(["Selesai"])
    GUIDE --> Z
    SAVE --> Z
```
