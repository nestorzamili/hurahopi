# Hurahopi — Activity Diagram: Pengalaman Anak dan Pembelajaran

Tanggal: 25 Oktober 2026. Status: spesifikasi alur untuk implementasi; belum merupakan fitur yang telah berjalan.

[Indeks diagram dan pemetaan fitur](../../README.md#activity-diagram). Aturan produk/parameter yang masih terbuka mengikuti [keputusan](../decisions.md).

## L01 — Tes penempatan dan hasil sementara

Pelaku: Anak, pendamping, sistem. Cakupan: FR-04; S05.

Menggunakan parameter uji placement-v0.1. Usia hanya titik awal. Tiga respons valid diperlukan untuk menyimpulkan satu probe; skip/teknis tidak dianggap salah.

```mermaid
flowchart TD
    S(["Mulai: sesi penempatan disiapkan lewat A04"]) --> R["Sistem: lanjutkan run terbuka atau titik acuan awal"]
    R --> P["Sajikan probe objektif dengan audio dan pilihan ketuk"]
    P --> A{"Respons atau berhenti?"}
    A -->|Jawaban| V["Server: nilai dan simpan satu bukti pada instans"]
    A -->|Lewat atau teknis gagal| X["Catat konteks; bukan salah"]
    A -->|Berhenti| PART["Simpan posisi run; hasil sementara/data terbatas"]
    V --> N{"Sudah tiga respons valid pada tingkat ini?"}
    X --> LIMIT{"Batas penyajian/bagian tercapai?"}
    N -->|Belum| LIMIT
    N -->|Sudah| D["Aturan: dua/tiga mandiri probe naik; nol/satu ke prasyarat"]
    D --> END{"Batas tingkat atau tingkat berikutnya sudah diuji?"}
    END -->|Tidak| LIMIT
    END -->|Ya| RES["Simpan tingkat awal berdasarkan bukti; bab belum diuji tetap terbatas"]
    LIMIT -->|Belum| P
    LIMIT -->|Ya| PART
    PART --> PR["Pendamping: lanjutkan bagian lain atau latihan dengan hasil sementara"]
    PR --> Z(["Selesai bagian ini"])
    RES --> Z
```

## L02 — Beranda anak, misi, dan pengantar bab

Pelaku: Pendamping, anak, sistem. Cakupan: FR-04/05/06; S04/S05.

Beranda Hopi ditampilkan setelah sesi disiapkan. Misi latihan menggunakan rencana sesi yang dibekukan. Penempatan memakai versi aturan/batas yang dibekukan dan memilih probe berikutnya berdasarkan hasil, dengan instans yang sudah disajikan tetap tersimpan. Pengantar interaktif/video opsional mengikuti D12. Tidak ada penawaran paket di layar anak.

```mermaid
flowchart TD
    S(["Mulai: sesi sah telah disiapkan lewat A04"]) --> H["Tampilkan beranda Hopi: avatar, misi, peta ringkas dan koleksi"]
    H --> B["Anak: ketuk Mulai atau Lanjutkan"]
    B --> P{"Purpose sesi penempatan?"}
    P -->|Ya| PLACE["Jalankan L01"]
    P -->|Tidak| C["Sistem: ambil instans saat ini dari rencana tersimpan"]
    C --> NEW{"Pertama memasuki bab dan pengantar tersedia?"}
    NEW -->|Ya| I["Putar pengantar audio/interaktif atau video; boleh dilewati"]
    NEW -->|Tidak| A["Tampilkan satu aktivitas"]
    I --> A
    A --> DONE{"Sisa slot sesi?"}
    DONE -->|Ada| C
    DONE -->|Tidak| REWARD["Tutup sesi dan tampilkan apresiasi L08"]
    PLACE --> Z(["Selesai sesi/bagian"])
    REWARD --> Z
```

## L03 — Pilihan visual dan soal cerita objektif

Pelaku: Anak, sistem. Cakupan: FR-06; S04/S06.

Juga pola dasar untuk memilih jumlah/bentuk, soal cerita audio, dan pilihan pada garis bilangan. Jangan mengirim kunci ke browser; memutar ulang instruksi bukan hint.

```mermaid
flowchart TD
    S(["Mulai: instans saat ini"]) --> P["Render objek/pilihan dan instruksi audio yang dapat diulang"]
    P --> A{"Tindakan anak"}
    A -->|Pilih dan konfirmasi jawaban| SEND["Kirim option ID dengan urutan respons dan key operasi"]
    A -->|Petunjuk| H["Server catat hint; tampilkan bantuan"]
    H --> P
    A -->|Lewat| SK["Terminalisasi skipped; bukan salah"]
    SEND --> AUTH{"Lease dan instans masih sah?"}
    AUTH -->|Tidak| STOP["Hentikan kiriman; jalankan recovery/perangkat sesuai sebab"]
    AUTH -->|Ya| G["Server: nilai respons; pertahankan nilai respons pertama"]
    G --> OK{"Jawaban benar?"}
    OK -->|Tidak| TRY["Umpan balik ramah; coba lagi, bantuan atau lewat"]
    TRY --> P
    OK -->|Ya| SAVE["Simpan hasil sekali; evaluasi L07; apresiasi partisipasi"]
    SAVE --> NEXT["Lanjut slot berikutnya atau L08"]
    SK --> NEXT
    NEXT --> Z(["Selesai aktivitas"])
    STOP --> Z
```

## L04 — Membuat kelompok, mencocokkan, dan menyusun

Pelaku: Anak, sistem. Cakupan: FR-06; S04/S06.

Pola input terstruktur untuk build_group, pasangan, urutan, bingkai sepuluh, potongan bentuk, memori pasangan, atau posisi garis bilangan. Kebenaran ditentukan dari representasi logis server, bukan koordinat/screenshot klien.

```mermaid
flowchart TD
    S(["Mulai: render instans dan objek yang tersedia"]) --> I{"Cara berinteraksi"}
    I -->|Ketuk atau keyboard| T["Pilih objek lalu tujuan; bisa dibatalkan"]
    I -->|Drag bila tersedia| D["Pindahkan objek dengan dnd-kit"]
    T --> STATE["Perbarui susunan lokal dan kontrol yang aksesibel"]
    D --> STATE
    STATE --> SUB{"Anak menekan Sudah?"}
    SUB -->|Belum| I
    SUB -->|Ya| SND["Kirim ID objek/pasangan/urutan, bukan skor"]
    SND --> V["Server: verifikasi lease, instans, ID unik dan parameter tersimpan"]
    V --> OK{"Memenuhi kunci/rubrik objektif?"}
    OK -->|Tidak| FB["Catat respons pertama; tawarkan bantuan/coba lagi/lewat"]
    FB --> I
    OK -->|Ya| SAVE["Simpan hasil idempoten; evaluasi L07; lanjut aktivitas"]
    SAVE --> Z(["Selesai"])
```

## L05 — Jawaban suara dan alternatif ketuk

Pelaku: Anak, sistem, worker/penyedia AI. Cakupan: FR-06/09; M3.

Fitur data anak memerlukan D2/D5, persetujuan, dan kesiapan penyedia. Penilaian isi jawaban tetap server; pelafalan tidak dinilai. Analisis tertunda tidak mengganti instans yang sedang tampil.

```mermaid
flowchart TD
    S(["Mulai: aktivitas mendukung suara"]) --> G{"Izin, persetujuan dan prasyarat AI/media siap?"}
    G -->|Tidak| TAP["Tawarkan jawaban ketuk dari L03/L04"]
    G -->|Ya| MIC{"Izin mikrofon diberikan?"}
    MIC -->|Tidak| TAP
    MIC -->|Ya| REC["Anak: rekam singkat, dengar ulang, kirim atau ulangi"]
    REC --> UP{"Unggahan valid dan berhasil?"}
    UP -->|Tidak| FAIL["Catat kegagalan teknis; retry atau ketuk"]
    FAIL --> TAP
    UP -->|Ya| Q["Simpan referensi media dan bukti tertunda; enqueue worker"]
    Q --> TR["Worker: cek ulang izin lalu transkripsikan"]
    TR --> CONF{"Isi dapat ditafsirkan dengan yakin?"}
    CONF -->|Tidak| C["Minta konfirmasi ketuk saat konteks masih sesuai; jika belum, bukti tetap terbatas"]
    CONF -->|Ya| GRADE["Server normalisasi isi, bandingkan kunci, simpan hasil logis sekali"]
    GRADE --> U["Perbarui bukti/laporan pada perhitungan berikutnya"]
    C --> Z(["Selesai pemrosesan ini"])
    U --> Z
    TAP --> Z
```

## L06 — Menggambar, mewarnai, dan karya terbuka

Pelaku: Anak, sistem, worker opsional. Cakupan: FR-06/09; M3.

Kanvas terstruktur dapat diperiksa deterministik. Portofolio dan pengamatan AI mempunyai izin/prasyarat terpisah; karya terbuka tidak mendapat nilai benar-salah.

```mermaid
flowchart TD
    S(["Mulai: render instruksi dan kanvas Konva"]) --> DRAW["Anak: gambar/warnai/susun; undo dan selesai tersedia"]
    DRAW --> TYPE{"Jenis tugas"}
    TYPE -->|Terstruktur| DET["Server periksa objek/bagian terpilih terhadap rubrik"]
    TYPE -->|Karya terbuka| OPEN["Catat kegiatan dan proses; tanpa skor benar-salah"]
    DET --> E["Simpan hasil dan bantuan sesuai rubrik"]
    OPEN --> E
    E --> PORT{"Persetujuan portofolio/media aktif?"}
    PORT -->|Tidak| NO["Simpan hanya bukti belajar yang diizinkan; jangan unggah karya portofolio"]
    PORT -->|Ya| UP["Unggah terlindungi; simpan referensi karya"]
    UP --> AI{"Pengamatan AI diizinkan dan penyedia siap?"}
    AI -->|Tidak| NEXT["Lanjutkan aktivitas; laporan memakai bukti tersedia"]
    AI -->|Ya| Q["Enqueue pengamatan; worker periksa ulang izin"]
    Q --> DESC["Simpan pengamatan berbukti atau data terbatas; tidak menyimpulkan bakat"]
    NO --> NEXT
    DESC --> NEXT
    NEXT --> Z(["Selesai"])
```

## L07 — Evaluasi penguasaan dan titik latihan berikutnya

Pelaku: Sistem, pendidik melalui parameter. Cakupan: FR-07; S07.

mastery-v0.1 adalah parameter uji. Ambang ringkasan laporan tiga aktivitas/dua sesi berbeda dari syarat naik tingkat 8/10. Pencapaian lama tidak dihapus saat perlu ulasan.

```mermaid
flowchart TD
    S(["Mulai: bukti objektif valid selesai/analisis tiba"]) --> D["Deduplikasi bukti dan pilih jendela keterampilan/tingkat/versi yang sama"]
    D --> Q{"Sepuluh bukti, minimal dua sesi dan dua templat cukup?"}
    Q -->|Ya| M{"Minimal 8 benar pertama tanpa petunjuk/bantuan?"}
    M -->|Ya| C{"Prasyarat dan konten tingkat berikutnya tersedia?"}
    C -->|Ya| UP["Jadwalkan tingkat berikutnya; catat alasan dan bukti"]
    C -->|Tidak| HOLD["Pertahankan latihan tersedia dan catat kesiapan"]
    Q -->|Tidak| LOW{"Pola kesulitan memenuhi parameter ulasan/penurunan?"}
    M -->|Tidak| LOW
    LOW -->|Ya| DOWN["Jadwalkan prasyarat/dukungan; pertahankan riwayat pencapaian"]
    LOW -->|Tidak| HOLD
    UP --> R["Perbarui laporan berbukti; perubahan berlaku pada sesi berikutnya"]
    DOWN --> R
    HOLD --> R
    R --> Z(["Selesai evaluasi"])
```

## L08 — Apresiasi, koleksi, dan akhir sesi

Pelaku: Anak, sistem, pendamping. Cakupan: FR-06/07; S06/S07.

Koleksi diberikan sekali per sumber. Personalisasi memakai item yang dimiliki dengan pemeriksaan konteks anak pada milestone fiturnya. Tidak ada loop sesi tambahan otomatis atau pesan membeli kepada anak.

```mermaid
flowchart TD
    S(["Mulai: slot sesi selesai atau anak berhenti"]) --> FULL{"Rencana selesai?"}
    FULL -->|Tidak| PAUSE["Simpan progres; pause/akhiri sesuai pilihan; tidak mengklaim tuntas"]
    FULL -->|Ya| SAVE["Tutup sesi; simpan apresiasi/hadiah dengan sumber unik"]
    SAVE --> DUP{"Hadiah sumber ini sudah pernah diberikan?"}
    DUP -->|Ya| SHOW["Tampilkan hasil yang sama tanpa hadiah tambahan"]
    DUP -->|Tidak| ADD["Tambahkan koleksi yang diperoleh"]
    ADD --> SHOW
    SHOW --> HOME["Beranda anak: misi selesai, koleksi dan peta ringkas"]
    HOME --> MORE{"Pendamping ingin putaran tambahan?"}
    MORE -->|Tidak| Z(["Selesai untuk sekarang"])
    MORE -->|Ya| AD["Kembali lewat PIN dan siapkan sesi baru lewat A04"]
    AD --> Z
    PAUSE --> Z
```

## L09 — Pause, muat ulang, dan koneksi terputus

Pelaku: Anak/pendamping, browser, sistem. Cakupan: FR-05/06; S05/S06.

Retry memakai instans, urutan respons, dan idempotency key yang sama. Lease transfer/logout/izin dicabut menolak replay; penutupan normal boleh memulihkan hasil tersimpan sesuai spesifikasi teknis.

```mermaid
flowchart TD
    S(["Mulai: pause, reload atau koneksi terputus"]) --> E{"Kejadian"}
    E -->|Pause| P["Server simpan posisi, tandai paused dan lepas lease"]
    E -->|Reload atau sambung kembali| C["Ambil status sesi tersimpan dan periksa konteks/lease"]
    C --> L{"Lease dipindah atau izin dicabut?"}
    L -->|Ya| STOP["Hentikan tulisan; tampilkan pesan sesuai sebab"]
    L -->|Tidak| DONE{"Operasi terakhir sudah tersimpan?"}
    DONE -->|Ya| R["Pulihkan hasil/instans yang sama; tanpa progres atau hadiah baru"]
    DONE -->|Belum| RET["Kirim ulang operasi logis yang sama bila masih berizin"]
    RET --> R
    P --> RES["Pendamping: lanjutkan melalui A04 dengan pemeriksaan akses ulang"]
    R --> Z(["Selesai pemulihan"])
    RES --> Z
    STOP --> Z
```
