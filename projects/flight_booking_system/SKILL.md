# Engineering Constraints & Standards

## 1. Concurrency & Seat Locking (Redis)

- To hold a seat: `SET seat_lock:{flightId}:{seatId} {userId} NX EX 600`.
- If key exists, reject hold immediately (Seat is already selected or locked).
- To release a seat: Delete key via Lua script ensuring only the lock owner can release it.

## 2. Stripe Webhook & Idempotency (PostgreSQL)

- Webhooks must verify Stripe signature first.
- Wrap ticket creation inside a PostgreSQL Transaction:
  1. Insert idempotency record or check existing `stripe_event_id`.
  2. Insert Passenger & Booking record.
  3. Mark seat status as `BOOKED` in the database.
  4. Release the Redis Lock.

## 3. Vector Search (pgvector via Drizzle)

- Use standard `cosineDistance` for policy/destination queries:
  `ORDER BY embedding <=> user_query_vector LIMIT 5`.
