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
ALTER TABLE schema_name.table_name
  ADD COLUMN IF NOT EXISTS column_name TYPE NOT NULL DEFAULT value;

-- Seed data if applicable
UPDATE schema_name.table_name SET column = value WHERE condition;

-- Comment column if non-obvious
COMMENT ON COLUMN schema_name.table_name.column_name IS
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
3. Write the header comment (name, project, what it does, date if verified)
4. Write one logical change — no bundling unrelated changes
5. Add `ADD COLUMN IF NOT EXISTS` (not bare `ADD COLUMN`) for safety on re-runs
6. Add seed data UPDATEs in the same file if the column needs values immediately
7. Add `COMMENT ON COLUMN` for any non-obvious column
8. Add schema prefix on all objects if the project uses namespacing
9. Commit the file before applying it to the database

### Applying a Migration
1. Open Supabase Dashboard → SQL Editor
2. Paste the migration content
3. Run — check for errors before assuming success
4. Verify the change in Table Editor or a `\d table_name` query
5. Note the migration in PROGRESS.md: migration number, date, what it does

### After Application
1. Update PROGRESS.md Quick Reference: `Key migration: NNN_description`
2. If a new column was added to a table used with Realtime — confirm `REPLICA IDENTITY FULL` is still set
3. If a new table was added — run the RLS checklist immediately (see `references/supabase-checklist.md`)

---

## Common Patterns

### New Table (minimal safe pattern)
```sql
-- NNN_add_[table_name].sql
-- [Project]: adds [table_name] for [purpose]

CREATE TABLE IF NOT EXISTS schema.table_name (
  id          BIGSERIAL PRIMARY KEY,
  -- columns...
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE schema.table_name ENABLE ROW LEVEL SECURITY;
-- RLS policies go in the same migration or a dedicated NNN+1 migration
```

### Add Column
```sql
ALTER TABLE schema.table_name
  ADD COLUMN IF NOT EXISTS column_name TYPE NOT NULL DEFAULT value;
```

### Seed Reference Data
```sql
-- Use ON CONFLICT DO NOTHING for idempotency on re-runs
INSERT INTO schema.table_name (col1, col2)
VALUES ('a', 1), ('b', 2)
ON CONFLICT (col1) DO NOTHING;
```

### pg_cron Job
```sql
-- Requires pg_cron extension enabled in Supabase
SELECT cron.schedule(
  'job-name',          -- unique name
  '* * * * *',         -- cron expression
  $$SELECT function_name()$$
);
```

### Realtime — REPLICA IDENTITY
```sql
ALTER TABLE schema.table_name REPLICA IDENTITY FULL;
```

---

## Watch-Outs From Real Projects

| Issue | What Happened | Prevention |
|-------|--------------|------------|
| Duplicate number | chronicle-worlds has two `015_` files | Always list the directory before creating a new file |
| Missing schema prefix | Objects land in `public` unexpectedly | Prefix every object: `schema.table_name` |
| Bare `ADD COLUMN` | Fails on re-run if column exists | Always use `ADD COLUMN IF NOT EXISTS` |
| ROLLBACK file applied to prod | Data integrity risk | Read the full file — check for `ROLLBACK` at the bottom before pasting |
| RLS not added with table | New table is open | Run RLS checklist immediately after `CREATE TABLE` |
| Realtime breaks after migration | Column changes invalidate subscription | Re-confirm `REPLICA IDENTITY FULL` after any ALTER on a Realtime table |

---

## Verification Gates
- [ ] File named `NNN_description.sql` with correct sequential number (no gaps, no duplicates)
- [ ] Header comment present: migration name, project, what it does
- [ ] One logical change per file
- [ ] `IF NOT EXISTS` used on all `ADD COLUMN` and `CREATE TABLE` statements
- [ ] Schema prefix on all objects (if project uses namespacing)
- [ ] ROLLBACK files are labeled and not applied to production
- [ ] File committed to repo before being applied to the database
- [ ] PROGRESS.md Quick Reference updated with migration number
- [ ] RLS checklist run if a new table was created
- [ ] `REPLICA IDENTITY FULL` confirmed if an altered table uses Realtime

---

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "It's a small change, I'll just run it in the SQL editor and skip the file" | There is no small change. A column dropped in the SQL editor with no migration file is invisible to every future session. |
| "I'll write the migration file after I test it" | The file IS the test record. Write it first, paste it in, run it. |
| "I'll bundle a few changes into one migration to save files" | When one part fails, you can't tell which. One logical change per file. |
| "The number doesn't matter, I'll fix it later" | Gaps and duplicates in the sequence cause confusion during debugging and are impossible to resolve cleanly. |

---

*Skill version: 1.0 — 2026-07-06 | Source evidence: chronicle-worlds (migrations 001–020), customer-sales-data-practical (csdp schema), community-infrastructure-development (schema/), industrial-maintenance-b2b (indB2B schema)*
