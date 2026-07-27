````markdown
# SKILL.md: Enterprise Frontend Engineering Standard

**Owner:** Principal Engineering & Architecture Group  
**Scope:** Frontend Web Applications (React, Next.js App Router, TypeScript)  
**Objective:** Define strict operational standards for secure, scalable, and high-performance frontend architectures.

---

## 1. Architectural Principles & File Organization

### 1.1 Scalable Architecture

We adhere to a pragmatic implementation of **Feature-Sliced Design (FSD)** combined with Next.js App Router paradigms. This enforces modularity and prevents monolithic "spaghetti" codebases.

**Directory Structure:**

```text
src/
 ├── app/              # Routing, Layouts, and Entry points (Next.js specific)
 ├── features/         # Feature modules (e.g., /auth, /checkout, /dashboard)
 │    └── [feature]/   # Co-located components, hooks, utils, and domain logic
 ├── shared/           # Cross-feature reusable abstractions
 │    ├── ui/          # Dumb components (Buttons, Inputs)
 │    ├── lib/         # Utility functions, API clients
 │    └── types/       # Global TypeScript declarations
```
````

**Component Hierarchy Rules:**

- **React Server Components (RSC) by Default:** Treat Server Components as "Smart" containers. They handle data fetching, backend authorization, and passing primitive props.
- **Client Components (Islands):** Use the `"use client"` directive strictly at the leaf nodes. Client components are "Dumb" views handling interactivity, browser APIs, and local state.
- **Slot Pattern (Inversion of Control):** Prevent prop-drilling and unnecessary client boundaries by passing Server Components as `children` or named slots to Client Components.

### 1.2 Type Safety & Maintainability

TypeScript is our primary static analysis tool. `any` is strictly prohibited.

**Strict Configuration (`tsconfig.json`):**

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "isolatedModules": true
  }
}
```

- **Boundary Enforcement:** Features cannot import from other features directly. Cross-feature communication must happen at the `app/` (page) level or via shared event buses.
- **Zero Circular Dependencies:** Enforced via `eslint-plugin-import` (`import/no-cycle`).

### 1.3 State Management Architecture

State is categorized and isolated based on its lifecycle and persistence requirements.

| State Type          | Responsibility                    | Approved Tooling                          | Anti-Pattern                             |
| ------------------- | --------------------------------- | ----------------------------------------- | ---------------------------------------- |
| **Server State**    | Async data, caching, revalidation | Next.js Cache, React Query, SWR           | Storing API responses in Redux/Zustand   |
| **Global UI State** | Theme, cross-feature UI state     | Zustand, React Context (if low frequency) | Prop-drilling 5+ levels deep             |
| **Local State**     | Form inputs, toggles, active tabs | `useState`, `useReducer`                  | Elevating local state to Global State    |
| **URL State**       | Pagination, filters, search       | Next.js `useSearchParams`, `nuqs`         | Syncing URL state to `useState` manually |

---

## 2. Secure Backend-First Data Processing

### 2.1 Client-Side Security (OWASP Top 10 Mitigations)

- **XSS (Cross-Site Scripting):** Never use `dangerouslySetInnerHTML` unless parsing explicitly sanitized markdown (via `DOMPurify`). React escapes variables by default.
- **CSP (Content Security Policy):** Must be configured in `next.config.js` or Edge Middleware to whitelist script execution, frame embedding, and API endpoints.

```javascript
// middleware.ts example header setup
const cspHeader = `
  default-src 'self';
  script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
  connect-src 'self' [https://api.enterprise.com](https://api.enterprise.com);
  frame-ancestors 'none';
`;
```

- **Clickjacking:** Enforce `X-Frame-Options: DENY` and CSP `frame-ancestors 'none'`.

### 2.2 Authentication & Authorization

- **JWT Token Lifecycle:** **Never** store access or refresh tokens in `localStorage` or `sessionStorage`. Tokens must be issued as `HttpOnly`, `Secure`, `SameSite=Strict` cookies.
- **Edge Middleware Guards:** Route protection must occur at the Edge before rendering begins.

```typescript
// proxy.ts
export default function proxy(req: NextRequest) {
  const session = req.cookies.get("session")?.value;
  if (!session && req.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
}
```

- **RBAC/ABAC:** Role-Based Access Control is evaluated on the server. Do not rely on client-side hiding (e.g., `if (isAdmin) <AdminPanel />`) as a security measure; backend endpoints must also verify roles.

### 2.3 Backend-First Data Safety

- **Input Validation:** All client-to-server data (e.g., Server Actions, API routes) must be validated using **Zod** or **Valibot**.

```typescript
const UpdateProfileSchema = z.object({
  email: z.string().email(),
  age: z.number().min(18).max(120),
});
```

- **Data Masking & Boundary Control:** Never pass whole database objects to client components. Pick/mask fields server-side to avoid leaking sensitive data (e.g., password hashes, internal IDs) to the browser via JSON serialization.

---

## 3. Performance & User Experience (High Performance & Scalable)

### 3.1 Core Web Vitals Optimization

We target **Green (Good)** thresholds for all Web Vitals across P75 percentiles.

- **LCP (Largest Contentful Paint):** Priority-load hero images. Avoid client-side rendering for LCP elements.

```tsx
<Image alt="Hero" fetchPriority="high" priority src="/hero.webp" />
```

- **INP (Interaction to Next Paint):** Keep main thread unblocked. Wrap expensive state updates in `startTransition`.
- **CLS (Cumulative Layout Shift):** Always reserve space for dynamic content. Define `width`, `height`, and `aspect-ratio` on all images, iframes, and ad slots.

### 3.2 Rendering & Code-Splitting Strategies

- **Rendering Matrix:**
- _Public Marketing Pages:_ Static Site Generation (SSG) / Incremental Static Regeneration (ISR).
- _Authenticated Dashboards:_ Server-Side Rendering (SSR) with streamed boundaries.

- **Streaming & Suspense:** Wrap independent async components in `<Suspense>` to unblock the initial HTML payload (preventing server waterfalls).

```tsx
<Suspense fallback="{<SkeletonRow"/>}>
  <HeavyDataComponent/>
</Suspense>

```

- **Dynamic Imports:** Lazy load non-critical client-side bundles (e.g., heavy charting libraries, modals, rich-text editors).

```tsx
const Chart = dynamic(() => import("@/features/analytics/Chart"), {
  ssr: false,
});
```

### 3.3 Asset & Memory Optimization

- **Bundle Budget:** Client bundles must not exceed 150KB (gzipped) per route.
- **DOM Virtualization:** Use `tanstack/react-virtual` for lists/tables exceeding 100 rows to prevent DOM bloat.
- **Memory Leaks:** Always clean up event listeners, intervals, and open WebSocket/SSE connections in `useEffect` cleanup functions. Pass `AbortController` signals to `fetch` calls to cancel requests on unmount.

---

## 4. Database Optimization & Smart Querying (Frontend & Edge)

### 4.1 Data Fetching Patterns

- **Parallel Fetching:** Avoid waterfalls by initializing requests concurrently when data isn't dependent.

```typescript
// ✅ Fast: Parallel execution
const [user, settings] = await Promise.all([getUser(id), getSettings(id)]);
```

- **Request Deduplication:** Rely on React's `fetch` patch or React Query to deduplicate identical requests within the same render pass.

### 4.2 Caching & Invalidation Strategies

- **Stale-While-Revalidate (SWR):** Utilize Next.js `revalidate` configurations to serve fast, slightly stale data while updating the cache in the background.
- **Tag-Based Invalidation:** Group related fetch requests using `next.tags` and invalidate them granularly on mutation.

```typescript
// Fetching
fetch("/api/users", { next: { tags: ["users"] } });

// Server Action Mutation
revalidateTag("users");
```

- **Optimistic UI:** Use `useOptimistic` (React 19) or React Query's `onMutate` to instantly update the UI while the server processes the request, rolling back on failure.

### 4.3 Database Indexing & Query Awareness for Frontend Devs

Frontend query patterns directly dictate database performance. Frontend architects must design UIs that align with backend index capabilities:

- **Pagination:** Avoid traditional `OFFSET` pagination for large datasets (O(N) time complexity). Use **Cursor-based pagination** (Infinite Scroll), which allows the database to leverage **B-Tree Indexes** efficiently.
- _Frontend Query:_ `?cursor=2023-10-01T12:00:00Z`
- _Required DB Index:_ Composite index on `(created_at, id)`.

- **Filtering & Sorting:** If the UI allows sorting by "Status" and "Date", the backend requires a Composite Index `(status, date)`. Exposing arbitrary combination filters requires advanced indexing (e.g., GIN indexes for JSONB, or Elasticsearch).
- **Search:** Avoid `LIKE '%query%'` patterns in UI design (forces full table scans). Ensure the UI restricts searches to exact matches or utilizes dedicated Full-Text Search infrastructure:
- **Dedicated Search Engines** (The Gold Standard) such as Elastic Search.
- **Native Full-Text Search** (FTS) such as (PostgreSQL FTS).
- **Trigram Indexes** (For true substring matching) such as pg_trgm in PostgreSQL.
- Frontend Debouncing and Constraints

```

```
