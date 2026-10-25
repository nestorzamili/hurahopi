# ERD — Sesi dan Pembelajaran

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    placement_runs {
        uuid placement_run_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid placement_rule_version_id FK "required"
    }
    placement_probes {
        uuid placement_probe_id PK "required"
        uuid placement_run_id FK "required"
        uuid level_id FK "required"
    }
    study_sessions {
        uuid study_session_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid started_by_user_id FK "required"
        uuid placement_run_id FK "nullable"
        uuid mastery_rule_version_id FK "required"
    }
    learning_leases {
        uuid learning_lease_id PK "required"
        uuid child_id FK "required"
        uuid study_session_id FK "required"
        uuid auth_session_id FK "required"
    }
    responses {
        uuid response_id PK "required"
        uuid attempt_id FK "required"
    }
    attempts {
        uuid attempt_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid study_session_id FK "required"
        uuid activity_version_id FK "required"
        uuid level_skill_id FK "required"
        uuid rubric_version_id FK "required"
        uuid placement_probe_id FK "nullable"
        uuid first_response_id FK "nullable; S06"
    }
    study_session_events {
        uuid study_session_event_id PK "required"
        uuid study_session_id FK "required"
        uuid attempt_id FK "nullable"
        uuid actor_user_id FK "nullable"
    }
    mastery_states {
        uuid mastery_state_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid level_skill_id FK "required"
        uuid mastery_rule_version_id FK "required"
        uuid rubric_version_id FK "required"
    }
    mastery_changes {
        uuid mastery_change_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid from_level_id FK "nullable"
        uuid to_level_id FK "required"
        uuid rule_version_id FK "nullable"
        uuid study_session_id FK "nullable"
        uuid actor_user_id FK "nullable"
    }
    mastery_change_evidence {
        uuid mastery_change_evidence_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid mastery_change_id FK "required"
        uuid attempt_id FK "required"
    }
    chapter_progress {
        uuid chapter_progress_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid chapter_id FK "required"
        uuid current_level_id FK "required"
        uuid pending_mastery_change_id FK "nullable"
    }
    reward_items {
        uuid reward_item_id PK "required"
        uuid product_asset_id FK "required"
    }
    reward_grants {
        uuid reward_grant_id PK "required"
        uuid child_id FK "required"
        uuid reward_item_id FK "required"
        uuid attempt_id FK "nullable"
        uuid study_session_id FK "nullable"
    }
    activity_versions {
        uuid activity_version_id PK "external module"
    }
    auth_sessions {
        uuid auth_session_id PK "external module"
    }
    chapters {
        uuid chapter_id PK "external module"
    }
    children {
        uuid child_id PK "external module"
    }
    courses {
        uuid course_id PK "external module"
    }
    families {
        uuid family_id PK "external module"
    }
    level_skills {
        uuid level_skill_id PK "external module"
    }
    levels {
        uuid level_id PK "external module"
    }
    product_assets {
        uuid product_asset_id PK "external module"
    }
    rubric_versions {
        uuid rubric_version_id PK "external module"
    }
    rule_versions {
        uuid rule_version_id PK "external module"
    }
    users {
        uuid user_id PK "external module"
    }
    families ||..o{ placement_runs : "family_id"
    children ||..o{ placement_runs : "child_id"
    courses ||..o{ placement_runs : "course_id"
    rule_versions ||..o{ placement_runs : "placement_rule_version_id"
    placement_runs ||..o{ placement_probes : "placement_run_id"
    levels ||..o{ placement_probes : "level_id"
    families ||..o{ study_sessions : "family_id"
    children ||..o{ study_sessions : "child_id"
    courses ||..o{ study_sessions : "course_id"
    users ||..o{ study_sessions : "started_by_user_id"
    placement_runs |o..o{ study_sessions : "placement_run_id"
    rule_versions ||..o{ study_sessions : "mastery_rule_version_id"
    children ||..o{ learning_leases : "child_id"
    study_sessions ||..o{ learning_leases : "study_session_id"
    auth_sessions ||..o{ learning_leases : "auth_session_id"
    attempts ||..o{ responses : "attempt_id"
    families ||..o{ attempts : "family_id"
    children ||..o{ attempts : "child_id"
    courses ||..o{ attempts : "course_id"
    study_sessions ||..o{ attempts : "study_session_id"
    activity_versions ||..o{ attempts : "activity_version_id"
    level_skills ||..o{ attempts : "level_skill_id"
    rubric_versions ||..o{ attempts : "rubric_version_id"
    placement_probes |o..o{ attempts : "placement_probe_id"
    responses |o..o{ attempts : "first_response_id"
    study_sessions ||..o{ study_session_events : "study_session_id"
    attempts |o..o{ study_session_events : "attempt_id"
    users |o..o{ study_session_events : "actor_user_id"
    children ||..o{ mastery_states : "child_id"
    courses ||..o{ mastery_states : "course_id"
    level_skills ||..o{ mastery_states : "level_skill_id"
    rule_versions ||..o{ mastery_states : "mastery_rule_version_id"
    rubric_versions ||..o{ mastery_states : "rubric_version_id"
    children ||..o{ mastery_changes : "child_id"
    courses ||..o{ mastery_changes : "course_id"
    levels |o..o{ mastery_changes : "from_level_id"
    levels ||..o{ mastery_changes : "to_level_id"
    rule_versions |o..o{ mastery_changes : "rule_version_id"
    study_sessions |o..o{ mastery_changes : "study_session_id"
    users |o..o{ mastery_changes : "actor_user_id"
    children ||..o{ mastery_change_evidence : "child_id"
    courses ||..o{ mastery_change_evidence : "course_id"
    mastery_changes ||..o{ mastery_change_evidence : "mastery_change_id"
    attempts ||..o{ mastery_change_evidence : "attempt_id"
    children ||..o{ chapter_progress : "child_id"
    courses ||..o{ chapter_progress : "course_id"
    chapters ||..o{ chapter_progress : "chapter_id"
    levels ||..o{ chapter_progress : "current_level_id"
    mastery_changes |o..o{ chapter_progress : "pending_mastery_change_id"
    product_assets ||..o{ reward_items : "product_asset_id"
    children ||..o{ reward_grants : "child_id"
    reward_items ||..o{ reward_grants : "reward_item_id"
    attempts |o..o{ reward_grants : "attempt_id"
    study_sessions |o..o{ reward_grants : "study_session_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [placement_runs](data-dictionary.md#placement_runs) | `placement_run_id` | S05 | Satu run tes penempatan yang bisa melintasi beberapa sesi belajar. |
| [placement_probes](data-dictionary.md#placement_probes) | `placement_probe_id` | S05 | Probe suatu tingkat pada run penempatan. |
| [study_sessions](data-dictionary.md#study_sessions) | `study_session_id` | S05 | Sesi anak dengan purpose, posisi, dan rencana yang dapat dilanjutkan. |
| [learning_leases](data-dictionary.md#learning_leases) | `learning_lease_id` | S05 | Riwayat pemberian lease perangkat; paling banyak satu aktif per anak. |
| [responses](data-dictionary.md#responses) | `response_id` | S06 | Respons berurutan; koreksi tidak menimpa respons pertama. |
| [attempts](data-dictionary.md#attempts) | `attempt_id` | S05 | Satu penyajian soal dan hasil logisnya; banyak respons koreksi tetap satu attempt. |
| [study_session_events](data-dictionary.md#study_session_events) | `study_session_event_id` | S06 | Interaksi, hint, bantuan, pause, atau skip yang relevan pada sesi. |
| [mastery_states](data-dictionary.md#mastery_states) | `mastery_state_id` | S07 | Status penguasaan anak pada keterampilan-tingkat dan versi aturan. |
| [mastery_changes](data-dictionary.md#mastery_changes) | `mastery_change_id` | S07 | Keputusan perubahan titik latihan dengan alasan dan bukti. |
| [mastery_change_evidence](data-dictionary.md#mastery_change_evidence) | `mastery_change_evidence_id` | S07 | Bukti relasional untuk satu perubahan tingkat. |
| [chapter_progress](data-dictionary.md#chapter_progress) | `chapter_progress_id` | S07 | Posisi latihan bab dan target perubahan sesi berikutnya. |
| [reward_items](data-dictionary.md#reward_items) | `reward_item_id` | S06 | Katalog stiker/koleksi yang diterbitkan produk. |
| [reward_grants](data-dictionary.md#reward_grants) | `reward_grant_id` | S06 | Pemberian satu item dengan sumber instans atau sesi yang unik. |
