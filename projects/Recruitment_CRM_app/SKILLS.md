```yaml
---
name: recruitment-crm-standards
description: >-
  Core standards for this multi-tenant Recruitment CRM SaaS project
  (Next.js + Supabase + Clerk + pgvector).
  Use this skill for ANY code change in this repo - new features, bug
  fixes, schema changes, or work on the
  AI agent ("Vetra"). Covers mandatory tenant isolation (RLS),
  Clerk↔Supabase sync rules, vector-search
  tenant filtering, Zod validation, and audit-logging requirements.
  Always consult this skill before writing
  or modifying any file in this project, even for small changes.
---
```

# Recruitment CRM — Project Standards

## Before Writing Any Code

1. Identify which tenant-scoped table(s) this change touches. A new table needs `tenant_id` + RLS from the start — not added later as an afterthought.
2. If this touches Clerk (organizations, users, seats, billing), check whether the Supabase webhook handler needs updating too, so the two stay in sync.
3. If this touches candidate/job matching or the AI Copilot, confirm the tenant filter is applied **before** the vector similarity search — never as a filter on the results afterward.

## Hard Rules

- The Supabase service role key is only ever used inside webhook handlers — never in a user-facing request path.
- Every new mutation ships with a matching Zod schema in the same change.
- Every Kanban stage transition is checked against the allowed state machine (Source $\rightarrow$ Screening $\rightarrow$ Interviewing $\rightarrow$ Offer $\rightarrow$ Placed/Hired, or Withdrawn from any stage). The client cannot silently skip stages.
- Any tool the AI agent (Vetra) can call must receive `tenant_id` as a fixed value from the server session – the LLM itself never chooses or supplies the tenant id.

## When Unsure

Ask before assuming: which role (Admin / Recruiter / Coordinator) is allowed to perform this action, and whether it needs to be written to `audit_logs`.
