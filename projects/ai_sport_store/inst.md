# 1. data creating

Act as a Principal Database Architect.

We are building a production-grade sports equipment store using Next.js App Router, Drizzle ORM, and Neon PostgreSQL.

Requirements:

1. Enable `pgvector` for product semantic search using Google Gemini embeddings (`text-embedding-001 @ 768`).
2. Design the relational schema for NextAuth users, products, persistent database carts, and orders.
3. Before writing any code, analyze the architecture, list key trade-offs, and ask me specific clarifying questions about indexing strategies, inventory handling, and schema relationships.

```

```

# 2. some work

Act as a Senior AI Systems Engineer.

Implement an AI shopping assistant using the Google Gemini :

Cart persistence and checkout must be DB-backed ..

Core Invariants:

1. Provide a hybrid semantic search tool that queries `pgvector` with relational pre-filtering (price, category, in-stock).
2. Provide a checkout tool that calculates cart totals directly from the database (zero hallucinated math) and prepares a PayTabs payment session.

```

```

# 3. payment webhook

-Build an idempotent Webhook Route Handler that validates signatures, locks database records to prevent race conditions, updates order status, and decrements stock atomically.

- Before writing code, propose your concurrency control strategy and ask me how we should handle failed/cancelled payment webhooks.
- We need to save every payment, success or fail inside our database and never trust URL success from the front end. Our source of truth must be the back end and database
