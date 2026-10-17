# Role & Architecture Definition: AeroFlow (Flight Engine)

## Overview

AeroFlow is a direct-carrier flight booking platform (Single Airline / Fleet Management) built with Next.js (App Router), Drizzle ORM, PostgreSQL
(with pgvector), Redis (Upstash), and Stripe.

## Tech Stack & Architecture

- **Framework:** Next.js (TypeScript, Server Actions, Route Handlers)
- **Database:** PostgreSQL with `pgvector` extension for semantic context retrieval.
- **ORM:** Drizzle ORM.
- **Seat Lock / Concurrency:** Redis key-value store with TTL (Two-Phase Seat Reservation).
- **Payments:** Stripe Checkout & Webhooks (Idempotent processing).
- **AI Concierge:** Pure Semantic Search & Travel Advisor (Tool calling via structured Drizzle filters + Cosine similarity over destinations).

## Core Rules

1. Never write a seat reservation directly to PostgreSQL before payment confirmation.
2. The seat MUST be locked in Redis (`NX` flag + 10-minute TTL) during the checkout flow.
3. Stripe Webhooks must be strictly idempotent to prevent duplicate ticket generation.
4. AI Concierge is read-only regarding state modifications; it only queries, matches semantics, and populates UI filters.
