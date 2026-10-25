# Hurahopi — Activity Diagram: Konten, Admin, dan Operasional

Tanggal: 25 Oktober 2026. Status: spesifikasi alur untuk implementasi; belum merupakan fitur yang telah berjalan.

[Indeks diagram dan pemetaan fitur](../../README.md#activity-diagram). Aturan produk/parameter yang masih terbuka mengikuti [keputusan](../decisions.md).

## O01 — Menyiapkan konten, aset, dan materi paket

Pelaku: Editor, sistem. Cakupan: FR-08/12; S09/M5.

Mencakup bab, tingkat, keterampilan, parameter generator, narasi/aset, pengantar/video, worksheet dan soal olimpiade. Konten yang belum tersedia tidak boleh dipilih mesin belajar.

```mermaid
flowchart TD
    S(["Mulai: editor membuka panel konten"]) --> C["Server periksa peran konten"]
    C --> F["Editor pilih bab/tingkat/keterampilan dan buat atau revisi draf"]
    F --> A["Siapkan instruksi, rubrik, kunci, rentang parameter dan aset berizin"]
    A --> V{"Schema, parameter, prasyarat dan aset valid?"}
    V -->|Tidak| ERR["Tampilkan kesalahan terperinci pada editor"]
    ERR --> F
    V -->|Ya| PRE["Preview pengalaman anak dan respons contoh"]
    PRE --> Q{"Siap diajukan?"}
    Q -->|Belum| F
    Q -->|Ya| SUB["Bekukan revisi untuk peninjauan O03"]
    SUB --> Z(["Selesai menyiapkan draf"])
```

## O02 — Pembuatan draf soal AI

Pelaku: Editor, worker/penyedia AI, sistem. Cakupan: FR-08; M3.

Hanya spesifikasi konten dikirim, tanpa data anak. Semua draf termasuk numerik wajib ditinjau pendidik; tidak ada generasi langsung saat anak bermain.

```mermaid
flowchart TD
    S(["Mulai: editor meminta variasi draf"]) --> C["Server cek peran, spesifikasi dan anggaran pemrosesan"]
    C --> Q["Enqueue job River; argumen hanya referensi spesifikasi konten"]
    Q --> AI["Worker memanggil penyedia; terima keluaran terstruktur"]
    AI --> V{"Schema, kunci yang dapat dihitung, kosakata, keamanan dan duplikasi lulus?"}
    V -->|Tidak| REJ["Tolak draf; catat alasan dan biaya untuk editor"]
    V -->|Ya| D["Simpan draf dan provenance model; bukan konten terbit"]
    D --> HUMAN["Masukkan antrean pendidik melalui O03"]
    REJ --> Z(["Selesai pemrosesan"])
    HUMAN --> Z
```

## O03 — Peninjauan, publikasi versi, dan pengarsipan

Pelaku: Pendidik/peninjau, editor, sistem. Cakupan: FR-08; layanan/peran/review minimum S04, UI lengkap S09.

Review terikat revisi dan hash seluruh isi efektif, termasuk binding/checksum aset. Perubahan draf membatalkan persetujuan; versi/binding/aset terbit tidak ditimpa. Layanan S04 mencatat review sebelum uji keluarga S08 meskipun UI S09 belum tersedia. Arsip mencegah pemilihan baru tetapi versi yang sudah dirujuk percobaan tetap dapat ditelusuri.

```mermaid
flowchart TD
    S(["Mulai: peninjau membuka draf in_review"]) --> CHECK["Periksa kunci, pembahasan, instruksi, rubrik, aset dan kesesuaian usia"]
    CHECK --> DEC{"Keputusan peninjau"}
    DEC -->|Perlu revisi| REV["Kembalikan ke draft dengan catatan; review lama tidak berlaku"]
    REV --> EDIT["Editor revisi lalu ajukan ulang O01"]
    DEC -->|Setujui| APPROVE["Simpan persetujuan untuk revisi dan hash termasuk versi aset"]
    APPROVE --> PUB["Peninjau meminta publish"]
    PUB --> SAME{"Revisi, hash dan aset sama; validasi masih lulus?"}
    SAME -->|Tidak| REV
    SAME -->|Ya| LIVE["Terbitkan versi immutable beserta provenance dan audit"]
    LIVE --> LATER{"Tindakan berikutnya"}
    LATER -->|Perubahan| NEW["Buat versi draft baru; jangan menimpa versi terbit"]
    LATER -->|Arsip| AR["Hentikan pemilihan baru; pertahankan referensi riwayat"]
    LATER -->|Tidak ada| Z(["Selesai"])
    EDIT --> Z
    NEW --> Z
    AR --> Z
```

## O04 — Harga, dukungan, penyesuaian akses, dan keluar paksa

Pelaku: Pengelola operasional, sistem. Cakupan: FR-12/13/14; M4.

Aksi memerlukan peran eksplisit dan alasan audit. Harga/refund mengikuti D7/D10; perubahan akses manual tidak boleh memalsukan status pembayaran gateway.

```mermaid
flowchart TD
    S(["Mulai: operasional membuka kasus berizin"]) --> ROLE["Server periksa peran operasional dan cakupan tindakan"]
    ROLE --> ACT{"Jenis tindakan"}
    ACT -->|Harga| PRICE["Buat versi harga baru untuk checkout berikutnya"]
    PRICE --> SNAP["Pesanan lama mempertahankan snapshot harga"]
    ACT -->|Dukungan akses atau refund| CASE["Periksa bukti pesanan, kebijakan D7/D10 dan alasan"]
    CASE --> CONF{"Tindakan dikonfirmasi dan sah?"}
    CONF -->|Tidak| NO["Tidak mengubah akses"]
    CONF -->|Ya| SEG["Catat penyesuaian segmen/refund melalui prosedur dukungan; audit"]
    ACT -->|Keluar paksa| SESS["Cabut sesi autentikasi/lease sesuai cakupan yang dipilih; audit"]
    SNAP --> Z(["Selesai"])
    NO --> Z
    SEG --> Z
    SESS --> Z
```

## O05 — Worker, kegagalan layanan, biaya, dan pemantauan

Pelaku: Worker River, sistem, operasional. Cakupan: FR-08/09/11/13/14; M3/M4.

Queue membawa ID referensi. Izin/penghapusan diperiksa lagi sebelum memproses dan menyajikan hasil; retry tidak menggandakan hasil logis atau biaya lewat panggilan berulang tanpa kontrol.

```mermaid
flowchart TD
    S(["Mulai: pekerjaan siap di antrean"]) --> C["Worker mengambil referensi dan memeriksa status/izin terbaru"]
    C --> OK{"Masih sah dan belum selesai?"}
    OK -->|Tidak| SK["Batalkan atau kembalikan hasil yang sudah ada"]
    OK -->|Ya| CALL["Panggil layanan dengan timeout/idempotency key bila didukung"]
    CALL --> R{"Hasil"}
    R -->|Berhasil| SAVE["Simpan satu hasil logis; catat status, latensi dan biaya"]
    R -->|Gagal sementara| LIM{"Batas retry belum habis?"}
    LIM -->|Ya| RET["Jadwalkan ulang dengan backoff"]
    LIM -->|Tidak| FAIL["Tandai gagal; beri jalur cadangan/status terbatas"]
    R -->|Status ambigu| REC["Catat ambigu dan rekonsiliasi sebelum mengulang efek eksternal"]
    SAVE --> MON["Log/Sentry tersanitasi dan metrik operasional tanpa data anak mentah"]
    FAIL --> MON
    REC --> MON
    SK --> Z(["Selesai putaran worker"])
    RET --> Z
    MON --> Z
```

## O06 — Cadangan, pemulihan, dan pencegahan data terhapus muncul kembali

Pelaku: Operasional, sistem database/media. Cakupan: FR-14; M4.

Target RPO/RTO dan jadwal retensi mengikuti D2/D8. Bukti uji restore disimpan; backup yang ada saja belum membuktikan pemulihan berhasil.

```mermaid
flowchart TD
    S(["Mulai: jadwal backup atau latihan pemulihan"]) --> B["Cadangkan PostgreSQL/WAL dan media sesuai kebijakan ke lokasi terpisah"]
    B --> V{"Backup terverifikasi dan dapat diakses?"}
    V -->|Tidak| ALERT["Peringatkan operasional; perbaiki dan ulang backup"]
    V -->|Ya| REST["Restore ke lingkungan terpisah dengan layanan keluar dikendalikan"]
    REST --> DEL["Terapkan catatan penghapusan sebelum membuka akses data"]
    DEL --> T["Uji integritas, isolasi, media, sesi/akses dan data yang telah dihapus"]
    T --> OK{"Sesuai target D8 dan kebijakan D2?"}
    OK -->|Tidak| FIX["Catat temuan; perbaiki prosedur dan ulang latihan"]
    FIX --> REST
    OK -->|Ya| E["Simpan bukti waktu/hasil restore dan prosedur operasional"]
    E --> Z(["Selesai latihan"])
    ALERT --> Z
```
