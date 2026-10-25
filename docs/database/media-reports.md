# ERD — Media dan Laporan

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    media_assets {
        uuid media_asset_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid attempt_id FK "nullable"
        uuid uploaded_by_user_id FK "nullable"
        uuid consent_event_id FK "nullable"
    }
    observations {
        uuid observation_id PK "required"
        uuid child_id FK "required"
        uuid attempt_id FK "required"
        uuid rubric_version_id FK "required"
        uuid processing_task_id FK "nullable"
    }
    report_snapshots {
        uuid report_snapshot_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid report_rule_version_id FK "required"
    }
    report_evidence {
        uuid report_evidence_id PK "required"
        uuid report_snapshot_id FK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid attempt_id FK "required"
        uuid observation_id FK "nullable"
    }
    report_exports {
        uuid report_export_id PK "required"
        uuid report_snapshot_id FK "required"
        uuid requested_by_user_id FK "required"
    }
    attempts {
        uuid attempt_id PK "external module"
    }
    children {
        uuid child_id PK "external module"
    }
    consent_events {
        uuid consent_event_id PK "external module"
    }
    courses {
        uuid course_id PK "external module"
    }
    families {
        uuid family_id PK "external module"
    }
    processing_tasks {
        uuid processing_task_id PK "external module"
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
    families ||..o{ media_assets : "family_id"
    children ||..o{ media_assets : "child_id"
    attempts |o..o{ media_assets : "attempt_id"
    users |o..o{ media_assets : "uploaded_by_user_id"
    consent_events |o..o{ media_assets : "consent_event_id"
    children ||..o{ observations : "child_id"
    attempts ||..o{ observations : "attempt_id"
    rubric_versions ||..o{ observations : "rubric_version_id"
    processing_tasks |o..o{ observations : "processing_task_id"
    families ||..o{ report_snapshots : "family_id"
    children ||..o{ report_snapshots : "child_id"
    courses ||..o{ report_snapshots : "course_id"
    rule_versions ||..o{ report_snapshots : "report_rule_version_id"
    report_snapshots ||..o{ report_evidence : "report_snapshot_id"
    children ||..o{ report_evidence : "child_id"
    courses ||..o{ report_evidence : "course_id"
    attempts ||..o{ report_evidence : "attempt_id"
    observations |o..o{ report_evidence : "observation_id"
    report_snapshots ||..o{ report_exports : "report_snapshot_id"
    users ||..o{ report_exports : "requested_by_user_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [media_assets](data-dictionary.md#media_assets) | `media_asset_id` | S09/M3 | Media pribadi anak, terpisah dari product_assets. |
| [observations](data-dictionary.md#observations) | `observation_id` | M3 | Pengamatan rubrik atas suatu attempt, deterministik/manusia/AI. |
| [report_snapshots](data-dictionary.md#report_snapshots) | `report_snapshot_id` | M3 | Snapshot laporan berbukti untuk ekspor/riwayat; ringkasan M1 boleh query langsung. |
| [report_evidence](data-dictionary.md#report_evidence) | `report_evidence_id` | M3 | Relasi bukti attempt/pengamatan ke key klaim lokal pada snapshot. |
| [report_exports](data-dictionary.md#report_exports) | `report_export_id` | M3 | Permintaan PDF sementara per snapshot dan pengguna. |
