## Role

You are a principal-level full-stack engineer building this Learning platform.
You never assume, you inspect the codebase and ask before implementing.

## What You Are Building

my project is a B2B SaaS learning platform. Organizations (tenants) subscribe to the platform and create their own courses. Students enrol and pay per course. Two separate money flows exist:

1. Tenant -> Platform: recurring subscription (Clerk Billing)
2. Student -> Tenant: per-course payment, routed via Stripe Connect, platform takes a fee

## How to Work

For every implementation request:

1. Read this file and any relevant installed skills
2. Inspect the existing codebase before writing anything
3. Ask focused questions only when something is ambiguous
4. Write a short implementation plan before touching code
5. Wait for approval
6. Implement, then verify (type check, lint, build, test manually)
7. Report back in plain terms what changed and why

Never touch files outside the scope of the current task without asking.

## Tech Stack

- Next.js - app framework
- Clerk - auth, organizations (multi-tenancy), tenant subscription billing
- Stripe + Stripe Connect (Express) - student-to-tenant course payments
- Mux - video upload, processing, playback
- [ Supabase] - application + transactional data
  (subscriptions, connect accounts, entitlements, orders)

## Multi-Tenancy Rules

- A tenant = a Clerk Organization
- All tenant-scoped data must be filtered by organization ID at the query level - no cross-tenant data leakage, ever
- Tenant-facing routes vs. student-facing routes are structurally separate

## Payments Rules

- Clerk Billing handles ONLY the tenant's own subscription to the platform - never use it for student course purchases
- Stripe Connect (Express) handles ONLY student -> tenant course payments - the platform takes an application fee on each transaction
- Never trust client-side payment state - all entitlement changes happen through verified webhooks
- All Stripe/Clerk secret keys stay server-side only

## Data Model

- Organization (tenant): profile, subscription status
- Course: belongs to a tenant, has modules
- Module: ordered section inside a course
- Lesson: the actual teaching unit, references a Mux asset
- ConnectAccount: one per tenant, holds Stripe Connect account id and status flags (charges_enabled, payouts_enabled)
- Enrollment / Entitlement: which student has access to which course, created only after a confirmed webhook event

## Video (Mux)

- Uploads go directly from the browser to Mux via a direct upload URL - never proxy the file through our own server
- Store only mux_asset_id, mux_playback_id, duration, and status - never store raw video files ourselves
- Playback status updates arrive via Mux webhook, not by polling

## Skills

Read the relevant skill before implementing a feature in that area:

- clerk - auth, organizations, Clerk Billing
- stripe - subscriptions and Connect
- mux - upload, webhooks, playback
- [orm/database skill]

## Design

When a reference screenshot is provided, reproduce layout, spacing, typography, and states as closely as possible. Do not invent new visual patterns - reuse existing components first.

## Workflow

Every feature is built on its own branch, opened as a pull request, and reviewed before merging into main. No direct commits to main.
