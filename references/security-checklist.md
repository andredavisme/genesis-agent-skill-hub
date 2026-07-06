# Security Checklist

> Pre-commit and pre-launch security gates for every project.

---

## Pre-Commit (Every Commit)
- [ ] No API keys, tokens, or passwords in committed files
- [ ] No Supabase `service_role` key in frontend code
- [ ] `.gitignore` covers `.env`, `*.key`, `secrets.*`
- [ ] Anon key is the only Supabase credential in frontend JS

## Supabase RLS
- [ ] RLS enabled on every table (`ENABLE ROW LEVEL SECURITY`)
- [ ] No table has `FOR ALL USING (true)` unless explicitly public read-only
- [ ] Authenticated user policies scope to `auth.uid()` not a hardcoded value
- [ ] `service_role` policies exist only for Edge Functions / server contexts

## Auth
- [ ] Magic link / OAuth flow tested end-to-end
- [ ] Session stored in `sessionStorage` (not `localStorage` for sensitive apps)
- [ ] No auth tokens logged to console
- [ ] Redirect URLs configured in Supabase Auth settings

## CORS
- [ ] Edge Functions return correct `Access-Control-Allow-Origin` for the Pages URL
- [ ] No wildcard `*` CORS in production Edge Functions unless intentionally public

## Pre-Launch
- [ ] All items above confirmed
- [ ] No debug `console.log` statements exposing data
- [ ] No test user credentials hardcoded anywhere

---

*Last updated: 2026-07-06*
