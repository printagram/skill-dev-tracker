# Decisions

_Architectural and convention decisions for the project. Format is ADR-lite. Status values: `active`, `deferred`, `rejected`, `superseded by D-###`._

## [D-001] Use Supabase instead of self-hosted Postgres for the order system
Date: 2026-02-10 | Status: active | Ref: DEVLOG 2026-02-10 14:20

Context: Access-based order management is being migrated to a cloud stack. Needed to choose a Postgres hosting option.
Alternatives: self-hosted Postgres on existing VPS, AWS RDS, Neon, Supabase
Decision: Supabase Free tier in eu-west-1 (Ireland), with pgvector enabled for future embedding work.
Rationale: built-in Auth, Storage, and row-level security reduced the amount of glue code. eu-west-1 matches Hostinger NL region for low latency. pgvector available without extra setup. Free tier sufficient for current volume; upgrade path is linear.
Conventions: all cloud data access goes through Supabase client. No direct Postgres connections from the app layer. Migrations live in `supabase/migrations/`.
Revisit when: row count exceeds 500k or Supabase pricing changes materially.

## [D-002] Sync model: Access → Supabase, not bidirectional
Date: 2026-02-18 | Status: active | Ref: DEVLOG 2026-02-18 11:05

Context: Access remains the source of truth for legacy order data during the migration window. Deciding whether to sync both ways.
Alternatives: bidirectional sync, one-way Access → Supabase, full cutover to Supabase now
Decision: one-way sync, Access → Supabase only. Supabase is read-only for external consumers during the migration window.
Rationale: bidirectional sync introduces conflict resolution complexity we do not need yet. Staff continues using Access; external tools read from Supabase. After cutover (Q3), direction reverses and Access becomes archive-only.
Conventions: all writes happen in Access. Supabase schema may have additional columns but must never lose a field that exists in Access.
Revisit when: cutover to Supabase-as-source is planned.

## [D-003] Canonical field name is `idx_Order`, not `id_Order`
Date: 2026-03-18 | Status: active | Ref: DEVLOG 2026-03-18 14:40

Context: Access exposes the primary key as `idx_Order` via ODBC. Several files in the sync layer used `id_Order`, causing silent sync failures that looked like empty result sets.
Alternatives: rename the Access field, add an ODBC alias, use `idx_Order` everywhere
Decision: use `idx_Order` everywhere in sync code. Do not rename the Access field.
Rationale: renaming the Access field would break downstream reports, VBA modules, and user-facing forms. The cost is paying attention to the `idx_` prefix in new code.
Conventions: all ODBC queries, Python ORM mappings, and Supabase column names use `idx_Order`. Code review should flag `id_Order` immediately.
Revisit when: Access is fully retired.

## [D-004] Rejected: webhook-based sync from Access
Date: 2026-02-22 | Status: rejected | Ref: DEVLOG 2026-02-22 16:30

Context: Considered replacing the polling sync with a webhook triggered by Access on record change.
Alternatives: polling every 5 min, Access trigger → HTTP webhook, change-data-capture on the Access file
Decision: rejected webhook approach. Kept polling.
Rationale: Access cannot reliably fire outbound HTTP requests without a custom add-in. CDC on a `.accdb` file is fragile. Polling every 5 min is good enough — delays are acceptable for the current workflow and the system is trivial to reason about.
Conventions: n/a (rejected).
Revisit when: polling latency becomes a user-visible problem, or Access is replaced.
