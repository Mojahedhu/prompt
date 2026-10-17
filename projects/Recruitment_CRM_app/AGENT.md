# AGENTS.md — Recruitment CRM (B2B SaaS)

## Tech Stack

- pnpm
- **Supabase (Postgres + pgvector)** — source of truth for all operational CRM data
- **Clerk** — multi-tenant auth, organizations, seats, billing (Pro plan, includes the AI Agent add-on)
- **Clerk Webhooks** — sync org/user/membership state from Clerk into Supabase
- **Vercel AI SDK (`streamText`) + OpenRouter (`@openrouter/ai-sdk-provider`)** — AI Copilot ("Astra"); model id is never hardcoded — see rule 8 below
- **pgvector** — used for two things: candidate $\rightleftharpoons$ job semantic matching, and Vetra's context retrieval
- Zod (validation), Vitest (unit tests), Playwright (integration tests)

## Non-Negotiable Rules

1. **Tenant isolation is mandatory.** Every table holding tenant data has a `tenant_id` column and a Row Level Security policy that filters by it. No user-facing code path may bypass RLS with the service role key.
2. **Never trust the client for tenant identity.** The tenant/org id always comes from the authenticated Clerk session on the server — never from a request body, query param, or client-side state.
3. **All mutations go through Zod schemas.** No unvalidated `request.json()` reaches the database layer.
4. **Clerk is the source of truth for identity. Supabase is the source of truth for CRM data.** Webhooks keep them in sync. Never write org/user state directly into Supabase outside the webhook flow — that causes drift.
5. **Vector search always filters by tenant first, then by similarity.** A candidate from Agency A must never appear in Agency B's matches,
