# Hurahopi — Kamus Data

Acuan metadata: [schema.json](schema.json). Status: rancangan, belum migrasi SQL; model, ERD, dan kamus data dipelihara bersama. [Dokumentasi utama](../../README.md#database) · [Konvensi dan aturan lintas tabel](design.md).

`Ya` pada Nullable berarti SQL NULL diperbolehkan; bukan berarti input API selalu opsional. PK selalu UUID dan NOT NULL. Seluruh tabel memiliki `created_at`; `updated_at` dipakai pada data yang melacak perubahan terakhir. Nilai awal, enum status, ON DELETE, serta indeks fisik dilengkapi pada migrasi milestone sesuai aturan di bawah dan spesifikasi desain.

## Akun dan Akses

### users

Identitas orang dewasa; anak tidak mempunyai akun login sendiri.

Mulai implementasi: **S03**. Entitas tunggal: `user`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Tidak | PK |  |
| `email` | `text` | Tidak |  | Alamat yang dinormalisasi server untuk pencocokan/keunikan; input email tetap harus diverifikasi sesuai alur identitas. |
| `name` | `text` | Ya |  |  |
| `pin_hash` | `text` | Ya |  |  |
| `status` | `text` | Tidak |  |  |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `roles` | `text[]` | Tidak |  | Peran global tetap editor/reviewer/operator; default array kosong, dinormalisasi sebagai himpunan. Bukan role owner/caregiver keluarga. Ditambahkan pada S04. |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (email)`

**CHECK per baris:**

- `status IN ('active','disabled','deleting')`
- `roles <@ ARRAY['editor','reviewer','operator']::text[]`
- `array_position(roles, NULL) IS NULL`
- `COALESCE(array_ndims(roles), 1) = 1`
- `cardinality(roles) <= 3`

**Aturan domain:**

- Email terverifikasi menjadi dasar penautan identitas melalui bukti server. PIN tersimpan sebagai hash password; tidak menyimpan PIN mentah.
- Perubahan roles hanya lewat layanan admin berizin dengan lock user dan audit actor/before/after. Default kosong; owner keluarga tidak otomatis admin. Fase 1 tidak mempunyai katalog role dinamis atau expiry terpisah per role.

### identities

Identitas Google atau email yang telah dibuktikan.

Mulai implementasi: **S03**. Entitas tunggal: `identity`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `identity_id` | `uuid` | Tidak | PK |  |
| `user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `provider` | `text` | Tidak |  |  |
| `provider_subject` | `text` | Tidak |  |  |
| `email_verified_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (provider, provider_subject)`

**Aturan domain:**

- Subject provider adalah teks eksternal, bukan foreign key ke pengguna lain.

### families

Batas kepemilikan data satu keluarga.

Mulai implementasi: **S03**. Entitas tunggal: `family`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `family_id` | `uuid` | Tidak | PK |  |
| `status` | `text` | Tidak |  |  |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Aturan domain:**

- Pembuatan keluarga dan membership owner berlangsung dalam satu transaksi; keluarga aktif harus mempunyai satu owner.
- Baris keluarga menjadi titik lock saat mengatur kuota pengasuh/undangan.

### family_memberships

Hubungan pengguna dewasa dengan keluarga dan perannya.

Mulai implementasi: **S03**. Entitas tunggal: `family_membership`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `family_membership_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `role` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `revoked_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (family_id, user_id)`
- `UNIQUE (family_id) WHERE role = 'owner' AND status = 'active'`
- `UNIQUE (family_membership_id, family_id)`

**CHECK per baris:**

- `role IN ('owner','caregiver')`
- `status IN ('active', 'revoked')`

**Indeks pencarian awal:**

- `INDEX (user_id, status, family_id)`

**Aturan domain:**

- Index parsial membatasi paling banyak satu owner; transaksi layanan memastikan keluarga aktif tidak kehilangan owner.

### children

Profil belajar anak, berada dalam tepat satu keluarga.

Mulai implementasi: **S03**. Entitas tunggal: `child`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `child_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `nickname` | `text` | Tidak |  |  |
| `age_years` | `smallint` | Tidak |  |  |
| `avatar` | `text` | Tidak |  | Pilihan avatar dari katalog tetap, misalnya hopi. Nama field tetap avatar; bukan unggahan atau URL bebas. |
| `status` | `text` | Tidak |  |  |
| `data_version` | `bigint` | Tidak |  |  |
| `selected_reward_grant_id` | `uuid` | Ya | FK → [reward_grants.reward_grant_id](#reward_grants) | Ditambahkan pada M3. |
| `preferences` | `jsonb` | Tidak |  | Preferensi dekorasi/aksesibilitas yang divalidasi; default objek JSON kosong, tanpa identitas relasional tersembunyi. Ditambahkan pada M3. |
| `lease_generation` | `bigint` | Tidak |  | Counter generasi lease yang tidak direset ketika riwayat lama dibersihkan; default 0. Ditambahkan pada S05. |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, family_id)`

**CHECK per baris:**

- `age_years BETWEEN 4 AND 10`
- `data_version >= 0`
- `lease_generation >= 0`

**Indeks pencarian awal:**

- `INDEX (family_id, status)`

**FK komposit tambahan:**

- `(selected_reward_grant_id, child_id)` → `reward_grants (reward_grant_id, child_id)`.

**Aturan domain:**

- Titik acuan usia bukan tingkat kemampuan. Penghapusan profil mengikuti D2/D7 dan tidak mendaur ulang child_id.
- Personalisasi adalah atribut profil satu-ke-satu, bukan tabel terpisah. FK komposit selected_reward_grant_id/child_id menjaga dekorasi milik anak; API hanya mengizinkan field preferensi yang didukung, bukan mengubah progres atau hak paket.
- data_version dinaikkan secara atomik pada perubahan profil/bukti/portofolio yang memengaruhi isi atau validitas laporan. Perubahan lease_generation saja tidak mengubah data_version. Penghapusan bukti dan invalidasi snapshot/ekspor terkait memakai lock child yang sama dengan publikasi hasil laporan.

### family_invitations

Undangan pengasuh dengan token sekali pakai.

Mulai implementasi: **M3**. Entitas tunggal: `family_invitation`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `family_invitation_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `invited_by_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `accepted_by_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `email` | `text` | Tidak |  | Alamat yang dinormalisasi server untuk pencocokan/keunikan; input email tetap harus diverifikasi sesuai alur identitas. |
| `token_hash` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `expires_at` | `timestamptz` | Tidak |  |  |
| `accepted_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (token_hash)`
- `UNIQUE (family_id, email) WHERE status = 'pending'`
- `UNIQUE (family_invitation_id, family_id)`

**Aturan domain:**

- Pending dan membership caregiver aktif bersama-sama memakai dua slot. Status undangan yang kedaluwarsa diperbarui di dalam transaksi kuota; jangan memakai now() sebagai predicate index parsial.

### child_permissions

Izin per anak: ditawarkan lewat undangan atau efektif lewat membership.

Mulai implementasi: **M3**. Entitas tunggal: `child_permission`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `child_permission_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `family_membership_id` | `uuid` | Ya | FK → [family_memberships.family_membership_id](#family_memberships) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `can_learn` | `boolean` | Tidak |  |  |
| `can_view_reports` | `boolean` | Tidak |  |  |
| `can_view_portfolio` | `boolean` | Tidak |  |  |
| `can_export_reports` | `boolean` | Tidak |  |  |
| `revoked_at` | `timestamptz` | Ya |  |  |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `family_invitation_id` | `uuid` | Ya | FK → [family_invitations.family_invitation_id](#family_invitations) |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (family_membership_id, child_id) WHERE family_membership_id IS NOT NULL`
- `UNIQUE (family_invitation_id, child_id) WHERE family_invitation_id IS NOT NULL`

**CHECK per baris:**

- `num_nonnulls(family_membership_id, family_invitation_id) = 1`

**FK komposit tambahan:**

- `(family_membership_id, family_id)` → `family_memberships (family_membership_id, family_id)`.
- `(child_id, family_id)` → `children (child_id, family_id)`.
- `(family_invitation_id, family_id)` → `family_invitations (family_invitation_id, family_id)`.

**Aturan domain:**

- Tepat satu pemilik: family_invitation_id untuk penawaran, atau family_membership_id untuk izin efektif. FK komposit memeriksa anak/pemilik pada keluarga yang sama. Row undangan tidak pernah memberi akses efektif.
- Penerimaan undangan mengunci keluarga dan memindahkan/upsert izin ke membership secara atomik. Jika membership lama diaktifkan kembali, cabut izin lama yang tidak ditawarkan, upsert izin tawaran, lalu hapus row penawaran dan audit penerimaan; tidak menggabungkan hak lama diam-diam.
- Izin awal belajar/laporan true, portofolio/ekspor false. Owner memakai membership, tidak perlu row caregiver. Semua operasi memeriksa membership dan revoked_at terbaru.

### auth_sessions

Sesi login satu browser milik orang dewasa.

Mulai implementasi: **S03**. Entitas tunggal: `auth_session`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `auth_session_id` | `uuid` | Tidak | PK |  |
| `user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `family_id` | `uuid` | Ya | FK → [families.family_id](#families) |  |
| `active_child_id` | `uuid` | Ya | FK → [children.child_id](#children) |  |
| `token_hash` | `text` | Tidak |  |  |
| `mode` | `text` | Tidak |  |  |
| `adult_unlocked_until` | `timestamptz` | Ya |  |  |
| `expires_at` | `timestamptz` | Tidak |  |  |
| `revoked_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (token_hash)`

**CHECK per baris:**

- `mode IN ('adult','child')`
- `mode <> 'child' OR (family_id IS NOT NULL AND active_child_id IS NOT NULL)`

**Indeks pencarian awal:**

- `INDEX (user_id, revoked_at, expires_at)`

**FK komposit tambahan:**

- `(active_child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- Jika active_child_id terisi, FK komposit memastikan anak di family_id yang dipilih. Login baru tidak mencabut sesi browser lain.
- Seluruh tab dengan auth_session yang sama berbagi mode dan active_child_id. Start/resume/transfer memeriksa ulang mode/PIN dan lease milik browser di bawah lock auth_session; bila ada lease anak lain, wajib pause eksplisit lebih dahulu. Gagal/konflik tidak mengubah cookie/konteks atau sesi anak lain.

### login_tokens

Tautan email login/reset PIN dengan purpose dan masa berlaku.

Mulai implementasi: **S03**. Entitas tunggal: `login_token`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `login_token_id` | `uuid` | Tidak | PK |  |
| `user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `email` | `text` | Tidak |  | Alamat yang dinormalisasi server untuk pencocokan/keunikan; input email tetap harus diverifikasi sesuai alur identitas. |
| `purpose` | `text` | Tidak |  |  |
| `token_hash` | `text` | Tidak |  |  |
| `expires_at` | `timestamptz` | Tidak |  |  |
| `consumed_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (token_hash)`

**Aturan domain:**

- Konsumsi atomik sekali pakai. Secret mentah tidak disimpan pada tabel, log atau argumen queue; pengiriman asinkron memakai payload sementara terenkripsi pada processing_tasks sesuai protokol email.

### oauth_login_states

State/nonce sementara untuk callback OAuth/OIDC.

Mulai implementasi: **S03**. Entitas tunggal: `oauth_login_state`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `oauth_login_state_id` | `uuid` | Tidak | PK |  |
| `provider` | `text` | Tidak |  |  |
| `state_hash` | `text` | Tidak |  |  |
| `nonce_hash` | `text` | Tidak |  |  |
| `pkce_verifier_encrypted` | `text` | Ya |  |  |
| `return_path` | `text` | Tidak |  |  |
| `expires_at` | `timestamptz` | Tidak |  |  |
| `consumed_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (state_hash)`

**Aturan domain:**

- Return path dibatasi daftar route lokal. Kunci dekripsi PKCE hanya berada di server.

### consent_events

Jejak pemberian/penarikan persetujuan per fitur dan versi kebijakan.

Mulai implementasi: **S03**. Entitas tunggal: `consent_event`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `consent_event_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `actor_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `kind` | `text` | Tidak |  |  |
| `policy_version` | `text` | Tidak |  |  |
| `sequence` | `bigint` | Tidak |  | Urutan server per anak/jenis persetujuan; dialokasikan di bawah lock anak. |
| `action` | `text` | Tidak |  |  |
| `occurred_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, kind, sequence)`

**CHECK per baris:**

- `action IN ('grant','withdraw')`
- `sequence > 0`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- Append-only selama retensi yang disepakati. Status efektif memakai sequence terbesar per anak/kind, dialokasikan di bawah lock anak; bukan timestamp klien atau urutan acak UUID. Grant lama tidak menggantikan pemeriksaan pencabutan.
- Pembacaan status efektif memakai UNIQUE (child_id, kind, sequence) sebagai indeks pencarian; tidak menambahkan indeks urutan timestamp yang tidak dipakai untuk menentukan persetujuan terbaru.

## Kurikulum dan Konten

### courses

Mata pelajaran; Fase 1 berisi math.

Mulai implementasi: **S02**. Entitas tunggal: `course`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `course_id` | `uuid` | Tidak | PK |  |
| `code` | `text` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (code)`

### chapters

Bab dalam suatu kursus.

Mulai implementasi: **S04**. Entitas tunggal: `chapter`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `chapter_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `code` | `text` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `position` | `integer` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (course_id, code)`
- `UNIQUE (chapter_id, course_id)`

### levels

Tingkat dalam satu bab.

Mulai implementasi: **S04**. Entitas tunggal: `level`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `level_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `chapter_id` | `uuid` | Tidak | FK → [chapters.chapter_id](#chapters) |  |
| `code` | `text` | Tidak |  |  |
| `position` | `integer` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (chapter_id, code)`
- `UNIQUE (level_id, course_id)`
- `UNIQUE (level_id, chapter_id)`

**FK komposit tambahan:**

- `(chapter_id, course_id)` → `chapters (chapter_id, course_id)`.

**Aturan domain:**

- FK komposit chapter_id/course_id memastikan bab berada di kursus yang sama.

### skills

Target keterampilan matematika yang stabil.

Mulai implementasi: **S04**. Entitas tunggal: `skill`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `skill_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `code` | `text` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (course_id, code)`
- `UNIQUE (skill_id, course_id)`

### level_skills

Relasi tingkat dengan keterampilan; satu keterampilan dapat dilatih pada beberapa tingkat.

Mulai implementasi: **S04**. Entitas tunggal: `level_skill`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `level_skill_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `skill_id` | `uuid` | Tidak | FK → [skills.skill_id](#skills) |  |
| `is_required` | `boolean` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (level_id, skill_id)`
- `UNIQUE (level_skill_id, course_id)`

**FK komposit tambahan:**

- `(level_id, course_id)` → `levels (level_id, course_id)`.
- `(skill_id, course_id)` → `skills (skill_id, course_id)`.

**Aturan domain:**

- FK komposit level/course dan skill/course mencegah keterampilan lintas kursus yang salah.

### level_prerequisites

Tingkat prasyarat, termasuk antar bab dalam satu kursus.

Mulai implementasi: **S04**. Entitas tunggal: `level_prerequisite`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `level_prerequisite_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `required_level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (level_id, required_level_id)`

**CHECK per baris:**

- `level_id <> required_level_id`

**FK komposit tambahan:**

- `(level_id, course_id)` → `levels (level_id, course_id)`.
- `(required_level_id, course_id)` → `levels (level_id, course_id)`.

**Aturan domain:**

- Kedua tingkat harus satu kursus. Layanan memeriksa graf tanpa siklus di bawah lock konten kursus; CHECK tidak memeriksa siklus lintas baris.

### rule_versions

Konfigurasi aturan penempatan, mastery, atau ringkasan laporan yang dibekukan.

Mulai implementasi: **S05**. Entitas tunggal: `rule_version`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `rule_version_id` | `uuid` | Tidak | PK |  |
| `kind` | `text` | Tidak |  |  |
| `version` | `text` | Tidak |  |  |
| `parameters` | `jsonb` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (kind, version)`

**CHECK per baris:**

- `kind IN ('placement', 'mastery', 'reporting')`

**Aturan domain:**

- Contoh version: placement-v0.1, mastery-v0.1. Parameter JSON tidak menyimpan FK/domain identity. Versi yang dirujuk tidak diubah; versi baru melalui evaluasi.
- Layanan memeriksa kind sesuai peran FK: placement, mastery, atau reporting; keberadaan UUID aturan saja tidak memverifikasi jenis aturan.

### rubric_versions

Rubrik objektif/observasi berversi.

Mulai implementasi: **S04**. Entitas tunggal: `rubric_version`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `rubric_version_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `code` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `criteria` | `jsonb` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (course_id, code, version)`
- `UNIQUE (rubric_version_id, course_id)`

**CHECK per baris:**

- `version > 0`

**Aturan domain:**

- Rubrik yang dirujuk versi konten immutable.

### activity_templates

Versi renderer/generator dan schema parameter suatu templat.

Mulai implementasi: **S04**. Entitas tunggal: `activity_template`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `activity_template_id` | `uuid` | Tidak | PK |  |
| `code` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `interaction_kind` | `text` | Tidak |  |  |
| `parameter_schema` | `jsonb` | Tidak |  |  |
| `generator_version` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (code, version)`

**CHECK per baris:**

- `version > 0`

**Aturan domain:**

- Templat yang dirujuk konten terbit immutable; perubahan renderer/generator direkam sebagai versi baru.

### product_assets

Aset produk orisinal/berizin; terpisah dari media pribadi anak.

Mulai implementasi: **S04**. Entitas tunggal: `product_asset`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `product_asset_id` | `uuid` | Tidak | PK |  |
| `code` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `kind` | `text` | Tidak |  |  |
| `locale` | `text` | Tidak |  |  |
| `storage_key` | `text` | Tidak |  |  |
| `delivery_policy` | `text` | Tidak |  |  |
| `checksum` | `text` | Tidak |  |  |
| `license_note` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (code, locale, version)`

**CHECK per baris:**

- `version > 0`

**Aturan domain:**

- Gambar/audio/video produk dapat publik sesuai kebijakan; PDF worksheet/kunci melewati endpoint hak fitur dan tidak memakai URL publik permanen.
- Versi aset yang dirujuk konten direview/terbit bersifat immutable, termasuk storage_key, checksum, lisensi dan kebijakan delivery. Objek storage tidak ditimpa pada key/versi yang sama; perubahan byte atau metadata yang memengaruhi review membuat versi aset baru.

### activity_versions

Konten, parameter, kunci server dan rubrik pada suatu revisi/versi.

Mulai implementasi: **S04**. Entitas tunggal: `activity_version`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `activity_version_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `code` | `text` | Tidak |  |  |
| `kind` | `text` | Tidak |  |  |
| `level_skill_id` | `uuid` | Tidak | FK → [level_skills.level_skill_id](#level_skills) |  |
| `activity_template_id` | `uuid` | Tidak | FK → [activity_templates.activity_template_id](#activity_templates) |  |
| `rubric_version_id` | `uuid` | Tidak | FK → [rubric_versions.rubric_version_id](#rubric_versions) |  |
| `content_generation_id` | `uuid` | Ya | FK → [content_generations.content_generation_id](#content_generations) | Ditambahkan pada M3. |
| `locale` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `revision` | `integer` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `instruction` | `text` | Tidak |  |  |
| `content` | `jsonb` | Tidak |  |  |
| `grading_config` | `jsonb` | Tidak |  |  |
| `content_hash` | `text` | Tidak |  |  |
| `published_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (course_id, code, locale, version)`
- `UNIQUE (activity_version_id, level_skill_id)`
- `UNIQUE (activity_version_id, rubric_version_id)`

**CHECK per baris:**

- `version > 0`
- `revision > 0`
- `status IN ('draft', 'in_review', 'published', 'archived')`
- `kind IN ('core','supporting','olympiad')`

**FK komposit tambahan:**

- `(level_skill_id, course_id)` → `level_skills (level_skill_id, course_id)`.
- `(rubric_version_id, course_id)` → `rubric_versions (rubric_version_id, course_id)`.

**Aturan domain:**

- FK komposit memeriksa level_skill dan rubric pada course yang sama. Content publik dan grading_config privat dipisahkan; kind/target/locale/code termasuk definisi yang direview.
- Published/archived mempertahankan isi immutable; draf boleh direvisi dengan revision dan hash baru yang membatalkan review sebelumnya.
- content_hash mencakup definisi efektif yang dinormalisasi: teks/payload/locale, konfigurasi penilaian, serta versi templat/rubrik/aset yang digunakan beserta checksum. Binding aset dan referensi video/PDF/kunci termasuk isi yang direview. Perubahan dependensi draf mengunci induk, menaikkan revision, menghitung hash baru dan membatalkan review; dependensi versi terbit tidak diubah.
- Identitas seri konten adalah (course_id, code); locale/version membedakan edisi. Seluruh FK belajar merujuk activity_version_id yang immutable, bukan identitas aktivitas terpisah.

### activity_version_assets

Aset produk yang dirujuk versi aktivitas.

Mulai implementasi: **S04**. Entitas tunggal: `activity_version_asset`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `activity_version_asset_id` | `uuid` | Tidak | PK |  |
| `activity_version_id` | `uuid` | Tidak | FK → [activity_versions.activity_version_id](#activity_versions) |  |
| `product_asset_id` | `uuid` | Tidak | FK → [product_assets.product_asset_id](#product_assets) |  |
| `role` | `text` | Tidak |  |  |
| `position` | `integer` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (activity_version_id, role, position)`

**Aturan domain:**

- Insert/update/delete binding melalui layanan revisi induk di bawah lock induk; perubahan menaikkan revision/hash dan menggugurkan review. Binding versi published/archived tidak dapat diubah; buat versi induk baru.

### chapter_introductions

Versi pengantar bab interaktif/video opsional.

Mulai implementasi: **M5**. Entitas tunggal: `chapter_introduction`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `chapter_introduction_id` | `uuid` | Tidak | PK |  |
| `chapter_id` | `uuid` | Tidak | FK → [chapters.chapter_id](#chapters) |  |
| `video_product_asset_id` | `uuid` | Ya | FK → [product_assets.product_asset_id](#product_assets) |  |
| `locale` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `revision` | `integer` | Tidak |  |  |
| `slides` | `jsonb` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `content_hash` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (chapter_id, locale, version)`

**CHECK per baris:**

- `status IN ('draft', 'in_review', 'published', 'archived')`
- `version > 0`
- `revision > 0`

**Aturan domain:**

- Slides menyimpan struktur/teks dan key role/urutan lokal; identitas aset memakai chapter_introduction_assets. Versi/revisi/hash dan review target tetap wajib.
- content_hash mencakup definisi efektif yang dinormalisasi: teks/payload/locale, konfigurasi penilaian, serta versi templat/rubrik/aset yang digunakan beserta checksum. Binding aset dan referensi video/PDF/kunci termasuk isi yang direview. Perubahan dependensi draf mengunci induk, menaikkan revision, menghitung hash baru dan membatalkan review; dependensi versi terbit tidak diubah.

### chapter_introduction_assets

Aset produk yang dirujuk layar pengantar bab.

Mulai implementasi: **M5**. Entitas tunggal: `chapter_introduction_asset`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `chapter_introduction_asset_id` | `uuid` | Tidak | PK |  |
| `chapter_introduction_id` | `uuid` | Tidak | FK → [chapter_introductions.chapter_introduction_id](#chapter_introductions) |  |
| `product_asset_id` | `uuid` | Tidak | FK → [product_assets.product_asset_id](#product_assets) |  |
| `role` | `text` | Tidak |  |  |
| `position` | `integer` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (chapter_introduction_id, role, position)`

**Aturan domain:**

- slides pada pengantar hanya memakai role/position lokal; identitas asset tersimpan pada relasi ini.
- Insert/update/delete binding melalui layanan revisi induk di bawah lock induk; perubahan menaikkan revision/hash dan menggugurkan review. Binding versi published/archived tidak dapat diubah; buat versi induk baru.

### worksheet_versions

Versi PDF worksheet dan kunci orang tua.

Mulai implementasi: **M4**. Entitas tunggal: `worksheet_version`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `worksheet_version_id` | `uuid` | Tidak | PK |  |
| `level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `worksheet_product_asset_id` | `uuid` | Tidak | FK → [product_assets.product_asset_id](#product_assets) |  |
| `answer_key_product_asset_id` | `uuid` | Ya | FK → [product_assets.product_asset_id](#product_assets) |  |
| `code` | `text` | Tidak |  |  |
| `locale` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `revision` | `integer` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `content_hash` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (level_id, code, locale, version)`

**CHECK per baris:**

- `status IN ('draft', 'in_review', 'published', 'archived')`
- `version > 0`
- `revision > 0`

**Aturan domain:**

- content_hash mencakup definisi efektif yang dinormalisasi: teks/payload/locale, konfigurasi penilaian, serta versi templat/rubrik/aset yang digunakan beserta checksum. Binding aset dan referensi video/PDF/kunci termasuk isi yang direview. Perubahan dependensi draf mengunci induk, menaikkan revision, menghitung hash baru dan membatalkan review; dependensi versi terbit tidak diubah.

### content_reviews

Persetujuan/penolakan manusia terhadap tepat satu target konten.

Mulai implementasi: **S04**. Entitas tunggal: `content_review`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `content_review_id` | `uuid` | Tidak | PK |  |
| `activity_version_id` | `uuid` | Ya | FK → [activity_versions.activity_version_id](#activity_versions) |  |
| `chapter_introduction_id` | `uuid` | Ya | FK → [chapter_introductions.chapter_introduction_id](#chapter_introductions) | Ditambahkan pada M5. |
| `worksheet_version_id` | `uuid` | Ya | FK → [worksheet_versions.worksheet_version_id](#worksheet_versions) | Ditambahkan pada M4. |
| `reviewer_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `revision` | `integer` | Tidak |  |  |
| `content_hash` | `text` | Tidak |  |  |
| `decision` | `text` | Tidak |  |  |
| `notes` | `text` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**CHECK per baris:**

- `num_nonnulls(activity_version_id, chapter_introduction_id, worksheet_version_id) = 1`
- `decision IN ('approved','changes_requested','rejected')`
- `revision > 0`

**Aturan domain:**

- Tiga target adalah FK nyata dengan CHECK tepat satu, bukan target_type/target_id tanpa integritas. Publish memeriksa review target/revisi/hash yang sama dan peran peninjau di bawah lock.
- Penyimpanan review dan layanan publish tersedia S04 sebelum uji keluarga S08; UI editorial lengkap masuk S09. Seed lokal sintetis tidak menjadi bukti persetujuan pendidik untuk konten uji keluarga.

## Sesi dan Pembelajaran

### placement_runs

Satu run tes penempatan yang bisa melintasi beberapa sesi belajar.

Mulai implementasi: **S05**. Entitas tunggal: `placement_run`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `placement_run_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `placement_rule_version_id` | `uuid` | Tidak | FK → [rule_versions.rule_version_id](#rule_versions) |  |
| `status` | `text` | Tidak |  |  |
| `age_years` | `smallint` | Tidak |  |  |
| `completed_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, course_id) WHERE status = 'in_progress'`
- `UNIQUE (placement_run_id, child_id, course_id)`

**CHECK per baris:**

- `status IN ('in_progress', 'completed', 'superseded')`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- Run berbeda dari study_session. Tes ulang supersede run lama; berhenti sebagian tetap menyimpan probe dan bukti.

### placement_probes

Probe suatu tingkat pada run penempatan.

Mulai implementasi: **S05**. Entitas tunggal: `placement_probe`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `placement_probe_id` | `uuid` | Tidak | PK |  |
| `placement_run_id` | `uuid` | Tidak | FK → [placement_runs.placement_run_id](#placement_runs) |  |
| `level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `status` | `text` | Tidak |  |  |
| `outcome` | `text` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (placement_run_id, level_id)`

**Aturan domain:**

- Kursus tingkat sama dengan run. Respons valid/count berasal dari instans/attempt; skip/teknis tidak menjadi salah.

### study_sessions

Sesi anak dengan purpose, posisi, dan rencana yang dapat dilanjutkan.

Mulai implementasi: **S05**. Entitas tunggal: `study_session`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `study_session_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `started_by_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `placement_run_id` | `uuid` | Ya | FK → [placement_runs.placement_run_id](#placement_runs) |  |
| `mastery_rule_version_id` | `uuid` | Tidak | FK → [rule_versions.rule_version_id](#rule_versions) |  |
| `purpose` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `plan_snapshot` | `jsonb` | Tidak |  |  |
| `last_interacted_at` | `timestamptz` | Tidak |  |  |
| `completed_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id) WHERE status IN ('active','paused')`
- `UNIQUE (study_session_id, child_id, course_id)`
- `UNIQUE (study_session_id, child_id)`

**CHECK per baris:**

- `purpose IN ('placement','practice','olympiad')`
- `(purpose = 'placement') = (placement_run_id IS NOT NULL)`
- `status IN ('active', 'paused', 'completed', 'ended')`

**Indeks pencarian awal:**

- `INDEX (child_id, course_id, created_at)`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.
- `(placement_run_id, child_id, course_id)` → `placement_runs (placement_run_id, child_id, course_id)`.

**Aturan domain:**

- FK komposit memeriksa child/family dan placement_run/child/course. Latihan membekukan rencana; penempatan membekukan aturan dan probe yang telah disajikan, bukan memprediksi jawaban probe berikutnya.

### learning_leases

Riwayat pemberian lease perangkat; paling banyak satu aktif per anak.

Mulai implementasi: **S05**. Entitas tunggal: `learning_lease`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `learning_lease_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `study_session_id` | `uuid` | Tidak | FK → [study_sessions.study_session_id](#study_sessions) |  |
| `auth_session_id` | `uuid` | Tidak | FK → [auth_sessions.auth_session_id](#auth_sessions) |  |
| `generation` | `bigint` | Tidak |  |  |
| `token_hash` | `text` | Tidak |  |  |
| `revoked_at` | `timestamptz` | Ya |  |  |
| `revocation_reason` | `text` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, generation)`
- `UNIQUE (token_hash)`
- `UNIQUE (child_id) WHERE revoked_at IS NULL`
- `UNIQUE (auth_session_id) WHERE revoked_at IS NULL`

**CHECK per baris:**

- `generation > 0`
- `(revoked_at IS NULL) = (revocation_reason IS NULL)`

**Indeks pencarian awal:**

- `INDEX (study_session_id, generation)`

**FK komposit tambahan:**

- `(study_session_id, child_id)` → `study_sessions (study_session_id, child_id)`.

**Aturan domain:**

- FK komposit study_session_id/child_id mengikat lease ke sesi anak. Rotasi menutup baris lama dan menambah baris baru di bawah lock child; riwayat hash mendukung recovery terminal dan audit.
- Generasi baru dialokasikan dengan menaikkan children.lease_generation secara atomik di bawah lock anak; tidak menghitung ulang dari MAX riwayat yang dapat dibersihkan. Rotasi secret recovery pada lease yang sama tidak menaikkan generasi.
- Satu auth_session hanya mempunyai satu lease aktif. Mutasi dari browser mengunci auth_sessions penerima lebih dahulu, lalu child, study_session/lease, dan instans. Transfer tidak mengubah/mengunci auth_sessions pemilik lama setelah child dikunci; perangkat lama mengetahui revocation pada request berikutnya.

### responses

Respons berurutan; koreksi tidak menimpa respons pertama.

Mulai implementasi: **S06**. Entitas tunggal: `response`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `response_id` | `uuid` | Tidak | PK |  |
| `attempt_id` | `uuid` | Tidak | FK → [attempts.attempt_id](#attempts) |  |
| `sequence` | `integer` | Tidak |  |  |
| `answer` | `jsonb` | Tidak |  |  |
| `answer_hash` | `text` | Tidak |  |  |
| `is_correct` | `boolean` | Ya |  |  |
| `submitted_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (attempt_id, sequence)`
- `UNIQUE (response_id, attempt_id)`

**CHECK per baris:**

- `sequence > 0`

**Aturan domain:**

- Urutan diperiksa saat transaksi instans dikunci. Payload dibatasi schema respons; tidak memuat skor/klaim kebenaran tepercaya dari klien.
- sequence 1 adalah jawaban pertama. Setelah respons ini disimpan, first_response_id pada attempt tidak boleh berpindah ke sequence berikutnya. Jawaban koreksi memakai sequence baru; retry sequence yang sama memverifikasi answer_hash.

### attempts

Satu penyajian soal dan hasil logisnya; banyak respons koreksi tetap satu attempt.

Mulai implementasi: **S05**. Entitas tunggal: `attempt`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `attempt_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `study_session_id` | `uuid` | Tidak | FK → [study_sessions.study_session_id](#study_sessions) |  |
| `activity_version_id` | `uuid` | Tidak | FK → [activity_versions.activity_version_id](#activity_versions) |  |
| `level_skill_id` | `uuid` | Tidak | FK → [level_skills.level_skill_id](#level_skills) |  |
| `rubric_version_id` | `uuid` | Tidak | FK → [rubric_versions.rubric_version_id](#rubric_versions) |  |
| `placement_probe_id` | `uuid` | Ya | FK → [placement_probes.placement_probe_id](#placement_probes) |  |
| `position` | `integer` | Tidak |  |  |
| `seed` | `text` | Tidak |  |  |
| `parameters` | `jsonb` | Tidak |  |  |
| `grading_snapshot` | `jsonb` | Tidak |  |  |
| `fingerprint` | `text` | Tidak |  |  |
| `evidence_kind` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `presented_at` | `timestamptz` | Ya |  |  |
| `completed_at` | `timestamptz` | Ya |  |  |
| `first_response_id` | `uuid` | Ya | FK → [responses.response_id](#responses) | Ditambahkan pada S06. |
| `first_response_correct` | `boolean` | Ya |  | Ditambahkan pada S06. |
| `independent_correct` | `boolean` | Ya |  | Ditambahkan pada S06. |
| `hint_count` | `integer` | Tidak |  | Ditambahkan pada S06. |
| `was_assisted` | `boolean` | Tidak |  | Ditambahkan pada S06. |
| `evaluated_at` | `timestamptz` | Ya |  | Ditambahkan pada S06. |
| `evaluation_status` | `text` | Tidak |  | pending/evaluated/not_applicable/failed; terpisah dari status penyelesaian soal. Default pending. Ditambahkan pada S06. |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (study_session_id, position)`
- `UNIQUE (attempt_id, study_session_id)`
- `UNIQUE (attempt_id, child_id, course_id)`

**CHECK per baris:**

- `position > 0`
- `status IN ('presented', 'in_progress', 'completed', 'skipped', 'technical_failure')`
- `hint_count >= 0`
- `evaluation_status IN ('pending','evaluated','not_applicable','failed')`

**Indeks pencarian awal:**

- `INDEX (child_id, course_id, evaluated_at)`

**FK komposit tambahan:**

- `(activity_version_id, level_skill_id)` → `activity_versions (activity_version_id, level_skill_id)`.
- `(activity_version_id, rubric_version_id)` → `activity_versions (activity_version_id, rubric_version_id)`.
- `(child_id, family_id)` → `children (child_id, family_id)`.
- `(first_response_id, attempt_id)` → `responses (response_id, attempt_id)`.
- `(study_session_id, child_id, course_id)` → `study_sessions (study_session_id, child_id, course_id)`.

**Aturan domain:**

- FK komposit memeriksa pasangan activity_version/level_skill dan activity_version/rubric. Probe bila terisi harus milik run pada sesi; pemeriksaan ini dilakukan layanan/trigger constraint.
- Public render DTO tidak membawa grading_snapshot. Instans yang telah disajikan tidak diganti saat reload/hasil AI terlambat.
- FK komposit first_response_id/attempt_id menjaga respons pertama milik attempt. FK komposit study_session_id/child_id/course_id serta child_id/family_id menjaga konteks keluarga/kursus.
- Mastery hanya memakai instans terminal objektif yang evaluated; pending/skip/teknis/karya terbuka dikecualikan sesuai rubrik.
- Attempt dibuat S05 saat soal disiapkan. S06 menambahkan respons/hasil; first_response merujuk respons sequence 1 pada attempt yang sama dan tidak ditimpa. Completed menyatakan aktivitas selesai; hasil benar/salah hanya tersedia setelah evaluation_status evaluated. Lewat/teknis/karya terbuka/pending tidak menjadi bukti objektif mastery.

### study_session_events

Interaksi, hint, bantuan, pause, atau skip yang relevan pada sesi.

Mulai implementasi: **S06**. Entitas tunggal: `study_session_event`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `study_session_event_id` | `uuid` | Tidak | PK |  |
| `study_session_id` | `uuid` | Tidak | FK → [study_sessions.study_session_id](#study_sessions) |  |
| `attempt_id` | `uuid` | Ya | FK → [attempts.attempt_id](#attempts) |  |
| `actor_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `kind` | `text` | Tidak |  |  |
| `data` | `jsonb` | Tidak |  |  |
| `occurred_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Indeks pencarian awal:**

- `INDEX (study_session_id, occurred_at)`

**FK komposit tambahan:**

- `(attempt_id, study_session_id)` → `attempts (attempt_id, study_session_id)`.

**Aturan domain:**

- FK komposit instans/sesi bila ada. Client mengirim study_session_event_id untuk deduplikasi; server memvalidasi konteks dan kind. Polling bukan interaksi belajar.

### mastery_states

Status penguasaan anak pada keterampilan-tingkat dan versi aturan.

Mulai implementasi: **S07**. Entitas tunggal: `mastery_state`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `mastery_state_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `level_skill_id` | `uuid` | Tidak | FK → [level_skills.level_skill_id](#level_skills) |  |
| `mastery_rule_version_id` | `uuid` | Tidak | FK → [rule_versions.rule_version_id](#rule_versions) |  |
| `status` | `text` | Tidak |  |  |
| `rubric_version_id` | `uuid` | Tidak | FK → [rubric_versions.rubric_version_id](#rubric_versions) |  |
| `first_mastered_at` | `timestamptz` | Ya |  |  |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, level_skill_id, mastery_rule_version_id, rubric_version_id)`

**FK komposit tambahan:**

- `(level_skill_id, course_id)` → `level_skills (level_skill_id, course_id)`.
- `(rubric_version_id, course_id)` → `rubric_versions (rubric_version_id, course_id)`.

**Aturan domain:**

- Kursus level_skill sama dengan course_id. Jendela bukti diquery dari attempts; tidak menyimpan array ID bukti sebagai satu-satunya relasi.
- Jendela bukti dan state dipisahkan juga per rubric_version_id. Pencapaian versi lama tetap historis; rubrik baru tidak mewarisi status penguasaan tanpa evaluasi/migrasi yang disetujui.

### mastery_changes

Keputusan perubahan titik latihan dengan alasan dan bukti.

Mulai implementasi: **S07**. Entitas tunggal: `mastery_change`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `mastery_change_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `from_level_id` | `uuid` | Ya | FK → [levels.level_id](#levels) |  |
| `to_level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `rule_version_id` | `uuid` | Ya | FK → [rule_versions.rule_version_id](#rule_versions) |  |
| `study_session_id` | `uuid` | Ya | FK → [study_sessions.study_session_id](#study_sessions) |  |
| `actor_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `kind` | `text` | Tidak |  |  |
| `reason` | `text` | Tidak |  | Identifier alasan domain sesuai kontrak endpoint; penamaan singkat tidak mengubah aturan validasi nilainya. |
| `decision_key` | `text` | Tidak |  |  |
| `effective_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, decision_key)`
- `UNIQUE (mastery_change_id, child_id, course_id)`

**Indeks pencarian awal:**

- `INDEX (child_id, course_id, created_at)`

**FK komposit tambahan:**

- `(from_level_id, course_id)` → `levels (level_id, course_id)`.
- `(to_level_id, course_id)` → `levels (level_id, course_id)`.
- `(study_session_id, child_id, course_id)` → `study_sessions (study_session_id, child_id, course_id)`.

**Aturan domain:**

- Source session bila ada harus anak/kursus yang sama. Parent override memiliki actor dan alasan, tetapi tidak menciptakan bukti mastery.
- Target prasyarat boleh pada bab lain dalam kursus yang sama; perbarui chapter_progress bab target, jangan memasang level pada bab yang salah.

### mastery_change_evidence

Bukti relasional untuk satu perubahan tingkat.

Mulai implementasi: **S07**. Entitas tunggal: `mastery_change_evidence`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `mastery_change_evidence_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `mastery_change_id` | `uuid` | Tidak | FK → [mastery_changes.mastery_change_id](#mastery_changes) |  |
| `attempt_id` | `uuid` | Tidak | FK → [attempts.attempt_id](#attempts) |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (mastery_change_id, attempt_id)`

**FK komposit tambahan:**

- `(mastery_change_id, child_id, course_id)` → `mastery_changes (mastery_change_id, child_id, course_id)`.
- `(attempt_id, child_id, course_id)` → `attempts (attempt_id, child_id, course_id)`.

**Aturan domain:**

- FK komposit ke change dan attempt memakai child_id/course_id yang sama.

### chapter_progress

Posisi latihan bab dan target perubahan sesi berikutnya.

Mulai implementasi: **S07**. Entitas tunggal: `chapter_progress`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `chapter_progress_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `chapter_id` | `uuid` | Tidak | FK → [chapters.chapter_id](#chapters) |  |
| `current_level_id` | `uuid` | Tidak | FK → [levels.level_id](#levels) |  |
| `pending_mastery_change_id` | `uuid` | Ya | FK → [mastery_changes.mastery_change_id](#mastery_changes) |  |
| `status` | `text` | Tidak |  |  |
| `updated_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, chapter_id)`

**FK komposit tambahan:**

- `(chapter_id, course_id)` → `chapters (chapter_id, course_id)`.
- `(current_level_id, chapter_id)` → `levels (level_id, chapter_id)`.
- `(pending_mastery_change_id, child_id, course_id)` → `mastery_changes (mastery_change_id, child_id, course_id)`.

**Aturan domain:**

- FK komposit menjaga current_level_id milik chapter_id dan chapter dalam course_id. Pending change harus milik anak/kursus; jika target beda bab, pemilihan sesi mengarahkan ke progress bab target.

### reward_items

Katalog stiker/koleksi yang diterbitkan produk.

Mulai implementasi: **S06**. Entitas tunggal: `reward_item`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `reward_item_id` | `uuid` | Tidak | PK |  |
| `code` | `text` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `product_asset_id` | `uuid` | Tidak | FK → [product_assets.product_asset_id](#product_assets) |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (code)`

### reward_grants

Pemberian satu item dengan sumber instans atau sesi yang unik.

Mulai implementasi: **S06**. Entitas tunggal: `reward_grant`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `reward_grant_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `reward_item_id` | `uuid` | Tidak | FK → [reward_items.reward_item_id](#reward_items) |  |
| `attempt_id` | `uuid` | Ya | FK → [attempts.attempt_id](#attempts) |  |
| `study_session_id` | `uuid` | Ya | FK → [study_sessions.study_session_id](#study_sessions) |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (child_id, attempt_id, reward_item_id) WHERE attempt_id IS NOT NULL`
- `UNIQUE (child_id, study_session_id, reward_item_id) WHERE study_session_id IS NOT NULL`
- `UNIQUE (reward_grant_id, child_id)`

**CHECK per baris:**

- `num_nonnulls(attempt_id, study_session_id) = 1`

**Aturan domain:**

- Sumber harus milik anak; pemeriksaan sumber lintas instans/sesi di bawah lock yang sama dengan penyelesaian.

## Media dan Laporan

### media_assets

Media pribadi anak, terpisah dari product_assets.

Mulai implementasi: **S09/M3**. Entitas tunggal: `media_asset`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `media_asset_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `attempt_id` | `uuid` | Ya | FK → [attempts.attempt_id](#attempts) |  |
| `uploaded_by_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `consent_event_id` | `uuid` | Ya | FK → [consent_events.consent_event_id](#consent_events) |  |
| `kind` | `text` | Tidak |  |  |
| `provider_asset_key` | `text` | Tidak |  |  |
| `resource_type` | `text` | Tidak |  |  |
| `delivery_type` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `byte_size` | `bigint` | Tidak |  |  |
| `duration_ms` | `integer` | Ya |  |  |
| `portfolio_saved_at` | `timestamptz` | Ya |  |  |
| `deleted_at` | `timestamptz` | Ya |  |  |
| `variants` | `jsonb` | Tidak |  | Default []; daftar varian milik media: name, provider_asset_key, transform, status. Tidak mempunyai ID domain/izin terpisah. Ditambahkan pada M3. |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (provider_asset_key)`

**CHECK per baris:**

- `byte_size >= 0`
- `delivery_type = 'authenticated'`
- `jsonb_typeof(variants) = 'array'`

**Indeks pencarian awal:**

- `INDEX (child_id, status, created_at)`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- Instans dan grant consent bila ada harus milik anak/keluarga yang sama. Persetujuan lama tidak menggantikan izin efektif terbaru. Provider key bukan URL publik atau API secret.
- Varian adalah daftar terstruktur milik media; nama unik per media divalidasi server. Original dan semua varian tetap authenticated. Update daftar diserialisasi dengan penghapusan di bawah lock media/child; job penghapusan menyimpan seluruh key remote sebelum row dibersihkan.

### observations

Pengamatan rubrik atas suatu attempt, deterministik/manusia/AI.

Mulai implementasi: **M3**. Entitas tunggal: `observation`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `observation_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `attempt_id` | `uuid` | Tidak | FK → [attempts.attempt_id](#attempts) |  |
| `rubric_version_id` | `uuid` | Tidak | FK → [rubric_versions.rubric_version_id](#rubric_versions) |  |
| `processing_task_id` | `uuid` | Ya | FK → [processing_tasks.processing_task_id](#processing_tasks) |  |
| `domain` | `text` | Tidak |  |  |
| `source` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `data` | `jsonb` | Tidak |  |  |
| `model_config` | `jsonb` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (observation_id, attempt_id)`

**Aturan domain:**

- Attempt harus milik child; rubrik/model yang dipakai dicatat. Karya terbuka tidak diubah menjadi skor benar/salah atau diagnosis.

### report_snapshots

Snapshot laporan berbukti untuk ekspor/riwayat; ringkasan M1 boleh query langsung.

Mulai implementasi: **M3**. Entitas tunggal: `report_snapshot`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `report_snapshot_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Tidak | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `report_rule_version_id` | `uuid` | Tidak | FK → [rule_versions.rule_version_id](#rule_versions) |  |
| `period_started_at` | `timestamptz` | Tidak |  |  |
| `period_ended_at` | `timestamptz` | Tidak |  |  |
| `child_data_version` | `bigint` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `snapshot_key` | `text` | Tidak |  |  |
| `claims` | `jsonb` | Tidak |  | Default []; daftar {key, text, evidence_status}. Key lokal unik per snapshot, tanpa array ID attempt tersembunyi. |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (snapshot_key)`
- `UNIQUE (report_snapshot_id, child_id, course_id)`

**CHECK per baris:**

- `period_ended_at > period_started_at`
- `jsonb_typeof(claims) = 'array'`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- Snapshot dibangun dari bukti valid; penghapusan bukti membatalkan snapshot/ekspor terdampak lalu membangun ulang, bukan membiarkan klaim tanpa sumber.
- Bangun snapshot dari satu pembacaan konsisten, misalnya transaksi REPEATABLE READ yang singkat, sambil merekam children.data_version. Finalisasi mengunci child dan hanya menandai ready bila data_version masih sama serta bukti masih sah; snapshot invalidated tidak dipulihkan menjadi ready oleh retry lama. Versi baru membuat snapshot baru.
- Klaim dimiliki snapshot dan tidak mempunyai lifecycle/API edit sendiri. Server memvalidasi key unik serta klaim sufficient memiliki report_evidence yang sah sebelum ready; snapshot siap immutable selain status invalidasi.
- Klaim dan report_evidence ditulis dalam transaksi penyusunan snapshot; key lokal di claims harus unik dan seluruh evidence harus menunjuk key yang ada. Array claims tidak menyimpan UUID attempt/anak/aset sebagai referensi tersembunyi.

### report_evidence

Relasi bukti attempt/pengamatan ke key klaim lokal pada snapshot.

Mulai implementasi: **M3**. Entitas tunggal: `report_evidence`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `report_evidence_id` | `uuid` | Tidak | PK |  |
| `report_snapshot_id` | `uuid` | Tidak | FK → [report_snapshots.report_snapshot_id](#report_snapshots) |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `claim_key` | `text` | Tidak |  |  |
| `attempt_id` | `uuid` | Tidak | FK → [attempts.attempt_id](#attempts) |  |
| `observation_id` | `uuid` | Ya | FK → [observations.observation_id](#observations) |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (report_snapshot_id, claim_key, attempt_id)`

**FK komposit tambahan:**

- `(observation_id, attempt_id)` → `observations (observation_id, attempt_id)`.
- `(report_snapshot_id, child_id, course_id)` → `report_snapshots (report_snapshot_id, child_id, course_id)`.
- `(attempt_id, child_id, course_id)` → `attempts (attempt_id, child_id, course_id)`.

**Aturan domain:**

- FK komposit memastikan snapshot, attempt, dan pengamatan berada pada anak/kursus/attempt yang sama. claim_key harus ada pada claims snapshot; ini key lokal dokumen yang divalidasi layanan, bukan FK ke entitas klaim terpisah.

### report_exports

Permintaan PDF sementara per snapshot dan pengguna.

Mulai implementasi: **M3**. Entitas tunggal: `report_export`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `report_export_id` | `uuid` | Tidak | PK |  |
| `report_snapshot_id` | `uuid` | Tidak | FK → [report_snapshots.report_snapshot_id](#report_snapshots) |  |
| `requested_by_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `include_artwork` | `boolean` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `output_storage_key` | `text` | Ya |  |  |
| `expires_at` | `timestamptz` | Ya |  |  |
| `cancelled_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Aturan domain:**

- Jika requester caregiver, include_artwork selalu false dan izin ekspor diperlukan. Key output bukan URL publik; worker dan endpoint unduh memeriksa izin terbaru.
- Gotenberg dijalankan tanpa menahan transaksi database. Output belum dapat diunduh selama rendering. Publikasi memakai transaksi singkat: lock child lalu snapshot/ekspor, periksa ulang data_version, izin requester dan status belum dibatalkan; transisi kondisional ke ready hanya bila semua sah. Jika gagal, hapus output sementara dan jangan menghidupkan kembali task cancelled. Endpoint unduh mengulangi pemeriksaan validitas snapshot, izin, expiry dan status ready.

## Paket dan Pembayaran

### plan_versions

Versi harga, durasi dan fitur paket yang dibekukan.

Mulai implementasi: **M4**. Entitas tunggal: `plan_version`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `plan_version_id` | `uuid` | Tidak | PK |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `code` | `text` | Tidak |  |  |
| `title` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `version` | `integer` | Tidak |  |  |
| `duration_months` | `smallint` | Tidak |  |  |
| `price` | `bigint` | Tidak |  |  |
| `currency` | `text` | Tidak |  |  |
| `includes_worksheets` | `boolean` | Tidak |  |  |
| `includes_olympiad` | `boolean` | Tidak |  |  |
| `effective_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (course_id, code, version)`
- `UNIQUE (course_id, code) WHERE status = 'active'`
- `UNIQUE (plan_version_id, course_id)`

**CHECK per baris:**

- `price >= 0`
- `duration_months IN (1,3,12)`
- `currency = 'IDR'`
- `status IN ('draft','active','archived')`

**Aturan domain:**

- Harga/fitur yang telah dirujuk order tidak diubah. D7 menetapkan nominal nyata; tidak memakai float untuk rupiah.
- Seri paket dikenali lewat course_id/code; version membekukan penawaran. Checkout baru hanya memakai versi active yang sudah efektif. Mengarsipkan penawaran tidak mengubah hak dari order/segmen lama yang sah. Perubahan harga/fitur/title membuat versi baru; status ketersediaan jual dapat berubah melalui layanan.

### orders

Pesanan satu anak/kursus dan satu versi paket.

Mulai implementasi: **M4**. Entitas tunggal: `order`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `order_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Ya | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Ya | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `buyer_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `plan_version_id` | `uuid` | Tidak | FK → [plan_versions.plan_version_id](#plan_versions) |  |
| `reference` | `text` | Tidak |  |  |
| `amount_due` | `bigint` | Tidak |  |  |
| `currency` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `expires_at` | `timestamptz` | Ya |  |  |
| `paid_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (reference)`

**CHECK per baris:**

- `amount_due >= 0`
- `currency = 'IDR'`
- `child_id IS NULL OR family_id IS NOT NULL`

**Indeks pencarian awal:**

- `INDEX (family_id, created_at)`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.
- `(plan_version_id, course_id)` → `plan_versions (plan_version_id, course_id)`.

**Aturan domain:**

- Pembuatan order wajib family/child/buyer non-null dan valid; nullable hanya untuk anonimisasi/hard-delete yang dibenarkan D2/D7 pada catatan terminal.
- Kursus paket, anak keluarga, snapshot harga dan buyer owner diperiksa dalam transaksi. Pesanan pending harus ditutup sebelum target profil dihapus.
- Saat penghapusan profil disahkan, child_id boleh dikosongkan sementara family_id dipertahankan untuk riwayat keluarga. Family_id hanya dikosongkan ketika penghapusan/anonimisasi keluarga disahkan; bukan otomatis saat satu anak dihapus.

### payment_transactions

Percobaan transaksi gateway untuk suatu order.

Mulai implementasi: **M4**. Entitas tunggal: `payment_transaction`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `payment_transaction_id` | `uuid` | Tidak | PK |  |
| `order_id` | `uuid` | Tidak | FK → [orders.order_id](#orders) |  |
| `provider` | `text` | Tidak |  |  |
| `provider_transaction_id` | `text` | Ya |  | Identitas penyedia/SDK, bukan FK domain. |
| `provider_session_id` | `text` | Ya |  | Identitas penyedia/SDK, bukan FK domain. |
| `status` | `text` | Tidak |  |  |
| `verified_amount` | `bigint` | Ya |  |  |
| `verified_currency` | `text` | Ya |  |  |
| `verified_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (provider, provider_transaction_id) WHERE provider_transaction_id IS NOT NULL`

**Aturan domain:**

- Provider ID adalah opaque text eksternal. Beberapa transaksi boleh terkait satu order; aktivasi akses tetap paling banyak satu dari pembayaran order tersebut.

### payment_events

Jejak callback/poll/reconciliation dengan data minimum.

Mulai implementasi: **M4**. Entitas tunggal: `payment_event`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `payment_event_id` | `uuid` | Tidak | PK |  |
| `order_id` | `uuid` | Ya | FK → [orders.order_id](#orders) |  |
| `payment_transaction_id` | `uuid` | Ya | FK → [payment_transactions.payment_transaction_id](#payment_transactions) |  |
| `source` | `text` | Tidak |  |  |
| `provider_event_key` | `text` | Ya |  |  |
| `payload_hash` | `text` | Tidak |  |  |
| `signature_valid` | `boolean` | Ya |  |  |
| `verification_status` | `text` | Tidak |  |  |
| `received_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (provider_event_key) WHERE provider_event_key IS NOT NULL`

**Indeks pencarian awal:**

- `INDEX (order_id, received_at)`

**Aturan domain:**

- Jika kedua FK terisi, transaction.order_id harus sama. Event gagal cocok tidak membuka akses. Raw callback/secret tidak disimpan tanpa kebutuhan retensi D2.

### trial_grants

Ledger penerima trial yang tidak direset dengan menghapus/mengganti profil.

Mulai implementasi: **M4**. Entitas tunggal: `trial_grant`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `trial_grant_id` | `uuid` | Tidak | PK |  |
| `family_id` | `uuid` | Ya | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Ya | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `trial_subject_key` | `uuid` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `started_at` | `timestamptz` | Tidak |  |  |
| `ended_at` | `timestamptz` | Tidak |  |  |
| `policy_version` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (trial_subject_key, course_id)`
- `UNIQUE (child_id, course_id) WHERE child_id IS NOT NULL`

**CHECK per baris:**

- `ended_at > started_at`
- `child_id IS NULL OR family_id IS NOT NULL`

**FK komposit tambahan:**

- `(child_id, family_id)` → `children (child_id, family_id)`.

**Aturan domain:**

- trial_subject_key adalah identitas ledger internal, bukan FK tabel yang hilang. Kuota/titik mulai/perpanjangan mengikuti D3; tidak mengunci angka tiga sebelum kebijakan final.
- FK nullable hanya untuk penghapusan/retensi yang disetujui D2. Tidak menyimpan nama anak sebagai snapshot ledger.
- Saat penghapusan profil disahkan, child_id boleh dikosongkan sementara family_id dipertahankan untuk riwayat keluarga. Family_id hanya dikosongkan ketika penghapusan/anonimisasi keluarga disahkan; bukan otomatis saat satu anak dihapus.
- Perhitungan kuota keluarga tetap mencakup ledger dengan child_id NULL. Menghapus atau membuat ulang profil tidak mengembalikan slot trial yang telah dipakai; angka kuota tetap mengikuti D3.

### refund_requests

Kasus refund manual melalui dukungan/penyedia sesuai D7.

Mulai implementasi: **M4**. Entitas tunggal: `refund_request`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `refund_request_id` | `uuid` | Tidak | PK |  |
| `order_id` | `uuid` | Tidak | FK → [orders.order_id](#orders) |  |
| `requested_by_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `requested_amount` | `bigint` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `reason` | `text` | Tidak |  | Identifier alasan domain sesuai kontrak endpoint; penamaan singkat tidak mengubah aturan validasi nilainya. |
| `provider_refund_reference` | `text` | Ya |  |  |
| `resolved_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**CHECK per baris:**

- `requested_amount >= 0`

**Aturan domain:**

- Tidak mengimplementasikan kebijakan refund otomatis yang belum disepakati. Total refund terverifikasi tidak boleh melebihi jumlah dibayar; pemeriksaan agregat di bawah lock order.

### access_adjustments

Catatan penyesuaian akses oleh operasional, termasuk akibat refund.

Mulai implementasi: **M4**. Entitas tunggal: `access_adjustment`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `access_adjustment_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `order_id` | `uuid` | Ya | FK → [orders.order_id](#orders) |  |
| `refund_request_id` | `uuid` | Ya | FK → [refund_requests.refund_request_id](#refund_requests) |  |
| `actor_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `kind` | `text` | Tidak |  |  |
| `reason` | `text` | Tidak |  | Identifier alasan domain sesuai kontrak endpoint; penamaan singkat tidak mengubah aturan validasi nilainya. |
| `details` | `jsonb` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Aturan domain:**

- Perubahan periode/status akses diaudit dan diserialisasi. Tidak memalsukan status pembayaran gateway.

### access_segments

Hak belajar/fitur pada interval waktu satu anak/kursus.

Mulai implementasi: **M4**. Entitas tunggal: `access_segment`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `access_segment_id` | `uuid` | Tidak | PK |  |
| `child_id` | `uuid` | Tidak | FK → [children.child_id](#children) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `plan_version_id` | `uuid` | Ya | FK → [plan_versions.plan_version_id](#plan_versions) |  |
| `order_id` | `uuid` | Ya | FK → [orders.order_id](#orders) |  |
| `trial_grant_id` | `uuid` | Ya | FK → [trial_grants.trial_grant_id](#trial_grants) |  |
| `access_adjustment_id` | `uuid` | Ya | FK → [access_adjustments.access_adjustment_id](#access_adjustments) |  |
| `source` | `text` | Tidak |  |  |
| `started_at` | `timestamptz` | Tidak |  |  |
| `ended_at` | `timestamptz` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (order_id) WHERE order_id IS NOT NULL`
- `UNIQUE (trial_grant_id) WHERE trial_grant_id IS NOT NULL`

**CHECK per baris:**

- `ended_at > started_at`
- `num_nonnulls(order_id, trial_grant_id, access_adjustment_id) = 1`
- `(source = 'purchase' AND order_id IS NOT NULL AND plan_version_id IS NOT NULL) OR (source = 'trial' AND trial_grant_id IS NOT NULL) OR (source = 'adjustment' AND access_adjustment_id IS NOT NULL)`

**Indeks pencarian awal:**

- `INDEX (child_id, course_id, status, ended_at)`

**Aturan domain:**

- Sumber dan anak/kursus harus cocok. Versi paket wajib untuk pembelian, opsional pada trial/manual sesuai policy. Sumber order hanya boleh diaplikasikan sekali setelah verifikasi.
- Periode efektif memakai batas [started_at, ended_at). Cegah interval aktif/terjadwal tumpang tindih per anak/kursus dengan exclusion constraint dan lock; D10 menentukan cara menyusun/mengubah segmen, bukan membiarkan tier efektif ambigu.

## AI, Audit, dan Operasional

### content_generations

Spesifikasi draf soal AI tanpa konteks/data anak.

Mulai implementasi: **M3**. Entitas tunggal: `content_generation`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `content_generation_id` | `uuid` | Tidak | PK |  |
| `requested_by_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `course_id` | `uuid` | Tidak | FK → [courses.course_id](#courses) |  |
| `level_skill_id` | `uuid` | Tidak | FK → [level_skills.level_skill_id](#level_skills) |  |
| `activity_template_id` | `uuid` | Tidak | FK → [activity_templates.activity_template_id](#activity_templates) |  |
| `requested_count` | `integer` | Tidak |  |  |
| `specification` | `jsonb` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `model_config` | `jsonb` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**CHECK per baris:**

- `requested_count > 0`

**FK komposit tambahan:**

- `(level_skill_id, course_id)` → `level_skills (level_skill_id, course_id)`.

**Aturan domain:**

- Kursus level_skill cocok. Spesifikasi dibatasi schema dan tidak memuat profil/media anak.

### processing_tasks

Status pekerjaan domain; tidak mengganti tabel queue milik River.

Mulai implementasi: **M3**. Entitas tunggal: `processing_task`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `processing_task_id` | `uuid` | Tidak | PK |  |
| `content_generation_id` | `uuid` | Ya | FK → [content_generations.content_generation_id](#content_generations) |  |
| `media_asset_id` | `uuid` | Ya | FK → [media_assets.media_asset_id](#media_assets) |  |
| `report_export_id` | `uuid` | Ya | FK → [report_exports.report_export_id](#report_exports) |  |
| `order_id` | `uuid` | Ya | FK → [orders.order_id](#orders) | Ditambahkan pada M4. |
| `login_token_id` | `uuid` | Ya | FK → [login_tokens.login_token_id](#login_tokens) |  |
| `deletion_request_id` | `uuid` | Ya | FK → [deletion_requests.deletion_request_id](#deletion_requests) | Ditambahkan pada M4. |
| `family_invitation_id` | `uuid` | Ya | FK → [family_invitations.family_invitation_id](#family_invitations) |  |
| `kind` | `text` | Tidak |  |  |
| `task_key` | `text` | Tidak |  |  |
| `river_job_id` | `bigint` | Ya |  | Identitas penyedia/SDK, bukan FK domain. |
| `status` | `text` | Tidak |  |  |
| `attempt_count` | `integer` | Tidak |  |  |
| `last_error` | `text` | Ya |  | Kode error teknis tersanitasi, tanpa body/token/media. |
| `provider_request_key` | `text` | Ya |  |  |
| `cost` | `numeric(20,8)` | Ya |  |  |
| `cost_currency` | `text` | Ya |  |  |
| `delivery_payload_encrypted` | `text` | Ya |  | Payload email sementara terenkripsi; hanya untuk task login/undangan, dibersihkan setelah pengiriman terminal. |
| `payload_expires_at` | `timestamptz` | Ya |  | Batas retry payload email; tidak melampaui expiry token/undangan. |
| `finished_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (task_key)`

**CHECK per baris:**

- `attempt_count >= 0`
- `num_nonnulls(content_generation_id, media_asset_id, report_export_id, order_id, login_token_id, family_invitation_id, deletion_request_id) = 1`
- `(delivery_payload_encrypted IS NULL) = (payload_expires_at IS NULL)`

**Indeks pencarian awal:**

- `INDEX (status, created_at)`

**Aturan domain:**

- Task kind menentukan FK target tepat satu. River job key adalah korelasi internal SDK, bukan FK agar purge queue tidak merusak riwayat domain.
- Delete/retensi membatalkan atau membersihkan task sebelum target dihapus. Biaya mendukung pecahan mata uang penyedia; order rupiah tetap bigint.
- Payload email terenkripsi hanya diizinkan pada task login_token/family_invitation. Secret tidak ada pada argumen River/log; key enkripsi dikelola terpisah. Retry mengirim payload dan provider_request_key yang sama, lalu membersihkan ciphertext setelah terminal sesuai protokol email.

### idempotency_records

Hasil operasi logis yang dapat dipulihkan tanpa duplikasi efek.

Mulai implementasi: **S05**. Entitas tunggal: `idempotency_record`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `idempotency_record_id` | `uuid` | Tidak | PK |  |
| `actor_user_id` | `uuid` | Tidak | FK → [users.user_id](#users) |  |
| `auth_session_id` | `uuid` | Tidak | FK → [auth_sessions.auth_session_id](#auth_sessions) |  |
| `learning_lease_id` | `uuid` | Ya | FK → [learning_leases.learning_lease_id](#learning_leases) | Lease asli operasi belajar; terisi pada submit dan penerbitan cookie agar replay terikat pemilik/generasi yang tepat. |
| `operation` | `text` | Tidak |  |  |
| `resource_key` | `text` | Tidak |  |  |
| `idempotency_key` | `text` | Tidak |  |  |
| `request_hash` | `text` | Tidak |  |  |
| `http_status` | `integer` | Tidak |  |  |
| `response` | `jsonb` | Tidak |  |  |
| `expires_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (auth_session_id, operation, resource_key, idempotency_key)`

**Indeks pencarian awal:**

- `INDEX (expires_at)`

**Aturan domain:**

- resource_key adalah kunci scope opaque, bukan FK generik domain. Verifikasi actor/lease/izin sebelum replay; secret lease/cookie tidak dicache. Respons tersimpan mengikuti retensi D2.
- actor_user_id harus sama dengan auth_sessions.user_id; diverifikasi saat transaksi/constraint join.
- Operasi belajar mengisi learning_lease_id dalam transaksi hasil. Recovery cookie hanya untuk lease aktif yang sama dengan record, auth_session pemilik, child, dan generation; bila sudah dipindah lalu kembali, key lama tidak boleh merotasi lease pengganti. Hapus record sesuai TTL sebelum lease yang dirujuk dibersihkan.

### audit_events

Jejak aksi sensitif dengan payload minimum tersanitasi.

Mulai implementasi: **S03**. Entitas tunggal: `audit_event`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `audit_event_id` | `uuid` | Tidak | PK |  |
| `actor_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `family_id` | `uuid` | Ya | FK → [families.family_id](#families) |  |
| `child_id` | `uuid` | Ya | FK → [children.child_id](#children) |  |
| `action` | `text` | Tidak |  |  |
| `target_table` | `text` | Tidak |  |  |
| `target_key` | `text` | Tidak |  |  |
| `reason` | `text` | Ya |  | Identifier alasan domain sesuai kontrak endpoint; penamaan singkat tidak mengubah aturan validasi nilainya. |
| `metadata` | `jsonb` | Tidak |  |  |
| `occurred_at` | `timestamptz` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Indeks pencarian awal:**

- `INDEX (family_id, occurred_at)`

**Aturan domain:**

- Target table/key adalah referensi historis audit, sengaja bukan FK agar jejak penghapusan bertahan sesuai D2. Tidak dipakai sebagai pengganti relasi bisnis/otorisasi.

### deletion_requests

Permintaan penghapusan dengan target historis dan progres.

Mulai implementasi: **M4**. Entitas tunggal: `deletion_request`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `deletion_request_id` | `uuid` | Tidak | PK |  |
| `requested_by_user_id` | `uuid` | Ya | FK → [users.user_id](#users) |  |
| `target_type` | `text` | Tidak |  |  |
| `target_key` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `policy_version` | `text` | Tidak |  |  |
| `completed_at` | `timestamptz` | Ya |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Aturan domain:**

- Target historis tidak menjadi FK keras ke objek yang justru akan dihapus. Catatan minimum dan retensinya mengikuti D2.

### deletion_items

Langkah penghapusan lokal/remote dan bukti retry.

Mulai implementasi: **M4**. Entitas tunggal: `deletion_item`.

| Kolom | Tipe PostgreSQL | Nullable | Kunci / referensi | Catatan |
| --- | --- | --- | --- | --- |
| `deletion_item_id` | `uuid` | Tidak | PK |  |
| `deletion_request_id` | `uuid` | Tidak | FK → [deletion_requests.deletion_request_id](#deletion_requests) |  |
| `storage_type` | `text` | Tidak |  |  |
| `target_storage_key_encrypted` | `text` | Tidak |  |  |
| `status` | `text` | Tidak |  |  |
| `retry_count` | `integer` | Tidak |  |  |
| `completed_at` | `timestamptz` | Ya |  |  |
| `target_fingerprint` | `text` | Tidak |  |  |
| `created_at` | `timestamptz` | Tidak |  |  |

**Keunikan:**

- `UNIQUE (deletion_request_id, storage_type, target_fingerprint)`

**CHECK per baris:**

- `retry_count >= 0`

**Aturan domain:**

- target_fingerprint adalah hash/HMAC stabil dari identitas objek; enkripsi acak target_storage_key_encrypted tidak dipakai untuk deduplikasi. Simpan kunci remote aman agar retry tetap dapat berjalan setelah baris aplikasi dibersihkan. Retensi dan pembersihan key mengikuti D2.
