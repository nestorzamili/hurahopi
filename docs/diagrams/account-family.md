# Hurahopi — Activity Diagram: Akun, Keluarga, dan Akses

Tanggal: 25 Oktober 2026. Status: spesifikasi alur untuk implementasi; belum merupakan fitur yang telah berjalan.

[Indeks diagram dan pemetaan fitur](../../README.md#activity-diagram). Aturan produk/parameter yang masih terbuka mengikuti [keputusan](../decisions.md).

## A01 — Masuk dengan Google atau tautan email

Pelaku: Orang dewasa, sistem, Google/Resend. Cakupan: FR-01; S03.

Login perangkat baru tidak mencabut login dewasa pada perangkat lain. Tautan email dikonsumsi melalui POST setelah konfirmasi, bukan otomatis saat pemindai membuka tautan.

```mermaid
flowchart TD
    S(["Mulai: orang dewasa membuka login"]) --> M{"Pilih metode"}
    M -->|Google| G["Sistem: buat state/nonce dan arahkan ke Google"]
    G --> GV{"Callback dan identitas valid?"}
    M -->|Email| E["Sistem: terima email, rate limit, kirim tautan melalui Resend"]
    E --> EC["Pengguna: buka tautan dan konfirmasi masuk"]
    EC --> EV{"Token belum dipakai dan belum kedaluwarsa?"}
    GV -->|Tidak| X["Tampilkan galat dan pilihan coba kembali"]
    EV -->|Tidak| X
    GV -->|Ya| I["Sistem: temukan/buat identitas dan sesi dewasa"]
    EV -->|Ya| I
    I --> P{"Setup keluarga sudah lengkap?"}
    P -->|Belum| U["Arahkan ke profil dan persetujuan A02/A03"]
    P -->|Sudah| H["Buka beranda orang tua atau area peran yang diizinkan"]
    X --> Z(["Selesai"])
    U --> Z
    H --> Z
```

## A02 — Membuat dan mengubah profil anak

Pelaku: Pemilik akun, sistem. Cakupan: FR-02; S03.

Nama panggilan, usia acuan, dan avatar cukup. Profil pengasuh hanya yang diizinkan; pengasuh tidak dapat mengubah profil. Penghapusan memakai A07.

```mermaid
flowchart TD
    S(["Mulai: pilih Kelola anak"]) --> C["Sistem: periksa pemilik akun dan unlock dewasa"]
    C --> Q{"Diizinkan?"}
    Q -->|Tidak| R["Tolak operasi"]
    Q -->|Ya| F["Orang tua: isi/ubah nama panggilan, usia, avatar"]
    F --> V{"Data valid?"}
    V -->|Tidak| F
    V -->|Ya| W["Sistem: simpan profil dalam keluarga dan catat perubahan"]
    W --> N{"Profil baru?"}
    N -->|Ya| P["Lanjut ke persetujuan A03"]
    N -->|Tidak| H["Perbarui kartu anak dan ringkasan yang relevan"]
    R --> Z(["Selesai"])
    P --> Z
    H --> Z
```

## A03 — Memberikan atau menarik persetujuan

Pelaku: Pemilik akun, sistem. Cakupan: FR-02/09/14; S03/M3.

Persetujuan AI, pemrosesan media, dan portofolio dipisahkan. Pilihan menolak media/AI menyediakan jalur belajar ketuk yang sesuai.

```mermaid
flowchart TD
    S(["Mulai: buka Persetujuan dan data"]) --> C["Sistem: verifikasi pemilik, profil dan PIN"]
    C --> F["Orang tua: baca tujuan/versi kebijakan dan pilih per fitur"]
    F --> D{"Berikan atau tarik persetujuan?"}
    D -->|Berikan| G["Catat event persetujuan sesuai versi kebijakan"]
    G --> P{"Prasyarat penyedia/retensi fitur terpenuhi?"}
    P -->|Ya| EN["Aktifkan kemampuan fitur yang disetujui"]
    P -->|Tidak| WAIT["Simpan pilihan; fitur tetap nonaktif sampai siap"]
    D -->|Tarik| RV["Catat pencabutan dan blokir pemrosesan baru"]
    RV --> J["Batalkan/cegah job terkait dan jalankan siklus data menurut D2"]
    J --> ALT["Tawarkan ketuk atau aktivitas deterministik"]
    EN --> Z(["Selesai"])
    WAIT --> Z
    ALT --> Z
```

## A04 — Menyiapkan mode anak dan kembali ke orang tua

Pelaku: Pendamping, anak, sistem. Cakupan: FR-03/05; S03/S05.

Pendamping menyiapkan sesi sebelum menyerahkan perangkat. Tombol Mulai pada beranda anak membuka aktivitas sesi yang sudah disiapkan; tidak membuat sesi backend baru tanpa otorisasi dewasa.

```mermaid
flowchart TD
    S(["Mulai: beranda orang tua"]) --> P["Pendamping: pilih anak dan tekan Mulai belajar"]
    P --> PIN{"Unlock dewasa/PIN valid?"}
    PIN -->|Tidak| G["Minta PIN dan batasi percobaan"]
    G --> PIN
    PIN -->|Ya| C["Sistem: periksa izin, persetujuan inti dan hak belajar"]
    C --> AC{"Boleh memulai atau melanjutkan?"}
    AC -->|Tidak| AD["Jelaskan kebutuhan di area dewasa"]
    AC -->|Ya| LS["Siapkan/lanjut sesi; tangani konflik perangkat lewat A06"]
    LS --> READY{"Lease berhasil diperoleh?"}
    READY -->|Tidak| AD
    READY -->|Ya| K["Masuk mode anak; bersihkan cache dewasa; tampilkan beranda Hopi"]
    K --> PLAY["Anak: Mulai misi atau lanjut aktivitas"]
    PLAY --> BACK["Pendamping: pilih Area orang tua"]
    BACK --> PV{"PIN valid?"}
    PV -->|Tidak| K
    PV -->|Ya| H["Sistem: buka konteks dewasa sesuai izin dan bersihkan cache anak"]
    H --> Z(["Selesai"])
    AD --> Z
```

## A05 — Undangan, izin, dan pencabutan pengasuh

Pelaku: Pemilik akun, pengasuh, sistem/Resend. Cakupan: FR-10; M3.

Maksimal dua slot termasuk undangan menunggu. Izin awal belajar/laporan; portofolio/PDF terpisah. Ketersediaan selama trial mengikuti finalisasi D3.

```mermaid
flowchart TD
    S(["Mulai: buka Pengasuh"]) --> M{"Tindakan orang tua"}
    M -->|Undang| Q{"Slot aktif dan menunggu kurang dari dua?"}
    Q -->|Tidak| FULL["Tampilkan slot penuh; kelola undangan lama"]
    Q -->|Ya| F["Pilih email, profil anak dan izin; konfirmasi PIN"]
    F --> I["Sistem: simpan undangan sekali pakai dan kirim email"]
    I --> LG["Pengasuh: masuk dengan identitas sendiri dan terima"]
    LG --> V{"Identitas, expiry dan kuota valid?"}
    V -->|Tidak| DENY["Tolak; jangan memberi akses"]
    V -->|Ya| A["Aktifkan membership dan izin per anak"]
    M -->|Ubah izin atau cabut| U["Sistem: verifikasi pemilik/PIN; simpan perubahan atomik"]
    U --> R["Cabut lease terkait; request/media/PDF berikutnya memeriksa izin baru"]
    A --> Z(["Selesai"])
    R --> Z
    FULL --> Z
    DENY --> Z
```

## A06 — Memindahkan sesi belajar ke perangkat lain

Pelaku: Pendamping, perangkat lama/baru, sistem. Cakupan: FR-05/06; S05/S06.

Pemindahan hanya setelah konfirmasi dan verifikasi dewasa; ID sesi/instans tetap. Login laporan orang tua tidak dicabut. Jawaban dan transfer diserialisasi di server. Semua tab satu auth_session berbagi konteks: penerima yang masih belajar pada anak lain harus pause eksplisit sebelum transfer; maksimal satu lease aktif per browser. Urutan lock: auth_session penerima → anak → sesi/lease, tanpa mengambil lock auth_session pemilik lama setelah child.

```mermaid
flowchart TD
    S(["Mulai: pendamping meminta belajar di perangkat baru"]) --> C["Sistem: cek izin, unlock, hak belajar dan lease anak"]
    C --> O{"Ada lease pada browser lain?"}
    O -->|Tidak| L["Terbitkan lease untuk sesi yang sah"]
    O -->|Ya| F["Tampilkan sesi sedang dipakai dan konfirmasi pemindahan"]
    F --> Y{"Pendamping setuju?"}
    Y -->|Tidak| KEEP["Pertahankan perangkat lama"]
    Y -->|Ya| V["Server: kunci auth penerima lalu anak dan lease; cek konteks, izin dan generasi"]
    V --> MATCH{"Generasi masih sama?"}
    MATCH -->|Tidak| F
    MATCH -->|Ya| ROT["Cabut lease lama; terbitkan lease baru; catat audit"]
    ROT --> L
    L --> CONT["Lanjutkan ID sesi, instans, dan progres yang sama"]
    ROT --> OLD["Perangkat lama: request berikutnya ditolak; tampilkan pesan netral"]
    CONT --> Z(["Selesai"])
    KEEP --> Z
    OLD --> Z
```

## A07 — Menghapus profil atau akun dan data turunannya

Pelaku: Pemilik akun, sistem, worker/penyedia media. Cakupan: FR-14; M4.

Retensi pengecualian transaksi/cadangan mengikuti D2/D7. Penghapusan karya tunggal ada pada P03. Restore tidak boleh menghidupkan data yang telah dihapus.

```mermaid
flowchart TD
    S(["Mulai: orang tua meminta hapus profil/akun"]) --> C["Verifikasi identitas/PIN, cakupan dan konsekuensi akses berbayar"]
    C --> Q{"Konfirmasi eksplisit?"}
    Q -->|Tidak| Z(["Selesai"])
    Q -->|Ya| T["Sistem: tandai penghapusan; tutup akses dan lease terkait"]
    T --> J["Batalkan/cegah job AI, laporan dan ekspor terkait"]
    J --> W["Worker: hapus media/turunan, invalidasi CDN dan hapus bukti sesuai D2"]
    W --> OK{"Seluruh langkah yang diwajibkan berhasil?"}
    OK -->|Tidak| RETRY["Catat langkah gagal; retry aman dan pantau"]
    RETRY --> W
    OK -->|Ya| AUD["Simpan catatan penghapusan minimum dan aturan pengecualian retensi"]
    AUD --> OUT["Sampaikan hasil; gunakan catatan ini saat restore O06"]
    OUT --> Z
```
