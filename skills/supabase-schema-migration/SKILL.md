---
name: supabase-schema-migration
description: Guides creating, naming, and applying Supabase PostgreSQL migration files. Use whenever a schema change is needed — new table, column add, seed data, trigger, view, or RLS policy.
---

# Supabase Schema Migration Skill

## Overview
Every schema change in a Supabase project is captured as a numbered SQL file committed to the repo. Migrations are the permanent, auditable record of how the database evolved. They are never edited after application — if something needs to change, a new migration is written.

## When to Use
- Creating a new table or view
- Adding or altering a column
- Adding a trigger, function, or pg_cron job
- Seeding reference/lookup data
- Adding or updating RLS policies
- Any structural change to the database

---

## File Conventions

### Naming
```
NNN_description_of_change.sql
```
- `NNN` is zero-padded, sequential, **never skipped or reused**
- Description is lowercase-hyphen or underscore, imperative (what it does)
- Examples: `019_add_max_health_to_characters.sql`, `020_seed_setting_grid_z_layers.sql`

### File Location
```
backend/migrations/       ← chronicle-worlds pattern
shared/schema/            ← community-infrastructure-development pattern
database/                 ← customer-sales-data-practical pattern
schema/                   ← industrial-maintenance-b2b pattern
```
Use whichever location is canonical for the project. Document it in PROGRESS.md.

---

## Table Naming Strategy

Formal PostgreSQL schemas (e.g. `CREATE SCHEMA csdp;`) are **not commonly used** across these projects. All tables live in `public`. Disambiguation is handled at the **table name level** using a prefix or by relying on the fact that the table is 100% unique to the Supabase project.

### Decision Tree

```
Is this Supabase project shared by multiple source repos / apps?
  ├─ YES → Use a prefix on ALL tables in this project (see Cross-Project below)
  └─ NO  → Will any table be read by a different project in the future?
              ├─ YES or MAYBE → Prefix the table from the start
              └─ NO, fully isolated → No prefix required; plain table names are fine
```

### Pattern A — No Prefix (isolated project)
Used when a Supabase project is owned entirely by one repo and no external consumer will ever query it.

```sql
-- chronicle-worlds: settings, characters, events, grid_cells, etc.
-- All in public schema, no prefix needed — project is self-contained.
CREATE TABLE public.characters ( ... );
CREATE TABLE public.events ( ... );
```

### Pattern B — Table Name Prefix (shared Supabase project)
Used when multiple repos or apps share the same Supabase project, OR when a table is intentionally exposed as a cross-project data source.

Prefix format: `[short_code]_table_name`
- Short code is 2–5 chars, lowercase, derived from the project or domain
- Applied consistently to **all** tables that belong to that domain group
- Lives in `public` schema — formal schema objects are not created

```sql
-- community-infrastructure-development: cid_ prefix
-- Same Supabase project (hhyhulqngdkwsxhymmcd) also used by virtual-pinball-wall
-- and potentially other projects — prefix prevents collision
CREATE TABLE public.cid_scenarios ( ... );
CREATE TABLE public.cid_communities ( ... );
CREATE TABLE public.cid_proposed_infrastructure ( ... );
```

### Cross-Project Tables
Some tables are explicitly designed to be **read by a different repo** via the Supabase anon key. These always use a prefix and require a `COMMENT ON TABLE` naming the consumer.

Real example — `cid_proposed_infrastructure`:
```sql
COMMENT ON TABLE public.cid_proposed_infrastructure IS
  'Proposed infrastructure projects linked to CID communities.
   Public display layer: maine-civic-tracker
   https://github.com/andredavisme/maine-civic-tracker
   proposals.html queries this table directly via Supabase anon key.';
```

Rules for cross-project tables:
1. Always prefixed — no exceptions
2. `COMMENT ON TABLE` names the consumer repo and the URL
3. RLS must include a `public read` SELECT policy (`FOR SELECT USING (true)`)
4. Write access locked to `service_role` only — the consumer never writes
5. Document the relationship in PROGRESS.md of **both** repos

### Known Prefix Registry

| Prefix | Project | Supabase Project ID |
|--------|---------|---------------------|
| `cid_` | community-infrastructure-development | hhyhulqngdkwsxhymmcd |
| `csdp_` | customer-sales-data-practical | (see project card) |
| `indB2B_` | industrial-maintenance-b2b | (see project card) |
| (none) | chronicle-worlds | hhyhulqngdkwsxhymmcd |
| (none) | rfq-course | nmemmfblpzrkwyljpmvp |

> ⚠️ chronicle-worlds and community-infrastructure-development share the same Supabase project (`hhyhulqngdkwsxhymmcd`). The `cid_` prefix is what prevents collision. Any new table added to this project that belongs to CID must carry the prefix.

---

## Two Migration Types

### Type 1 — COMMIT (structural change, applied to prod)

Header format:
```sql
-- NNN_description.sql
-- [Project Name]: [what this migration does]
-- [Optional: key decision or constraint noted inline]
-- ⚠️ Verified against live DB YYYY-MM-DD: [what was confirmed]
```

Body format:
```sql
-- One logical change per migration
ALTER TABLE public.table_name
  ADD COLUMN IF NOT EXISTS column_name TYPE NOT NULL DEFAULT value;

-- Seed data if applicable
UPDATE public.table_name SET column = value WHERE condition;

-- Comment column if non-obvious
COMMENT ON COLUMN public.table_name.column_name IS
  'What this column means and why it exists.';
```

Real example — migration 018 from chronicle-worlds:
```sql
-- Migration 018: add conflict_modifier to z_properties
-- Makes height-advantage damage data-driven instead of hardcoded.

ALTER TABLE z_properties
  ADD COLUMN IF NOT EXISTS conflict_modifier NUMERIC NOT NULL DEFAULT 0;

UPDATE z_properties SET conflict_modifier =  0.0 WHERE z_layer = 0;
UPDATE z_properties SET conflict_modifier = -0.5 WHERE z_layer = -1;
-- ...

COMMENT ON COLUMN z_properties.conflict_modifier IS
  'Bonus/penalty added to introduce_conflict damage when actor is at this z_layer.';
```

---

### Type 2 — ROLLBACK (test fixtures / destructive preview)

Header format:
```sql
-- =============================================================
-- Migration NNN — [Title]
-- [Project Name]
-- =============================================================
-- HOW TO RUN:
--   1. Replace PLACEHOLDER_UUID with a real value.
--   2. Paste into Supabase SQL Editor and run.
--   3. Ends with ROLLBACK — change to COMMIT to persist fixtures.
-- =============================================================

BEGIN;

-- ... test inserts / selects ...

ROLLBACK;  -- ← change to COMMIT only to persist
```

**ROLLBACK files are reference-only. Never apply to production without reading the full file first.**

---

## Process

### Writing a New Migration
1. Check the last migration number in `backend/migrations/` (or equivalent) — increment by 1
2. Create file: `NNN_description.sql`
3. Decide prefix strategy using the decision tree above
4. Write the header comment (name, project, what it does, date if verified)
5. Write one logical change — no bundling unrelated changes
6. Use `ADD COLUMN IF NOT EXISTS` (not bare `ADD COLUMN`) for safety on re-runs
7. Add seed data UPDATEs in the same file if the column needs values immediately
8. Add `COMMENT ON COLUMN` for any non-obvious column; `COMMENT ON TABLE` for any cross-project table
9. Commit the file before applying it to the database

### Applying a Migration
1. Open Supabase Dashboard → SQL Editor
2. Paste the migration content
3. Run — check for errors before assuming success
4. Verify the change in Table Editor or a `SELECT * FROM table_name LIMIT 5` query
5. Note the migration in PROGRESS.md: migration number, date, what it does

### After Application
1. Update PROGRESS.md Quick Reference: `Key migration: NNN_description`
2. If a new column was added to a Realtime table — confirm `REPLICA IDENTITY FULL` is still set
3. If a new table was added — run the RLS checklist immediately (see `references/supabase-checklist.md`)
4. If a cross-project table was added — update PROGRESS.md of the consumer repo too

---

## Common Patterns

### New Table (minimal safe pattern)
```sql
-- NNN_add_[prefix_table_name].sql
-- [Project]: adds [table] for [purpose]

CREATE TABLE IF NOT EXISTS public.prefix_table_name (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  -- columns...
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE public.prefix_table_name ENABLE ROW LEVEL SECURITY;
-- RLS policies go in the same migration or NNN+1
```

### Add Column
```sql
ALTER TABLE public.table_name
  ADD COLUMN IF NOT EXISTS column_name TYPE NOT NULL DEFAULT value;
```

### Seed Reference Data
```sql
-- Use ON CONFLICT DO NOTHING for idempotency on re-runs
INSERT INTO public.table_name (col1, col2)
VALUES ('a', 1), ('b', 2)
ON CONFLICT (col1) DO NOTHING;
```

### pg_cron Job
```sql
SELECT cron.schedule(
  'job-name',
  '* * * * *',
  $$SELECT function_name()$$
);
```

### Realtime — REPLICA IDENTITY
```sql
ALTER TABLE public.table_name REPLICA IDENTITY FULL;
```

---

## Watch-Outs From Real Projects

| Issue | What Happened | Prevention |
|-------|--------------|------------|
| Duplicate migration number | chronicle-worlds has two `015_` files | Always list the directory before creating a new file |
| Shared project, no prefix | chronicle-worlds + CID share `hhyhulqngdkwsxhymmcd` | Use the decision tree — if the project is shared, prefix everything |
| Bare `ADD COLUMN` | Fails on re-run if column exists | Always use `ADD COLUMN IF NOT EXISTS` |
| ROLLBACK file applied to prod | Data integrity risk | Read the full file; check for `ROLLBACK` at the bottom |
| RLS not added with table | New table is open | Run RLS checklist immediately after `CREATE TABLE` |
| Cross-project table undocumented | Consumer breaks when table changes | `COMMENT ON TABLE` + note in both PROGRESS.md files |
| Realtime breaks after migration | Column changes invalidate subscription | Re-confirm `REPLICA IDENTITY FULL` after any ALTER on a Realtime table |

---

## Verification Gates
- [ ] File named `NNN_description.sql` with correct sequential number (no gaps, no duplicates)
- [ ] Header comment present: migration name, project, what it does
- [ ] One logical change per file
- [ ] Prefix decision made and documented (prefixed vs. plain; why)
- [ ] If shared Supabase project: prefix applied to all tables in this domain
- [ ] `IF NOT EXISTS` used on all `ADD COLUMN` and `CREATE TABLE` statements
- [ ] ROLLBACK files labeled; not applied to production
- [ ] File committed to repo before being applied to the database
- [ ] PROGRESS.md Quick Reference updated with migration number
- [ ] RLS checklist run if a new table was created
- [ ] If cross-project table: `COMMENT ON TABLE` names consumer + URL; both PROGRESS.md files updated
- [ ] `REPLICA IDENTITY FULL` confirmed if an altered table uses Realtime

---

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "It's a small change, I'll just run it in the SQL editor and skip the file" | There is no small change. A column run in the SQL editor with no migration file is invisible to every future session. |
| "I'll write the migration file after I test it" | The file IS the test record. Write it first, paste it in, run it. |
| "I'll bundle a few changes into one migration to save files" | When one part fails, you can't tell which. One logical change per file. |
| "The number doesn't matter, I'll fix it later" | Gaps and duplicates in the sequence cause confusion during debugging and are impossible to resolve cleanly. |
| "This table is just for this project, no prefix needed" | Use the decision tree. If the Supabase project is shared now or ever will be, prefix it. Renaming a table later is a breaking change for every consumer. |

---

*Skill version: 1.1 — 2026-07-06 | Source evidence: chronicle-worlds (migrations 001–020), community-infrastructure-development (cid_ prefix, cross-project table pattern), customer-sales-data-practical, industrial-maintenance-b2b, rfq-course*
