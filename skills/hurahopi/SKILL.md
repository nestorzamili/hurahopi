---
name: hurahopi
description: Implement or review Hurahopi product, UX, database, API, and application changes using its approved scope, learning rules, design direction, and development documents.
---

# Hurahopi

Read [the central README](../../README.md) for the current state and next milestone. Inspect the actual workspace before using application commands: planned folders, contracts, migrations, and tests may not exist yet.

## Load the relevant context

- Product scope or policy: [PRD](../../docs/prd.md) and [decisions](../../docs/decisions.md). Preserve open decisions and distinguish prototype parameters from validated learning standards.
- UI, components, or user-facing text: [desain.md](../../docs/desain.md) for visual direction, [UX](../../docs/ux-spec.md) for behavior, and the relevant [feature flows](../../README.md#activity-diagram). Read anti-slop through [AGENTS.md](../../AGENTS.md) and reuse a mode already selected in the conversation.
- Data or endpoints: [schema source](../../docs/database/schema.json), [database design](../../docs/database/design.md), and [technical specification](../../docs/technical-spec.md).
- Learning or grading: [learning specification](../../docs/learning-spec.md). A presented question and its result are one `attempts` record; ordered answers belong to `responses`. Preserve first-answer, assistance, and evidence semantics.
- Implementation: [architecture](../../docs/architecture.md), the applicable story in [delivery](../../docs/delivery.md), and its verification requirements in [operations](../../docs/operations.md).

## Preserve the project's distinctions

The child's Hopi world is the main experience. Adult dashboards support preparation and evidence review; they are not a template for the child's activity screen. The adult prepares the session and lease before the child opens the mission. Data, offers, and controls on each screen follow that context.

Use the selected React/Vite/TanStack and Go/Fiber/PostgreSQL stack. Adapt parent/admin components as specified; do not import demo authentication, a second application router, or a template's ownership model.

Keep table/field names in snake_case. PKs use singular entity + `_id`; FKs retain the target PK name with meaningful role prefixes. Prefer contextual fields such as `avatar`, `role`, `provider`, and `locale`; retain hash, timestamp, and unit suffixes where meaningful. Read current schema names instead of recreating obsolete entities. Table count is not a quota: create only the tables and columns needed by the current milestone.

Preserve D9's per-child session/lease limits and the browser-context transaction rules. Preserve D13's review of the full content definition, including immutable assets. AI providers and child-data processing follow their existing gates; frontend state and fake fixtures do not grant production access.

## Finish a change

When behavior, scope, or schema changes, update the authoritative specification and its affected examples, diagrams, and backlog. A fix that preserves documented behavior need not change unrelated specifications. Maintain `schema.json`, ERDs, and the data dictionary together by editing their affected entries in the same change. Keep one README and link new topic documents from it.

Review documentation links, skill instructions, schema/ERD/dictionary consistency, and examples directly. Use the contrast formula/reference table in antislop-human for palette changes. Run the applicable application checks when application code exists; UI delivery includes browser, keyboard, responsive, and state checks. State what was verified and what still requires implementation or user validation. Do not add approval steps or global configuration changes beyond the user's request.
