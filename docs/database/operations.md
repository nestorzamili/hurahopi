# ERD — AI, Audit, dan Operasional

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    content_generations {
        uuid content_generation_id PK "required"
        uuid requested_by_user_id FK "required"
        uuid course_id FK "required"
        uuid level_skill_id FK "required"
        uuid activity_template_id FK "required"
    }
    processing_tasks {
        uuid processing_task_id PK "required"
        uuid content_generation_id FK "nullable"
        uuid media_asset_id FK "nullable"
        uuid report_export_id FK "nullable"
        uuid order_id FK "nullable; M4"
        uuid login_token_id FK "nullable"
        uuid deletion_request_id FK "nullable; M4"
        uuid family_invitation_id FK "nullable"
    }
    idempotency_records {
        uuid idempotency_record_id PK "required"
        uuid actor_user_id FK "required"
        uuid auth_session_id FK "required"
        uuid learning_lease_id FK "nullable"
    }
    audit_events {
        uuid audit_event_id PK "required"
        uuid actor_user_id FK "nullable"
        uuid family_id FK "nullable"
        uuid child_id FK "nullable"
    }
    deletion_requests {
        uuid deletion_request_id PK "required"
        uuid requested_by_user_id FK "nullable"
    }
    deletion_items {
        uuid deletion_item_id PK "required"
        uuid deletion_request_id FK "required"
    }
    activity_templates {
        uuid activity_template_id PK "external module"
    }
    auth_sessions {
        uuid auth_session_id PK "external module"
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
    family_invitations {
        uuid family_invitation_id PK "external module"
    }
    learning_leases {
        uuid learning_lease_id PK "external module"
    }
    level_skills {
        uuid level_skill_id PK "external module"
    }
    login_tokens {
        uuid login_token_id PK "external module"
    }
    media_assets {
        uuid media_asset_id PK "external module"
    }
    orders {
        uuid order_id PK "external module"
    }
    report_exports {
        uuid report_export_id PK "external module"
    }
    users {
        uuid user_id PK "external module"
    }
    users ||..o{ content_generations : "requested_by_user_id"
    courses ||..o{ content_generations : "course_id"
    level_skills ||..o{ content_generations : "level_skill_id"
    activity_templates ||..o{ content_generations : "activity_template_id"
    content_generations |o..o{ processing_tasks : "content_generation_id"
    media_assets |o..o{ processing_tasks : "media_asset_id"
    report_exports |o..o{ processing_tasks : "report_export_id"
    orders |o..o{ processing_tasks : "order_id"
    login_tokens |o..o{ processing_tasks : "login_token_id"
    deletion_requests |o..o{ processing_tasks : "deletion_request_id"
    family_invitations |o..o{ processing_tasks : "family_invitation_id"
    users ||..o{ idempotency_records : "actor_user_id"
    auth_sessions ||..o{ idempotency_records : "auth_session_id"
    learning_leases |o..o{ idempotency_records : "learning_lease_id"
    users |o..o{ audit_events : "actor_user_id"
    families |o..o{ audit_events : "family_id"
    children |o..o{ audit_events : "child_id"
    users |o..o{ deletion_requests : "requested_by_user_id"
    deletion_requests ||..o{ deletion_items : "deletion_request_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [content_generations](data-dictionary.md#content_generations) | `content_generation_id` | M3 | Spesifikasi draf soal AI tanpa konteks/data anak. |
| [processing_tasks](data-dictionary.md#processing_tasks) | `processing_task_id` | M3 | Status pekerjaan domain; tidak mengganti tabel queue milik River. |
| [idempotency_records](data-dictionary.md#idempotency_records) | `idempotency_record_id` | S05 | Hasil operasi logis yang dapat dipulihkan tanpa duplikasi efek. |
| [audit_events](data-dictionary.md#audit_events) | `audit_event_id` | S03 | Jejak aksi sensitif dengan payload minimum tersanitasi. |
| [deletion_requests](data-dictionary.md#deletion_requests) | `deletion_request_id` | M4 | Permintaan penghapusan dengan target historis dan progres. |
| [deletion_items](data-dictionary.md#deletion_items) | `deletion_item_id` | M4 | Langkah penghapusan lokal/remote dan bukti retry. |
