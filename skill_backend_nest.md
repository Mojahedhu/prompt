````markdown
# SKILL.md: Enterprise Backend Engineering Standard (NestJS)

**Owner:** Principal Backend Systems Architecture Group  
**Scope:** Backend Services, APIs, and Distributed Systems utilizing the NestJS Framework  
**Objective:** Establish strict operational and architectural standards for building highly secure, resilient, and horizontally scalable NestJS backend systems.

---

## 1. Architectural Principles & File Organization

### 1.1 Domain-Driven Design (DDD) & Modular Architecture

NestJS’s `@Module()` system natively supports Bounded Contexts. We strictly enforce a **Separation of Concerns** using DDD principles. Modules must encapsulate specific domains, and business logic must be isolated from framework-specific HTTP adapters.

- **Domain Layer:** Enterprise rules, entities, and interfaces. Must remain framework-agnostic where possible.
- **Application Layer (Services/CQRS):** Orchestrates use cases (Commands/Queries). Uses `@Injectable()` services or `@nestjs/cqrs` command handlers.
- **Infrastructure Layer:** External transports (`@Controller()`, `@MessagePattern()`), Repositories (Prisma/TypeORM adapters).

**Microservices vs. Modular Monolith Decision Matrix:**
Start with a **Modular Monolith** using decoupled `@Module()` boundaries. Extract to `@nestjs/microservices` only when independent scalability, independent deployment, or strict fault isolation is required.

### 1.2 Scalable Directory Structures

Code is organized by **Domain (Feature)**, adhering to a strict internal layer separation.

```text
src/
 ├── core/                        # Global interceptors, guards, filters, custom decorators
 ├── shared/                      # Cross-domain utilities (e.g., RedisModule, PrismaModule)
 └── modules/
      └── billing/                # Domain Boundary (BillingModule)
           ├── billing.module.ts
           ├── application/       # CQRS Handlers, Application Services
           ├── domain/            # Entities, Value Objects, Domain Events
           └── infrastructure/
                ├── controllers/  # REST / gRPC controllers
                ├── persistence/  # Prisma/TypeORM Repositories
                └── messaging/    # Kafka / RabbitMQ Event Handlers
```
````

**Dependency Inversion Principle (DIP):**
Use NestJS Dependency Injection (DI) tokens to inject infrastructure into application services.

- **Rule:** Application services depend on interfaces (e.g., `IBillingRepository`), which are provided via custom providers (`{ provide: 'IBillingRepository', useClass: PostgresBillingRepository }`).

### 1.3 Maintainability & Robust Design

- **Anti-Circular Dependencies:** The use of `forwardRef()` is considered an architectural smell and is heavily restricted. Modules should communicate via a shared domain or in-memory event buses (`EventEmitter2`) if boundaries are crossed.
- **Contract-First APIs:** Use `@nestjs/swagger` with strict DTO decorators (`@ApiProperty`) to generate OpenAPI 3.x specs. For microservices, use `.proto` files as the source of truth.
- **Graceful Degradation:** Implement Circuit Breakers via custom Interceptors or libraries like `nest-status-monitor` and `resilience4j` Node ports for all cross-service HTTP/gRPC calls.

---

## 2. Secure Backend-First Data Processing

### 2.1 Zero-Trust Security & Hardening

- **Input Validation at the Edge:** Global validation pipes are mandatory to strip malicious or unknown payloads.

```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true, // Strips properties without decorators
    forbidNonWhitelisted: true, // Throws error on unknown properties
    transform: true, // Auto-transforms payloads to DTO instances
  }),
);
```

- **Rate Limiting:** Enforce distributed rate limiting using `@nestjs/throttler` backed by Redis (`throttler-storage-redis`) to prevent DDoS and brute-force attacks across clustered instances.
- **Edge Hardening:** Mount `helmet` middleware globally. Fastify users must use `@fastify/helmet`.

### 2.2 Authentication, Authorization & Identity

- **Token Lifecycle:** Use `@nestjs/jwt` for short-lived access tokens (RS256). Refresh tokens must be opaque strings stored in the database with device fingerprinting and family-rotation enabled.
- **Declarative RBAC/ABAC:** Use Custom Guards and Decorators to enforce authorization at the controller or method level.

```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(Role.ADMIN)
@Post('process')
async processPayment(@Body() dto: PaymentDto) { ... }

```

- **Internal Communication:** NestJS microservice transports (TCP, Redis) are strictly for internal VPC use. For cross-cluster communication, use gRPC or Kafka with mTLS configurations enforced at the transport layer.

### 2.3 Data Protection & Compliance

- **Secrets Management:** Never use `process.env` directly in services. Use `@nestjs/config` with custom schema validation (Joi/Zod) to ensure the app fails at boot if secrets are missing. Secrets must be injected via Vault/KMS.
- **PII Masking:** Implement a global `@Injectable()` Interceptor (`CallHandler`) that uses a redaction library (like `fast-redact`) to mask sensitive data (SSN, credit cards) before serializing the response or sending it to the logger.

---

## 3. Performance & System Scalability

### 3.1 High-Throughput Concurrency & Async Processing

- **Fastify Engine:** For high-throughput I/O services, replace the default Express engine with the `FastifyAdapter` (`@nestjs/platform-fastify`), yielding up to 20-30% better request/sec performance.
- **Background Workers:** Defer heavy CPU/IO tasks using `@nestjs/bullmq` backed by Redis. Controllers must return HTTP 202 Accepted while workers process jobs asynchronously.
- **Event-Driven Architecture:** Use `@nestjs/microservices` with the Kafka or RabbitMQ transporters for inter-service choreographies. Do not use synchronous `@nestjs/axios` calls across bounded contexts unless absolutely necessary.

### 3.2 Distributed Caching & Invalidation

- **Multi-Layer Caching:** Use `@nestjs/cache-manager` configured with a Redis store.
- **Cache-Aside with Interceptors:** Cache declarative HTTP responses using `@UseInterceptors(CacheInterceptor)`. Use custom cache keys derived from the request object and JWT tenant IDs.
- **Cache Stampede Protection:** For heavily queried aggregations, implement a custom provider that utilizes a Redis Mutex (SETNX) lock or probabilistic early expiration to ensure only one NestJS instance regenerates the cache.

### 3.3 Reliability & Telemetry

- **Observability:** Integrate `nestjs-otel` (OpenTelemetry) to automatically instrument controllers, TypeORM/Prisma queries, and HTTP requests. Ensure `trace-id` propagation across message queues.
- **Health Probes:** Use `@nestjs/terminus` to expose `/health/liveness` and `/health/readiness` endpoints. Include DB pings, Redis connections, and Kafka consumer lag in the readiness checks.
- **Graceful Shutdown:** You must explicitly enable shutdown hooks to allow NestJS to close DB pools and drain queues on `SIGTERM`.

```typescript
// main.ts
const app = await NestFactory.create(AppModule);
app.enableShutdownHooks();
```

---

## 4. Database Optimization & Smart Querying

### 4.1 Database Indexing Strategies

- **Index Selectivity:** Indexes are managed via ORM schema files (Prisma `.schema` or TypeORM `@Index()`). Follow the **Equality, Sort, Range** rule for composite indexing.
- **Monitoring Scans:** Utilize APM tools (e.g., Datadog, New Relic) hooked into TypeORM logger/Prisma extensions to flag queries running sequential scans over large tables.

### 4.2 Query Optimization & Anti-Pattern Prevention

- **Eradicating N+1 Queries:** TypeORM lazy relations are banned. Use `@nestjs/graphql` with `DataLoader` to batch and deduplicate nested entity resolution. For REST, utilize query builders or Prisma `include` to fetch relations explicitly in a single tick.
- **Cursor-Based Pagination:** Offset pagination (`skip`/`take` with large offsets) is forbidden for enterprise feeds. Implement Keyset/Cursor pagination utilizing B-Tree indexes.
- **Connection Pooling:** NestJS instances scale horizontally. You must connect TypeORM/Prisma through an external connection pooler (e.g., **PgBouncer**) rather than relying solely on the ORM’s internal pool, preventing DB connection exhaustion during auto-scaling events.

### 4.3 Data Consistency & Schema Evolution

- **Saga Pattern (Distributed Transactions):** `@nestjs/cqrs` provides a `Saga` implementation using RxJS Observables. Use this to orchestrate cross-module distributed transactions with explicit compensating events, avoiding database-level locking (2PC).
- **Local Transactions:** When updating multiple tables within the same database, wrap calls in a TypeORM `QueryRunner` transaction or Prisma `$transaction` API.
- **Zero-Downtime Migrations:** Execute structural schema changes using the **Expand-Contract Pattern**. Schema migrations (e.g., Prisma Migrate, TypeORM Migrations) must be entirely decoupled from the NestJS application lifecycle and executed via independent CI/CD jobs prior to application deployment.

```

```
