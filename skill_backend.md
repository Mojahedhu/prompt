````markdown
# SKILL.md: Enterprise Backend Engineering Standard

**Owner:** Principal Backend Systems Architecture Group  
**Scope:** Backend Services, APIs, Distributed Systems, and Databases  
**Objective:** Establish strict operational and architectural standards for building highly secure, resilient, and horizontally scalable backend systems.

---

## 1. Architectural Principles & File Organization

### 1.1 Domain-Driven Design (DDD) & Modular Architecture

We strictly enforce a **Separation of Concerns** using DDD principles. Business logic must be entirely isolated from framework-specific code, databases, and external transports.

- **Domain Layer:** Enterprise business rules, entities, and value objects. Zero external dependencies.
- **Application Layer:** Orchestrates use cases. Interacts with the domain layer.
- **Infrastructure Layer:** Database adapters, API controllers, external service clients (gRPC/REST).

**Microservices vs. Modular Monolith Decision Matrix:**
Start with a **Modular Monolith**. Extract to Microservices _only_ when one of the following thresholds is met:

1.  **Independent Scalability:** A specific domain requires drastically different compute/memory profiles (e.g., heavy PDF generation vs. CRUD).
2.  **Independent Deployment:** Teams are blocked by monolithic release cycles.
3.  **Fault Isolation:** A volatile domain threatens the stability of core systems.

### 1.2 Scalable Directory Structures

Code is organized by **Domain (Feature)**, not by technical role.

```text
src/
 ├── shared/                      # Cross-domain utilities (logging, base entities)
 └── modules/
      └── billing/                # Domain Boundary
           ├── application/       # Use cases (e.g., ProcessPaymentCommand.ts)
           ├── domain/            # Entities, Value Objects, Domain Events
           └── infrastructure/
                ├── controllers/  # REST / gRPC / GraphQL endpoints
                ├── persistence/  # TypeORM / Prisma repositories
                └── messaging/    # Kafka / RabbitMQ consumers
```
````

**Dependency Inversion Principle (DIP):**
The Application layer defines interfaces (Ports); the Infrastructure layer implements them (Adapters).

- **Rule:** Controllers and Use Cases must _never_ import database drivers directly.

### 1.3 Maintainability & Robust Design

- **Anti-Circular Dependencies:** Enforced via `madge` or ESLint boundary rules. Cross-domain communication in a monolith must happen via in-memory event buses, not direct service imports.
- **Contract-First APIs:** API definitions (OpenAPI 3.x, gRPC Protobufs) must be defined _before_ code is written. Code is generated from the contract, ensuring clients and servers never drift.
- **Graceful Degradation:** Use **Circuit Breakers** (e.g., `resilience4j` or Node equivalent) for all synchronous cross-service HTTP/gRPC calls.

```typescript
// Anti-Pattern: Unprotected external call
const data = await externalApi.fetch();

// Standard: Wrapped in a Circuit Breaker with Fallback
const data = await circuitBreaker.execute(
  () => externalApi.fetch(),
  fallbackData,
);
```

---

## 2. Secure Backend-First Data Processing

### 2.1 Zero-Trust Security & Hardening

Assume the internal network is compromised. All requests, whether external or internal (service-to-service), must be authenticated and authorized.

- **Input Validation at the Edge:** All incoming payloads (REST, gRPC, queues) must be strictly validated against a schema (e.g., Zod, Class-Validator) stripping unknown properties.
- **Rate Limiting:** Implemented at the API Gateway and Service level using a **Token Bucket** algorithm (e.g., via Redis) to absorb bursts while preventing DDoS.

```typescript
// Redis-backed Token Bucket Ratelimit
await redisRateLimiter.consume(req.ip, 10); // Max 10 req/sec
```

### 2.2 Authentication, Authorization & Identity

- **Token Lifecycle:**
- **Access Tokens:** Stateless JWTs, short-lived (maximum 15 minutes), signed with asymmetric keys (RS256).
- **Refresh Tokens:** Opaque strings, long-lived, stored in a backend database with device fingerprinting and family-rotation enabled (for immediate revocation).

- **Password Hashing:** **Argon2id** is mandatory for new systems. Bcrypt is legacy.
- **Internal Communication:** Mutual TLS (mTLS) is required for all service-to-service communication to guarantee transit encryption and cryptographic identity verification.

### 2.3 Data Protection & Compliance

- **Secrets Management:** Environment variables must _never_ contain plaintext secrets. Use HashiCorp Vault or AWS KMS to inject credentials at runtime.
- **Encryption:**
- _At-Rest:_ AES-256-GCM.
- _In-Transit:_ TLS 1.3 minimum.

- **PII Masking & Audit:** All logs containing PII (emails, phone numbers, SSNs) must be masked via structured logging redaction hooks. High-risk mutations must generate immutable audit logs appended to a WORM (Write Once, Read Many) datastore.

---

## 3. Performance & System Scalability

### 3.1 High-Throughput Concurrency & Async Processing

- **Event-Driven Architecture:** Synchronous request-response chains across multiple services are strictly prohibited. Use **Kafka** or **RabbitMQ** for cross-service choreographies.
- **The Outbox Pattern:** To guarantee data consistency between the database and the message broker, do not dual-write. Write the domain entity and an "Outbox Event" in a single ACID database transaction, then use a background worker (e.g., Debezium) to publish the event.
- **Background Workers:** Defer heavy I/O or CPU-bound tasks (email dispatch, PDF generation) to distributed worker queues (e.g., BullMQ, Temporal).

### 3.2 Distributed Caching & Invalidation

- **Cache-Aside Pattern:** Services read from Cache (Redis). On miss, read from DB, write to Cache, return to client.
- **Defeating Cache Stampedes:** High-traffic keys expiring simultaneously will crash the database.
- _Rule:_ Implement **Probabilistic Early Expiration (PERF)** or use a **Redis SETNX Mutex** so only one worker fetches the new data while others serve stale data.

```typescript
// Mutex Cache Lock Strategy
if (await redis.setnx("lock:reports", 1)) {
  redis.expire("lock:reports", 10);
  const data = await db.query("...");
  await redis.set("reports", data);
  await redis.del("lock:reports");
}
```

### 3.3 Reliability & Telemetry

- **Distributed Tracing:** OpenTelemetry is mandatory. A `trace-id` must be injected at the API Gateway and propagated through all HTTP headers, message queues, and database logs.
- **Structured Logging:** Standardize on JSON formatted logs (e.g., `pino` or `zap`). String concatenation logging is forbidden.
- **Graceful Shutdown:** On `SIGTERM`, applications must:

1. Fail Kubernetes Readiness probes.
2. Stop accepting new HTTP/Queue requests.
3. Allow inflight requests to finish (max timeout 30s).
4. Close database and Redis connection pools gracefully.

---

## 4. Database Optimization & Smart Querying

### 4.1 Database Indexing Strategies

Indexes speed up reads but penalize writes. Every index must justify its existence.

- **Composite Index Rule:** Follow the **Equality, Sort, Range** rule.
- Order columns in the index by: `(1) Equals checks -> (2) Sort checks -> (3) Range checks`.

- **Index Selectivity:** Do not index boolean columns or low-cardinality data (e.g., `status = 'ACTIVE'`) unless combined with a high-cardinality column, or created as a **Partial Index** (`WHERE status = 'PENDING'`).
- **Query Analysis:** CI/CD pipelines should run `EXPLAIN ANALYZE` on long-running queries to catch Sequential Scans before they hit production.

### 4.2 Query Optimization & Anti-Pattern Prevention

- **Eradicating N+1 Queries:** ORM lazy-loading is strictly banned. Use the **DataLoader** pattern to batch and deduplicate secondary lookups in a single tick.
- **Cursor-Based Pagination:** Offset pagination (`OFFSET 10000 LIMIT 50`) forces the DB to scan and discard 10,000 rows. Use Keyset/Cursor pagination for infinite feeds.

```sql
-- Fast, relies on B-Tree Index
SELECT * FROM users WHERE (created_at, id) < ('2023-10-01', 543) ORDER BY created_at DESC LIMIT 50;

```

- **Connection Pooling:** Applications must connect to databases through a transaction-level connection pooler (e.g., **PgBouncer** or **HikariCP**). Direct infinite connections from stateless workers will exhaust DB memory.

### 4.3 Data Consistency & Schema Evolution

- **Distributed Transactions:** Two-Phase Commit (2PC) is highly discouraged due to locking overhead. Use the **Saga Pattern** (Choreographed or Orchestrated) with explicit compensating transactions for rollbacks.
- **Zero-Downtime Migrations (Expand-Contract Pattern):** Renaming or deleting columns natively locks tables and causes downtime.
- _Phase 1 (Expand):_ Add the new column `v2`. Update code to write to both `v1` and `v2`, but read from `v1`.
- _Phase 2:_ Run a background script to backfill `v2` data from `v1`.
- _Phase 3:_ Update code to read from `v2`.
- _Phase 4 (Contract):_ Drop `v1` in a future release.

```

```
