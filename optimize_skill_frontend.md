# SKILL.md: Enterprise Frontend Engineering Standard

**Owner:** Principal Engineering & Architecture Group
**Scope:** Frontend Web Applications (React 19+, Next.js 16+ App Router, TypeScript 5+)
**Last Reviewed:** September 2026 · Aligned with Next.js 16.3
**Objective:** Define strict operational standards for secure, scalable, high-performance, and AI-agent-compatible frontend architectures.

---

## 0. Package Manager Detection & Consistency

### 0.1 Detect the Active Package Manager

Before writing any terminal command, the agent **must** identify which package manager the project uses and apply it consistently to every command in this document. Hard-coding `npm` or `pnpm` arbitrarily causes broken installs, duplicate lock files, and CI failures.

**Detection priority order — check each signal in sequence and stop at the first match:**

| Priority | Signal | Detected manager |
| -------- | ------ | ---------------- |
| 1 | `pnpm-lock.yaml` exists at the repo root | `pnpm` |
| 2 | `package-lock.json` exists at the repo root | `npm` |
| 3 | `yarn.lock` exists at the repo root | `yarn` |
| 4 | `bun.lockb` or `bun.lock` exists at the repo root | `bun` |
| 5 | `packageManager` field in `package.json` (e.g. `"pnpm@9.x"`) | value from field |
| 6 | User's prompt explicitly names a package manager | stated manager |
| 7 | None of the above | default to `npm` and warn |

```text
# Detection pseudo-logic (agent internal step — do not echo to user)
IF   pnpm-lock.yaml present         → PM = pnpm
ELIF package-lock.json present      → PM = npm
ELIF yarn.lock present              → PM = yarn
ELIF bun.lockb or bun.lock present  → PM = bun
ELIF package.json[packageManager]   → PM = that value
ELIF prompt mentions "pnpm"/"npm"   → PM = stated value
ELSE                                → PM = npm  # fallback + emit warning
```

### 0.2 Command Translation Table

Use the detected `PM` variable to translate every generic command below. **Never mix managers in a single project.**

| Action | `npm` | `pnpm` | `yarn` | `bun` |
| ------ | ----- | ------ | ------ | ----- |
| Install all deps | `npm install` | `pnpm install` | `yarn` | `bun install` |
| Add a dependency | `npm install <pkg>` | `pnpm add <pkg>` | `yarn add <pkg>` | `bun add <pkg>` |
| Add a dev dependency | `npm install -D <pkg>` | `pnpm add -D <pkg>` | `yarn add -D <pkg>` | `bun add -d <pkg>` |
| Add a global tool | `npm install -g <pkg>` | `pnpm add -g <pkg>` | `yarn global add <pkg>` | `bun add -g <pkg>` |
| Run a script | `npm run <script>` | `pnpm run <script>` | `yarn <script>` | `bun run <script>` |
| Run a one-off binary | `npx <bin>` | `pnpm dlx <bin>` | `yarn dlx <bin>` | `bunx <bin>` |
| Audit deps | `npm audit --audit-level=high` | `pnpm audit --audit-level high` | `yarn npm audit` | `bun audit` |
| Remove a dep | `npm uninstall <pkg>` | `pnpm remove <pkg>` | `yarn remove <pkg>` | `bun remove <pkg>` |

### 0.3 Enforcement Rules

- **One lock file per repo.** If both `pnpm-lock.yaml` and `package-lock.json` exist, flag it as a conflict and ask the user which manager is canonical before proceeding.
- **Never generate a second lock file.** Running `npm install` in a `pnpm` project (or vice versa) corrupts the dependency graph. Always use the detected manager.
- **CI must mirror local.** The same manager and version (pinned via `packageManager` field or `.tool-versions` / `mise.toml`) must be used in CI and local dev. Do not let CI default to a different manager than what the lock file expects.
- **`engines` field:** Recommend adding an `engines` field to `package.json` pinning the expected Node.js and manager versions:

```json
{
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=9.0.0"
  },
  "packageManager": "pnpm@9.15.0"
}
```

- **Corepack (Node.js 16.9+):** For `pnpm` and `yarn` projects, enable [Corepack](https://nodejs.org/api/corepack.html) so the correct manager version is enforced automatically:

```bash
# Enable once per machine — then use whichever manager package.json declares
corepack enable
corepack use pnpm@9.15.0
```

---

## 1. Architectural Principles & File Organization

### 1.1 Scalable Architecture

We adhere to a pragmatic implementation of **Feature-Sliced Design (FSD)** combined with Next.js App Router paradigms. This enforces modularity and prevents monolithic "spaghetti" codebases.

**Directory Structure:**

```text
src/
 ├── app/              # Routing, Layouts, Entry points, and Cache boundaries (Next.js App Router)
 ├── features/         # Feature modules (e.g., /auth, /checkout, /dashboard)
 │    └── [feature]/   # Co-located components, hooks, utils, types, and domain logic
 ├── core/             # App-wide setup (providers, theme, global middleware)
 ├── shared/           # Cross-feature reusable abstractions
 │    ├── ui/          # Dumb/presentational components (Buttons, Inputs, Skeletons)
 │    ├── lib/         # Utility functions, API clients, schema validators
 │    └── types/       # Global TypeScript declarations and branded types
 └── AGENTS.md         # Agent rules pointer (managed by `next dev` — do not remove)
```

**Component Hierarchy Rules:**

- **React Server Components (RSC) by Default:** Treat Server Components as "Smart" containers. They handle data fetching, backend authorization, and pass only serializable primitive props to the client.
- **Client Components (Islands):** Use the `"use client"` directive strictly at the leaf nodes closest to interactivity. Client components are "Dumb" views responsible for browser APIs, event handlers, and local state only.
- **Slot Pattern (Inversion of Control):** Prevent prop-drilling and unnecessary client bundle inflation by passing Server Components as `children` or named slots into Client Components.
- **Cache Components (`"use cache"`):** For data-heavy Server Components that do not need to re-render on every request, annotate with `"use cache"` to opt into static shell rendering and instant navigations.

### 1.2 Type Safety & Maintainability

TypeScript is our primary static analysis tool. `any` is strictly prohibited across all production code.

**Strict Configuration (`tsconfig.json`):**

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "isolatedModules": true,
    "moduleResolution": "bundler",
    "target": "ES2022"
  }
}
```

- **Boundary Enforcement:** Features cannot import from sibling features directly. Cross-feature communication happens at the `app/` (page/layout) level or via shared event buses / Zustand slices.
- **Zero Circular Dependencies:** Enforced via `eslint-plugin-import` (`import/no-cycle`).
- **Branded Types:** Use branded/nominal types for domain identifiers (e.g., `UserId`, `OrderId`) to prevent accidental ID mix-ups at compile time.

### 1.3 State Management Architecture

State is categorized and isolated by lifecycle, ownership, and persistence.

| State Type          | Responsibility                       | Approved Tooling                                    | Anti-Pattern                                  |
| ------------------- | ------------------------------------ | --------------------------------------------------- | --------------------------------------------- |
| **Server State**    | Async data, caching, revalidation    | Next.js Cache (`"use cache"`), React Query, SWR     | Storing API responses in Redux/Zustand         |
| **Global UI State** | Theme, cross-feature UI flags        | Zustand, React Context (low-frequency only)         | Prop-drilling 5+ levels                       |
| **Local State**     | Form inputs, toggles, active tabs    | `useState`, `useReducer`, `useActionState`          | Elevating ephemeral local state globally       |
| **URL State**       | Pagination, filters, search params   | Next.js `useSearchParams`, `useParams`, `nuqs`      | Syncing URL state to `useState` manually       |
| **Optimistic UI**   | Immediate feedback before server ack | React 19 `useOptimistic`, React Query `onMutate`    | Re-fetching after every mutation without rollback |

---

## 2. Secure Backend-First Data Processing

### 2.1 Client-Side Security (OWASP Top 10 Mitigations)

- **XSS (Cross-Site Scripting):** Never use `dangerouslySetInnerHTML` unless rendering explicitly sanitized markup (via `DOMPurify` with a strict allowlist). React escapes variables by default — do not bypass this.
- **CSP (Content Security Policy):** Must be configured in `next.config.ts` or Edge Middleware to whitelist script sources, frame embedding, and API origins. Use nonce-based CSP for dynamic scripts.

```typescript
// middleware.ts — nonce-based CSP headers
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { v4 as uuid } from "uuid";

export function middleware(req: NextRequest) {
  const nonce = uuid();
  const csp = [
    "default-src 'self'",
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    "connect-src 'self' https://api.enterprise.com",
    "frame-ancestors 'none'",
    "upgrade-insecure-requests",
  ].join("; ");

  const res = NextResponse.next();
  res.headers.set("Content-Security-Policy", csp);
  res.headers.set("X-Nonce", nonce);
  res.headers.set("X-Frame-Options", "DENY");
  return res;
}
```

- **Clickjacking:** Enforce both `X-Frame-Options: DENY` and `frame-ancestors 'none'` in CSP for defense-in-depth.
- **Dependency Auditing:** Run the audit command for the **detected package manager** (see Section 0.2) on every PR and in CI — e.g. `npm audit --audit-level=high` for npm projects or `pnpm audit --audit-level high` for pnpm projects. Pin critical dependency versions and treat high-severity findings as blocking.

### 2.2 Authentication & Authorization

- **JWT Token Lifecycle:** **Never** store access or refresh tokens in `localStorage` or `sessionStorage`. Tokens must be issued as `HttpOnly`, `Secure`, `SameSite=Strict` cookies set by the server.
- **Edge Middleware Guards:** All route protection must occur at the Edge layer before any rendering begins. This eliminates flash-of-unauthenticated-content.

```typescript
// middleware.ts — route guard
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const session = req.cookies.get("session")?.value;
  const isProtected = req.nextUrl.pathname.startsWith("/dashboard");
  if (!session && isProtected) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

- **RBAC/ABAC:** Role-Based Access Control is always evaluated on the server. Never rely on client-side UI hiding (`if (isAdmin) <AdminPanel />`) as a security measure. Backend endpoints must independently verify roles on every request.

### 2.3 Backend-First Data Safety

- **Input Validation:** Every client-to-server data path (Server Actions, Route Handlers) must validate input with **Zod** or **Valibot** before any business logic executes.

```typescript
// Shared schema — used by both client (form) and server (action)
const UpdateProfileSchema = z.object({
  email: z.string().email(),
  displayName: z.string().min(2).max(64),
  age: z.number().int().min(18).max(120),
});

// Server Action
export async function updateProfile(data: unknown) {
  "use server";
  const parsed = UpdateProfileSchema.safeParse(data);
  if (!parsed.success) {
    return { error: parsed.error.flatten() };
  }
  // ... business logic
}
```

- **Data Masking & Boundary Control:** Never pass full database rows or ORM models to Client Components. Pick/mask fields server-side to prevent leaking sensitive columns (password hashes, internal IDs, PII) via JSON serialization.
- **Server Actions Security:** All Server Actions that mutate data must verify CSRF origin headers and re-authenticate the session. Do not trust any data payload from the client.

---

## 3. Performance & User Experience

### 3.1 Core Web Vitals Targets

Target **Green (Good)** thresholds for all Web Vitals at the P75 percentile. Monitor continuously via Vercel Speed Insights or a Real-User Monitoring (RUM) provider.

| Metric | Target   | Common Cause of Failure             |
| ------ | -------- | ----------------------------------- |
| LCP    | < 2.5s   | Unoptimized hero images, SSR delays |
| INP    | < 200ms  | Long main-thread tasks, excessive re-renders |
| CLS    | < 0.1    | Images without dimensions, late-injected banners |
| TTFB   | < 800ms  | Slow server, no edge caching        |

- **LCP:** Priority-load hero images using `priority` and `fetchPriority="high"`. Avoid client-side rendering for above-the-fold content.

```tsx
<Image
  src="/hero.webp"
  alt="Hero"
  width={1440}
  height={720}
  fetchPriority="high"
  priority
/>
```

- **INP:** Keep the main thread clear. Wrap expensive non-urgent state updates in `startTransition`. Debounce user input before triggering data fetching. Avoid synchronous layout-thrashing in event handlers.
- **CLS:** Always declare explicit `width`, `height`, and `aspect-ratio` on all images, iframes, video embeds, and ad slots. Use CSS `content-visibility: auto` for off-screen sections.

### 3.2 Rendering & Code-Splitting Strategies

**Rendering Decision Matrix:**

| Route Type                  | Strategy                                     | Reason                                           |
| --------------------------- | -------------------------------------------- | ------------------------------------------------ |
| Public marketing pages      | Static (SSG) / ISR                           | Maximum CDN cacheability, fast TTFB              |
| Authenticated dashboards    | SSR with Suspense streaming                  | Fresh data, progressive hydration                |
| High-traffic product pages  | Cache Components (`"use cache"`) + PPR       | Instant navigations via pre-rendered static shells |
| Real-time feeds             | Client-side with SWR/polling or Server-Sent Events | Live data requirements                      |

- **Cache Components & Instant Insights (Next.js 16.3+):** When Cache Components are enabled, every `await` on the server is an explicit product choice. Next.js 16.3 surfaces uncached data as actionable errors with three labeled fix options:
  - **`[stream]`** — Wrap the data access in a `<Suspense>` boundary.
  - **`[cache]`** — Annotate the component or fetch with `"use cache"`.
  - **`[block]`** — Export `export const instant = false` to allow a deliberately blocking route.

  Use the **Copy prompt** button in the Instant Insights overlay to generate a paste-ready agent prompt that walks your coding agent through identifying, fixing, and verifying the issue end-to-end.

- **Streaming & Suspense:** Wrap independent async Server Components in `<Suspense>` with meaningful skeleton fallbacks to unblock the initial HTML payload and prevent server waterfalls.

```tsx
// ✅ Wrap independent async sections — not the whole page body
<Suspense fallback={<SkeletonCard />}>
  <UserActivityFeed userId={userId} />
</Suspense>
```

> ⚠️ **Suspense placement matters:** A `<Suspense>` boundary placed too high (wrapping the entire page body, or with `fallback={null}`) produces an empty static shell. Always place boundaries as close as possible to the actual dynamic data read.

- **Dynamic Imports:** Lazy-load non-critical client bundles (charting libraries, modals, rich-text editors) to keep initial bundle size minimal.

```tsx
const RichTextEditor = dynamic(
  () => import("@/features/editor/RichTextEditor"),
  { ssr: false, loading: () => <Skeleton className="h-48" /> }
);
```

### 3.3 Asset & Memory Optimization

- **Bundle Budget:** Client bundles must not exceed **150 KB (gzipped)** per route. Measure with `next build --analyze` (via `@next/bundle-analyzer`) and enforce in CI.
- **DOM Virtualization:** Use `@tanstack/react-virtual` for lists and tables exceeding 100 rows to prevent DOM node bloat and memory pressure.
- **Memory Leak Prevention:** Always clean up subscriptions, event listeners, intervals, and WebSocket/SSE connections in `useEffect` cleanup functions. Pass `AbortController` signals to `fetch` calls to cancel in-flight requests on component unmount.

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetchData({ signal: controller.signal }).then(setData).catch(() => {});
  return () => controller.abort();
}, [id]);
```

- **Font Optimization:** Use `next/font` with `display: "swap"` and `preload: true` for all custom fonts. Never load fonts from third-party CDNs at runtime.

### 3.4 Bundle Optimization & Tree-Shaking

To prevent bloating the bundle with dead code and to solve the "barrel file" problem when using heavy UI or icon libraries (like `lucide-react`, `@mui/material`, `lodash`, etc.), always enforce the automated Next.js approach. 

Specifically, ensure that large libraries are added to the `optimizePackageImports` array in `next.config.ts` (or `next.config.js`) so Next.js can handle the tree-shaking automatically, rather than using manual deep-path imports across the codebase.

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {
    optimizePackageImports: ["lucide-react", "@mui/icons-material", "lodash", "recharts"],
  },
};

export default nextConfig;
```
*Note: Depending on the Next.js version (e.g. 15+ or 16+), `optimizePackageImports` may have been stabilized outside of `experimental` or be the default for specific packages. Always adhere to the active Next.js docs.*

---

## 4. Data Fetching, Caching & Querying

### 4.1 Data Fetching Patterns

- **Parallel Fetching First:** Avoid sequential waterfalls. Initiate all independent requests concurrently.

```typescript
// ✅ Correct: Parallel — total time = max(user, settings, posts)
const [user, settings, posts] = await Promise.all([
  getUser(id),
  getSettings(id),
  getRecentPosts(id),
]);

// ❌ Wrong: Sequential waterfall — total time = user + settings + posts
const user = await getUser(id);
const settings = await getSettings(id);
const posts = await getRecentPosts(id);
```

- **Request Deduplication:** Rely on React's built-in `fetch` deduplication cache (within a single render pass) or React Query/SWR deduplication to prevent duplicate network calls for the same resource.

### 4.2 Caching & Invalidation Strategies

- **`"use cache"` Directive (Next.js 16+):** Annotate Server Components or data-fetching functions with `"use cache"` to opt them into the static rendering cache and enable instant navigations. Pair with `cacheLife()` and `cacheTag()` for fine-grained TTL and invalidation control.

```typescript
import {
  unstable_cacheTag as cacheTag,
  unstable_cacheLife as cacheLife,
} from "next/cache";

async function getProducts() {
  "use cache";
  cacheTag("products");
  cacheLife("hours");
  return db.query("SELECT * FROM products");
}
```

- **Tag-Based Invalidation:** Group fetches by domain tag and invalidate granularly on mutation.

```typescript
// In a Server Action after mutation
import { revalidateTag } from "next/cache";
revalidateTag("products");
```

- **Stale-While-Revalidate (ISR):** Use `revalidate` on route segments or `fetch` options to serve fast, slightly stale content while updating the cache in the background.
- **Optimistic UI:** Use React 19's `useOptimistic` or React Query's `onMutate` to immediately reflect mutations in the UI. Always implement a rollback on failure.

### 4.3 Database & Query Awareness for Frontend Engineers

Frontend query patterns directly dictate database performance. Frontend architects must design UIs that align with backend index capabilities.

- **Pagination:** Never use `OFFSET`-based pagination for large datasets (O(N) scan complexity). Use **cursor-based pagination** instead.
  - Frontend query: `?cursor=2024-06-01T12:00:00Z`
  - Required DB index: Composite `(created_at DESC, id DESC)`.

- **Filtering & Sorting:** Every filter/sort combination exposed in the UI requires a matching composite index. Exposing arbitrary combinations requires advanced infrastructure (GIN indexes, Elasticsearch).

- **Search:** Avoid `LIKE '%query%'` patterns — they force full table scans. Route search through dedicated infrastructure:
  - **Dedicated Search Engines** (Gold Standard): Elasticsearch, Typesense, Algolia.
  - **Native Full-Text Search**: PostgreSQL FTS with `tsvector`.
  - **Trigram Indexes**: `pg_trgm` for true substring matching in PostgreSQL.

- **Frontend Debouncing:** Debounce all search and filter inputs (minimum 300ms) before triggering API calls.

```tsx
const debouncedSearch = useDebouncedCallback((value: string) => {
  router.push(`?q=${encodeURIComponent(value)}`);
}, 300);
```

---

## 5. AI-Agent-Compatible Development (Next.js 16.3+)

Next.js 16.3 extends the framework's first-class support for agent-driven development. These standards ensure your project integrates cleanly with coding agents such as Claude Code, Cursor, and Codex.

### 5.1 AGENTS.md — Version-Matched Docs for Agents

`next dev` in 16.3 automatically writes and maintains an `AGENTS.md` pointer block that directs coding agents to read the locally bundled Next.js docs (`node_modules/next/dist/docs/`) instead of potentially outdated training data.

```md
<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ
from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before
writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->
```

**Rules:**
- **Never remove** the `<!-- BEGIN:nextjs-agent-rules -->` block. Removing it only recreates the uncommitted change on the next `next dev` run.
- **Commit the block** alongside your work to keep the repository tree clean.
- To opt out entirely: set `agentRules: false` in `next.config.ts`.
- For projects on Next.js 16.1 or earlier, run the codemod to bootstrap the file:

```bash
# npm / default
npx @next/codemod@canary agents-md

# pnpm
pnpm dlx @next/codemod@canary agents-md
```

### 5.2 First-Party Agent Skills

Next.js 16.3 ships four first-party [Skills](https://www.skills.sh/vercel/next.js) for driving multi-step workflows that bundled docs alone cannot fully orchestrate.

| Skill                                 | Purpose                                                                                      |
| ------------------------------------- | -------------------------------------------------------------------------------------------- |
| `next-dev-loop`                       | Full dev feedback loop: browser control, console, network, React DevTools introspection      |
| `next-cache-components-adoption`      | Incrementally adopts Cache Components across routes with per-feature PRs or in-place         |
| `next-cache-components-optimizer`     | Converts target routes to instant navigations and commits regression-catching tests           |
| `next-partial-prefetching-adoption`   | Enables Partial Prefetching; works until all links share a shared loading shell              |

Use the `npx` / `pnpm dlx` / `yarn dlx` / `bunx` equivalent from the **detected package manager** (Section 0.2):

```bash
# npm / default
npx skills add vercel/next.js --skill next-dev-loop
npx skills add vercel/next.js --skill next-cache-components-adoption
npx skills add vercel/next.js --skill next-cache-components-optimizer
npx skills add vercel/next.js --skill next-partial-prefetching-adoption

# pnpm
pnpm dlx skills add vercel/next.js --skill next-dev-loop
pnpm dlx skills add vercel/next.js --skill next-cache-components-adoption
pnpm dlx skills add vercel/next.js --skill next-cache-components-optimizer
pnpm dlx skills add vercel/next.js --skill next-partial-prefetching-adoption
```

**Recommended agent prompts:**

```text
After every edit, verify the page still works at runtime using the next-dev-loop skill.
```

```text
Adopt Cache Components in this project using the next-cache-components-adoption skill.
```

```text
Make the navigation from /settings to /dashboard instant using the next-cache-components-optimizer skill. The header and project list should be part of the instant UI.
```

> **Note:** The older knowledge-only Skills from `skills.sh` (e.g., for App Router conventions) are retired now that bundled docs serve that purpose. Run `npx skills update` (or `pnpm dlx skills update` for pnpm projects) to remove them.

### 5.3 Agent Browser with React DevTools Introspection

`agent-browser` v0.27+ (the successor to the experimental `next-browser`) provides agents with full browser observability: DOM, console logs, network requests, Web Vitals, **and** React DevTools introspection.

Use the **detected package manager** (Section 0.2) to install globally:

```bash
# npm projects
npm install -g agent-browser@^0.27

# pnpm projects
pnpm add -g agent-browser@^0.27

# yarn projects
yarn global add agent-browser@^0.27

# bun projects
bun add -g agent-browser@^0.27
```

Available React commands (requires `--enable react-devtools` at launch):

| Command                                  | Description                                   |
| ---------------------------------------- | --------------------------------------------- |
| `react tree`                             | List the full React component tree            |
| `react inspect <fiberId>`                | Inspect props and state of a specific component |
| `react renders start` / `stop`           | Profile component re-renders                  |
| `react suspense --only-dynamic --json`   | Identify what's blocking a render             |

The `next-dev-loop` Skill automatically launches `agent-browser` with React DevTools enabled.

### 5.4 Actionable Errors (Instant Insights)

When Cache Components are enabled, Next.js 16.3 presents uncached `await` calls as **Instant Insights** — actionable errors with three labeled fix options, visible in both the dev overlay and terminal output.

**Fix options:**
- **`[stream]`** — Wrap in `<Suspense fallback={...}>` around the data access.
- **`[cache]`** — Cache the data access with `"use cache"`.
- **`[block]`** — Set `export const instant = false` to allow a deliberately blocking route.

**Copy prompt button:** Each Insight in the overlay includes a **Copy prompt** button that generates a paste-ready agent prompt covering: identifying the failing code, reading the per-error docs page, applying the canonical fix pattern, and verifying the result at runtime via `next-dev-loop`. Use this to delegate fixes to your coding agent.

**Structured terminal output:** The same fix menu appears in the terminal during `next dev` and `next build`, so agents reading from CI logs or terminal sessions still get the same labeled, doc-linked fixes without the overlay.

**Per-error documentation:** Each error type has its own page at `nextjs.org/docs/messages/<error-code>` with canonical fix **Patterns**, **Trade-offs** against alternative fixes, and **Gotchas** that are easy to miss. Append `.md` to the URL for plain Markdown (e.g., `https://nextjs.org/docs/messages/blocking-prerender-dynamic.md`).

### 5.5 DevTools MCP Server

The Next.js DevTools MCP server (`next-devtools-mcp`) exposes live compilation state from the running `next dev` server, so agents can diagnose issues without running a full `next build`.

**Key tools in 16.3:**
- `get_compilation_issues` — returns all current compilation errors for the project.
- `compile_route` — compiles a single route and returns its issues. Much faster than a full build when iterating.

```json
// .mcp.json — add to your project root
{
  "mcpServers": {
    "nextjs": {
      "command": "npx",
      "args": ["next-devtools-mcp"]
    }
  }
}
```

Skills like `next-dev-loop` call the underlying `/_next/mcp` endpoints directly, so they work without extra MCP client setup.

### 5.6 Docs as Markdown

Append `.md` to any `nextjs.org/docs` URL to retrieve it as plain Markdown — useful when constructing agent prompts that need authoritative reference material inline.

| Resource                  | URL                                             |
| ------------------------- | ----------------------------------------------- |
| Full docs index           | `https://nextjs.org/docs/llms.txt`              |
| All docs bundled          | `https://nextjs.org/docs/llms-full.txt`         |
| Per-error page (example)  | `https://nextjs.org/docs/messages/blocking-prerender-dynamic.md` |
| Any docs page             | Append `.md` to the URL                         |

Both `llms.txt` and `llms-full.txt` follow the [`llms.txt` convention](https://llmstxt.org/), compatible with any agent that already reads `llms.txt` for other tools.

---

## 6. Developer Experience & Toolchain

### 6.1 Linting & Formatting

- **ESLint:** Use `eslint-config-next` as the base. Layer on `@typescript-eslint/recommended-type-checked`, `import/no-cycle`, and `unicorn/prefer-module`.
- **Prettier:** Enforce consistent formatting with a project-level `.prettierrc`. Integrate with `eslint-config-prettier` to avoid rule conflicts.
- **Husky + lint-staged:** Run linting and type checking on staged files before every commit. Block commits that introduce type errors or lint violations.

### 6.2 Testing Strategy

| Layer       | Tooling                          | Coverage Target                              |
| ----------- | -------------------------------- | -------------------------------------------- |
| Unit        | Vitest + React Testing Library   | All shared utilities and hooks               |
| Integration | Vitest + MSW (API mocking)       | Feature-level data flows and Server Actions  |
| E2E         | Playwright                       | Critical user journeys (auth, checkout, etc.)|
| Visual      | Chromatic / Percy                | Shared UI component regressions              |

- **Prefer testing user behavior** over implementation details. Test what users see and do, not component internals.
- **Co-locate tests:** Keep test files adjacent to the source they cover (`ComponentName.test.tsx` next to `ComponentName.tsx`).
- **Snapshot tests:** Use sparingly. Prefer assertion-based tests for behavior; snapshot only for serializable outputs like generated HTML or config objects.

### 6.3 CI/CD Requirements

Every pull request must pass all of the following gates before merge:

1. `tsc --noEmit` — zero TypeScript errors
2. `eslint .` — zero lint warnings (treated as errors in CI)
3. `vitest run` — all unit and integration tests pass
4. `next build` — production build succeeds with no compilation errors
5. Bundle size check — no route exceeds the 150 KB gzipped budget
6. `<PM> audit` — no high-severity vulnerabilities (use the command from Section 0.2 for the detected manager, e.g. `npm audit --audit-level=high` or `pnpm audit --audit-level high`)

### 6.4 Git Branching Strategy & AI Workflow

The project strictly follows an environment-based branching model: `main` (Production) <== `dev` (Staging/Integration) <== `features/*` (Active Development).

**AI Agent Git Rules (Mandatory):**
1. **Branch Detection:** At the start of a session or task, the AI must check the existing branches (`git branch -a`). If the foundational branches (`main` or `dev`) are missing, the AI must alert the user and ask for permission to initialize them.
2. **Permission First:** The AI must **always ask for explicit user permission** before executing any Git commands that mutate state (e.g., `git checkout -b`, `git commit`, `git push`, merging, or creating PRs).
3. **Feature Completion:** When the user indicates a feature is complete on a `features/*` branch, the AI must pause and ask for permission to initiate the CI/CD workflow (e.g., creating a PR to `dev` to trigger GitHub Actions) before moving on to write new application code.

### 6.5 GitHub Actions (CI/CD Pipeline)

When instructed to set up CI/CD, the AI must first detect the local environment versions by running commands like `node -v` and `<PM> -v`. It must then generate the following workflow at `.github/workflows/ci.yml`, injecting the exactly matched versions to ensure local-to-CI parity. This enforces the gates from Section 6.3 on any PR targeting `dev` or `main`.

```yaml
name: Frontend CI/CD Pipeline

on:
  pull_request:
    branches: [main, dev]
  push:
    branches: [main, dev]

jobs:
  quality-gates:
    name: Lint, Test, and Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Package Manager
        uses: pnpm/action-setup@v3 # (AI must adapt this to the detected package manager)
        with:
          version: <INJECT_DETECTED_PM_VERSION_HERE> # e.g., output of `pnpm -v`

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '<INJECT_DETECTED_NODE_VERSION_HERE>' # e.g., output of `node -v`
          cache: 'pnpm' # (Adapt to detected manager)

      - name: Install Dependencies
        run: pnpm install --frozen-lockfile # (Adapt to detected manager)

      - name: TypeScript Check
        run: pnpm tsc --noEmit

      - name: ESLint
        run: pnpm eslint .

      - name: Security Audit
        run: pnpm audit --audit-level high

      - name: Run Tests
        run: pnpm vitest run

      - name: Next.js Build
        run: pnpm build
        env:
          NEXT_TELEMETRY_DISABLED: 1
```

---

## 7. Accessibility (a11y)

Accessibility is a **correctness requirement**, not an enhancement.

- **Semantic HTML:** Use the correct HTML element for its semantic role. Never use `<div>` or `<span>` for interactive controls.
- **ARIA:** Add ARIA attributes only when native HTML semantics are insufficient. Never use ARIA to override correct semantics.
- **Keyboard Navigation:** All interactive elements must be reachable and fully operable via keyboard alone. Focus order must follow the visual reading order.
- **Color Contrast:** All text must meet WCAG 2.2 AA contrast ratios (4.5:1 for normal text, 3:1 for large text). Never convey meaning through color alone.
- **Focus Management:** When modals, drawers, or dialogs open, move focus to the first interactive element inside. Restore focus to the trigger element on close.
- **Automated Auditing:** Integrate `axe-core` (via `jest-axe` or the Playwright axe plugin) into the CI pipeline. Treat any axe violation as a blocking CI failure.

---

## 8. Production Compliance Checklist

Before any production deployment, verify all of the following:

- [ ] CSP headers are configured and tested for every deployment environment
- [ ] Auth tokens are stored in `HttpOnly`, `Secure`, `SameSite=Strict` cookies only
- [ ] No sensitive data (PII, hashes, internal IDs) is serialized to Client Components
- [ ] All Server Actions validate input with Zod or Valibot before executing business logic
- [ ] Core Web Vitals are in the Green (Good) range at P75 on RUM data
- [ ] Bundle size budgets pass for all modified routes (`next build --analyze`)
- [ ] `<PM> audit` returns zero high-severity vulnerabilities (Section 0.2 for the correct command)
- [ ] All `axe-core` accessibility violations are resolved
- [ ] `AGENTS.md` agent rules block is present and committed to the repository
- [ ] Instant Insights are resolved (stream / cache / block) for all routes with Cache Components enabled
- [ ] Cache Component adoption is verified via `agent-browser` React DevTools after any `"use cache"` migration
