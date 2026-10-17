# First Database

## Role: Senior Backend Engineer.

## Task: Create the complete Drizzle ORM schema for AeroFlow with PostgreSQL and pgvector with neon db.

### Requirements:

1. Enable `pgvector` extension in PostgreSQL.
2. Tables required:

- `airports`: code (IATA), name, city, country.
- `flights`: flightNumber, originId, destinationId, departureTime, arrivalTime, basePrice.
- `aircraft_layouts`: aircraftModel, totalSeats, seatConfiguration (JSON).
- `seats`: flightId, seatNumber (e.g., '12A'), class (Economy, Business), baggageAllowanceKg,
  extraBaggagePricePerKg.
- `bookings`: pnr (unique), flightId, userId, totalPrice, status ('PENDING', 'CONFIRMED', 'CANCELLED'),
  stripePaymentIntentId.
- `passengers`: bookingId, fullName, passportNumber, selectedSeatId.
- `destination_guides`: city, country, content, embedding (vector(1536)).
- `processed_events`: id (stripe_event_id), processedAt (for webhook idempotency).

```

```

## Second db use redis

### Role: Senior Backend Engineer.

### Task: Build the atomic seat locking module using Redis (Upstash) in TypeScript.

#### Provide 3 functions:

1. holdSeat(flightId, seatId, userId): Lock a seat for 10 minutes. Return true if locked successfully, false if already taken.
2. releaseSeat(flightId, seatId, userId): Unlock the seat ONLY if it belongs to this user.
3. getLockedSeatsForFlight(flightId): Return a list of all currently locked seat IDs for this flight (to show on the UI map).

Requirements:

- Keep the code clean, modular, and well-typed.

```

```

# Third Database payment

Role: Senior Backend Engineer.
Task: Create the Stripe Webhook Route Handler in Next.js App Router for checkout completion.

Requirements:

1. Verify the request really came from Stripe using the webhook signature.

2. When a payment succeeds:
   - Check if this event was already processed before. If yes, stop early and return 200 OK (to avoid creating double tickets).
   - In a single database transaction (using Drizzle ORM):
     - Save the event ID so it won't run again.
     - Create the booking with a 6-character PNR code and status 'CONFIRMED'.
     - Save the passenger details with their chosen seat
     - Delete the temporary seat lock from Redis

```

```

# Four Database & AI

## Role: AI & Backend Engineer.

## Task: Implement the AI Concierge query engine using pgvector and Drizzle ORM.

### Requirements:

1. Create an API route that accepts a user prompt (e.g., "أريد رحلة شاطئية عائلية مع وزن زائد للأمتعة").
2. Generate embeddings for the prompt.
3. Use Drizzle's cosineDistance with pgvector to find the top matching destinations from `destination_guides` using the query embedding...

4. Query Flights & Seats:

- Use the matched destination cities and user criteria (dates, budget, baggage) to query the `flights` and `seats` tables with Drizzle

5. Format the Response:

- A list of recommended flights with prices and baggage details.
- Suggested filtered values (destination, fates, baggage) so the frontend search can auto-fill them.
- Keep the code clean, modular, and well-typed.

```

```

# Five Frontend

read design.md
agents.md and skills carefully
.. your are senior full stack ...

## Task: Build the visual Flight Seat-Map and Baggage selection UI in Next.js using Tailwind CSS.

### Requirements:

1. Display the aircraft cabin layout (Aisle, Windows, Row numbers).
2. Distinguish seat states visually: Available, Temporarily Held (Locked in Redis), and Booked (from PostgreSQL).
3. Include dynamic baggage selector (calculates extra weight cost per Kg based on the selected seat's `extraBaggagePricePerKg`).
4. On seat click:
   - Call Server Action to execute Redis lock (`holdSeat`).
   - Start a 10-minute visual countdown timer.
   - Redirect to Stripe Checkout session if lock succeeds; show instant alert if seat was taken.
5. Design all dropdown and calender by tailwind css using shadcn ui components...
6. keep the code clean, modular, and well-typed.

```

```

# Six Problems

## one (real time seat selection)

the problem is i need to refresh the page to see the locked seats when 2 users try to select the same seat at once .. we want sth real-time without refreshing

## two (add to the processed_event table)

extends the processed_event (payment) table to add more information about the event

```

```

# Seven (QR Code)

## Task: Implement an electronic boarding pass verification system using a QR Code.

When a flight booking is confirmed, generate a secure verification QR code linked to the unique booking PNR and embed it dynamically into both the Resend confirmation email template and the customer's web boarding-pass page. When the QR code is scanned using any mobile device camera, it must redirect to a dedicated, responsive verification route that instantly fetches the booking state from the database and visually confirms the ticket validity, passenger name, flight number, seat assignment, and luggage allowance without exposing sensitive data.

```

```
