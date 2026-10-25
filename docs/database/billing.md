# ERD — Paket dan Pembayaran

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    plan_versions {
        uuid plan_version_id PK "required"
        uuid course_id FK "required"
    }
    orders {
        uuid order_id PK "required"
        uuid family_id FK "nullable"
        uuid child_id FK "nullable"
        uuid course_id FK "required"
        uuid buyer_user_id FK "nullable"
        uuid plan_version_id FK "required"
    }
    payment_transactions {
        uuid payment_transaction_id PK "required"
        uuid order_id FK "required"
    }
    payment_events {
        uuid payment_event_id PK "required"
        uuid order_id FK "nullable"
        uuid payment_transaction_id FK "nullable"
    }
    trial_grants {
        uuid trial_grant_id PK "required"
        uuid family_id FK "nullable"
        uuid child_id FK "nullable"
        uuid course_id FK "required"
    }
    refund_requests {
        uuid refund_request_id PK "required"
        uuid order_id FK "required"
        uuid requested_by_user_id FK "nullable"
    }
    access_adjustments {
        uuid access_adjustment_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid order_id FK "nullable"
        uuid refund_request_id FK "nullable"
        uuid actor_user_id FK "nullable"
    }
    access_segments {
        uuid access_segment_id PK "required"
        uuid child_id FK "required"
        uuid course_id FK "required"
        uuid plan_version_id FK "nullable"
        uuid order_id FK "nullable"
        uuid trial_grant_id FK "nullable"
        uuid access_adjustment_id FK "nullable"
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
    users {
        uuid user_id PK "external module"
    }
    courses ||..o{ plan_versions : "course_id"
    families |o..o{ orders : "family_id"
    children |o..o{ orders : "child_id"
    courses ||..o{ orders : "course_id"
    users |o..o{ orders : "buyer_user_id"
    plan_versions ||..o{ orders : "plan_version_id"
    orders ||..o{ payment_transactions : "order_id"
    orders |o..o{ payment_events : "order_id"
    payment_transactions |o..o{ payment_events : "payment_transaction_id"
    families |o..o{ trial_grants : "family_id"
    children |o..o{ trial_grants : "child_id"
    courses ||..o{ trial_grants : "course_id"
    orders ||..o{ refund_requests : "order_id"
    users |o..o{ refund_requests : "requested_by_user_id"
    children ||..o{ access_adjustments : "child_id"
    courses ||..o{ access_adjustments : "course_id"
    orders |o..o{ access_adjustments : "order_id"
    refund_requests |o..o{ access_adjustments : "refund_request_id"
    users |o..o{ access_adjustments : "actor_user_id"
    children ||..o{ access_segments : "child_id"
    courses ||..o{ access_segments : "course_id"
    plan_versions |o..o{ access_segments : "plan_version_id"
    orders |o..o| access_segments : "order_id"
    trial_grants |o..o| access_segments : "trial_grant_id"
    access_adjustments |o..o{ access_segments : "access_adjustment_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [plan_versions](data-dictionary.md#plan_versions) | `plan_version_id` | M4 | Versi harga, durasi dan fitur paket yang dibekukan. |
| [orders](data-dictionary.md#orders) | `order_id` | M4 | Pesanan satu anak/kursus dan satu versi paket. |
| [payment_transactions](data-dictionary.md#payment_transactions) | `payment_transaction_id` | M4 | Percobaan transaksi gateway untuk suatu order. |
| [payment_events](data-dictionary.md#payment_events) | `payment_event_id` | M4 | Jejak callback/poll/reconciliation dengan data minimum. |
| [trial_grants](data-dictionary.md#trial_grants) | `trial_grant_id` | M4 | Ledger penerima trial yang tidak direset dengan menghapus/mengganti profil. |
| [refund_requests](data-dictionary.md#refund_requests) | `refund_request_id` | M4 | Kasus refund manual melalui dukungan/penyedia sesuai D7. |
| [access_adjustments](data-dictionary.md#access_adjustments) | `access_adjustment_id` | M4 | Catatan penyesuaian akses oleh operasional, termasuk akibat refund. |
| [access_segments](data-dictionary.md#access_segments) | `access_segment_id` | M4 | Hak belajar/fitur pada interval waktu satu anak/kursus. |
