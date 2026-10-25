# ERD — Kurikulum dan Konten

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    courses {
        uuid course_id PK "required"
    }
    chapters {
        uuid chapter_id PK "required"
        uuid course_id FK "required"
    }
    levels {
        uuid level_id PK "required"
        uuid course_id FK "required"
        uuid chapter_id FK "required"
    }
    skills {
        uuid skill_id PK "required"
        uuid course_id FK "required"
    }
    level_skills {
        uuid level_skill_id PK "required"
        uuid course_id FK "required"
        uuid level_id FK "required"
        uuid skill_id FK "required"
    }
    level_prerequisites {
        uuid level_prerequisite_id PK "required"
        uuid course_id FK "required"
        uuid level_id FK "required"
        uuid required_level_id FK "required"
    }
    rule_versions {
        uuid rule_version_id PK "required"
    }
    rubric_versions {
        uuid rubric_version_id PK "required"
        uuid course_id FK "required"
    }
    activity_templates {
        uuid activity_template_id PK "required"
    }
    product_assets {
        uuid product_asset_id PK "required"
    }
    activity_versions {
        uuid activity_version_id PK "required"
        uuid course_id FK "required"
        uuid level_skill_id FK "required"
        uuid activity_template_id FK "required"
        uuid rubric_version_id FK "required"
        uuid content_generation_id FK "nullable; M3"
    }
    activity_version_assets {
        uuid activity_version_asset_id PK "required"
        uuid activity_version_id FK "required"
        uuid product_asset_id FK "required"
    }
    chapter_introductions {
        uuid chapter_introduction_id PK "required"
        uuid chapter_id FK "required"
        uuid video_product_asset_id FK "nullable"
    }
    chapter_introduction_assets {
        uuid chapter_introduction_asset_id PK "required"
        uuid chapter_introduction_id FK "required"
        uuid product_asset_id FK "required"
    }
    worksheet_versions {
        uuid worksheet_version_id PK "required"
        uuid level_id FK "required"
        uuid worksheet_product_asset_id FK "required"
        uuid answer_key_product_asset_id FK "nullable"
    }
    content_reviews {
        uuid content_review_id PK "required"
        uuid activity_version_id FK "nullable"
        uuid chapter_introduction_id FK "nullable; M5"
        uuid worksheet_version_id FK "nullable; M4"
        uuid reviewer_user_id FK "required"
    }
    content_generations {
        uuid content_generation_id PK "external module"
    }
    users {
        uuid user_id PK "external module"
    }
    courses ||..o{ chapters : "course_id"
    courses ||..o{ levels : "course_id"
    chapters ||..o{ levels : "chapter_id"
    courses ||..o{ skills : "course_id"
    courses ||..o{ level_skills : "course_id"
    levels ||..o{ level_skills : "level_id"
    skills ||..o{ level_skills : "skill_id"
    courses ||..o{ level_prerequisites : "course_id"
    levels ||..o{ level_prerequisites : "level_id"
    levels ||..o{ level_prerequisites : "required_level_id"
    courses ||..o{ rubric_versions : "course_id"
    courses ||..o{ activity_versions : "course_id"
    level_skills ||..o{ activity_versions : "level_skill_id"
    activity_templates ||..o{ activity_versions : "activity_template_id"
    rubric_versions ||..o{ activity_versions : "rubric_version_id"
    content_generations |o..o{ activity_versions : "content_generation_id"
    activity_versions ||..o{ activity_version_assets : "activity_version_id"
    product_assets ||..o{ activity_version_assets : "product_asset_id"
    chapters ||..o{ chapter_introductions : "chapter_id"
    product_assets |o..o{ chapter_introductions : "video_product_asset_id"
    chapter_introductions ||..o{ chapter_introduction_assets : "chapter_introduction_id"
    product_assets ||..o{ chapter_introduction_assets : "product_asset_id"
    levels ||..o{ worksheet_versions : "level_id"
    product_assets ||..o{ worksheet_versions : "worksheet_product_asset_id"
    product_assets |o..o{ worksheet_versions : "answer_key_product_asset_id"
    activity_versions |o..o{ content_reviews : "activity_version_id"
    chapter_introductions |o..o{ content_reviews : "chapter_introduction_id"
    worksheet_versions |o..o{ content_reviews : "worksheet_version_id"
    users ||..o{ content_reviews : "reviewer_user_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [courses](data-dictionary.md#courses) | `course_id` | S02 | Mata pelajaran; Fase 1 berisi math. |
| [chapters](data-dictionary.md#chapters) | `chapter_id` | S04 | Bab dalam suatu kursus. |
| [levels](data-dictionary.md#levels) | `level_id` | S04 | Tingkat dalam satu bab. |
| [skills](data-dictionary.md#skills) | `skill_id` | S04 | Target keterampilan matematika yang stabil. |
| [level_skills](data-dictionary.md#level_skills) | `level_skill_id` | S04 | Relasi tingkat dengan keterampilan; satu keterampilan dapat dilatih pada beberapa tingkat. |
| [level_prerequisites](data-dictionary.md#level_prerequisites) | `level_prerequisite_id` | S04 | Tingkat prasyarat, termasuk antar bab dalam satu kursus. |
| [rule_versions](data-dictionary.md#rule_versions) | `rule_version_id` | S05 | Konfigurasi aturan penempatan, mastery, atau ringkasan laporan yang dibekukan. |
| [rubric_versions](data-dictionary.md#rubric_versions) | `rubric_version_id` | S04 | Rubrik objektif/observasi berversi. |
| [activity_templates](data-dictionary.md#activity_templates) | `activity_template_id` | S04 | Versi renderer/generator dan schema parameter suatu templat. |
| [product_assets](data-dictionary.md#product_assets) | `product_asset_id` | S04 | Aset produk orisinal/berizin; terpisah dari media pribadi anak. |
| [activity_versions](data-dictionary.md#activity_versions) | `activity_version_id` | S04 | Konten, parameter, kunci server dan rubrik pada suatu revisi/versi. |
| [activity_version_assets](data-dictionary.md#activity_version_assets) | `activity_version_asset_id` | S04 | Aset produk yang dirujuk versi aktivitas. |
| [chapter_introductions](data-dictionary.md#chapter_introductions) | `chapter_introduction_id` | M5 | Versi pengantar bab interaktif/video opsional. |
| [chapter_introduction_assets](data-dictionary.md#chapter_introduction_assets) | `chapter_introduction_asset_id` | M5 | Aset produk yang dirujuk layar pengantar bab. |
| [worksheet_versions](data-dictionary.md#worksheet_versions) | `worksheet_version_id` | M4 | Versi PDF worksheet dan kunci orang tua. |
| [content_reviews](data-dictionary.md#content_reviews) | `content_review_id` | S04 | Persetujuan/penolakan manusia terhadap tepat satu target konten. |
