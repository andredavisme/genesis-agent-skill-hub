---
name: supabase-edge-functions
description: Guides writing, structuring, and deploying Supabase Edge Functions (Deno/TypeScript). Use whenever a server-side operation is needed — action resolution, form handling, procedural generation, or any write that bypasses RLS.
---

# Supabase Edge Functions Skill

## Overview
Edge Functions are Deno TypeScript deployed to Supabase's global edge network. They run server-side with `SUPABASE_SERVICE_ROLE_KEY` access, bypassing RLS. Use them when the frontend cannot or should not perform the operation directly: multi-step writes, business logic, validation, or service-role inserts.

## When to Use
- Multi-step database writes that must succeed atomically
- Validation before insert (required field checks, enum checks, game rules)
- Writes that require service_role (trigger bypass, modifying other users' rows)
- Action resolution with side effects (stat changes, position updates, broadcasts)
- Any time the frontend needs to call a backend operation without exposing logic

---

## File Structure

```
functions/
  function-name/
    index.ts          ← entry point; must export a serve() or Deno.serve() handler
```

- One directory per function, kebab-case name
- No shared `_shared/` layer currently used — helpers are co-located or inlined
- File header comment: version, what it handles, version changelog, migration dependencies

---

## Two Import Styles in Use

Both are valid. Use whichever is already established in the project.

```ts
// Style A — deno.land CDN (resolve-turn, submit-proposal)
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

// Style B — jsr: (discover-cell — newer, preferred for new functions)
import 'jsr:@supabase/functions-js/edge-runtime.d.ts';
import { createClient } from 'jsr:@supabase/supabase-js@2';
```

> Prefer `jsr:` imports for new functions. `deno.land` imports work but are the older pattern.

---

## Required Boilerplate

Every function needs these three things. No exceptions.

### 1. CORS Headers
```ts
const CORS_HEADERS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};
```

Optionally add `'Access-Control-Allow-Methods': 'POST, OPTIONS'` if you want to be explicit (discover-cell pattern).

### 2. OPTIONS Preflight Handler
```ts
// resolve-turn / submit-proposal style:
if (req.method === 'OPTIONS')
  return new Response('ok', { headers: CORS_HEADERS });

// discover-cell style (returns 204, explicit method block):
if (req.method === 'OPTIONS')
  return new Response(null, { status: 204, headers: corsHeaders });
if (req.method !== 'POST')
  return new Response(JSON.stringify({ error: 'Method not allowed' }), { status: 405, headers: corsHeaders });
```

### 3. Supabase Client with Service Role
```ts
const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);
```

This must use `SERVICE_ROLE_KEY`, not the anon key. The function runs as service_role — it bypasses RLS.

---

## Skeleton Template

```ts
// function-name/index.ts  v1
// What this function does.
// Depends on: migrations NNN–NNN.

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const CORS_HEADERS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};

serve(async (req: Request) => {
  if (req.method === 'OPTIONS')
    return new Response('ok', { headers: CORS_HEADERS });

  try {
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    );

    let body: Record<string, unknown>;
    try {
      body = await req.json();
    } catch {
      return new Response(
        JSON.stringify({ error: 'Invalid JSON' }),
        { status: 400, headers: CORS_HEADERS }
      );
    }

    // Required field validation
    const required = ['field_one', 'field_two'];
    for (const field of required) {
      if (!body[field]) {
        return new Response(
          JSON.stringify({ error: `Missing required field: ${field}` }),
          { status: 400, headers: CORS_HEADERS }
        );
      }
    }

    // --- business logic here ---

    return new Response(
      JSON.stringify({ status: 'ok' }),
      { status: 200, headers: { ...CORS_HEADERS, 'Content-Type': 'application/json' } }
    );

  } catch (err) {
    return new Response(
      JSON.stringify({ error: String(err) }),
      { status: 500, headers: CORS_HEADERS }
    );
  }
});
```

---

## Function Patterns From Real Projects

### Pattern 1 — Validate → Insert (simple write)
Used by: `submit-proposal`

```ts
// 1. Parse JSON (wrap in try/catch)
// 2. Loop over required fields, return 400 for any missing
// 3. Validate enum values explicitly
// 4. Insert with service_role client
// 5. Return { status: 'submitted' } or the error message
```

Key detail from `submit-proposal`: JWT verification is disabled for Phase 1 (public form).
Phase 2 hook is commented in-place:
```ts
// Phase 2: validate invite_id here before insert
// const invite = await validateInvite(supabase, body.invite_id);
// if (!invite.valid) return new Response(invite.reason, { status: 403 });
```
Leave Phase 2 stubs in as comments rather than deleting them.

---

### Pattern 2 — Multi-step Action Resolution
Used by: `resolve-turn`

Step order matters — follow this sequence:
1. Race/queue check (return 202 if not player's turn)
2. Resolve actor (player → character lookup)
3. Validate action type; compute duration
4. Guard checks (branch fork limit, etc.) — fail fast before any writes
5. Insert event row (`resolution_state: 'pending'`)
6. Action-specific side effects (stat deltas, position updates)
7. If side effect fails → delete the event row and return error
8. Insert chronicle entry
9. Mark event `resolved`
10. Broadcast via Realtime channel
11. Return response with all deltas

Broadcast pattern:
```ts
await supabase.channel('turns').send({
  type: 'broadcast',
  event: 'turn_resolved',
  payload: { player_id, character_id, action, stat_deltas, ... },
});
```

---

### Pattern 3 — Idempotent Upsert / Procedural Generation
Used by: `discover-cell`

```ts
// 1. Check if entity already exists → return it immediately if so (no scaffold check)
// 2. Determine assignment (setting, parent, etc.)
// 3. Validate preconditions (z-scaffold check, capacity, etc.)
// 4. Insert new entity
// 5. Ensure dependent entities exist (entity_copy, etc.) — create if missing
// 6. Return { spawned: true/false, blocked: true/false, ... }
```

`discover-cell` also initializes the client at module scope (outside the handler), not inside:
```ts
// Module-scope client — fine for stateless read-heavy functions
const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);
```
For functions with complex per-request state or auth context, initialize inside the handler instead.

---

## Response Conventions

| Status | When |
|--------|------|
| 200 | Success — always include `Content-Type: application/json` |
| 202 | Accepted but not yet processed (e.g. queued turn) |
| 400 | Bad request — missing field, invalid JSON, unknown action |
| 403 | Forbidden — game rule violation (e.g. travel blocked) |
| 404 | Entity not found |
| 405 | Method not allowed (if you block non-POST explicitly) |
| 409 | Conflict — limit reached (e.g. branch fork limit) |
| 500 | Server error — always return `String(err)` as the error message |

Always spread CORS headers onto every response including errors:
```ts
{ status: 400, headers: CORS_HEADERS }
{ status: 200, headers: { ...CORS_HEADERS, 'Content-Type': 'application/json' } }
```

---

## Deploying a Function

```bash
# Deploy one function
supabase functions deploy function-name

# Deploy all functions
supabase functions deploy

# Verify JWT disabled (for public-facing functions)
# Set in Supabase Dashboard → Edge Functions → [function] → Verify JWT: off
```

> `submit-proposal` has JWT verification disabled — the form is public. `resolve-turn` expects an authenticated player. Check the function's header comment for the intended auth posture before deploying.

---

## Environment Variables

Edge Functions get `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` automatically from the Supabase runtime. No `.env` needed for those.

For any custom secrets:
```bash
supabase secrets set MY_SECRET=value
```
Access in function:
```ts
Deno.env.get('MY_SECRET')!
```

---

## Watch-Outs From Real Projects

| Issue | Source | What Happened | Fix |
|-------|--------|--------------|-----|
| Missing CORS on error response | Any function | Browser silently drops error — frontend sees network error, not 400/500 | Spread `CORS_HEADERS` onto every return, including errors |
| OPTIONS not handled | Any function | Preflight fails, all POST calls blocked | Always handle `req.method === 'OPTIONS'` first |
| `supabase.single()` throws on no row | resolve-turn player lookup | Unhandled rejection crashes function | Always destructure `{ data, error }` and check both |
| Event insert not rolled back on side effect failure | resolve-turn | Orphan `pending` event in DB | Delete the event row before returning the error |
| Version comment not updated | resolve-turn (v10) | Impossible to know which version is deployed | Increment version in header comment on every meaningful change |
| JWT verify not checked after Phase 2 stub | submit-proposal | Phase 2 invite validation never wired up | Stubs exist — don't delete them; wire them up when the phase arrives |
| `jsr:` vs `deno.land` mixed in same function | n/a | Import resolution errors | Pick one style per function; don't mix |

---

## Verification Gates
- [ ] `CORS_HEADERS` defined and spread onto every response (including errors)
- [ ] `OPTIONS` preflight handled at the top of the handler
- [ ] `SUPABASE_SERVICE_ROLE_KEY` used (not anon key)
- [ ] JSON parse wrapped in try/catch with 400 response
- [ ] All required fields validated with explicit 400 per missing field
- [ ] Guard checks (limits, rules) happen before any DB writes
- [ ] On side effect failure: event/insert rolled back before returning error
- [ ] Version comment in file header incremented
- [ ] `Depends on: migrations NNN–NNN` noted in header
- [ ] JWT verify setting confirmed in Supabase Dashboard matches intended auth posture
- [ ] Function deployed with `supabase functions deploy function-name`
- [ ] PROGRESS.md updated with function name and what it does

---

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "I'll add the CORS headers to the success response but not the errors — errors won't reach the browser anyway" | They will. The browser checks CORS on every response including 400/500. Missing CORS on an error = silent network failure. |
| "I'll skip the OPTIONS handler since my frontend always sends credentials" | Browsers send a preflight OPTIONS before the real request when headers like `authorization` are present. Skip it and all calls fail. |
| "I don't need to delete the event row if the side effect fails — I'll just mark it failed" | A `pending` event that never resolves corrupts the turn queue and chronicle permanently. Delete it. |
| "I'll use the anon key since this is just a read" | Edge Functions exist to run as service_role. If you only need anon reads, use the Supabase client directly from the frontend. |

---

*Skill version: 1.0 — 2026-07-06 | Source evidence: chronicle-worlds (resolve-turn v10, submit-proposal, discover-cell)*
