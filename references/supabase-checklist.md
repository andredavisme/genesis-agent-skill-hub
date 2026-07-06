# Supabase Checklist

> Quick-reference gates for any Supabase table, migration, or Edge Function.

---

## New Table Checklist
- [ ] Schema prefix applied (e.g. `csdp.orders`, not `public.orders`)
- [ ] RLS enabled: `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`
- [ ] SELECT policy added for `anon` or `authenticated` as appropriate
- [ ] INSERT/UPDATE/DELETE policy scoped to `service_role` or authenticated user
- [ ] `REPLICA IDENTITY FULL` set if table used with Realtime
- [ ] Migration file created: `NNN_description.sql` with COMMIT/ROLLBACK header
- [ ] Migration tested on dev before applying to production

## New Migration Checklist
- [ ] File named `NNN_description.sql` (sequential, no gaps)
- [ ] Header comment includes: migration name, date, status (COMMIT/ROLLBACK), notes
- [ ] ROLLBACK file created alongside COMMIT file if risky change
- [ ] Never apply a `-- ROLLBACK` file to production
- [ ] Schema prefix on all objects

## New Edge Function Checklist
- [ ] File: `functions/[name]/index.ts`
- [ ] Version comment at top of file (e.g. `// resolve-turn v9`)
- [ ] Uses `service_role` key (never anon key server-side)
- [ ] CORS headers set for GitHub Pages origin
- [ ] Error handling returns structured JSON with status code
- [ ] Tagged ACTIVE in PROGRESS.md Quick Reference after deploy

## Pre-Launch Supabase Gate
- [ ] All tables have RLS enabled
- [ ] No anon key used server-side
- [ ] No secrets committed to the repo
- [ ] pg_cron jobs confirmed ACTIVE if required
- [ ] Realtime tables have REPLICA IDENTITY FULL

---

*Last updated: 2026-07-06*
