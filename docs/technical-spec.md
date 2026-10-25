# Hurahopi — Spesifikasi Teknis Alur Pertama

[Dokumentasi utama](../README.md).

Tanggal: 26 Oktober 2026. Status: rancangan siap diimplementasikan dengan data sintetis. Kontrak executable dan migrasi dibuat pada S02; dokumen ini tidak menyatakan keduanya sudah tersedia.

Acuan: [PRD](prd.md), [keputusan](decisions.md), [arsitektur](architecture.md), [pembelajaran](learning-spec.md), dan [backlog](delivery.md). Spesifikasi ini merinci Tahap 2; fitur lain tetap mengikuti cakupan Fase 1.

## Susunan repo yang akan dibuat

```text
apps/web/                    React, Vite, TypeScript
  src/app/                   routing, provider, pemilihan konteks pengguna
  src/features/              auth, profiles, learning, reports, content-admin
  src/components/            komponen umum dan primitive UI
  src/activities/            renderer templat aktivitas anak
  src/locales/id-ID/          teks antarmuka dan referensi narasi
  src/generated/             tipe dari openapi-typescript; tidak diedit manual
  src/lib/api/               client openapi-fetch, cookie/CSRF, galat dan key operasi
  src/admin/                 Shadcn Admin Kit, data/auth provider ke API Go
backend/
  cmd/api/                   entry point HTTP
  cmd/worker/                dibuat saat pekerjaan latar pertama diperlukan
  internal/                  identity, family, content, learning, reports, access
  migrations/                migrasi SQL berversi memakai Goose
  testdata/                  profil dan konten sintetis
contracts/openapi.yaml       kontrak HTTP executable, dibuat S02
infra/                       Compose, Caddy, contoh konfigurasi tanpa secret
docs/                        dokumen pengembangan
```

Satu modul Go di `backend/`, satu aplikasi frontend, dan backend modular dalam satu repo. Handler memvalidasi DTO lalu memanggil layanan domain; query memakai pgx. Transaksi bisnis berada di layanan domain. Jangan membuat layanan terpisah per modul pada milestone awal.

S01 mengambil scaffold backend minimum dari Go Blueprint dan frontend dari template resmi Vite React TypeScript. Gunakan pnpm dengan versi yang dikunci untuk frontend; catat versi/commit generator dan komponen sumber. Sesuaikan hasil generate dengan struktur di atas dan tabel stack di arsitektur; tidak membawa aplikasi frontend kedua, auth demo, atau layanan template yang tidak dipakai.

Saat bootstrap, perbarui [README utama](../README.md) untuk cara menjalankan; jangan membuat README di subdirektori. Tambahkan `.env.example`, lockfile frontend, `go.mod`/`go.sum`, dan perintah pemeriksaan yang bisa diulang di CI. Abaikan `.env`, token, log sensitif, media anak, dan hasil build dari version control. Path aplikasi di atas adalah target struktur; dokumentasi dan README utama sudah tersedia.

## Integrasi Modul yang Ditetapkan

Pilihan dan sumber modul mengacu pada [arsitektur](architecture.md#stack-inti-dan-layanan-pendukung). Aturan implementasinya:

- **UI:** satu fondasi shadcn/ui varian Base UI dan Tailwind. Adaptasi layout/tabel Satnaing untuk orang tua; satukan primitive dan token. Shadcn Admin Kit hanya pada `/admin`; gunakan adapter `ra-router-tanstack` untuk TanStack Router yang sama, bukan memasang router aplikasi lain.
- **Server state:** TanStack Query. Query key mencakup konteks pengguna/keluarga/anak/kursus; batalkan request dan bersihkan data saat konteks atau izin berubah. Jangan persist cache keluarga di browser. Jangan retry mutasi jawaban, transfer, hadiah, atau pembayaran secara otomatis; ulangi secara eksplisit dengan key operasi yang sama.
- **HTTP:** `src/generated/` hanya memuat tipe hasil openapi-typescript. Konfigurasi openapi-fetch di `src/lib/api/` memakai tipe tersebut dan menambahkan cookie/CSRF, pemetaan galat, serta idempotency key. Library client tidak dihasilkan ulang sebagai source dan tipe TypeScript tidak menggantikan validasi server.
- **Admin:** data provider menerjemahkan request/list ke API Go dan memetakan respons menjadi data ra-core; auth provider memanggil status sesi/izin backend. `record.id` adalah alias lokal dari PK resource, misalnya `activity_version_id`; adapter menerjemahkannya kembali ke nama PK saat request. Alias `id` tidak menjadi kolom database atau field identitas API. Aksi review/publish memakai endpoint domain dan revisi eksplisit, bukan generic edit status. Semua izin diuji kembali di Go.
- **Database:** pgxpool untuk aplikasi; Goose CLI sebagai proses migrasi terpisah dengan kredensial migrasi. API tidak membuat tabel atau menjalankan migrasi otomatis pada startup. Query SQL eksplisit dan transaksi layanan domain menjadi baseline; ORM atau sqlc tidak ditambahkan sebagai lapisan awal.
- **Identitas:** Google OAuth2 memakai `golang.org/x/oauth2` dan validasi ID token memakai go-oidc. Magic link, PIN, server session, dan lease tetap domain aplikasi. Resend melalui adapter pengirim; fake sender lokal/test tidak menentukan pilihan provider produksi.
- **Aktivitas:** dua templat awal memakai React/DOM dan ketuk/keyboard. dnd-kit hanya dipasang ketika drag dikerjakan. Konva/react-konva masuk M3 untuk kanvas, dengan state terstruktur dan pemulihan sendiri.
- **Pekerjaan:** River dibuat saat job tahan restart pertama dibutuhkan, termasuk email bila dijalankan asinkron; worker berbagi PostgreSQL dan domain service dengan API. Queue membawa ID referensi, bukan media, PIN, cookie, atau token login mentah.
- **PDF:** worker menyusun HTML/CSS dan aset laporan lalu memanggil Gotenberg internal. Tidak menerima URL/render instruction arbitrer dari browser; hasil disimpan sementara sesuai D2. Converter tidak memegang sesi pengguna dan bukan endpoint unduh publik.
- **Error:** `slog` untuk log JSON server; SDK Sentry Go/React untuk error teknis tersanitasi. Tidak mengirim body, token, jawaban, nama/media anak atau input PIN. Session replay/screenshot/pengambilan PII otomatis dinonaktifkan. Adapter/DSN baru diaktifkan setelah pengujian sanitasi.

## Konvensi data dan waktu

### Batas Layar dan Konteks Browser

| Route UI | Konteks | Sumber data dan tindakan |
| --- | --- | --- |
| `/orang-tua` | Dewasa yang berizin | Pemilih anak, ringkasan dan tombol menyiapkan belajar; pengasuh dibatasi profil/fitur yang diizinkan |
| `/belajar/{study_session_id}` | Mode anak dengan sesi yang disiapkan atau recovery terminal yang sah | Beranda Hopi; Mulai/Lanjutkan membuka instans pada sesi yang sama, bukan membuat sesi baru |
| `/belajar/{study_session_id}/aktivitas` | Lease aktif yang sah | Render instans saat ini; submit/hint/skip melalui endpoint belajar |
| `/belajar/{study_session_id}/petualangan` | Konteks anak yang sah | Tampilan ringkas perjalanan; tidak memilih tingkat di luar mesin belajar |
| `/belajar/{study_session_id}/koleksi` | Konteks anak yang sah | Tampilan item/apresiasi yang diizinkan; aksi personalisasi ditambahkan sesuai kontrak milestone fitur |
| `/admin` | Peran admin eksplisit pada backend | Area Shadcn Admin Kit; bukan akses otomatis pemilik akun keluarga |

Sesudah POST menyiapkan sesi berhasil, server memasang lease dan mode anak, lalu frontend menampilkan beranda anak. GET sesi menyertakan DTO anak terbatas: nama panggilan/avatar anak tersebut, ringkasan misi, posisi perjalanan, dan apresiasi yang boleh dilihat. Tidak menyertakan profil saudara, laporan dewasa, harga, pesanan, izin pengasuh, atau kunci soal. Tampilan ringkas peta/koleksi M1 dapat memakai DTO sesi yang sama; endpoint lanjutan ditambahkan saat fitur penuh dikerjakan.

Pada kunjungan ulang, frontend memeriksa status sesi/konteks server sebelum memilih beranda, instans tersimpan, apresiasi, atau pesan sesi dipindahkan. Selesai/pause tidak memberi izin membuat sesi baru dari mode anak. Tombol Area orang tua/ganti profil membuka PIN dan berpindah konteks setelah server mengizinkan. Hasil terminal hanya dapat dibaca sesuai aturan recovery yang telah ditetapkan. URL dan menyembunyikan menu tidak menggantikan otorisasi endpoint.

### Ketentuan Data

- Tabel/kolom memakai `snake_case`; PK nama entitas tunggal + `_id` dan FK mengikuti PK tujuan. ID resource berupa UUID opaque; slug seperti `math`, `counting-1-10`, dan `L2` adalah kode domain, bukan pengganti pemeriksaan izin.
- Timestamp database memakai `timestamptz`; API memakai ISO 8601 UTC. Tampilan periode belajar memakai `Asia/Jakarta`.
- Nama panggilan, usia dalam tahun, dan avatar cukup untuk profil awal. Hindari tanggal lahir, alamat, sekolah, atau identitas tambahan pada alur pertama.
- `family_id` diturunkan dari sesi dan resource tersimpan; request tidak boleh menentukan kepemilikan keluarga secara bebas.
- Setiap relasi anak–sesi–percobaan–bukti harus berada pada keluarga dan kursus yang sama. Gunakan foreign key/unique constraint serta scope query; UUID yang sulit ditebak tidak menggantikan otorisasi.
- Nilai harga kelak memakai integer rupiah dan versi harga immutable. Hak belajar diperiksa berdasarkan anak dan kursus, bukan hanya status akun.

## Model data dan migrasi inti

Acuan tunggal tabel, kolom, PK/FK, nullable, dan constraint adalah [ERD/database](database/design.md) beserta [kamus data](database/data-dictionary.md). Jangan mempertahankan model minimum kedua yang berbeda nama atau kardinalitas. Seluruh tabel/kolom memakai `snake_case`, PK nama entitas tunggal + `_id`, dan FK mengikuti PK tujuan.

| Kelompok alur pertama | Tabel utama dan PK contoh | Invariant penting |
| --- | --- | --- |
| Identitas dan keluarga | `users.user_id`, `identities.identity_id`, `families.family_id`, `children.child_id` | Membership/consent/sesi mengikat akses keluarga; mode anak dibatasi server |
| Konten | `levels.level_id`, `level_skills.level_skill_id`, `activity_versions.activity_version_id` | Kurikulum/rubrik satu kursus; versi terbit immutable |
| Penempatan dan belajar | `placement_runs.placement_run_id`, `study_sessions.study_session_id`, `learning_leases.learning_lease_id` | Satu sesi terbuka dan satu lease aktif per anak; lease lama dipertahankan sebagai riwayat |
| Respons dan hasil | `attempts.attempt_id`, `responses.response_id` | Unik slot sesi dan sequence respons; penyajian/hasil pada attempt yang sama |
| Progres dan bukti | `mastery_states.mastery_state_id`, `mastery_changes.mastery_change_id`, `chapter_progress.chapter_progress_id` | State per anak/level_skill/versi aturan/rubrik; bukti di `mastery_change_evidence`; progres pada bab yang benar |
| Keandalan | `idempotency_records.idempotency_record_id`, `audit_events.audit_event_id` | Replay tetap memeriksa izin/lease; efek logis tidak digandakan |

S02 membuat kerangka Goose, konvensi, dan tabel/seed kursus. Identitas ditambahkan S03, konten/roles/review serta layanan publish minimum S04, sesi/attempt/respons/progres S05–S07, editor/media sintetis S09. Ikuti [urutan implementasi database](database/design.md#urutan-implementasi), termasuk kolom FK yang baru ditambah saat target tersedia. Constraint setiap kelompok diuji saat migrasinya dibuat; S02 tidak membuat seluruh tabel Fase 1. S01 tidak perlu membangun worker. Hak belajar pada S05 diperoleh dari interface `AccessPolicy`; adapter internal memakai fixture sintetis dan adapter produksi kelak membaca segmen akses.

Fixture akses hanya tersedia pada environment lokal/test yang eksplisit. Startup produksi wajib gagal bila dev-login atau fixture akses diaktifkan. Jangan menyisipkan hak belajar gratis permanen sebagai default ketika data langganan tidak ditemukan.

## Autentikasi, mode, dan izin

Google login memakai Go OAuth2 dan go-oidc, dengan pemeriksaan state/nonce, signature, issuer, audience, dan expiry ID token pada backend. Tautan email memakai token sekali pakai dengan expiry; konsumsi token dilakukan melalui POST setelah pengguna membuka layar konfirmasi agar pemindai tautan email tidak otomatis menghabiskannya. Respons permintaan tautan tidak mengungkap apakah email sudah terdaftar.

Browser memakai cookie sesi `HttpOnly`, `Secure` pada HTTPS, dan `SameSite`; request perubahan data memerlukan token CSRF dan pemeriksaan origin. Login merotasi ID sesi. Token login, cookie, PIN, jawaban mentah, dan media tidak masuk log. PIN di-hash dengan fungsi password hashing, diberi pembatasan percobaan, dan tidak menjadi pengganti identitas akun.

Mode anak melekat pada sesi autentikasi browser tersebut, bukan seluruh sesi pengguna. Mulai belajar dilakukan dari konteks dewasa yang telah diverifikasi PIN, kemudian sesi browser masuk mode anak. Cookie lease opaque terpisah mengizinkan operasi belajar pada satu anak/sesi saja. Backend menolak endpoint dewasa dari mode anak walaupun cookie login orang tua masih valid. Browser lain dapat tetap berada di area dewasa.

Keluar ke area orang tua memakai verifikasi PIN server; default teknis unlock untuk operasi sensitif adalah lima menit dan dapat dikonfigurasi. Pengasuh memakai PIN sendiri serta izin keluarga/profil sendiri. Pembukaan PIN tidak menaikkan peran atau memberi izin yang belum dimiliki. Reset PIN memerlukan autentikasi ulang orang dewasa, bukan dapat dilakukan dari mode anak.

Satu sesi browser menampilkan satu konteks anak pada satu waktu; seluruh tab dengan cookie yang sama berbagi mode/konteks. Jika pendamping mengganti anak, konfirmasikan dan jalankan pause untuk sesi sebelumnya sebelum menerbitkan lease anak berikutnya. Start/resume/transfer memeriksa ulang konteks di bawah lock auth_session. Mode anak tetap ditolak dengan `403 ADULT_UNLOCK_REQUIRED`; setelah konteks dewasa sah, adanya lease aktif anak lain menghasilkan `409 BROWSER_LEARNING_ACTIVE`. Belajar bersamaan untuk kakak/adik memakai browser/perangkat dengan sesi autentikasi masing-masing; tidak membutuhkan akun keluarga terpisah.

Respons resource milik keluarga lain menggunakan `404 RESOURCE_NOT_FOUND`. Resource keluarga sendiri yang operasinya tidak diizinkan menggunakan `403 FORBIDDEN`; mode anak menggunakan `403 ADULT_UNLOCK_REQUIRED`. Pola ini mengikuti pemeriksaan izin setiap request dalam [panduan OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html).

### Identitas soal dan hasil

Render, respons, hint, media dan bukti memakai `attempt_id` yang sama. Satu attempt sudah menyimpan penyajian serta hasil logis; jawaban koreksi menambah row `responses` dengan sequence baru. Respons `sequence=1` menjadi `first_response_id` yang immutable. `status` penyelesaian dipisahkan dari `evaluation_status`; mastery tidak membaca pending/skip/teknis/karya terbuka sebagai salah. Rincian [attempt dan penilaian](database/design.md#attempt-respons-dan-penilaian).

## Status dan transisi

| Objek | Transisi | Aturan |
| --- | --- | --- |
| Sesi belajar | dibuat → `active` | Periksa izin dan akses; latihan membekukan rencana, penempatan membekukan aturan/batas dan instans yang sudah disajikan; maksimal satu sesi terbuka per anak |
| Sesi belajar | `active` → `paused` → `active` | Pause melepas lease; resume memeriksa ulang hak belajar dan menerbitkan lease baru |
| Sesi belajar | `active` → `completed` | Semua slot rencana selesai/dilewati; satu apresiasi sesi; lease dicabut |
| Sesi belajar | `active`/`paused` → `ended` | Penghentian eksplisit dari area dewasa; pertahankan hasil, jangan dianggap tuntas |
| Lease | aktif generasi N → aktif generasi N+1 | Pemindahan terkonfirmasi; ID sesi dan instans tetap; secret lama ditolak |
| Instans | `presented` → `in_progress` → terminal | Terminal adalah `completed`, `skipped`, atau `technical_failure`; `presented` boleh langsung terminal |
| Respons salah | `in_progress` → `in_progress` | Boleh koreksi dengan nomor respons baru; nilai respons pertama tetap tersimpan |
| Konten | `draft` → `in_review` → `published` → `archived` | Penolakan kembali ke draft; mengubah konten yang ditinjau membatalkan review; perubahan konten terbit membuat versi baru |

Sesi belajar penempatan dan `placement_run` adalah objek berbeda. Jika anak beralih ke latihan sebelum penempatan lengkap, akhiri sesi belajar penempatan sebagai `ended` dengan alasan `continue_with_provisional_placement`, tetapi pertahankan run/probe yang belum lengkap. Latihan lalu memakai sesi baru dan hasil sementara. Kembali ke penempatan membuat sesi belajar baru yang melanjutkan run tersebut. Start dengan purpose berbeda ketika masih ada sesi `active`/`paused` menghasilkan `409 SESSION_PURPOSE_CONFLICT` sampai pendamping mengonfirmasi penghentian sesi lama. Tes ulang secara eksplisit menandai run lama `superseded`, mempertahankan buktinya, dan membuka run baru setelah sesi terbuka ditutup.

`completed` pada attempt berarti anak menyelesaikan aktivitas; bedakan dari `first_response_correct` dan `independent_correct`. Aktivitas yang diselesaikan setelah salah/petunjuk memperoleh apresiasi partisipasi tetapi tidak menjadi benar mandiri. Untuk prototipe, setelah dua respons salah tawarkan contoh terbimbing atau lewat; jangan paksa mengulang tanpa akhir. Contoh terbimbing menandai bantuan, dan bukan respons mandiri.

Pada transisi terminal latihan, lanjutkan ke instans berikutnya dalam rencana. Pada penempatan, pilih probe berikutnya dari hasil yang sudah ada dalam batas versi aturan; instans yang sudah disajikan tetap immutable. Bila semua slot dilewati, tutup sesi tanpa klaim kemajuan kemampuan. Mengakhiri lebih awal memakai pause atau `ended` sesuai tindakan pengguna; tidak otomatis mengubah slot tersisa menjadi jawaban salah.

## Transaksi yang wajib atomik

1. **Mulai/resume:** kunci auth_session pemanggil, periksa ulang mode/PIN/konteks dan lease browser, lalu kunci anak dan cari sesi terbuka. Periksa akses dan buat/lanjut sesi serta lease. Partial unique per child dan per auth_session menolak sesi/lease ganda, termasuk dua tab yang memilih anak berbeda. Perubahan cookie/konteks hanya mengikuti transaksi yang berhasil.
2. **Pemindahan:** kunci auth_session penerima → anak → sesi/lease; periksa kembali PIN/izin/akses dan `expected_generation`; naikkan `children.lease_generation`, cabut baris lease lama, tambahkan baris lease generasi baru, serta tulis audit. Jika generasi sudah berubah, kembalikan konflik tanpa mencabut lease terbaru. Jangan mengunci/mengubah auth_session pemilik lama setelah child; login lama tetap ada dan mengetahui pencabutan melalui lease.
3. **Jawaban/petunjuk:** kunci auth_session → anak → sesi/lease → instans; verifikasi ulang mode, active_child_id, token, generasi, status, dan versi. Simpan respons atau bantuan serta update hasil logis secara atomik. Request petunjuk yang commit lebih dahulu membuat respons berikutnya tidak mandiri.
4. **Penyelesaian instans:** terminalisasi, pembaruan progres, hadiah unik, dan evaluasi aturan berada dalam satu transaksi. Bila perlu worker, enqueue pada transaksi yang sama. Perubahan tingkat berlaku untuk rencana sesi berikutnya.
5. **Pencabutan akses:** perubahan izin terbaru dan pencabutan lease terkait diserialisasi dengan jawaban. Layanan tanpa perubahan auth_session boleh mulai pada anak lalu lease; jangan mengambil lock auth_session setelah anak. Logout/perubahan sesi browser mengambil auth_session lebih dahulu. Bila beberapa baris sejenis diperlukan, urutkan menurut PK. Query tidak mengandalkan permission yang hanya disalin ke token lama.

Periksa otorisasi/lease sebelum mengembalikan hasil idempotensi. Jawaban yang commit sebelum pemindahan tetap sah; jawaban dari lease lama yang diproses setelah pemindahan ditolak. Urutan ini harus dibuktikan dengan pengujian integrasi PostgreSQL.

Evaluasi membandingkan bukti dengan tingkat sumber instans. Jika target perubahan yang sama sudah dijadwalkan, jangan membuat perubahan/hadiah tambahan dari sisa instans pada tingkat itu. `chapter_progress` menyimpan target sesi berikutnya; baru saat sesi berikutnya dibuat target menjadi tingkat latihan efektif. Membuka keterampilan berikutnya membuat state bukti baru untuk keterampilan tersebut, sementara pencapaian keterampilan sebelumnya tetap tersedia.

Lease yang berakhir karena `completed` atau `paused` tidak lagi menerima tulisan baru, tetapi hash token dan alasan penutupannya dipertahankan untuk pemulihan hasil. Retry operasi yang sudah commit boleh membaca hasil tersimpan jika token/generasi dan sesi autentikasi pemilik masih cocok, izin profil masih berlaku, serta belum ada pemindahan/pencabutan izin. Ini tidak memberi izin menulis setelah sesi selesai. Lease yang dicabut karena transfer, logout, atau pencabutan izin tetap ditolak sebelum replay; klien mengambil status melalui konteks dewasa bila perlu.

Jika respons penerbitan cookie lease hilang, retry start/resume/transfer dengan key yang sama dari sesi autentikasi penerima dapat menerbitkan ulang lease milik browser itu saja melalui rotasi token atomik. Record operasi belajar merujuk `learning_lease_id` asli. Recovery cookie hanya merotasi lease aktif yang ID, pemilik, anak, dan generasinya sama dengan hasil operasi tersebut. Setelah transfer keluar lalu kembali, key lama ditolak untuk penerbitan cookie lease pengganti. Ini hanya memulihkan hasil operasi yang telah tercatat; tidak membuat sesi/percobaan baru dan tidak memindahkan lease yang telah beralih ke browser lain. Jalur recovery boleh diakses dari mode anak hanya untuk operasi tercatat tersebut; tidak membuka operasi dewasa lain atau menghapus pemeriksaan PIN untuk pemindahan baru. Secret lease tidak disimpan di cache respons atau log.

## Rancangan kontrak API

Base path `/api/v1`, JSON `snake_case`, `request_id` untuk korelasi log. Daftar admin memakai `page` dan `per_page`, parameter sort/filter berdaftar izin, serta respons `{items, total, page, per_page}` agar data provider ra-core dapat memetakan paginasi secara langsung. Daftar profil kecil tidak perlu paginasi; daftar riwayat anak yang besar dapat memakai cursor dengan schema endpoint tersendiri. `Idempotency-Key` wajib pada operasi POST bisnis yang menambah/mengubah hasil logis. Scope key: pengguna, sesi autentikasi, operasi, dan resource. Retry dengan payload sama mengembalikan hasil logis semula; metadata token/generasi boleh berubah pada pemulihan penerbitan lease di atas. Key sama dengan payload berbeda menghasilkan `409 IDEMPOTENCY_KEY_REUSED`. Default teknis retensi record 24 jam untuk prototipe; constraint domain tetap mencegah duplikasi setelah record dibersihkan. TTL produksi ditinjau bersama D2.

| Metode dan path | Konteks | Input/hasil utama |
| --- | --- | --- |
| `GET /auth/session` | Publik/dewasa/anak | Status login dan CSRF; mode anak hanya mendapat konteks minimal, tanpa data pembayaran/laporan |
| `GET /auth/google/start`, `GET /auth/google/callback` | Publik | Redirect OAuth; state/nonce dan validasi identitas backend |
| `POST /auth/email-links` | Publik | `{email}` → respons generik `202`; rate limit |
| `POST /auth/email-links/consume` | Publik | Token dan konfirmasi pengguna → sesi; token tidak masuk query log |
| `POST /auth/logout` | Login | Cabut sesi browser serta lease yang dimilikinya, bukan seluruh perangkat |
| `POST /auth/pin` | Dewasa dengan autentikasi ulang | Atur/ganti PIN; verifikasi bukti login ulang, hash server |
| `POST /auth/adult-unlock` | Login, termasuk mode anak | `{pin}` → konteks dewasa; rate limit; tidak mengubah peran |
| `GET /children` | Dewasa | Hanya profil sesuai membership/izin; mode anak tidak mendapat daftar keluarga lengkap |
| `POST /children`, `PATCH /children/{child_id}` | Pemilik + unlock | Nama panggilan, usia, avatar; perubahan tercatat |
| `POST /children/{child_id}/consents` | Pemilik + unlock | Jenis, versi kebijakan, grant/withdraw; event tercatat |
| `GET /children/{child_id}/learning-state` | Dewasa berizin | Penempatan, tingkat latihan, sesi dapat dilanjutkan, status akses; tanpa secret lease |
| `POST /children/{child_id}/placement-runs/retest` | Pemilik + unlock | Mulai run baru dengan alasan setelah sesi belajar aktif/paused diakhiri dengan konfirmasi; run lama superseded, riwayat hasil dipertahankan |
| `PATCH /children/{child_id}/practice-level` | Pemilik + unlock | Bab/tingkat dan alasan; hanya tingkat dengan konten tersedia; diterapkan sesi berikutnya tanpa menambah bukti mastery |
| `POST /children/{child_id}/study-sessions` | Dewasa berizin + unlock | `{course, purpose}` → buat/resume sesi dan cookie lease; `purpose=placement` melanjutkan run terbuka |
| `POST /study-sessions/{study_session_id}/transfer` | Dewasa berizin + unlock | `{expected_generation}` → lease baru; konflik meminta konfirmasi ulang |
| `GET /study-sessions/{study_session_id}` | Lease valid atau recovery terminal milik browser yang sama | Rencana ringkas, status, instans saat ini; selesai/pause memberi hasil tersanitasi; lease dipindah/dicabut izin → galat khusus |
| `POST /study-sessions/{study_session_id}/interactions` | Lease belajar valid | Heartbeat interaksi `{study_session_event_id, kind}`; polling tidak memperpanjang idle; bukan bukti jawaban/kemampuan |
| `POST /study-sessions/{study_session_id}/pause` | Lease belajar valid | Simpan posisi, pause, lepas lease |
| `POST /study-sessions/{study_session_id}/end` | Dewasa berizin + unlock | Akhiri sesi aktif/paused; jangan membuat klaim tuntas |
| `GET /attempts/{attempt_id}` | Lease belajar valid | Render DTO instans saat ini, tanpa kunci atau skor tersembunyi |
| `POST /attempts/{attempt_id}/hints` | Lease belajar valid | Catat hint sebelum respons; kembalikan hint tahap berikutnya |
| `POST /attempts/{attempt_id}/assistance` | Lease belajar valid | `source` bernilai `adult` atau `guided_example`; catat bantuan yang diketahui |
| `POST /attempts/{attempt_id}/responses` | Lease belajar valid | `{sequence, answer}`; penilaian server, hasil, dan status instans |
| `POST /attempts/{attempt_id}/skip` | Lease belajar valid | `reason` bernilai `child_choice` atau `technical_failure`; bukan salah |
| `GET /children/{child_id}/reports/summary` | Dewasa berizin | Periode, status bukti, ringkasan, alasan, referensi percobaan |
| `GET /children/{child_id}/reports/evidence/{attempt_id}` | Dewasa berizin | Bukti ringkas milik anak tersebut; tanpa secret/kunci bank soal |
| `GET /admin/content-catalog` | Editor/peninjau | Bab, tingkat, keterampilan, templat, dan parameter untuk editor minimum |
| `GET /admin/activity-versions`, `GET /admin/activity-versions/{activity_version_id}` | Editor/peninjau | Daftar/filter antrean dan detail preview sesuai peran |
| `POST /admin/activity-versions` | Editor konten | Draf konten tervalidasi |
| `PATCH /admin/activity-versions/{activity_version_id}` | Editor konten | Ubah draf dengan `expected_revision`; review lama digugurkan; versi terbit tidak dapat ditimpa |
| `POST /admin/activity-versions/{activity_version_id}/submit-review` | Editor konten | Bekukan revisi untuk peninjauan |
| `POST /admin/activity-versions/{activity_version_id}/reviews` | Peninjau konten | `{decision, notes, revision}`; keputusan terikat revisi |
| `POST /admin/activity-versions/{activity_version_id}/publish` | Peninjau konten | Hanya revisi yang lolos validasi dan review |

S02 menuliskan path, enum, schema request/response, cookie, CSRF, status code, dan contoh di `contracts/openapi.yaml` sebelum handler fitur dibuat. openapi-typescript menghasilkan tipe dan openapi-fetch menggunakannya; CI memeriksa schema valid, tipe/client dapat dikompilasi, serta regenerasi tipe tidak menghasilkan diff. Endpoint billing/media/pengasuh/PDF ditambahkan ketika milestone terkait siap dengan aturan PRD sebagai acuan.

### Contoh profil anak

Request pembuatan profil dari area dewasa:

```json
{"nickname":"Alya","age_years":6,"avatar":"hopi"}
```

`avatar` adalah pilihan katalog tetap yang divalidasi server. Bentuk nilainya tetap string; tidak perlu akhiran code pada nama field. `users.roles` juga berisi nilai tetap, default kosong; izin keluarga tetap berasal dari membership dan child_permissions, bukan dari daftar role global.

### Contoh pemindahan sesi

Request membuat/melanjutkan belajar di laptop ketika tablet masih aktif:

```json
{"course":"math","purpose":"practice"}
```

Respons `409` berikut tidak membocorkan token browser tablet:

```json
{
  "error": {
    "code": "LEARNING_ACTIVE_ELSEWHERE",
    "message": "Sesi belajar Alya sedang terbuka di perangkat lain.",
    "details": {"study_session_id":"11111111-1111-4111-8111-111111111111","generation":4}
  },
  "request_id":"req-demo-01"
}
```

Setelah konfirmasi, POST transfer mengirim `{"expected_generation":4}` dengan cookie dewasa dan CSRF yang sah. Server menerbitkan cookie lease generasi 5, mengubah browser penerima ke mode anak, dan mempertahankan ID sesi/instans. Nomor generasi adalah kontrol konflik, bukan bukti otorisasi.

### Contoh render dan jawaban

DTO render publik anak untuk templat pilihan kelompok:

```json
{
  "attempt_id":"22222222-2222-4222-8222-222222222222",
  "template":"choose_group",
  "instruction":"Pilih piring berisi tiga apel.",
  "instruction_audio":"id-count-apples-three-v1",
  "options":[
    {"option_id":"a","asset":"apple","count":2},
    {"option_id":"b","asset":"apple","count":3},
    {"option_id":"c","asset":"apple","count":4}
  ],
  "next_sequence":1,
  "hint_available":true
}
```

Instruksi, jumlah objek, dan opsi adalah informasi yang memang dilihat anak; kunci `correct_option_id` tidak dikirim. Klien mengirim `{"sequence":1,"answer":{"selected_option_id":"b"}}`. Untuk `build_group`, kirim ID objek unik yang dipilih, bukan `correct:true` atau skor dari klien. Server memeriksa ID terhadap objek yang tersedia lalu menghitung jumlah.

Respons sukses memuat `attempt_id`, `first_response_correct`, `independent_correct`, `status`, `feedback`, dan `next_attempt_id` nullable. Anak menerima umpan balik dari `feedback`; angka penilaian internal tidak perlu tampil. Jika respons pertama salah lalu respons kedua benar, `first_response_correct=false` tetap dipertahankan.

### Galat bisnis

| HTTP | Kode | Perilaku klien |
| --- | --- | --- |
| 401 | `AUTH_REQUIRED` | Kembali login; progres tersimpan tetap ada |
| 403 | `ADULT_UNLOCK_REQUIRED` | Tampilkan PIN tanpa membuka data dewasa terlebih dahulu |
| 403 | `FORBIDDEN`, `CONSENT_REQUIRED`, `ACCESS_EXPIRED` | Jelaskan kebutuhan di area dewasa; mode anak menerima pesan netral |
| 404 | `RESOURCE_NOT_FOUND` | Tidak mengungkap resource keluarga lain |
| 409 | `LEARNING_ACTIVE_ELSEWHERE`, `LEASE_CHANGED` | Tampilkan atau ulangi konfirmasi pemindahan |
| 409 | `BROWSER_LEARNING_ACTIVE` | Pendamping pause sesi anak yang masih dimiliki browser sebelum pindah profil; tidak mengganti konteks diam-diam |
| 409 | `SESSION_PURPOSE_CONFLICT` | Pendamping mengakhiri sesi purpose lama dengan konfirmasi; run penempatan belum lengkap tetap tersimpan |
| 409 | `LEARNING_LEASE_REVOKED` | Hentikan pengiriman; sesi dipindahkan; jangan retry otomatis untuk mengambil alih |
| 409 | `IDEMPOTENCY_KEY_REUSED`, `RESPONSE_SEQUENCE_CONFLICT`, `ATTEMPT_NOT_CURRENT` | Ambil status resmi; jangan membuat instans pengganti diam-diam |
| 422 | `VALIDATION_ERROR`, `CONTENT_UNAVAILABLE` | Tampilkan kesalahan form atau aktivitas cadangan yang sudah terbit |
| 429 | `RATE_LIMITED` | Hormati `Retry-After`; jangan menyalahkan anak |
| 503 | `TEMPORARILY_UNAVAILABLE` | Pertahankan instans dan retry operasi logis yang sama setelah koneksi pulih |

## Laporan dan bukti

Respons ringkasan memuat `child_id`, `course`, `period_started_at`, `period_ended_at`, `updated_at`, `evidence_status`, `report_rule_version_id`, `report_rule_version`, `current_practice_level`, serta daftar klaim dengan `evidence_attempt_ids`. Contoh output narasi dan sumber bukti ada di [contoh laporan](learning-spec.md#contoh-laporan-orang-tua).

Default prototipe memakai query/templat deterministik. Ringkasan kemampuan minimal tiga aktivitas pada dua sesi; syarat mastery tetap 8/10. Karya terbuka dan jawaban dibantu tidak boleh dipromosikan menjadi bukti penguasaan objektif. Report endpoint tetap memeriksa membership meskipun masa belajar habis.

## Pemeriksaan implementasi pertama

- Kontrak valid, tipe hasil openapi-typescript dan konfigurasi openapi-fetch sesuai, lint/typecheck/build frontend serta pemeriksaan Go lulus.
- Migrasi Goose berjalan pada PostgreSQL kosong dan upgrade dari skema milestone sebelumnya; API tidak menjalankan migrasi saat startup; constraint menolak sesi/lease/hadiah ganda.
- Uji otorisasi mencoba ID keluarga lain, mode anak, pengasuh dicabut, dan konteks dewasa tanpa PIN.
- Uji transaksi mencakup dua tab satu auth_session yang memilih anak berbeda, lalu menjalankan dua start, dua transfer, hint vs jawaban, dan jawaban vs transfer bersamaan.
- Uji recovery mencakup cookie lease yang responsnya hilang, jawaban terakhir sudah commit tetapi respons hilang, serta resume setelah pause; replay tidak boleh mengambil alih perangkat lain.
- Uji domain menutup kasus 7/10, 8/10 satu sesi, 8/10 dua sesi/dua templat, bantuan, duplikat, dan bukti teknis gagal.
- Playwright menjalankan profil → penempatan → misi → apresiasi → laporan, pemulihan reload, dan pemindahan perangkat dengan dua browser context.

- Layanan review/publish S04 menolak review dengan aset/revisi/hash berbeda; fixture keluarga memiliki review pendidik sebelum S08.
- Integrasi UI/admin menggunakan satu router dan satu fondasi primitive; adapter daftar, galat, CSRF, dan aksi review/publish diverifikasi dengan API nyata pada S09.
- Impor sumber mencatat asal/lisensi; startup/build tidak membutuhkan Clerk, Redis, S3/MinIO, atau fork LMS. Modul kanvas/PDF/drag baru dibundel/dijalankan ketika milestone-nya memerlukannya.

Rincian gate dan pemetaan ke cerita ada di [delivery](delivery.md); prasyarat penggunaan data anak dan produksi ada di [operasional](operations.md).

Referensi alur: [indeks activity diagram](../README.md#activity-diagram). Penempatan diagram di dokumen tidak menyatakan API/M3/M4 sudah diimplementasikan.
