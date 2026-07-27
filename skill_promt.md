# 1. Base Prompt

Generate a comprehensive, reusable "skill.md' markdown file for building enterprise production-ready, highly secure, scalable, and optimized Front-end Web Applications

## Divide the 'skill.md' file into the following mandatory sections:

- Architectural Principles & Files Organization.
- Secure Backend-First Data Processing.
- Performance and User Experience (High Performance, Scalable).
- Database Optimization and Smart Querying (Database Indexing).

---

Here is an optimized and polished version of your prompt designed to elicit an expert-level, enterprise-grade response from an AI:

---

### 2. Frontend Prompt

### Optimized Prompt

> **Role & Persona:**
> Act as a Principal Frontend Engineer and Enterprise Software Architect. Your task is to generate a comprehensive, enterprise-grade, and reusable `SKILL.md` file that serves as an operational engineering standard for building highly secure, performant, scalable, and production-ready Frontend Web Applications.
> **Instructions & Guidelines:**
> Create a structured, highly actionable Markdown guide using clear headings, code examples, best practices, anti-patterns, and decision matrices. The standards must adhere strictly to modern frontend engineering (e.g., Next.js App Router, React, TypeScript, state management standards, and Web Vitals optimization).
> You **must** structure the `SKILL.md` file using the following mandatory sections:
>
> ---
>
> ### 1. Architectural Principles & File Organization
>
> - **Scalable Architecture:** Modular layout rules, feature-based directory structures (e.g., Domain-Driven / Feature-sliced design), and component hierarchy (Smart vs. Dumb components, slot patterns).
> - **Type Safety & Maintainability:** Strict TypeScript configurations, clean code rules, and boundary rules (e.g., zero circular dependencies, isolation of pure business logic).
> - **State Management Architecture:** Clean separation of Server State (e.g., React Query, SWR, Server Actions), Local UI State, and Global State, avoiding unnecessary re-renders.
>
> ### 2. Secure Backend-First Data Processing
>
> - **Client-Side Security:** OWASP Top 10 mitigations for frontend (XSS, CSRF, Clickjacking), CSP header implementations, and secure handling of sensitive data.
> - **Authentication & Authorization:** Secure JWT token lifecycle (HTTP-Only, SameSite cookies vs. memory storage), RBAC/ABAC route protection, and edge middleware auth guards.
> - **Backend-First Data Safety:** Server-side input validation/sanitization (Zod/Valibot schemas), data masking, avoiding sensitive prop drilling, and secure API boundary handling.
>
> ### 3. Performance & User Experience (High Performance & Scalable)
>
> - **Core Web Vitals Optimization:** Actionable strategies for LCP, INP, and CLS optimization (e.g., font loading, priority hints, layout stability strategies).
> - **Rendering & Code-Splitting Strategies:** Server-Side Rendering (SSR), Static Site Generation (SSG), Incremental Static Regeneration (ISR), Dynamic Imports, and `@defer` / Suspense boundary placement.
> - **Asset & Memory Optimization:** Efficient bundle budget management, tree-shaking, DOM virtualization, image optimization pipelines, and preventing memory leaks in client subscriptions.
>
> ### 4. Database Optimization & Smart Querying (Frontend & Edge Integration)
>
> - **Data Fetching Patterns:** Minimizing network waterfalls using parallel data fetching, streaming UI, optimistic UI updates, and request deduplication.
> - **Caching & Invalidation Strategies:** Stale-While-Revalidate configurations, aggressive edge caching, mutation invalidation keys, and offline-first/indexedDB persistence when applicable.
> - **Database Indexing & Query Awareness for Frontend Devs:** How frontend query structures (pagination, cursor-based feeds, filtering, sorting) dictate database index design (Composite Indexes, B-Tree, Full-Text Search) and backend response shapes.
>
> ---
>
> **Format Output:**
> Output **only** the complete `SKILL.md` file content inside a single continuous Markdown block. Ensure the tone is authoritative, highly technical, concise, and immediately actionable for senior development teams.

---

### Why this optimized prompt works better:

1. **Defines Expert Role:** Sets the target persona to a _Principal Frontend Engineer_, ensuring the output uses professional architectural depth.
2. **Clarifies Next-Gen Tech Stack:** Guides the AI to use modern patterns (TypeScript, Server Components, Zod, Edge Middleware, Signals/React Query) rather than legacy paradigms.
3. **Elevates Section #4 (Database Awareness for Frontend):** Directly links how frontend pagination/filtering patterns mandate specific backend index choices (like B-Tree vs Composite indexes), resolving the bridge between frontend queries and database performance.
4. **Enforces Output Precision:** Demands code examples, anti-patterns, and strict Markdown format so you get a drop-in ready `SKILL.md` file.

# ==================================================

# 3. Backend Prompt

# ==================================================

Here is an optimized and polished version of your prompt, crafted to ensure an AI generates an authoritative, deeply technical, enterprise-grade `SKILL.md` file for backend engineering:

---

### Optimized Prompt

> **Role & Persona:**
> Act as a Principal Backend Systems Architect and Enterprise Infrastructure Lead. Your task is to generate a comprehensive, enterprise-grade, and reusable `SKILL.md` file that serves as an operational engineering standard for building highly secure, resilient, performant, and scalable Backend Web Applications and Distributed Systems.
> **Instructions & Guidelines:**
> Create a structured, highly actionable Markdown guide using clear headings, code snippets, architectural patterns, anti-patterns, and concrete technical rules. The standards must reflect modern backend engineering practices (e.g. Nest best practices, Microservices/Modular Monoliths, REST/gRPC/GraphQL API design, Domain-Driven Design, distributed caching, event-driven architectures, and zero-trust security).
> You **must** structure the `SKILL.md` file using the following mandatory sections:
>
> ---
>
> ### 1. Architectural Principles & File Organization
>
> - **Domain-Driven Design (DDD) & Modular Architecture:** Strict boundary enforcement, separation of concerns (Presentation, Application, Domain, Infrastructure layers), and guidelines choosing between Modular Monoliths vs. Microservices.
> - **Scalable Directory Structures:** Feature-first / Domain-sliced file organization, dependency inversion principle (DIP), and clean abstraction layers for database driver and third-party integrations.
> - **Maintainability & Robust Design:** Anti-circular dependency strategies, strong typing schemas, contract-driven API design (OpenAPI/gRPC proto specifications), and graceful degradation strategies (Circuit Breakers, Bulkheads).
>
> ### 2. Secure Backend-First Data Processing
>
> - **Zero-Trust Security & Hardening:** OWASP Backend Top 10 mitigation strategies, strict input sanitization/validation at the edge, defense-in-depth API gateway policies, rate limiting (Token Bucket/Leaky Bucket), and DDoS defense.
> - **Authentication, Authorization & Identity:** Secure token lifecycles (short-lived JWTs, opaque refresh tokens, token revocation lists), RBAC/ABAC policy engines, secure password hashing (Argon2id/Bcrypt), and mTLS/internal service communication security.
> - **Data Protection & Compliance:** Secrets management (Vault/KMS), Encryption-at-Rest (AES-256) and Encryption-in-Transit (TLS 1.3), PCI-DSS/GDPR-compliant PII data masking, and tamper-proof audit logging.
>
> ### 3. Performance & System Scalability
>
> - **High-Throughput Concurrency & Async Processing:** Non-blocking I/O, event loop optimization, thread pool management, background worker queues (BullMQ, Celery, Temporal), and event-driven architectures (Kafka, RabbitMQ).
> - **Distributed Caching & Invalidation:** Multi-layer caching strategies (In-memory L1, Distributed L2 Redis/Memcached), cache Stampede protection (mutex locking, probabilistic early expiration), and cache invalidation patterns (Write-Through, Cache-Aside).
> - **Reliability & Telemetry:** Observability standards (Distributed Tracing with OpenTelemetry, Prometheus metrics, structured JSON logging), health check probes (Liveness/Readiness), and graceful shutdown sequences (draining connections, flushing queues).
>
> ### 4. Database Optimization & Smart Querying
>
> - **Database Indexing Strategies:** In-depth indexing patterns (B-Tree, Hash, GIN, GiST, Composite Indexes), avoiding index bloat, index selectivity analysis, and identifying/fixing sequential scans via `EXPLAIN ANALYZE`.
> - **Query Optimization & Anti-Pattern Prevention:** Eradicating N+1 query bugs, efficient cursor-based pagination over offset pagination, connection pooling configuration (PgBouncer, HikariCP), and managing read/write database replicas with lag awareness.
> - **Data Consistency & Schema Evolution:** Distributed transaction handling (Saga pattern, Two-Phase Commit), ACID vs. BASE trade-offs, zero-downtime database migrations (Expand-Contract/Parallel Run pattern), and partitioning/sharding large datasets.
>
> ---
>
> **Format Output:**
> Output **only** the complete `SKILL.md` file content inside a single continuous Markdown block. Ensure the tone is authoritative, highly technical, concise, and immediately actionable for staff engineers and technical leads.

---

### Key Improvements in this Optimized Version:

1. **Defines Seniority & Context:** Sets the persona to a _Principal Backend Systems Architect_ to guarantee enterprise-level depth rather than basic CRUD patterns.
2. **Covers Modern Infrastructure:** Expands topics to include distributed tracing (OpenTelemetry), queueing engines (Kafka/BullMQ), secrets managers, and connection poolers.
3. **Elevates Database & Security Mechanics:** Specifically prompts for real-world production topics like `EXPLAIN ANALYZE`, composite indexing, zero-downtime database migrations (Expand-Contract), circuit breakers, and rate-limiting algorithms.
4. **Strict Format Control:** Directs the AI to produce a clean, ready-to-use Markdown document without introductory conversational fluff.
