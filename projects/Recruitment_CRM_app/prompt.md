# first

Design the Supabase schema for a multi-tenant recruitment CRM. Tenants are recruitment agencies (Clerk organizations). Core entities: candidates, clients (employer companies), jobs, pipeline stages, interviews, offers, audit_logs. Every tenant-owned table must have tenant_id and RLS policies scoped to it. Propose the schema and RLS policies, and flag any entity where tenant isolation is not straightforward.

- 1. How should tenant_id be represented across every table?

**Clerk org id as text PK (Recommended)**
tenants.id is text holding the Clerk org id (org_xxx). Every tenant table has tenant_id text references tenants(id). RLS is a direct string compare against the JWT claim — no lookup, no helper function, fastest possible policy.

- 2. Tenant key **Stages** Candidate ID Roles

**Are pipeline stages fixed, or configurable per agency?**

**Per-tenant table + canonical kind (Recommended)**
pipeline_stages is a tenant-scoped table agencies can rename/reorder, but each row carries a canonical kind enum. The state machine in SKILLS.md is enforced on kind, so customization can never let a client skip a stage.

- 3. **Same human candidate applies via two different agencies. How is that stored?**

**Fully duplicated per tenant (Recommended)**
Each agency owns its own candidate row, even for the same person. Nothing is shared. Absolute isolation, trivial RLS, and one agency's notes/resume can never leak. Cost: no cross-tenant dedup, and a GDPR erasure request must fan out across tenants.

- 4. **How should role checks (Admin / Recruiter / Coordinator) be enforced in RLS?**

**Clerk JWT claim + mirrored table (Recommended)**
RLS reads the role straight from the Clerk token (o.rol) so policies need no join. Memberships are also mirrored into Supabase via webhook, used only for joins and audit display — never as the auth decision.

# Second

- Set up Clerk Webhooks to sync organization and user state into Supabase (organizations, memberships, roles: Admin/Recruiter/Coordinator). Focus on: webhook signature verification, idempotency (a webhook can be delivered more than once), and what should happen to a user's UI access if they're removed from an org mid-session.

# three

- Step 3 — Billing Logic (Clerk: Free vs Pro plan)

We have two plans: Free (no AI agent access) and Pro (includes the AI agent add-on). Review the logic for: what happens server-side when an agency hits its seat limit, what happens to existing candidates/jobs data if an agency downgrades or cancels, and — specifically — how do we make sure a Free-plan tenant can never trigger an AI agent call even if they know the API route, not just have the button hidden in the UI. Flag any other billing edge case specific to a recruitment CRM.

_I haven't created or activated any organization, any billing inside the clerk. I've just only created the authentication. So guide me what do I need inside clerk dashboard to activate all of them._

# four

- **Free limits** | **Seat limit** | **Downgrade** | **Past due**

**What does the Free plan actually cap?**

- **Seats only, unlimited records (Recommended)**
  Free gets 2 seats, no AI agent, but unlimited candidates/jobs. Simplest to enforce and to explain, and record caps in a CRM tend to punish exactly the trial usage that leads to conversion.

---

**An agency on a 2-seat plan tries to add a 3rd recruiter. What should the server do?**

- **Block, with upgrade prompt (Recommended)**
  Server rejects the invite with a 402 naming the limit. Deterministic and never produces a surprise invoice. Pending (unaccepted) invitations count against the limit so three simultaneous invites can't race past two seats.

---

**An agency downgrades to Free or cancels. They have 4,000 candidates and 30 open jobs. What happens to that data?**

- **Read-only + export, retained (Recommended)**
  All data stays and stays readable and exportable; writes are refused. For a recruitment CRM the pipeline IS the customer's business, so deleting or hiding it on a failed card is both a support catastrophe and a data-protection problem.

---

**A Pro agency's card fails and the subscription goes past_due. When does access change?**

- **Grace period, then read-only (Recommended)**
  Keep full access for a grace window (e.g. 14 days) while Clerk retries the card, showing an in-app banner. Then fall to read-only. Cutting a recruiting agency off mid-placement over an expired card loses the customer, not just the payment.

# five

- Step 4 — Kanban Pipeline Logic

Define the state machine for candidate pipeline stages (Source $\rightarrow$ Screening $\rightarrow$ Interviewing $\rightarrow$ Offer $\rightarrow$ Placed/Hired, or Withdrawn from any stage). Which transitions should be blocked, which need confirmation, and which must always be logged to audit_logs? Also flag any race-condition risk if two recruiters move the same candidate at the same time.

# six

[https://clerk.com/docs/nextjs/reference/components/billing/pricing-table](https://clerk.com/docs/nextjs/reference/components/billing/pricing-table)

we need to remove the mock pricing table inside the app and replace it with pricingTable component from clerk ...check this image and url so you can add any related components also like CheckoutButton

# seven

We need to make sure that the webhook is working between clerk and supabase after i made integration between them so we can see the plan pro or free inside supabase and everything related
Also create a organization table in supabase to store all related information.

# eight

Step 5 — Candidate–Job Matching (pgvector)

Design the semantic matching logic between candidates and open jobs using pgvector. The critical constraint: matching must never return a candidate from a different tenant, even with a high similarity score. Show how the query structure guarantees tenant filtering happens before the vector similarity comparison, not after.

---

Astra is an AI agent (Vercel AI SDK streamText via the OpenRouter provider) that summarizes interviews, drafts client emails, and suggests matched candidates.

# ten

The candidate's pipeline stage must have a single source of truth — one table/column, read by Talent Pool, Client Ledger, and Interviews,
