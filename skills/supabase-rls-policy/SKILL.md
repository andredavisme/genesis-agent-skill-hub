---
name: supabase-rls-policy
description: Guides writing, applying, and verifying Supabase Row Level Security policies. Use whenever a table is created, a new access pattern is needed, or a permission error is being debugged.
---

# Supabase RLS Policy Skill

## Overview
Row Level Security (RLS) is non-negotiable on every Supabase table. A table with RLS disabled is fully open to anyone with the anon key. RLS must be enabled at creation and policies must be explicit — no implicit access, no wildcard shortcuts in production.

## When to Use
- Creating any new table
- Adding a new access pattern (e.g. authenticated users can now write)
- Debugging a 403 / permission denied error from the frontend
- Debugging service_role operations failing inside Edge Functions or triggers
- Auditing a project's security posture

---

## The Four RLS Patterns

All policies across current projects fall into one of these four patterns. Pick the right one before writing any SQL.

---

### Pattern 1 — Public Read, No Write
Used for: reference data, game world state, public analysis data, cross-project display tables.
Anyone with the anon key can SELECT. Nobody can write via the API.

```sql
ALTER TABLE public.table_name ENABLE ROW LEVEL SECURITY;

CREATE POLICY "public read"
  ON public.table_name
  FOR SELECT
  USING (true);

-- No INSERT/UPDATE/DELETE policy = no write access via API
-- Writes go through service_role in Edge Functions only
```

Real example — chronicle-worlds world state tables (migration 011): [cite:35]
```sql
CREATE POLICY "Public read world_tick_state"
  ON public.world_tick_state
  FOR SELECT
  USING (true);
```

Real example — CID cross-project table (migration 005):
```sql
CREATE POLICY "public read" ON public.cid_proposed_infrastructure FOR SELECT USING (true);
CREATE POLICY "service role write" ON public.cid_proposed_infrastructure
  FOR ALL USING (auth.role() = 'service_role');
```

---

### Pattern 2 — Service Role Write (Edge Functions / triggers)
Used for: any table written to by Edge Functions, pg_cron jobs, or trigger functions. Frontend never writes directly.

```sql
ALTER TABLE public.table_name ENABLE ROW LEVEL SECURITY;

-- Public or authenticated read (choose one)
CREATE POLICY "public read table_name"
  ON public.table_name FOR SELECT USING (true);

-- Service role INSERT
CREATE POLICY "service role insert table_name"
  ON public.table_name
  FOR INSERT
  TO service_role
  WITH CHECK (true);

-- Service role UPDATE (if needed)
CREATE POLICY "service role update table_name"
  ON public.table_name
  FOR UPDATE
  USING (auth.role() = 'service_role');
```

Real example — chronicle-worlds `players` and `characters` (migration 008): [cite:32]
```sql
CREATE POLICY service_role_insert_characters ON characters
  FOR INSERT TO service_role WITH CHECK (true);
```

> ⚠️ If service_role operations fail with "relation not found" inside trigger functions, the issue is `search_path` — not RLS. Set `SET search_path = public` on the trigger function (see Watch-Outs).

---

### Pattern 3 — Authenticated User Scoped (own rows only)
Used for: player records, user profiles, any row that belongs to a specific user.

```sql
ALTER TABLE public.table_name ENABLE ROW LEVEL SECURITY;

-- User can only read their own row
CREATE POLICY "user read own"
  ON public.table_name
  FOR SELECT
  TO authenticated
  USING (user_id_column = auth.uid());

-- User can only update their own row
CREATE POLICY "user update own"
  ON public.table_name
  FOR UPDATE
  TO authenticated
  USING (user_id_column = auth.uid());
```

Real example — chronicle-worlds `players` (migration 008): [cite:32]
```sql
CREATE POLICY player_read_own ON players
  FOR SELECT TO authenticated
  USING (player_id = auth.uid());

CREATE POLICY player_update_own ON players
  FOR UPDATE TO authenticated
  USING (player_id = auth.uid());
```

Real example — scoping UPDATE through a join (migration 015): [cite:34]
```sql
-- Only update the character YOU control
CREATE POLICY authenticated_update_own_character
  ON public.characters FOR UPDATE TO authenticated
  USING (
    character_id IN (
      SELECT controlled_character_id
      FROM public.players
      WHERE player_id = auth.uid()
    )
  );
```

---

### Pattern 4 — Formal Schema + GRANT (non-public schemas)
Used only when a formal PostgreSQL schema (not `public`) was created. RLS alone is not enough — `GRANT USAGE` and `GRANT SELECT` on the schema and tables are also required for anon/authenticated access.

```sql
-- Step 1: Enable RLS (same as always)
ALTER TABLE schema_name.table_name ENABLE ROW LEVEL SECURITY;
CREATE POLICY "public_read_table" ON schema_name.table_name FOR SELECT USING (true);

-- Step 2: Grant schema access (REQUIRED for non-public schemas)
GRANT USAGE ON SCHEMA schema_name TO anon, authenticated;

-- Step 3: Grant table access
GRANT SELECT ON schema_name.table_name TO anon, authenticated;

-- Step 4: Grant view access if views exist in this schema
GRANT SELECT ON schema_name.view_name TO anon, authenticated;
```

Real example — customer-sales-data-practical `csdp` schema (migrations 004 + 005): [cite:37][cite:38]
```sql
-- Migration 004
ALTER TABLE csdp.customer ENABLE ROW LEVEL SECURITY;
CREATE POLICY "public_read_customer" ON csdp.customer FOR SELECT USING (true);

-- Migration 005 (separate file — GRANTs often need their own migration)
GRANT USAGE ON SCHEMA csdp TO anon, authenticated;
GRANT SELECT ON csdp.customer TO anon, authenticated;
GRANT SELECT ON csdp.v_open_orders TO anon, authenticated;
-- ... all tables and views
```

> ⚠️ This is the pattern most often missed. If a table is in a non-public schema and the frontend gets 404 or empty results (not a 403), the missing piece is almost always `GRANT USAGE ON SCHEMA` or a missing `GRANT SELECT`.

---

## Combining Patterns

A single table can have multiple policies for different roles and operations. The patterns combine freely:

```sql
-- entity_copies in chronicle-worlds:
-- Public can read. Service_role OR authenticated can insert.
CREATE POLICY "Public read entity_copies"
  ON public.entity_copies FOR SELECT USING (true);

CREATE POLICY "Service role or authenticated insert entity_copies"
  ON public.entity_copies FOR INSERT
  WITH CHECK (
    auth.role() = 'service_role'
    OR auth.uid() IS NOT NULL
  );
```

---

## Process

### Adding RLS to a New Table
1. `ENABLE ROW LEVEL SECURITY` immediately after `CREATE TABLE` — same migration
2. Decide which pattern applies (use the four patterns above)
3. Write policies for every operation that needs to be permitted
4. Verify: any operation NOT covered by a policy is **denied by default**
5. If non-public schema: add `GRANT USAGE` and `GRANT SELECT` in same or next migration
6. Test from the frontend using the anon key (not service_role) to confirm expected behavior

### Debugging a Permission Error

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| 403 on SELECT | RLS enabled, no SELECT policy | Add `FOR SELECT USING (true)` or scoped policy |
| Empty result, no error | Missing `GRANT USAGE ON SCHEMA` | Add GRANT migration (Pattern 4) |
| 403 on INSERT from Edge Function | Missing service_role INSERT policy | Add Pattern 2 INSERT policy |
| Trigger INSERT fails silently | `search_path` not set on function | Add `SET search_path = public` to trigger function |
| Authenticated user can't read own row | `auth.uid()` column mismatch | Verify column name matches what `auth.uid()` returns |
| Policy exists but still denied | Policy defined `TO authenticated` but anon key used | Add a separate anon policy or remove role restriction |

### Replacing a Policy
Policies cannot be edited — they must be dropped and recreated.
```sql
DROP POLICY IF EXISTS "old policy name" ON public.table_name;
CREATE POLICY "new policy name" ON public.table_name ...;
```
Always use `DROP POLICY IF EXISTS` to avoid errors on re-run.

---

## Policy Naming Convention

```
[role]_[operation]_[table]        ← snake_case (chronicle-worlds style)
"[role] [operation] [table]"      ← quoted sentence (cid / csdp style)
```

Both work. Be consistent within a project. The quoted style is more readable in the Supabase dashboard.

Examples:
- `service_role_insert_characters`
- `"Public read entity_copies"`
- `"public_read_customer"`
- `authenticated_update_own_character`

---

## Watch-Outs From Real Projects

| Issue | Source | What Happened | Fix |
|-------|--------|--------------|-----|
| Trigger INSERT blocked by RLS | chronicle-worlds migration 008 | service_role trigger couldn't insert — RLS was blocking it | Add explicit `service_role INSERT` policy |
| `search_path` error in trigger | chronicle-worlds migration 008 | `provision_player()` trigger couldn't find `public.players` | Add `SET search_path = public` to trigger function |
| Non-public schema, silent empty result | csdp (migration 005) | RLS was correct but `GRANT USAGE ON SCHEMA` was missing | Separate GRANT migration required |
| Policy replaced without DROP | Any project | `CREATE POLICY` fails if policy name already exists | Always `DROP POLICY IF EXISTS` first |
| `FOR ALL` covers too much | CID migration | `FOR ALL USING (auth.role() = 'service_role')` grants service_role SELECT too | Fine if intentional; use specific `FOR INSERT` if SELECT should be public |

---

## Verification Gates
- [ ] `ENABLE ROW LEVEL SECURITY` is present on every new table
- [ ] Every table has at least one explicit policy (no RLS-enabled table with zero policies — that blocks everything)
- [ ] Pattern selected matches the access requirement (public / service_role / scoped / schema)
- [ ] If non-public schema: `GRANT USAGE ON SCHEMA` and `GRANT SELECT` included
- [ ] DROP POLICY IF EXISTS used before any policy replacement
- [ ] Policy tested from the frontend using the anon key (not service_role)
- [ ] If trigger function writes to a table: `SET search_path = public` present on the function
- [ ] Policy added to migration file and committed — not only applied in the SQL editor

---

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "I'll add RLS after I get it working" | A table without RLS is open to the internet the moment it's created. Add it in the same migration. |
| "The frontend isn't using auth, so RLS doesn't matter" | The anon key is public. Anyone can call your Supabase URL. RLS is what prevents them from reading or writing everything. |
| "service_role bypasses RLS so I don't need policies for it" | service_role bypasses RLS by default — but trigger functions do NOT automatically run as service_role unless `SECURITY DEFINER` is set. Write explicit policies. |
| "I can't get the policy to work, I'll just disable RLS temporarily" | "Temporarily" disabled RLS has shipped to production more than once. Debug with the pattern table above instead. |

---

*Skill version: 1.0 — 2026-07-06 | Source evidence: chronicle-worlds (migrations 008, 011, 014, 015), community-infrastructure-development (migration 005), customer-sales-data-practical (migrations 004, 005)*
