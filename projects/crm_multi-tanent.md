# B2B CRM Engineering & Security Guidelines

## 1. System Context & Architecture

- Multi-Tenant B2B CRM application built with Next.js App Router, shadcn/ui, Clerk, Supabase, and Vercel Eve.
- Primary Value Proposition: Unlimited seats per organization with AI-driven deal qualification and sales automation.

## 2. Multi-Tenancy & Security Rules (CRITICAL)

- Tenant Isolation: Every database query, tool execution, and state modification MUST be scoped to the active Organization ID (`org_id`) retrieved from Clerk session.
- Row Level Security (RLS): Enforce strict RLS policies on Supabase tables so that authenticated users can only access data belonging to their `org_id`.
- Never trust client-side organization identifiers; always derive `org_id` securely on the server or inside Vercel Eve context.

## 3. Vercel Eve & AI Subagents Architecture

- Structure agents clean and modular under the Eve directory structure (`instructions.md`, `agent.ts`, `tools/`, `subagents/`).
- `sales-copilot`: Handles natural language workspace queries (e.g., deal status, summaries, pipeline analytics) restricted strictly to the user's `org_id`.
- `qualifier-agent`: Evaluates incoming lead criteria, scores intent (Hot/Warm/Cold), and updates deal stages automatically via tools.
- All agent tools must validate the active subscription plan and tenant boundaries before executing mutations.

## 4. Subscription & Billing Workflows (Stripe & Clerk)

- Pricing Model: Unlimited users per organization; tiers are gated by lead capacity and AI usage credits.
- Free Tier Limits: 100 leads max, basic `sales-copilot` access only, zero automated background triggers.
- Pro Tier Gateways: Unlimited leads, full `qualifier-agent` access, automated email/CRM actions.
- Cancellation & Downgrade Behavior:
  - On subscription cancellation or payment failure (via Stripe Webhook), revoke Pro subagent triggers immediately.
  - Do not delete existing tenant data on downgrade; set workspace to read-only for excess leads beyond Free limits.
- Idempotency: Always handle Stripe webhooks idempotently using event ID verification.

## 5. UI/UX & Design System Principles

- Build interfaces using shadcn/ui and Tailwind CSS.
- Ensure all tables (Leads, Deals) use TanStack Table for fast filtering, sorting, and inline editing.
- Maintain a clean corporate SaaS aesthetic appropriate for B2B users.
