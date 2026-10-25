# ERD — Akun dan Akses

Acuan metadata: [schema.json](schema.json). Perbarui model, diagram, dan kamus data bersama ketika rancangan berubah.

[Dokumentasi utama](../../README.md#database) · [Aturan desain](design.md) · [Kamus data lengkap](data-dictionary.md)

Diagram menampilkan PK/FK. Kotak `external module` hanya menunjukkan PK tabel modul lain. Garis mencakup seluruh FK tunggal milik modul ini; FK komposit dan batasan unik bersyarat dirinci di kamus data. Kardinalitas adalah batas struktural FK, bukan jaminan bahwa proses bisnis sudah selesai.

```mermaid
erDiagram
    direction LR
    users {
        uuid user_id PK "required"
    }
    identities {
        uuid identity_id PK "required"
        uuid user_id FK "required"
    }
    families {
        uuid family_id PK "required"
    }
    family_memberships {
        uuid family_membership_id PK "required"
        uuid family_id FK "required"
        uuid user_id FK "required"
    }
    children {
        uuid child_id PK "required"
        uuid family_id FK "required"
        uuid selected_reward_grant_id FK "nullable; M3"
    }
    family_invitations {
        uuid family_invitation_id PK "required"
        uuid family_id FK "required"
        uuid invited_by_user_id FK "required"
        uuid accepted_by_user_id FK "nullable"
    }
    child_permissions {
        uuid child_permission_id PK "required"
        uuid family_id FK "required"
        uuid family_membership_id FK "nullable"
        uuid child_id FK "required"
        uuid family_invitation_id FK "nullable"
    }
    auth_sessions {
        uuid auth_session_id PK "required"
        uuid user_id FK "required"
        uuid family_id FK "nullable"
        uuid active_child_id FK "nullable"
    }
    login_tokens {
        uuid login_token_id PK "required"
        uuid user_id FK "nullable"
    }
    oauth_login_states {
        uuid oauth_login_state_id PK "required"
    }
    consent_events {
        uuid consent_event_id PK "required"
        uuid family_id FK "required"
        uuid child_id FK "required"
        uuid actor_user_id FK "required"
    }
    reward_grants {
        uuid reward_grant_id PK "external module"
    }
    users ||..o{ identities : "user_id"
    families ||..o{ family_memberships : "family_id"
    users ||..o{ family_memberships : "user_id"
    families ||..o{ children : "family_id"
    reward_grants |o..o{ children : "selected_reward_grant_id"
    families ||..o{ family_invitations : "family_id"
    users ||..o{ family_invitations : "invited_by_user_id"
    users |o..o{ family_invitations : "accepted_by_user_id"
    families ||..o{ child_permissions : "family_id"
    family_memberships |o..o{ child_permissions : "family_membership_id"
    children ||..o{ child_permissions : "child_id"
    family_invitations |o..o{ child_permissions : "family_invitation_id"
    users ||..o{ auth_sessions : "user_id"
    families |o..o{ auth_sessions : "family_id"
    children |o..o{ auth_sessions : "active_child_id"
    users |o..o{ login_tokens : "user_id"
    families ||..o{ consent_events : "family_id"
    children ||..o{ consent_events : "child_id"
    users ||..o{ consent_events : "actor_user_id"
```

## Tabel dan waktu implementasi

| Tabel | Primary key | Mulai | Tujuan |
| --- | --- | --- | --- |
| [users](data-dictionary.md#users) | `user_id` | S03 | Identitas orang dewasa; anak tidak mempunyai akun login sendiri. |
| [identities](data-dictionary.md#identities) | `identity_id` | S03 | Identitas Google atau email yang telah dibuktikan. |
| [families](data-dictionary.md#families) | `family_id` | S03 | Batas kepemilikan data satu keluarga. |
| [family_memberships](data-dictionary.md#family_memberships) | `family_membership_id` | S03 | Hubungan pengguna dewasa dengan keluarga dan perannya. |
| [children](data-dictionary.md#children) | `child_id` | S03 | Profil belajar anak, berada dalam tepat satu keluarga. |
| [family_invitations](data-dictionary.md#family_invitations) | `family_invitation_id` | M3 | Undangan pengasuh dengan token sekali pakai. |
| [child_permissions](data-dictionary.md#child_permissions) | `child_permission_id` | M3 | Izin per anak: ditawarkan lewat undangan atau efektif lewat membership. |
| [auth_sessions](data-dictionary.md#auth_sessions) | `auth_session_id` | S03 | Sesi login satu browser milik orang dewasa. |
| [login_tokens](data-dictionary.md#login_tokens) | `login_token_id` | S03 | Tautan email login/reset PIN dengan purpose dan masa berlaku. |
| [oauth_login_states](data-dictionary.md#oauth_login_states) | `oauth_login_state_id` | S03 | State/nonce sementara untuk callback OAuth/OIDC. |
| [consent_events](data-dictionary.md#consent_events) | `consent_event_id` | S03 | Jejak pemberian/penarikan persetujuan per fitur dan versi kebijakan. |
