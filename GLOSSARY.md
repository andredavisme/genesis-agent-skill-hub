# Glossary

> Project-specific terms, abbreviations, and concepts used across Andre's work.
> Add new terms as they emerge. Alphabetical order within sections.

---

## General

| Term | Definition |
|------|------------|
| Agent brain | The collection of context files (AGENTS.md + project card + skills) pasted into an AI session |
| Genesis Hub | This repository — `andredavisme/genesis-agent-skill-hub` |
| Living document | A file maintained as the single source of truth, updated continuously rather than archived |
| No-build-step | A frontend approach where HTML/JS/CSS are served directly without a bundler or compiler |
| Session handoff | The practice of ending a session with a `SESSION_HANDOFF.md` update so any agent can resume |
| Skill | A SKILL.md file encoding a reusable workflow — steps, rules, verification gates |

## Supabase

| Term | Definition |
|------|------------|
| anon key | Supabase publishable key — safe for frontend, SELECT only by convention |
| Edge Function | Serverless TypeScript function running on Deno at the Supabase edge |
| Migration | A numbered SQL file (`NNN_description.sql`) that evolves the database schema |
| pg_cron | PostgreSQL extension for scheduled SQL jobs — used for world ticks and automation |
| REPLICA IDENTITY | PostgreSQL setting required on tables used with Supabase Realtime (`ALTER TABLE ... REPLICA IDENTITY FULL`) |
| RLS | Row Level Security — Supabase/PostgreSQL policy controlling who can read/write each row |
| service_role | Supabase server-side key — bypasses RLS, used only in Edge Functions or secure server contexts |

## Project Codes

| Code | Project |
|------|---------|
| csdp | customer-sales-data-practical |
| indB2B | industrial-maintenance-b2b |
| cid_ | community-infrastructure-development |

## Status Indicators

| Symbol | Meaning |
|--------|---------|
| ✅ | Complete |
| 🔼 | Next / in queue |
| 🟡 | In progress |
| 🔴 | Blocked |
| ⚡ | Partial adoption |
| ❌ | Trimmed / not applicable |

---

*Last updated: 2026-07-06*
