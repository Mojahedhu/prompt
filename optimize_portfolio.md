# Portfolio Build Plan

<!-- AGENT INSTRUCTION: Read this entire document before writing any code.
     The Global Context block (Section 0) contains non-negotiable rules that
     apply to every step. Do not skip it. -->

---

## Document Meta

| Field                    | Value                                                                        |
| ------------------------ | ---------------------------------------------------------------------------- |
| **App Name**             | Personal Developer Portfolio + Admin CMS                                     |
| **Owner**                | Mojahed Mohammed — Frontend Engineer                                         |
| **Package Manager**      | `pnpm` (detected via `pnpm-lock.yaml` — see `optimize_skill_frontend.md §0`) |
| **Engineering Standard** | `optimize_skill_frontend.md` (Enterprise Frontend Engineering Standard)      |
| **Design Source**        | Google Stitch / MCP — hand off design via MCP tool at build time             |
| **Target Environment**   | Vercel (Edge Network + Serverless Functions)                                 |
| **Last Updated**         | September 2026                                                               |

---

## 0. Global Context (Read Once, Apply Everywhere)

> **AGENT:** This section is your source of truth. Every step references it.
> Do not derive assumptions from your training data — use these definitions.

### 0.1 Technology Stack

| Layer            | Technology                             | Version / Notes                                                                                               |
| ---------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Framework        | Next.js App Router                     | 16.3+                                                                                                         |
| Language         | TypeScript                             | 5+ · `strict: true` · no `any`                                                                                |
| UI Library       | shadcn/ui                              | Latest                                                                                                        |
| Styling          | Tailwind CSS                           | v4 + `tw-animate-css` package                                                                                 |
| Animation        | CSS-First Strategy                     | 1. `tw-animate-css` (basic) 2. `tailwindcss-motion` (complex) 3. `framer-motion` (only if JS strictly needed) |
| Database ORM     | Drizzle ORM                            | Turso (libSQL) adapter                                                                                        |
| Database         | Turso (SQLite)                         | Edge-compatible                                                                                               |
| Auth             | Auth.js v5 (NextAuth)                  | GitHub OAuth · single admin user                                                                              |
| Validation       | Zod                                    | Shared between client forms and server actions                                                                |
| Forms            | React Hook Form                        | Integrated with `useActionState` (React 19)                                                                   |
| Error Tracking   | Sentry (`@sentry/nextjs`)              | Client + Edge runtime                                                                                         |
| File Storage     | Vercel Blob                            | Images, CV PDF                                                                                                |
| Static Config    | Vercel Edge Config                     | Read-only: name, tagline, social links                                                                        |
| Icons            | lucide-react                           |                                                                                                               |
| Theme            | next-themes                            | Light / Dark / System                                                                                         |
| Canvas FX        | HTML5 Canvas + `requestAnimationFrame` | Custom `<RibbonCanvas />` client component                                                                    |
| Real-time Events | Ably                                   | WebSocket pub/sub — admin inbox live updates · Turso stays as DB                                              |
| AI Assistant     | Gemini 3.5 Flash + Vercel AI SDK       | Context-stuffed RAG portfolio agent (no vector DB)                                                            |
| i18n             | next-intl v3                           | EN (default) + AR · subpath routing `/en` `/ar` · RSC + Server Actions support                                |
| Sidebar          | shadcn/ui Sidebar                      | Collapsible icon-mode · cookie-persisted state · RTL-aware · used in portfolio & admin layouts                |

### 0.2 Data Layer Map

> ⚠️ **Critical rule:** Vercel Edge Config is **read-only at runtime**.
> Any content that the Admin CMS mutates must live in **Turso**, not Edge Config.

| Data Entity                          | Store              | Access Pattern                                                                      |
| ------------------------------------ | ------------------ | ----------------------------------------------------------------------------------- |
| Skills                               | Turso (SQLite)     | RSC → Drizzle → `"use cache"` + `cacheTag('skills')`                                |
| Experiences                          | Turso (SQLite)     | RSC → Drizzle → `"use cache"` + `cacheTag('experiences')`                           |
| Projects                             | Turso (SQLite)     | RSC → Drizzle → `"use cache"` + `cacheTag('projects')`                              |
| Contact Messages                     | Turso (SQLite)     | Server Action writes to DB **then** publishes Ably event → admin inbox updates live |
| Real-time Events                     | Ably (pub/sub)     | Server publishes via REST · Client subscribes via WebSocket (token auth)            |
| Project Images, CV                   | Vercel Blob        | Uploaded via Admin Server Action → served via signed URL                            |
| Site Config (name, tagline, socials) | Vercel Edge Config | Read-only at build/request time                                                     |

### 0.3 Cache Strategy

All public portfolio data uses **Next.js 16 Cache Components** (`"use cache"`) with named tags.
Admin mutation Server Actions must call `revalidateTag(tag)` immediately after a successful write.

```typescript
// Tag constants — single source of truth
export const CACHE_TAGS = {
  skills: "skills",
  experiences: "experiences",
  projects: "projects",
} as const;
```

### 0.4 Project Folder Structure (Feature-Sliced Design)

```text
src/
├── app/                          # Next.js App Router — routes, layouts, boundaries
│   ├── [locale]/                 # Dynamic locale segment — added in Step 9 (i18n)
│   │   ├── (portfolio)/          # Route group: public-facing pages (no auth)
│   │   │   ├── layout.tsx        # Public shell: SidebarCookieProvider + NavBar
│   │   │   ├── page.tsx          # /[locale]  → Home / Hero
│   │   │   ├── about/page.tsx
│   │   │   ├── skills/page.tsx
│   │   │   ├── projects/page.tsx
│   │   │   ├── experience/page.tsx
│   │   │   ├── recommendations/page.tsx
│   │   │   ├── education/page.tsx
│   │   │   └── contact/page.tsx
│   │   ├── (admin)/              # Route group: protected CMS (English-only)
│   │   │   ├── layout.tsx        # Admin shell: SidebarCookieProvider + AdminSidebar + auth check
│   │   │   ├── admin/page.tsx    # /admin → Dashboard overview
│   │   │   ├── admin/skills/     # CRUD: skills
│   │   │   ├── admin/experience/ # CRUD: experiences
│   │   │   ├── admin/projects/   # CRUD: projects + image upload
│   │   │   └── admin/messages/   # Read-only: contact inbox
│   │   └── layout.tsx            # Locale layout: sets lang + dir on <html>, NextIntlClientProvider
│   ├── layout.tsx                # Root shell: <ThemeProvider> + <RibbonCanvas />
│   ├── not-found.tsx             # /404
│   └── api/                      # Route Handlers
│       └── chat/                 # AI Assistant streaming route
├── messages/                     # i18n translation files — added in Step 9
│   ├── en.json                   # English UI strings (default locale)
│   └── ar.json                   # Arabic UI strings
├── i18n/                         # next-intl config — added in Step 9
│   ├── routing.ts                # Locale list + default locale definition
│   ├── request.ts                # Server-side message loader (RSC)
│   └── navigation.ts             # Typed Link, redirect, useRouter wrappers
├── features/                     # Feature modules — co-located logic
│   ├── ai-assistant/             # <ChatWidget /> + RAG context builder
│   ├── canvas/                   # <RibbonCanvas /> component + physics logic
│   ├── contact/                  # <ContactForm /> + submitContact Server Action
│   ├── nav/                      # <NavBar /> — collapsible shadcn/ui Sidebar (Step 8)
│   ├── admin/                    # Admin DataTables, CRUD modals, upload logic
│   │   ├── AdminSidebar.tsx      # Admin collapsible sidebar — added in Step 8
│   │   └── actions/
│   │       └── translate.ts      # Gemini auto-translate Server Action — added in Step 9
│   └── auth/                     # Auth.js config, session utilities
├── shared/
│   ├── lib/
│   │   ├── db/
│   │   │   ├── client.ts         # Drizzle + Turso client singleton
│   │   │   └── schema.ts         # All table definitions (+ AR locale columns in Step 9)
│   │   ├── cache-tags.ts         # CACHE_TAGS constant (see §0.3)
│   │   └── validators/           # Shared Zod schemas
│   ├── ui/                       # Dumb shadcn/ui wrappers (Button, Input, Skeleton…)
│   │   ├── LocaleSwitcher.tsx    # Language toggle — added in Step 9
│   │   └── SidebarCookieProvider.tsx  # RSC: reads cookie → passes defaultOpen — added in Step 8
│   └── types/                    # Global TypeScript declarations
├── proxy.ts                       # Edge: CSP nonce + /admin auth guard + next-intl middleware
└── AGENTS.md                     # Next.js agent rules pointer (do not remove)
```

### 0.5 Agent Operating Rules

1. **Package manager is `pnpm`.** Use `pnpm add`, `pnpm dlx`, `pnpm run`. Never use `npm install` or `npx` unless explicitly stated.
2. **RSC by default.** Every component is a Server Component unless it needs browser APIs, event handlers, or local state. Add `"use client"` only at the leaf node closest to interactivity.
3. **No `any`.** TypeScript strict mode is enforced. Use `unknown` + Zod parse for external data.
4. **Zod on every boundary.** Validate on the client (form UX) AND re-validate on the server (Server Action) independently.
5. **Never trust the client.** Re-authenticate the session inside every mutation Server Action.
6. **Cache every public RSC.** Annotate with `"use cache"` + `cacheTag(CACHE_TAGS.x)`. Call `revalidateTag` on every successful admin mutation.
7. **Design comes from MCP.** When building UI, call the MCP design tool to read layout tokens, spacing, and component specs from the Stitch design. Do not invent layouts.

---

## Step 1 — Project Initialization & Infrastructure

**Goal:** Bootstrap the project, wire up all services, and establish the database schema before writing any UI.

### Decisions

- **Why Turso over Vercel Postgres?** Turso (libSQL/SQLite) is edge-compatible with zero cold-start overhead, and `@libsql/client` works natively with Drizzle ORM.
- **Why Auth.js v5 over Clerk?** Single-admin use case; Auth.js v5 adds zero user management complexity, and GitHub OAuth requires no paid plan.

### Tasks

**1a — Bootstrap**

```bash
pnpm create next-app@latest portfolio --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd portfolio
```

**1b — Install core dependencies**

```bash
# UI + Styling
pnpm add shadcn@latest lucide-react next-themes class-variance-authority clsx tailwind-merge
pnpm add -D tw-animate-css

# Database
pnpm add drizzle-orm @libsql/client
pnpm add -D drizzle-kit

# Auth
pnpm add next-auth@beta

# Validation + Forms
pnpm add zod react-hook-form @hookform/resolvers

# Storage
pnpm add @vercel/blob @vercel/edge-config

# Error Tracking (run wizard separately)
pnpm add @sentry/nextjs
```

**1c — Initialize shadcn/ui**

```bash
pnpm dlx shadcn@latest init
# Select: Dark theme · CSS variables · src/shared/ui as component directory
```

**1d — Database Schema** (`src/shared/lib/db/schema.ts`)

```typescript
import { sqliteTable, text, integer, real } from "drizzle-orm/sqlite-core";

export const skills = sqliteTable("skills", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  label: text("label").notNull(),
  category: text("category").notNull(), // e.g. "Frontend", "DevOps"
  proficiency: real("proficiency").notNull(), // 0.0 → 1.0
  iconUrl: text("icon_url"),
  sortOrder: integer("sort_order").notNull().default(0),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(
    () => new Date(),
  ),
});

export const experiences = sqliteTable("experiences", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  company: text("company").notNull(),
  role: text("role").notNull(),
  description: text("description").notNull(),
  startDate: text("start_date").notNull(), // ISO date string
  endDate: text("end_date"), // null = "Present"
  logoUrl: text("logo_url"),
  sortOrder: integer("sort_order").notNull().default(0),
});

export const projects = sqliteTable("projects", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  title: text("title").notNull(),
  description: text("description").notNull(),
  techStack: text("tech_stack", { mode: "json" }).$type<string[]>().notNull(),
  liveUrl: text("live_url"),
  repoUrl: text("repo_url"),
  imageUrl: text("image_url"),
  featured: integer("featured", { mode: "boolean" }).notNull().default(false),
  sortOrder: integer("sort_order").notNull().default(0),
});

export const contactMsgs = sqliteTable("contact_msgs", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  name: text("name").notNull(),
  email: text("email").notNull(),
  message: text("message").notNull(),
  read: integer("read", { mode: "boolean" }).notNull().default(false),
  createdAt: integer("created_at", { mode: "timestamp" }).$defaultFn(
    () => new Date(),
  ),
});
```

**1e — Environment Variables** (`.env.local`)

```bash
# Turso
TURSO_DATABASE_URL=libsql://your-db.turso.io
TURSO_AUTH_TOKEN=your-token

# Auth.js v5
AUTH_SECRET=your-secret
AUTH_GITHUB_ID=your-github-app-id
AUTH_GITHUB_SECRET=your-github-app-secret
ALLOWED_ADMIN_EMAIL=your@email.com

# Vercel Blob
BLOB_READ_WRITE_TOKEN=your-token

# Sentry
SENTRY_DSN=https://...
NEXT_PUBLIC_SENTRY_DSN=https://...
```

**1f — Configure Sentry**

```bash
pnpm dlx @sentry/wizard@latest -i nextjs
# Answer: Vercel deployment · Yes to source maps · No to performance profiling (add manually if needed)
```

**1g — Configure Tailwind CSS v4 (CSS-first — no `tailwind.config.ts`)**

> ℹ️ **Tailwind CSS v4** (default in Next.js 15+) is **CSS-first**. There is no `tailwind.config.ts`.
> All configuration — theme tokens, dark mode, plugins — lives in `src/app/globals.css`.
> Content detection is **automatic**; no `content` array is needed.
> The legacy `tailwindcss-animate` plugin and `animate-tailwind` package are replaced by
> **`tw-animate-css`**, which is shadcn/ui's official v4-native animation library (migrated March 2025).

**Install the animation package:**

```bash
pnpm add -D tw-animate-css
```

**`src/app/globals.css`**

```css
/* 1. Tailwind v4 base import — replaces @tailwind base/components/utilities */
@import "tailwindcss";

/* 2. tw-animate-css — v4-native replacement for tailwindcss-animate / animate-tailwind
      Provides: animate-in, animate-out, fade-in, fade-out, zoom-in, zoom-out,
                slide-in-from-*, slide-out-to-*, animate-delay-*, etc. */
@import "tw-animate-css";

/* 3. Custom theme tokens (replaces tailwind.config.ts `theme.extend`) */
@theme {
  --font-sans: "Inter", sans-serif;
  --font-mono: "JetBrains Mono", monospace;

  /* Dark mode surface colors (used by shadcn/ui CSS variables) */
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --border: oklch(1 0 0 / 10%);
  --ring: oklch(0.556 0 0);
  --radius: 0.625rem;
}

/* 4. Dark mode variant — Tailwind v4 uses @variant instead of darkMode config */
@variant dark (&:is(.dark *));
```

> ⚠️ **`next.config.ts` is not involved in Tailwind configuration.** Tailwind v4 integrates
> via `@tailwindcss/postcss` in `postcss.config.mjs` (auto-generated by `create-next-app`).
> Do not add Tailwind settings to `next.config.ts`.

### Acceptance Criteria

- [ ] `pnpm run dev` starts without errors.
- [ ] `drizzle-kit push` creates all four tables in Turso.
- [ ] Sentry DSN is active and a test event appears in the dashboard.
- [ ] No `package-lock.json` or `yarn.lock` present (pnpm only).

### Agent Prompt

```
Act as a senior full-stack engineer.

Using the tasks in Step 1 of the portfolio build plan:
1. Run all pnpm install commands exactly as listed.
2. Generate the complete Drizzle schema file at `src/shared/lib/db/schema.ts` with the four tables: skills, experiences, projects, contact_msgs.
3. Generate the Drizzle client singleton at `src/shared/lib/db/client.ts` using @libsql/client.
4. Generate `drizzle.config.ts` pointing to the Turso URL from env.
5. Generate `sentry.client.config.ts` and `sentry.server.config.ts` for @sentry/nextjs — keep the client bundle impact minimal (no session replay unless explicitly requested).
6. Generate `src/app/globals.css` with the Tailwind v4 CSS-first setup: @import "tailwindcss", @import "tw-animate-css", @theme block with font and color tokens, and @variant dark for dark mode. Do NOT generate a tailwind.config.ts — it does not exist in v4.

Constraints: TypeScript strict mode, no `any`, pnpm only.
```

---

## Step 2 — Fluid Ribbon Canvas Background

**Goal:** Build the animated canvas background as an isolated client island that has zero impact on SSR, bundle size of non-canvas routes, or pointer interactivity.

### Decisions

- **Client island (`"use client"`)** — canvas requires `window`, `document`, and `requestAnimationFrame`. It cannot run on the server.
- **`pointer-events: none`** — the canvas sits above content (`z-50`) but must never block clicks, links, or form inputs beneath it.
- **Pause on blur** — stop the `requestAnimationFrame` loop when `document.visibilityState === 'hidden'` to save CPU.
- **Plan A — Spring + Velocity Ribbon (primary):** A physically-weighted ribbon with spring easing and velocity-aware stroke thickness. Looks like a premium calligraphic ink trail. No extra libraries.
- **Plan B — Particle Trail (fallback):** If the ribbon feel is judged too rigid after visual testing, switch to a particle emitter. Each trail point spawns particles that drift and fade, creating a light smoke-dispersal feel. Still purely canvas, no extra libraries. Switch by changing the `CANVAS_MODE` constant.

### Tasks

**Component location:** `src/features/canvas/RibbonCanvas.tsx`

**Mode switch constant (top of file):**

```typescript
// Change to 'particles' to activate Plan B fallback
const CANVAS_MODE: "ribbon" | "particles" = "ribbon";
```

---

#### Plan A — Spring + Velocity Ribbon (Primary)

**Physics model:**

- Maintain an array of `N` trailing points (recommended: 30).
- Each frame, move each point toward the previous with a **spring factor** of `0.12` (not simple lerp — spring overshoots slightly and settles, giving "weight"):
  ```typescript
  point.vx += (target.x - point.x) * SPRING;
  point.vy += (target.y - point.y) * SPRING;
  point.vx *= DAMPING; // 0.75 — kills overshoot quickly
  point.vy *= DAMPING;
  point.x += point.vx;
  point.y += point.vy;
  ```
- **Velocity-based thickness:** Calculate mouse speed per frame (`Δx² + Δy²`). Map fast movement → thin stroke (`lineWidth: 1`), slow movement → thick stroke (`lineWidth: 6`). Apply smoothly with lerp.
- **Every frame:** call `ctx.clearRect(0, 0, width, height)` to make the canvas fully transparent — this ensures the canvas never visually covers the content beneath it.
- Draw the trail as **N-1 individual segments** (NOT one single path). Each segment gets its own `strokeStyle` with a decreasing alpha to produce the smooth fading tail:
  ```typescript
  // segment i draws from points[i] to points[i+1]
  const alpha = (1 - i / N) * 0.9; // tip brightest, tail nearly invisible
  ctx.strokeStyle = hexToRgba(ACCENT_COLORS[colorIndex], alpha);
  ctx.lineWidth = velocityLineWidth;
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
  ctx.beginPath();
  ctx.moveTo(points[i].x, points[i].y);
  ctx.lineTo(points[i + 1].x, points[i + 1].y);
  ctx.stroke();
  ```

**Color model (WattVision palette cycling):**

- Maintain an array of accent colors: `['#00E5FF', '#32D74B', '#FF453A']` (Cyan, Green, Red).
- Maintain a `colorIndex` ref (starts at 0).
- On every `focus` event: `colorIndex = (colorIndex + 1) % 3`.
- Apply a canvas glow matching the current color: `ctx.shadowColor = ACCENT_COLORS[colorIndex]`, `ctx.shadowBlur = 8`.
- Alpha per segment: `alpha = (1 - i / N) * 0.9` — tip is brightest, tail fades.
- Helper: `hexToRgba(hex, alpha)` converts the current palette hex to an `rgba()` string for the strokeStyle.

> **Why `clearRect` + segments instead of `fillRect` fade?**
> Because the canvas sits at `z-50` over all content, using `fillRect` with a translucent color would stack opacity 60× per second and eventually paint an opaque rectangle over the entire website. `clearRect` resets the canvas to fully transparent every frame, and the fading tail is achieved mathematically through per-segment alpha — the canvas never covers content.

---

#### Plan B — Particle Trail (Fallback — activate via `CANVAS_MODE`)

Only implement if Plan A fails visual QA.

**Particle model:**

- Each `mousemove` event, spawn 2–3 `Particle` objects at the cursor position:
  ```typescript
  type Particle = {
    x: number;
    y: number;
    vx: number;
    vy: number;
    alpha: number;
    size: number;
  };
  ```
- Each particle gets a random velocity with slight upward drift (`vy: -0.3 to -0.8`) and lateral scatter (`vx: ±0.5`).
- Each frame: `clearRect` the canvas, update position, decay `alpha -= 0.015`, decay `size *= 0.97`.
- Remove particles with `alpha <= 0`.
- Draw each as a filled circle with its current alpha: `ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)`.
- Color: use the same `ACCENT_COLORS[colorIndex]` logic + matching glow.

---

**Resize handling (both plans):**

- Listen to `window.resize` and update `canvas.width` / `canvas.height`.
- Re-center the initial mouse position after resize.

**Cleanup (both plans):**

- Cancel `animationFrameId` in `useEffect` cleanup.
- Remove all event listeners (`mousemove`, `focus`, `blur`, `resize`, `visibilitychange`).

### Acceptance Criteria

- [ ] Canvas renders over all content on every page without layout shift.
- [ ] Clicking links and buttons through the canvas works normally (`pointer-events: none` verified).
- [ ] Text and images behind the canvas remain fully crisp and unobscured at all times.
- [ ] Chrome Task Manager shows ≤ 2% CPU when mouse is idle for 5 seconds.
- [ ] Animation pauses when browser tab is backgrounded.
- [ ] Ribbon visibly gets thinner during fast mouse sweeps and thicker during slow drags (Plan A).
- [ ] Glow effect matches the current accent color and is visible in both Dark and Light themes.
- [ ] No memory leaks detectable in Chrome DevTools Memory panel after 5 page navigations.

### Agent Prompt

```
Write a React component at `src/features/canvas/RibbonCanvas.tsx` using the "use client" directive and HTML5 Canvas.

At the top of the file, declare:
  const CANVAS_MODE: 'ribbon' | 'particles' = 'ribbon';
This lets us switch to the particle fallback without rewriting the component.

--- PLAN A: ribbon mode ---
Implement a spring-physics ribbon (active when CANVAS_MODE === 'ribbon'):
- 30 trailing points. Each point has: x, y, vx, vy.
- Spring factor SPRING = 0.12, damping DAMPING = 0.75.
  Each frame: vx += (targetX - x) * SPRING; vx *= DAMPING; x += vx. Same for y.
- Track mouse speed each frame (dx² + dy²). Lerp a `lineWidth` value between 1 (fast) and 6 (slow).
- CRITICAL — every frame: call ctx.clearRect(0, 0, canvas.width, canvas.height) FIRST.
  Do NOT use fillRect with a translucent color — the canvas is a z-50 overlay and fillRect
  would stack opacity and cover all website content beneath it.
- Draw the trail as N-1 individual line segments (NOT one single path):
    for each segment i from 0 to N-2:
      alpha = (1 - i / N) * 0.9
      ctx.strokeStyle = hexToRgba(ACCENT_COLORS[colorIndex], alpha)
      ctx.lineWidth = currentLineWidth
      ctx.lineCap = 'round'; ctx.lineJoin = 'round'
      ctx.beginPath(); ctx.moveTo(points[i].x, points[i].y)
      ctx.lineTo(points[i+1].x, points[i+1].y); ctx.stroke()
- Cycle colors on window focus: const ACCENT_COLORS = ['#00E5FF', '#32D74B', '#FF453A']; update colorIndex ref.
- Apply glow matching current color: ctx.shadowColor = ACCENT_COLORS[colorIndex], ctx.shadowBlur = 8.
- Add a hexToRgba(hex: string, alpha: number): string helper function.

--- PLAN B: particle mode (fallback) ---
Implement a particle emitter (active when CANVAS_MODE === 'particles'):
- On mousemove, spawn 2-3 Particle objects: { x, y, vx: ±0.5 random, vy: -0.3 to -0.8, alpha: 1, size: 3 }
- Each frame: ctx.clearRect first, then update position, alpha -= 0.015, size *= 0.97. Remove dead particles.
- Draw each as ctx.arc filled circle with per-particle alpha. Same ACCENT_COLORS cycling + glow.

--- SHARED requirements ---
- Canvas className: "fixed inset-0 z-50 pointer-events-none" — full viewport, above content, non-blocking.
- Pause RAF loop on visibilitychange hidden, resume on visible.
- Handle window resize: update canvas dimensions.
- Clean up all event listeners and cancel animationFrameId in useEffect return.
- TypeScript strict: no `any`, all points and particles typed, all refs typed.
- pnpm, no extra libraries — pure HTML5 Canvas only.
```

---

## Step 3 — Root Layout, Navigation & Theme

**Goal:** Build the persistent shell that wraps every public page: root layout with `<RibbonCanvas />`, responsive Side Navbar, and light/dark theme system.

**Design source:** Read the Stitch design via MCP before writing layout or navigation code.

### Decisions

- **Side Navbar (Desktop):** Fixed left sidebar matching the portfolio design — circular avatar, vertical nav links (Home, About, Skills, Projects, Experience, Education, Recommendations, Contact), Download CV button at bottom.
- **Mobile Drawer:** shadcn/ui `Sheet` component triggered by a hamburger icon — replaces the sidebar on screens below `lg` breakpoint.
- **Theme:** `next-themes` with `attribute="class"` for Tailwind dark mode. System preference is the default. User toggle stored in a cookie to prevent flash.
- **`<RibbonCanvas />`** is placed directly in the root `layout.tsx` so it persists across all client navigations without re-mounting.

### Tasks

**3a — Install theme support**

```bash
pnpm add next-themes
```

**3b — Root layout** (`src/app/layout.tsx`)

- Wrap with `<ThemeProvider attribute="class" defaultTheme="system" enableSystem>`.
- Render `<RibbonCanvas />` as a sibling of `{children}` (not wrapping it).
- Apply `next/font` for Inter and JetBrains Mono — never load fonts from CDN.

**3c — NavBar component** (`src/features/nav/NavBar.tsx`)

- Desktop: `<nav>` with `position: fixed; left: 0; top: 0; height: 100vh; width: 240px`.
- Active link detection: use `usePathname()` for current route highlight.
- Download CV: Server-rendered `<a>` tag pointing to the Vercel Blob signed URL for the CV PDF.
- Mobile: Replace with shadcn/ui `<Sheet>` triggered by a `lucide-react` `Menu` icon.

**3d — Theme toggle** (`src/shared/ui/ThemeToggle.tsx`)

- Use `useTheme()` from `next-themes`. Render `Sun` / `Moon` icon from `lucide-react`.
- Mark as `"use client"` — it reads browser state.

### Acceptance Criteria

- [ ] Nav persists across all page navigations (no re-mount flash).
- [ ] Active nav link is visually distinct from inactive links.
- [ ] `<RibbonCanvas />` is present and interactive on every public page.
- [ ] Dark mode toggle works without flash of unstyled content (FOUC).
- [ ] Mobile drawer opens and closes without layout shift.
- [ ] No fonts loaded from external CDNs (verified in Network tab).

### Agent Prompt

```
Using the design tokens and layout from the Stitch MCP design tool:

1. Create `src/app/layout.tsx` — root layout wrapping all pages with:
   - next/font for Inter (body) and JetBrains Mono (code/metrics).
   - ThemeProvider from next-themes (attribute="class", defaultTheme="system").
   - <RibbonCanvas /> rendered as a fixed overlay.
   - <NavBar /> rendered as the left sidebar.

2. Create `src/features/nav/NavBar.tsx`:
   - Fixed left sidebar (240px wide, full height) on lg+ screens.
   - Circular avatar at top, vertical nav links in middle, Download CV <a> at bottom.
   - Links: Home (/), About (/about), Skills (/skills), Projects (/projects),
     Experience (/experience), Education (/education),
     Recommendations (/recommendations), Contact (/contact).
   - Active state via usePathname().
   - On screens below lg: render a shadcn/ui Sheet with a Menu trigger icon.

3. Create `src/shared/ui/ThemeToggle.tsx` — "use client" — Sun/Moon icon toggle using useTheme().

Constraints: TypeScript strict, FSD imports (no cross-feature imports), pnpm, RSC by default.
```

---

## Step 4 — Public Portfolio Pages & Animations

**Goal:** Build all 8 public-facing pages as React Server Components, fetching data from Turso via Drizzle ORM, cached with `"use cache"` and tag-based invalidation.

**Design source:** Read each page's layout from the Stitch MCP design tool before building it.

### Decisions

- **RSC by default.** All 8 pages are Server Components. Client islands are added only for interactive elements (contact form, theme toggle).
- **`"use cache"` + `cacheTag`.** Data-fetching RSCs are annotated to produce static shells that are instantly navigable and invalidated by admin mutations.
- **Animations on initial load.** `animate-tailwind` classes are applied inline on the server-rendered HTML — no JS required for entrance animations. Use stagger delays (`animate-delay-100`, `animate-delay-200`, …).

### Page Specifications

| Route              | Page            | Key UI Blocks                            | Data Source                          |
| ------------------ | --------------- | ---------------------------------------- | ------------------------------------ |
| `/`                | Home / Hero     | Name, title, intro, CTA buttons          | Vercel Edge Config                   |
| `/about`           | About           | Portrait + bio, bracket frame            | Vercel Edge Config                   |
| `/skills`          | Skills          | Responsive grid of tech logo cards       | Turso `skills` table                 |
| `/projects`        | Projects        | Feature cards: image, title, tags        | Turso `projects` table               |
| `/experience`      | Experience      | Vertical timeline with company cards     | Turso `experiences` table            |
| `/education`       | Education       | Education cards with institution logos   | Vercel Edge Config                   |
| `/recommendations` | Recommendations | Testimonial cards with avatars           | Vercel Edge Config                   |
| `/contact`         | Contact         | Form (left) + contact info cards (right) | Static (form posts to Server Action) |

### Tasks

**4a — Shared data access layer** (`src/shared/lib/db/queries.ts`)

```typescript
import { db } from "./client";
import { skills, experiences, projects } from "./schema";
import { CACHE_TAGS } from "../cache-tags";
import { unstable_cacheTag as cacheTag } from "next/cache";
import { asc } from "drizzle-orm";

export async function getSkills() {
  "use cache";
  cacheTag(CACHE_TAGS.skills);
  return db.select().from(skills).orderBy(asc(skills.sortOrder));
}

export async function getExperiences() {
  "use cache";
  cacheTag(CACHE_TAGS.experiences);
  return db.select().from(experiences).orderBy(asc(experiences.sortOrder));
}

export async function getProjects() {
  "use cache";
  cacheTag(CACHE_TAGS.projects);
  return db.select().from(projects).orderBy(asc(projects.sortOrder));
}
```

**4b — Skills page animation pattern** (`src/app/(portfolio)/skills/page.tsx`)

```tsx
import { getSkills } from "@/shared/lib/db/queries";

export default async function SkillsPage() {
  const skills = await getSkills();
  return (
    <section>
      <h1>Skills</h1>
      <ul className="grid grid-cols-3 gap-4 md:grid-cols-5">
        {skills.map((skill, i) => (
          <li
            key={skill.id}
            className={`animate-zoom-in animate-delay-${(i % 10) * 100}`}
          >
            {/* Skill card UI from MCP design */}
          </li>
        ))}
      </ul>
    </section>
  );
}
```

**4c — Projects page** — Large feature cards with `image_url` from Vercel Blob, pill-shaped `techStack` badges, `featured` items rendered first.

**4d — Experience page** — Vertical timeline: thin left border line, dark cards with `company`, `role`, date range (`start_date` → `end_date ?? "Present"`).

### Acceptance Criteria

- [ ] All 8 pages render as RSCs (zero `"use client"` in page files).
- [ ] Lighthouse Performance score ≥ 95 on the Home page.
- [ ] LCP < 2.5s, CLS < 0.1, INP < 200ms on P75 (measured via Vercel Speed Insights).
- [ ] Client JS bundle ≤ 150 KB gzipped per route (verified with `next build --analyze`).
- [ ] `tw-animate-css` entrance animations fire on initial paint without layout shift.

### Agent Prompt

```
Using the Stitch MCP design tool to read the exact layout specs for each page:

Generate React Server Component page files for all 8 portfolio routes:
/, /about, /skills, /projects, /experience, /education, /recommendations, /contact

Rules:
- Every page file must be a Server Component (no "use client").
- Skills, Projects, Experience: call the matching getX() query from src/shared/lib/db/queries.ts.
- Home, About, Education, Recommendations: read static content from Vercel Edge Config.
- Apply tw-animate-css stagger classes to list items (animate-zoom-in, animate-delay-N).
- Use next/image with explicit width/height and priority on hero images.
- FSD: import only from @/shared or @/features — no direct cross-feature imports.
- TypeScript strict: all props typed, no implicit returns.
```

---

## Step 5 — Contact Form

**Goal:** Build the contact form as a hybrid: React Hook Form client island for validation UX, React 19 Server Action for submission, Turso for message storage.

### Decisions

- **React Hook Form + Zod client-side** — instant inline validation without a round-trip.
- **`useActionState` + `useFormStatus` (React 19)** — manage loading/success/error state natively without external state management.
- **Server Action re-validates with Zod** — never trust the client payload; parse again server-side before writing to DB.
- **Rate limiting** — add a simple IP-based rate limit (e.g. 3 submissions per hour) in the Server Action using Vercel's Edge API or `@upstash/ratelimit`.

### Shared Zod Schema (`src/shared/lib/validators/contact.ts`)

```typescript
import { z } from "zod";

export const ContactSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  message: z.string().min(10).max(2000),
});

export type ContactInput = z.infer<typeof ContactSchema>;
```

### Server Action (`src/features/contact/actions.ts`)

```typescript
"use server";

import { ContactSchema } from "@/shared/lib/validators/contact";
import { db } from "@/shared/lib/db/client";
import { contactMsgs } from "@/shared/lib/db/schema";

export async function submitContact(_prevState: unknown, formData: FormData) {
  const raw = {
    name: formData.get("name"),
    email: formData.get("email"),
    message: formData.get("message"),
  };

  const parsed = ContactSchema.safeParse(raw);
  if (!parsed.success) {
    return { success: false, errors: parsed.error.flatten().fieldErrors };
  }

  await db.insert(contactMsgs).values(parsed.data);
  return { success: true };
}
```

### Client Component (`src/features/contact/ContactForm.tsx`)

```typescript
"use client";
// useActionState wraps submitContact
// useFormStatus drives the submit button disabled/loading state
// React Hook Form + zodResolver for instant inline validation
// On success: show toast notification, reset form
```

### Acceptance Criteria

- [ ] Form shows inline Zod errors without submitting.
- [ ] Submit button shows loading spinner via `useFormStatus`.
- [ ] On success: toast appears, form resets.
- [ ] Message appears in Turso `contact_msgs` table.
- [ ] Server Action rejects malformed payloads (test with curl).
- [ ] Submitting the form 4+ times in an hour returns a 429-equivalent error message.

### Agent Prompt

```
Generate the Contact form feature:

1. Shared Zod schema at `src/shared/lib/validators/contact.ts` — name (2-100), email, message (10-2000).

2. Server Action at `src/features/contact/actions.ts`:
   - "use server" at the top.
   - Accept FormData, re-validate with ContactSchema.safeParse().
   - On valid: insert into contactMsgs table via Drizzle.
   - Return { success: boolean, errors?: FieldErrors }.

3. Client Component at `src/features/contact/ContactForm.tsx`:
   - "use client".
   - React Hook Form with zodResolver(ContactSchema) for instant inline validation.
   - useActionState(submitContact, null) for submission state.
   - useFormStatus() inside a SubmitButton sub-component for loading state.
   - On success: call toast() (shadcn/ui Toaster) and reset().

Use the Stitch MCP design for the form layout (left: inputs, right: contact info cards).
TypeScript strict, no `any`, pnpm.
```

---

## Step 5.1 — Real-time Events with Ably

**Goal:** Wire Ably as a thin WebSocket event bus on top of Turso. Turso remains the database of record. Ably only carries lightweight event notifications — never raw data payloads — so the admin UI re-fetches from Turso on each event (source of truth stays in the DB).

> **Architecture principle:** Server Action → write to Turso → publish Ably event (tiny notification) → client receives event → re-fetch from Turso (or update local state). Never send sensitive DB rows through Ably.

### Decisions

- **Ably over Pusher** — 6M messages/month free tier vs Pusher's 100 concurrent connections; Ably's `LiveSync` keeps UI reactive to DB state with minimal code.
- **Token authentication** — never expose `ABLY_API_KEY` to the browser. The admin client requests a short-lived token from a protected Route Handler (`/api/ably-token`) before connecting.
- **Two channels in use:**
  - `admin-inbox` — fires when a new contact message is submitted
  - `portfolio-updates` — (optional) fires when admin mutates skills/experience/projects, can drive live UI updates for visitors

### Tasks

**5.1a — Install Ably**

```bash
pnpm add ably
```

**5.1b — Channel constants** (`src/shared/lib/ably-channels.ts`)

```typescript
// Single source of truth for all Ably channel + event names
export const ABLY_CHANNELS = {
  adminInbox: "admin-inbox",
  portfolioUpdates: "portfolio-updates",
} as const;

export const ABLY_EVENTS = {
  newMessage: "new-message",
  contentUpdated: "content-updated",
} as const;
```

**5.1c — Server-side publisher utility** (`src/shared/lib/ably-publisher.ts`)

```typescript
import Ably from "ably";

// Ably.Rest is safe in Serverless / Edge — it uses HTTP, not a persistent socket
const ably = new Ably.Rest(process.env.ABLY_API_KEY!);

export async function publishAblyEvent(
  channel: string,
  event: string,
  data: Record<string, unknown>,
) {
  await ably.channels.get(channel).publish(event, data);
}
```

**5.1d — Update `submitContact` Server Action** (`src/features/contact/actions.ts`)

Add the Ably publish call immediately after the successful Turso insert:

```typescript
"use server";

import { ContactSchema } from "@/shared/lib/validators/contact";
import { db } from "@/shared/lib/db/client";
import { contactMsgs } from "@/shared/lib/db/schema";
import { publishAblyEvent } from "@/shared/lib/ably-publisher";
import { ABLY_CHANNELS, ABLY_EVENTS } from "@/shared/lib/ably-channels";

export async function submitContact(_prevState: unknown, formData: FormData) {
  const raw = {
    name: formData.get("name"),
    email: formData.get("email"),
    message: formData.get("message"),
  };

  const parsed = ContactSchema.safeParse(raw);
  if (!parsed.success) {
    return { success: false, errors: parsed.error.flatten().fieldErrors };
  }

  const [inserted] = await db
    .insert(contactMsgs)
    .values(parsed.data)
    .returning();

  // Notify admin inbox in real-time — only send non-sensitive preview data
  await publishAblyEvent(ABLY_CHANNELS.adminInbox, ABLY_EVENTS.newMessage, {
    id: inserted.id,
    name: parsed.data.name,
    preview: parsed.data.message.slice(0, 80),
    createdAt: inserted.createdAt?.toISOString(),
  });

  return { success: true };
}
```

**5.1e — Token auth Route Handler** (`src/app/api/ably-token/route.ts`)

Admin-only endpoint — never accessible to public visitors:

```typescript
import Ably from "ably";
import { auth } from "@/features/auth/auth";
import { NextResponse } from "next/server";

export async function GET() {
  const session = await auth();

  // Only the admin gets an Ably token
  if (session?.user?.email !== process.env.ALLOWED_ADMIN_EMAIL) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const rest = new Ably.Rest(process.env.ABLY_API_KEY!);
  const tokenRequest = await rest.auth.createTokenRequest({
    clientId: session.user.email!,
    capability: {
      "admin-inbox": ["subscribe"],
      "portfolio-updates": ["subscribe"],
    },
    ttl: 3_600_000, // 1 hour
  });

  return NextResponse.json(tokenRequest);
}
```

**5.1f — Admin inbox real-time hook** (`src/features/admin/hooks/useAdminInbox.ts`)

```typescript
"use client";

import Ably from "ably";
import { useEffect, useRef, useState } from "react";
import { ABLY_CHANNELS, ABLY_EVENTS } from "@/shared/lib/ably-channels";

export type InboxPreview = {
  id: number;
  name: string;
  preview: string;
  createdAt: string;
};

export function useAdminInbox(initial: InboxPreview[]) {
  const [inbox, setInbox] = useState(initial);
  const clientRef = useRef<Ably.Realtime | null>(null);

  useEffect(() => {
    // Token auth — never exposes ABLY_API_KEY to the browser
    const client = new Ably.Realtime({ authUrl: "/api/ably-token" });
    clientRef.current = client;

    const channel = client.channels.get(ABLY_CHANNELS.adminInbox);

    channel.subscribe(ABLY_EVENTS.newMessage, (msg) => {
      setInbox((prev) => [msg.data as InboxPreview, ...prev]);
    });

    return () => {
      channel.unsubscribe();
      client.close();
    };
  }, []);

  return inbox;
}
```

**5.1g — Wire the hook into the admin messages page**

```typescript
// src/app/(admin)/admin/messages/page.tsx — RSC loads initial data
import { db } from '@/shared/lib/db/client';
import { contactMsgs } from '@/shared/lib/db/schema';
import { desc } from 'drizzle-orm';
import { MessagesClient } from '@/features/admin/MessagesClient';

export default async function MessagesPage() {
  const messages = await db
    .select()
    .from(contactMsgs)
    .orderBy(desc(contactMsgs.createdAt));

  return <MessagesClient initial={messages} />;
}
```

```typescript
// src/features/admin/MessagesClient.tsx — "use client" island
"use client";

import { useAdminInbox } from "./hooks/useAdminInbox";
import type { InboxPreview } from "./hooks/useAdminInbox";

export function MessagesClient({ initial }: { initial: InboxPreview[] }) {
  const inbox = useAdminInbox(initial);
  // Render master-detail list — inbox is live-updated
}
```

### Acceptance Criteria

- [ ] Submitting the contact form from a browser → new message appears in admin inbox **without a page refresh**.
- [ ] Ably token request to `/api/ably-token` returns 401 for unauthenticated requests.
- [ ] `ABLY_API_KEY` is never visible in browser Network tab (only the short-lived token appears).
- [ ] Closing the browser tab cleanly unsubscribes the Ably channel (no zombie connections).
- [ ] Ably dashboard shows zero error events after 10 test submissions.

### Agent Prompt

```
Add real-time admin inbox using Ably on top of the existing Turso + Drizzle stack.

1. Install: pnpm add ably

2. Create channel + event name constants at `src/shared/lib/ably-channels.ts`.

3. Create a server-side Ably publisher utility at `src/shared/lib/ably-publisher.ts`
   using Ably.Rest (HTTP-based, safe for serverless).

4. Update `src/features/contact/actions.ts` (submitContact Server Action):
   After the Drizzle insert, call publishAblyEvent() with a small preview payload
   (id, name, first 80 chars of message, createdAt). Never send the full message body.

5. Create a protected Route Handler at `src/app/api/ably-token/route.ts`:
   - Verify session.user.email === ALLOWED_ADMIN_EMAIL.
   - Return a short-lived Ably TokenRequest (1 hour TTL) with subscribe-only capability
     on admin-inbox and portfolio-updates channels.
   - Return 401 for any other caller.

6. Create `src/features/admin/hooks/useAdminInbox.ts`:
   - "use client"
   - Ably.Realtime initialized with { authUrl: '/api/ably-token' } — never raw API key.
   - Subscribe to admin-inbox / new-message events.
   - Prepend incoming messages to local state (setInbox prev => [msg.data, ...prev]).
   - Clean up (unsubscribe + client.close()) in useEffect return.

7. Refactor the admin messages page to: RSC (fetches initial rows from Turso) +
   <MessagesClient initial={messages} /> "use client" island that calls useAdminInbox().

TypeScript strict, no `any`, pnpm, FSD structure.
```

---

## Step 5.2 — AI Portfolio Assistant (RAG via Context Stuffing)

**Goal:** Provide visitors with a floating, dark-themed AI chat widget ("Mojahed's AI Assistant") that can answer questions about your skills, experience, projects, and availability in real-time.

> **Architecture principle:** The portfolio data is small enough to fit entirely within the LLM's context window. Instead of a complex Vector DB / Embedding pipeline, we use **Context Stuffing**. A Server Route Handler fetches your data from Turso, builds a massive system prompt, and sends it to the LLM.

### Decisions

- **LLM:** Google Gemini 3.5 Flash (via `@ai-sdk/google`). Selected for its massive 1M token context window and highly generous free tier (1,500 req/day), making context-stuffing completely viable and free.
- **SDK:** Vercel AI SDK (`ai`). Provides the standard `useChat` hook and `streamText` response for instant ChatGPT-like typewriter effects.
- **UI:** A floating chat popover widget adhering to the WattVision dark mode design, rendering markdown responses. Includes "Suggested Inquiries" chips for zero-typing onboarding.

### Tasks

**5.2a — Install AI dependencies**

```bash
pnpm add ai @ai-sdk/google
```

**5.2b — Build the Context Fetcher** (`src/features/ai-assistant/buildContext.ts`)

```typescript
import { db } from "@/shared/lib/db/client";
import { skills, experiences, projects } from "@/shared/lib/db/schema";

export async function buildPortfolioContext(): Promise<string> {
  const [allSkills, allExp, allProj] = await Promise.all([
    db.select().from(skills),
    db.select().from(experiences),
    db.select().from(projects),
  ]);

  const formatSkills = allSkills.map((s) => `- ${s.name}`).join("\\n");
  const formatExp = allExp
    .map(
      (e) =>
        `- **${e.role}** at ${e.company} (${e.duration}): ${e.description}`,
    )
    .join("\\n");
  const formatProj = allProj
    .map((p) => `- **${p.title}**: ${p.summary} (Tech: ${p.techStack})`)
    .join("\\n");

  return `
    You are Mojahed's AI Portfolio Assistant.
    Answer questions about him accurately using ONLY the data below.
    Be professional, concise, and helpful.
    
    [SKILLS]
    ${formatSkills}
    
    [EXPERIENCE]
    ${formatExp}
    
    [PROJECTS]
    ${formatProj}
  `;
}
```

**5.2c — Streaming Route Handler** (`src/app/api/chat/route.ts`)

```typescript
import { streamText } from "ai";
import { google } from "@ai-sdk/google";
import { buildPortfolioContext } from "@/features/ai-assistant/buildContext";

export async function POST(req: Request) {
  const { messages } = await req.json();
  const context = await buildPortfolioContext();

  const result = await streamText({
    model: google("gemini-3.5-flash"),
    system: context,
    messages,
  });

  return result.toDataStreamResponse();
}
```

**5.2d — Chat Widget UI** (`src/features/ai-assistant/ChatWidget.tsx`)

- Implement a floating FAB (Floating Action Button) that toggles a chat popover.
- Use the `useChat` hook from `ai/react`.
- Include pre-defined chip buttons (e.g., "What is his tech stack?", "Is he open to full-time roles?").
- Mount this globally in the root `layout.tsx` so it persists across page navigation.

### Acceptance Criteria

- [ ] Widget appears in the bottom corner globally across the public portfolio.
- [ ] Clicking a suggested chip immediately sends the message and streams the response.
- [ ] The LLM accurately quotes the user's latest Turso-stored project data without hallucination.
- [ ] Responses stream smoothly token-by-token.

### Agent Prompt

```
Build the AI Portfolio Assistant feature using Gemini 3.5 Flash and Vercel AI SDK.

1. Install dependencies: pnpm add ai @ai-sdk/google

2. Create `src/features/ai-assistant/buildContext.ts` that fetches all rows from
   skills, experiences, and projects in Turso (Drizzle) and returns a large
   stringified system prompt containing all of Mojahed's data.

3. Create the route handler `src/app/api/chat/route.ts` (POST) that:
   - Extracts `messages` from req.json()
   - Calls buildPortfolioContext() to get the system prompt
   - Calls streamText() using model google('gemini-3.5-flash')
   - Returns result.toDataStreamResponse()

4. Build `src/features/ai-assistant/ChatWidget.tsx` (Client Component):
   - A floating button (bottom right) that opens a chat window.
   - Use the `useChat` hook from 'ai/react'.
   - The chat window must match the WattVision dark UI aesthetic.
   - Add a "Suggested Inquiries" section with 2-3 clickable chips that append
     and submit the question immediately.
   - Render the LLM responses cleanly (support basic markdown if possible).

5. Add <ChatWidget /> to the public root `src/app/layout.tsx`.

TypeScript strict, no `any`, pnpm, FSD structure.
```

---

## Step 6 — Admin CMS Dashboard & Authentication

**Goal:** Build a fully protected personal CMS at `/admin` for managing portfolio content (skills, experience, projects) and reading contact messages.

### Decisions

- **Auth.js v5 with GitHub OAuth** — zero user table, single allowed email, production-grade session management.
- **Edge Proxy guard (`proxy.ts`)** — protection happens at the network edge before any rendering, eliminating flash-of-unauthenticated content.
- **Side Navbar** — Admin layout uses its own fixed left sidebar (separate from the public nav) matching the WattVision-inspired dark dashboard design.
- **`useOptimistic`** — admin mutations update the UI instantly; server confirms or rolls back.
- **`revalidateTag` on every write** — public cached pages automatically purge when admin saves a change.
- **Ably for messages inbox** — real-time live updates handled by Step 5.1; Step 6 wires it into the full admin layout.

### 6.1 Authentication Setup

**`src/features/auth/auth.ts`**

```typescript
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [GitHub],
  callbacks: {
    signIn({ profile }) {
      return profile?.email === process.env.ALLOWED_ADMIN_EMAIL;
    },
  },
});
```

**`src/proxy.ts`** — combined: CSP nonce + admin guard

```typescript
import { auth } from "@/features/auth/auth";
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export const proxy = auth((req) => {
  const isAdminRoute = req.nextUrl.pathname.startsWith("/admin");
  if (isAdminRoute && !req.auth) {
    return NextResponse.redirect(new URL("/api/auth/signin", req.url));
  }
});

export const config = { matcher: ["/admin/:path*"] };
```

### 6.2 Admin Routes

| Route               | Purpose            | Key UI                                                                           |
| ------------------- | ------------------ | -------------------------------------------------------------------------------- |
| `/admin`            | Dashboard overview | 3 KPI cards (projects, skills, unread messages) + Activity log + Message preview |
| `/admin/skills`     | Manage skills      | DataTable + Add/Edit Dialog + Delete confirm                                     |
| `/admin/experience` | Manage experience  | DataTable + Add/Edit Dialog + Delete confirm                                     |
| `/admin/projects`   | Manage projects    | DataTable + Add/Edit Dialog + Vercel Blob image upload                           |
| `/admin/messages`   | Contact inbox      | Master-Detail layout: list + message viewer + mark-read                          |

### 6.3 UI Patterns

**DataTable** (`src/features/admin/DataTable.tsx`)

- Built on shadcn/ui `DataTable` (TanStack Table v9).
- Columns: sortable, searchable inline.
- Row actions: Edit (Dialog), Delete (AlertDialog confirm).

**CRUD Dialog** (`src/features/admin/CrudDialog.tsx`)

- shadcn/ui `Dialog` + `Form`.
- React Hook Form + Zod schema matching the DB table.
- Server Action called on submit.
- `useOptimistic` updates the parent list instantly.

**Image Upload** (projects only)
_Rule: Enforce client-side compression (e.g., using `browser-image-compression` or HTML5 canvas) in the Client Component to resize/compress images to < 500KB before appending to `FormData`._

```typescript
// Server Action
async function uploadProjectImage(formData: FormData) {
  "use server";
  const file = formData.get("image") as File; // Already compressed by client
  const blob = await put(file.name, file, { access: "public" });
  return blob.url;
}
```

### 6.4 Mutation + Cache Invalidation Pattern

```typescript
// Example: create a skill
export async function createSkill(_: unknown, formData: FormData) {
  "use server";
  const session = await auth();
  const allowedEmails = (process.env.ALLOWED_ADMIN_EMAILS || "").split(",");
  if (!session?.user?.email || !allowedEmails.includes(session.user.email)) {
    throw new Error("Unauthorized");
  }
  const parsed = SkillSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.flatten() };
  await db.insert(skills).values(parsed.data);
  revalidateTag(CACHE_TAGS.skills); // ← purges public /skills page cache
  return { success: true };
}
```

### Acceptance Criteria

- [ ] Visiting `/admin` without a GitHub session redirects to `/api/auth/signin`.
- [ ] Signing in with a non-allowed GitHub account is rejected.
- [ ] Creating a skill in admin causes the public `/skills` page to show updated data on next visit.
- [ ] Deleting a project removes it from both the admin table (optimistic) and the public page (after revalidation).
- [ ] Image upload stores to Vercel Blob and URL is saved to DB.
- [ ] Every mutation Server Action re-authenticates the session independently.

### Agent Prompt

```
Act as a full-stack engineer. Build the Admin CMS feature.

1. Auth config at `src/features/auth/auth.ts` — Auth.js v5, GitHub OAuth provider,
   signIn callback restricts to `ALLOWED_ADMIN_EMAILS` env var (parsed as comma-separated array).

2. Route handler at `src/app/api/auth/[...nextauth]/route.ts` — export handlers.

3. Proxy at `src/proxy.ts` (Next 16+) — guard /admin/:path* using auth() with `export const proxy`.
   Redirect unauthenticated requests to /api/auth/signin.

4. Admin layout at `src/app/(admin)/layout.tsx`:
   - Fixed left sidebar (240px) with: logo, nav links (Dashboard, Skills, Experience,
     Projects, Messages [unread badge]), Sign Out button, "← View Portfolio" link.
   - Main content area to the right.
   - No <RibbonCanvas /> — admin area is clean and focused.

5. For each entity (skills, experiences, projects):
   - Page: `src/app/(admin)/admin/[entity]/page.tsx` — RSC, fetch all rows, render <DataTable />.
   - DataTable: shadcn/ui DataTable (TanStack Table v9) with Edit and Delete row actions.
   - CRUD Dialog: shadcn/ui Dialog with React Hook Form + zodResolver.
   - Server Actions: create, update, delete — each re-authenticates session, validates with Zod,
     writes to Drizzle, calls revalidateTag(CACHE_TAGS.[entity]).
   - useOptimistic in the parent Client Component for instant row add/remove.

6. Messages page: RSC master-detail layout. Server Actions for mark-as-read (flips boolean) and delete. Both must re-authenticate the session, write to Drizzle, and use `useOptimistic` for instant UI updates.

TypeScript strict, no `any`, pnpm, FSD structure.
```

---

## Step 7 — Optimization, Accessibility & Deployment

**Goal:** Verify production readiness across performance, accessibility, security, and observability before deploying to Vercel.

### 7.1 Performance Checklist

| Check            | Target                    | Tool                                    |
| ---------------- | ------------------------- | --------------------------------------- |
| LCP              | < 2.5s                    | Vercel Speed Insights                   |
| INP              | < 200ms                   | Chrome DevTools → Performance tab       |
| CLS              | < 0.1                     | Lighthouse                              |
| Bundle per route | ≤ 150 KB gzipped          | `pnpm build --analyze`                  |
| Canvas RAF CPU   | ≤ 2% at idle              | Chrome Task Manager                     |
| Canvas FPS       | Stable 60fps on mousemove | `requestAnimationFrame` timestamp delta |

**Canvas performance check:**

```bash
# In Chrome DevTools → Performance:
# Record 5s of mouse movement across the portfolio
# Verify: no long tasks (>50ms) in the Main thread
# Verify: "Rendering" time < 4ms per frame
```

### 7.2 Accessibility Checklist

- [ ] All interactive elements are keyboard-reachable and visible focus ring present.
- [ ] Color contrast ≥ 4.5:1 for body text (verified with axe DevTools).
- [ ] All `<img>` / `<Image>` have descriptive `alt` text.
- [ ] Modal dialogs (admin CRUD) trap focus and restore on close.
- [ ] `axe-core` CI integration reports zero violations.

### 7.3 Security Checklist

- [ ] CSP nonce headers active on all routes (verified in Response Headers).
- [ ] Auth tokens in `HttpOnly` cookies only (verify: no tokens in `localStorage`).
- [ ] Admin Server Actions re-authenticate session on every call.
- [ ] Sentry source maps active: trigger a deliberate error and verify demangled stack trace.

**Sentry test:**

```typescript
// Add temporarily to any page, then remove after verifying
throw new Error("[TEST] Sentry source map verification — delete me");
```

### 7.4 Pre-Deployment Gates (CI)

All PRs must pass before merge (mirrors `optimize_skill_frontend.md §6.3`):

```bash
pnpm tsc --noEmit                    # Zero TypeScript errors
pnpm eslint .                        # Zero lint warnings
pnpm vitest run                      # All tests pass
pnpm build                           # Production build succeeds
pnpm audit --audit-level high        # Zero high-severity vulnerabilities
```

### Agent Prompt

```
Perform a production readiness audit on the portfolio project.

1. Run `pnpm build` and report any warnings or errors.
2. Run `pnpm tsc --noEmit` and fix any type errors found.
3. Check all public page routes with Lighthouse (target: Performance ≥ 95, Accessibility = 100).
4. Identify any client component that could be converted to a Server Component to reduce JS payload.
5. Add a temporary throw new Error('Sentry test') to the Home page Server Component,
   verify the Sentry dashboard shows a demangled TypeScript stack trace, then remove it.
6. Confirm the canvas component pauses correctly on tab blur using Chrome DevTools
   Performance Profiler — show that no RAF callbacks fire while the tab is hidden.
7. Run axe-core on each public page and report any violations.

Report findings as: PASS / FAIL / WARN with fix recommendations for each FAIL.
```

---

## Step 8 — Collapsible Sidebar (Portfolio + Admin)

**Goal:** Upgrade both the public portfolio sidebar and the admin sidebar to support collapsing into icon-only mode, giving users extra screen real-estate and a polished UX touch. State persists across navigations via a cookie so the sidebar never flashes on load.

### Decisions

- **shadcn/ui `Sidebar` component** — the project already uses shadcn/ui; its first-class `Sidebar` primitives (`SidebarProvider`, `SidebarMenu`, `SidebarRail`, `SidebarTrigger`, etc.) handle collapse state, cookie persistence, RTL mirroring, keyboard a11y, and mobile `Sheet` fallback out of the box. No custom state management is required.
- **`collapsible="icon"` mode** — sidebar shrinks from 240 px (expanded, icon + label) to 64 px (collapsed, icon only). Labels are auto-hidden by shadcn/ui CSS; `Tooltip` components show them on hover in collapsed state.
- **Cookie-persisted state** — a `<SidebarCookieProvider>` RSC reads the `sidebar_state` cookie server-side and passes `defaultOpen` to `<SidebarProvider>`, preventing any flash between navigations.
- **RTL-aware** — the sidebar uses CSS logical properties (`inset-inline-start`) internally. When `dir="rtl"` is set on `<html>` (Step 9), the sidebar automatically renders on the right side and the collapse rail flips to the correct edge. No extra CSS required.
- **Independent state** — portfolio and admin sidebars use separate `<SidebarCookieProvider>` instances, so collapsing one does not affect the other.
- **Mobile unchanged** — below the `lg` breakpoint the sidebar continues to render as a shadcn/ui `Sheet` (full-screen drawer). The collapsible icon-mode only applies to `lg+` screens.

### Tasks

**8a — Install sidebar + tooltip components**

```bash
pnpm dlx shadcn@latest add sidebar tooltip
```

**8b — `SidebarCookieProvider`** (`src/shared/ui/SidebarCookieProvider.tsx`)

An RSC that reads the persisted cookie and passes `defaultOpen` to the client `SidebarProvider` — no client JS needed for initial render:

```typescript
import { cookies } from 'next/headers';
import { SidebarProvider } from '@/shared/ui/sidebar'; // shadcn generated file

export async function SidebarCookieProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  const cookieStore = await cookies();
  const defaultOpen = cookieStore.get('sidebar_state')?.value !== 'collapsed';
  return <SidebarProvider defaultOpen={defaultOpen}>{children}</SidebarProvider>;
}
```

**8c — Rebuild portfolio NavBar** (`src/features/nav/NavBar.tsx`)

Replace the existing fixed-`div` sidebar with shadcn/ui Sidebar primitives:

```typescript
// Structure (not full implementation — read MCP design for exact tokens)
<Sidebar collapsible="icon">
  <SidebarHeader>
    {/* Circular avatar — text label hidden when collapsed */}
  </SidebarHeader>

  <SidebarContent>
    <SidebarMenu>
      {NAV_LINKS.map((link) => (
        <SidebarMenuItem key={link.href}>
          <Tooltip>
            <TooltipTrigger asChild>
              <SidebarMenuButton asChild isActive={pathname === link.href}>
                <Link href={link.href}>
                  <link.Icon />
                  <span>{link.label}</span>  {/* auto-hidden when collapsed */}
                </Link>
              </SidebarMenuButton>
            </TooltipTrigger>
            <TooltipContent side="right">{link.label}</TooltipContent>
          </Tooltip>
        </SidebarMenuItem>
      ))}
    </SidebarMenu>
  </SidebarContent>

  <SidebarFooter>
    <ThemeToggle />
    {/* LocaleSwitcher added here in Step 9 */}
    <DownloadCVButton />
  </SidebarFooter>

  <SidebarRail /> {/* draggable collapse handle on the sidebar edge */}
</Sidebar>
```

- Replace all `ml-` / `mr-` / `pl-` / `pr-` / `left-` / `right-` Tailwind utilities with logical equivalents (`ms-`, `me-`, `ps-`, `pe-`, `start-`, `end-`) to support RTL in Step 9.
- Active link detection: `usePathname()` from `next/navigation` (unchanged from original).
- Mobile: shadcn/ui `Sidebar` automatically falls back to a `Sheet` drawer below `lg`. Wire `SidebarTrigger` to a `Menu` icon in the mobile header.

**8d — Wrap portfolio layout** (`src/app/[locale]/(portfolio)/layout.tsx`)

```typescript
import { SidebarCookieProvider } from '@/shared/ui/SidebarCookieProvider';
import { NavBar } from '@/features/nav/NavBar';

export default async function PortfolioLayout({ children }) {
  return (
    <SidebarCookieProvider>
      <NavBar />
      <main className="ps-[var(--sidebar-width)]">{children}</main>
    </SidebarCookieProvider>
  );
}
// Note: --sidebar-width CSS custom property is set automatically by SidebarProvider
// and transitions smoothly between expanded (240px) and collapsed (64px) values.
```

**8e — Build admin sidebar** (`src/features/admin/AdminSidebar.tsx`)

Same Sidebar primitives as the portfolio NavBar but with admin-specific links and no locale switcher:

```typescript
// Admin nav links
const ADMIN_LINKS = [
  { href: "/admin", label: "Dashboard", Icon: LayoutDashboard },
  { href: "/admin/skills", label: "Skills", Icon: Zap },
  { href: "/admin/experience", label: "Experience", Icon: Briefcase },
  { href: "/admin/projects", label: "Projects", Icon: FolderKanban },
  {
    href: "/admin/messages",
    label: "Messages",
    Icon: Inbox,
    badge: unreadCount,
  },
];
// SidebarFooter: SignOut button + "← View Portfolio" link
```

- The unread messages badge renders on the `Inbox` icon in both expanded and collapsed states.
- No `LocaleSwitcher` — admin is English-only.

**8f — Wrap admin layout** (`src/app/[locale]/(admin)/layout.tsx`)

```typescript
import { SidebarCookieProvider } from '@/shared/ui/SidebarCookieProvider';
import { AdminSidebar } from '@/features/admin/AdminSidebar';

export default async function AdminLayout({ children }) {
  return (
    <SidebarCookieProvider>  {/* independent state from portfolio sidebar */}
      <AdminSidebar />
      <main className="ps-[var(--sidebar-width)]">{children}</main>
    </SidebarCookieProvider>
  );
}
```

### Acceptance Criteria

- [ ] Portfolio sidebar collapses to 64 px on toggle — only icons visible.
- [ ] Hovering a nav icon in collapsed state shows a `Tooltip` with the link label.
- [ ] Expanded/collapsed state persists across full page navigations (cookie, no flash).
- [ ] `SidebarRail` drag-to-resize works on desktop.
- [ ] Mobile: sidebar renders as a full-screen sheet drawer; collapse mode is not active below `lg`.
- [ ] Admin sidebar collapses independently from portfolio sidebar.
- [ ] Unread messages badge is visible on the Inbox icon in both collapsed and expanded states.
- [ ] No `pointer-events` are blocked — clicking nav links works normally.
- [ ] Keyboard navigation: all sidebar items are reachable and operable via Tab + Enter.
- [ ] No layout shift (CLS = 0) when sidebar transitions between states.

### Agent Prompt

```
Act as a senior frontend engineer.

Using the shadcn/ui Sidebar component (already installed via `pnpm dlx shadcn@latest add sidebar tooltip`):

1. Create `src/shared/ui/SidebarCookieProvider.tsx`:
   - RSC (no "use client").
   - Read `sidebar_state` cookie via `cookies()` from next/headers.
   - Return <SidebarProvider defaultOpen={cookieValue !== 'collapsed'}>{children}</SidebarProvider>.

2. Rebuild `src/features/nav/NavBar.tsx` using shadcn/ui Sidebar primitives:
   - collapsible="icon": expanded = 240px (icon + label), collapsed = 64px (icon only).
   - SidebarRail for drag-to-resize.
   - Wrap each nav item in a Tooltip (shows label when sidebar is collapsed).
   - Use SidebarMenuButton with isActive={pathname === link.href}.
   - SidebarFooter: ThemeToggle, placeholder comment for LocaleSwitcher (Step 9), DownloadCVButton.
   - Replace ALL left/right margin/padding utilities with logical equivalents (ms-, me-, ps-, pe-, start-, end-).
   - Mobile fallback: SidebarTrigger in a mobile header bar triggers the Sheet drawer.

3. Wrap `src/app/[locale]/(portfolio)/layout.tsx` with <SidebarCookieProvider> and render <NavBar /> inside.

4. Create `src/features/admin/AdminSidebar.tsx`:
   - Same Sidebar primitives as NavBar.
   - Admin links: Dashboard, Skills, Experience, Projects, Messages (with unread badge).
   - SidebarFooter: Sign Out button + "← View Portfolio" link.
   - No LocaleSwitcher.

5. Wrap `src/app/[locale]/(admin)/layout.tsx` with a separate <SidebarCookieProvider> and render <AdminSidebar /> inside.

Constraints: TypeScript strict, no `any`, pnpm, RSC by default ("use client" only at leaf interactivity nodes), FSD imports.
Read the Stitch MCP design for exact spacing tokens before writing layout code.
```

---

## Step 9 — Internationalisation (EN + AR)

**Goal:** Add Arabic as a second locale alongside the English default. All public portfolio pages, navigation, and the AI chat widget adapt to the selected locale. The Admin CMS stays English-only but gains Gemini-powered auto-translation so the admin can publish Arabic content without writing it manually.

### Decisions

- **Library: `next-intl` v3** — the only i18n library with first-class Next.js App Router + RSC support. Handles locale routing, server-side message loading for RSCs, typed `useTranslations` / `getTranslations`, and `NextIntlClientProvider` without any `"use client"` overhead on pages. Replaces any hand-rolled i18n approach.
- **Routing: subpath (`/en/…` and `/ar/…`)** — root `/` redirects to the user's browser language (falling back to `/en`). URLs are locale-specific, SEO-indexable, and shareable.
- **RTL: CSS logical properties** — Arabic requires `dir="rtl"`. All layout utilities in the NavBar (already refactored in Step 8) use logical properties. The `<html>` element receives `dir="rtl"` for `/ar` routes via the `[locale]/layout.tsx`, which cascades through the entire page automatically.
- **Arabic font: `Cairo`** — loaded via `next/font/google` (never CDN), applied conditionally when `locale === 'ar'`. English pages continue to use `Inter`.
- **Dynamic content translation (Q2 = Yes)** — skills labels, experience roles/descriptions, and project titles/descriptions get nullable `*Ar` columns in Turso. The Admin CMS CRUD dialogs expose an English input field + an Arabic field with a **✨ Generate** button that calls a Gemini Server Action to fill the Arabic field automatically. The admin can review and edit before saving.
- **Locale toggle: Side Navbar** — `<LocaleSwitcher />` component placed in the `SidebarFooter` of the portfolio NavBar (below ThemeToggle). Switches between `/en` and `/ar` while preserving the current path. Not shown in the Admin sidebar.
- **AI assistant (Q3 = Yes)** — `buildPortfolioContext()` receives the active locale and injects a language instruction into the Gemini system prompt, making the chat widget respond in Arabic on `/ar/*`.
- **Admin CMS (Q4 = English-only)** — Admin UI strings remain English. Only the _data content_ (skills, projects, experience) is translated via the auto-generate flow.

### Tasks

**9a — Install next-intl**

```bash
pnpm add next-intl
```

**9b — i18n routing config** (`src/i18n/routing.ts`)

```typescript
import { defineRouting } from "next-intl/routing";

export const routing = defineRouting({
  locales: ["en", "ar"],
  defaultLocale: "en",
});
```

**9c — Server request config** (`src/i18n/request.ts`)

```typescript
import { getRequestConfig } from "next-intl/server";
import { routing } from "./routing";

export default getRequestConfig(async ({ requestLocale }) => {
  const locale = (await requestLocale) ?? routing.defaultLocale;
  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default,
  };
});
```

**9d — Typed navigation wrappers** (`src/i18n/navigation.ts`)

```typescript
import { createNavigation } from "next-intl/navigation";
import { routing } from "./routing";

export const { Link, redirect, useRouter, usePathname, getPathname } =
  createNavigation(routing);
// Use these everywhere instead of bare next/navigation — they include locale automatically.
```

**9e — Translation files**

`src/messages/en.json` (English — default):

```json
{
  "nav": {
    "home": "Home",
    "about": "About",
    "skills": "Skills",
    "projects": "Projects",
    "experience": "Experience",
    "education": "Education",
    "recommendations": "Recommendations",
    "contact": "Contact",
    "downloadCV": "Download CV"
  },
  "home": {
    "greeting": "Hi, I'm Mojahed",
    "tagline": "Frontend Engineer",
    "ctaPrimary": "View Projects",
    "ctaSecondary": "Contact Me"
  },
  "contact": {
    "title": "Get in Touch",
    "name": "Name",
    "email": "Email",
    "message": "Message",
    "submit": "Send Message",
    "successTitle": "Message sent!",
    "successBody": "I'll get back to you soon."
  },
  "skills": { "title": "Skills" },
  "projects": {
    "title": "Projects",
    "liveDemo": "Live Demo",
    "viewCode": "View Code"
  },
  "experience": { "title": "Experience", "present": "Present" },
  "education": { "title": "Education" },
  "recommendations": { "title": "Recommendations" },
  "about": { "title": "About Me" },
  "theme": { "toggle": "Toggle theme" },
  "locale": { "switchTo": "العربية" },
  "sidebar": { "collapse": "Collapse sidebar", "expand": "Expand sidebar" }
}
```

`src/messages/ar.json` (Arabic):

```json
{
  "nav": {
    "home": "الرئيسية",
    "about": "عني",
    "skills": "المهارات",
    "projects": "المشاريع",
    "experience": "الخبرة",
    "education": "التعليم",
    "recommendations": "التوصيات",
    "contact": "تواصل",
    "downloadCV": "تحميل السيرة الذاتية"
  },
  "home": {
    "greeting": "مرحباً، أنا مجاهد",
    "tagline": "مهندس واجهات أمامية",
    "ctaPrimary": "عرض المشاريع",
    "ctaSecondary": "تواصل معي"
  },
  "contact": {
    "title": "تواصل معي",
    "name": "الاسم",
    "email": "البريد الإلكتروني",
    "message": "الرسالة",
    "submit": "إرسال الرسالة",
    "successTitle": "تم الإرسال!",
    "successBody": "سأرد عليك قريباً."
  },
  "skills": { "title": "المهارات" },
  "projects": {
    "title": "المشاريع",
    "liveDemo": "معاينة مباشرة",
    "viewCode": "عرض الكود"
  },
  "experience": { "title": "الخبرة", "present": "الحاضر" },
  "education": { "title": "التعليم" },
  "recommendations": { "title": "التوصيات" },
  "about": { "title": "عني" },
  "theme": { "toggle": "تبديل المظهر" },
  "locale": { "switchTo": "English" },
  "sidebar": {
    "collapse": "طي الشريط الجانبي",
    "expand": "توسيع الشريط الجانبي"
  }
}
```

> ⚠️ **Translation key parity is enforced in CI.** A Vitest test asserts that `en.json` and `ar.json` have identical key sets. Any missing key in either file is a blocking test failure.

**9f — Locale layout** (`src/app/[locale]/layout.tsx`)

Reads locale from params, sets `lang` + `dir` on `<html>`, loads the Arabic or Latin font, and wraps with `NextIntlClientProvider`:

```typescript
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';
import { Inter, Cairo } from 'next/font/google';

const inter = Inter({ subsets: ['latin'], display: 'swap' });
const cairo = Cairo({ subsets: ['arabic'], display: 'swap' });

export default async function LocaleLayout({ children, params }) {
  const { locale } = await params;
  const messages = await getMessages();
  const isRTL = locale === 'ar';

  return (
    <html
      lang={locale}
      dir={isRTL ? 'rtl' : 'ltr'}
      className={isRTL ? cairo.className : inter.className}
      suppressHydrationWarning
    >
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

**9g — Update middleware** (`src/proxy.ts`)

Combine next-intl locale routing with the existing admin auth guard:

```typescript
import createMiddleware from "next-intl/middleware";
import { auth } from "@/features/auth/auth";
import { routing } from "@/i18n/routing";
import { NextResponse } from "next/server";

const intlMiddleware = createMiddleware(routing);

export const proxy = auth((req) => {
  const isAdminRoute = /\/(en|ar)\/admin/.test(req.nextUrl.pathname);
  if (isAdminRoute && !req.auth) {
    return NextResponse.redirect(new URL("/api/auth/signin", req.url));
  }
  return intlMiddleware(req);
});

export const config = {
  matcher: ["/((?!api|_next|.*\\..*).*)"],
};
```

**9h — `next.config.ts` — wire next-intl plugin**

```typescript
import createNextIntlPlugin from "next-intl/plugin";
const withNextIntl = createNextIntlPlugin("./src/i18n/request.ts");

import type { NextConfig } from "next";
const nextConfig: NextConfig = {
  experimental: {
    optimizePackageImports: ["lucide-react"],
  },
};

export default withNextIntl(nextConfig);
```

**9i — `LocaleSwitcher` component** (`src/shared/ui/LocaleSwitcher.tsx`)

Placed in the portfolio NavBar `SidebarFooter` (below ThemeToggle). Not present in Admin sidebar.

```typescript
'use client';
import { useLocale, useTranslations } from 'next-intl';
import { usePathname, Link } from '@/i18n/navigation';
import { Globe } from 'lucide-react';
import { SidebarMenuButton } from '@/shared/ui/sidebar';

export function LocaleSwitcher() {
  const locale = useLocale();
  const t = useTranslations('locale');
  const pathname = usePathname();
  const nextLocale = locale === 'en' ? 'ar' : 'en';

  return (
    <SidebarMenuButton asChild tooltip={t('switchTo')}>
      <Link href={pathname} locale={nextLocale}>
        <Globe />
        <span>{t('switchTo')}</span>
      </Link>
    </SidebarMenuButton>
  );
}
```

**9j — Update all public page RSCs**

In each of the 8 public pages, replace hardcoded strings with `getTranslations()`:

```typescript
// Example: src/app/[locale]/(portfolio)/skills/page.tsx
import { getTranslations } from 'next-intl/server';
import { getSkills } from '@/shared/lib/db/queries';

export default async function SkillsPage({ params }) {
  const { locale } = await params;
  const t = await getTranslations('skills');
  const skills = await getSkills(locale);          // locale-aware query

  return (
    <section>
      <h1>{t('title')}</h1>
      {/* ... */}
    </section>
  );
}
```

Use `Link` from `@/i18n/navigation` wherever `next/link` was used in public pages.

**9k — Database schema — AR locale columns** (`src/shared/lib/db/schema.ts`)

Add nullable Arabic columns to the three dynamic-content tables:

```typescript
export const skills = sqliteTable("skills", {
  // ... all existing columns unchanged
  labelAr: text("label_ar"), // nullable — Gemini-generated
});

export const experiences = sqliteTable("experiences", {
  // ... all existing columns unchanged
  roleAr: text("role_ar"),
  descriptionAr: text("description_ar"),
});

export const projects = sqliteTable("projects", {
  // ... all existing columns unchanged
  titleAr: text("title_ar"),
  descriptionAr: text("description_ar"),
});
```

```bash
pnpm drizzle-kit generate   # generates migration SQL
pnpm drizzle-kit push       # applies to Turso
```

**9l — Locale-aware queries** (`src/shared/lib/db/queries.ts`)

```typescript
export async function getSkills(locale: string) {
  "use cache";
  cacheTag(locale === "ar" ? CACHE_TAGS.skillsAr : CACHE_TAGS.skills);
  const rows = await db.select().from(skills).orderBy(asc(skills.sortOrder));
  return rows.map((s) => ({
    ...s,
    label: locale === "ar" && s.labelAr ? s.labelAr : s.label,
  }));
}
// Same locale-aware pattern for getExperiences() and getProjects()
```

**9m — Update cache tags** (`src/shared/lib/cache-tags.ts`)

```typescript
export const CACHE_TAGS = {
  skills: "skills",
  skillsAr: "skills-ar",
  experiences: "experiences",
  experiencesAr: "experiences-ar",
  projects: "projects",
  projectsAr: "projects-ar",
} as const;
```

Admin mutation Server Actions must call `revalidateTag` for **both** the EN and AR tag when content is updated:

```typescript
revalidateTag(CACHE_TAGS.skills);
revalidateTag(CACHE_TAGS.skillsAr);
```

**9n — Auto-translation Server Action** (`src/features/admin/actions/translate.ts`)

```typescript
"use server";
import { auth } from "@/features/auth/auth";
import { google } from "@ai-sdk/google";
import { generateText } from "ai";

export async function translateToArabic(text: string): Promise<string> {
  const session = await auth();
  if (session?.user?.email !== process.env.ALLOWED_ADMIN_EMAIL) {
    throw new Error("Unauthorized");
  }
  const { text: translated } = await generateText({
    model: google("gemini-3.5-flash"),
    prompt: `Translate the following to Modern Standard Arabic.
Return ONLY the translation — no explanation, no quotes, no markdown:

${text}`,
  });
  return translated.trim();
}
```

**9o — Admin CRUD dialogs — Arabic field pair pattern**

Each CRUD dialog (skills, experiences, projects) adds an Arabic field below each English field. The auto-translate trigger fires on blur of the English field (if the Arabic field is still empty) or when the admin clicks the **✨ Generate** button:

```typescript
// English field
<FormField name="label" render={({ field }) => (
  <FormItem>
    <FormLabel>Label (English)</FormLabel>
    <FormControl>
      <Input
        {...field}
        dir="ltr"
        onBlur={async () => {
          if (field.value && !form.getValues('labelAr')) {
            const ar = await translateToArabic(field.value);
            form.setValue('labelAr', ar);
          }
        }}
      />
    </FormControl>
  </FormItem>
)} />

// Arabic field — editable after generation
<FormField name="labelAr" render={({ field }) => (
  <FormItem>
    <div className="flex items-center gap-2">
      <FormLabel>Label (Arabic)</FormLabel>
      <Button type="button" size="sm" variant="ghost"
              onClick={async () => {
                const en = form.getValues('label');
                if (en) form.setValue('labelAr', await translateToArabic(en));
              }}>
        ✨ Generate
      </Button>
    </div>
    <FormControl>
      <Input {...field} dir="rtl" placeholder="سيتم التوليد تلقائياً…" />
    </FormControl>
  </FormItem>
)} />
```

**9p — AI Assistant — locale-aware system prompt**

`src/features/ai-assistant/buildContext.ts`:

```typescript
export async function buildPortfolioContext(locale: string): Promise<string> {
  const langInstruction =
    locale === "ar"
      ? "You MUST respond in Modern Standard Arabic (العربية الفصحى). All answers in Arabic only."
      : "Respond in English.";
  // ... existing Turso data fetches unchanged
  return `${langInstruction}\n\nYou are Mojahed's AI Portfolio Assistant...`;
}
```

`src/app/api/chat/route.ts` — receive locale from the client:

```typescript
export async function POST(req: Request) {
  const { messages, locale } = await req.json();
  const context = await buildPortfolioContext(locale ?? "en");
  // ... rest unchanged
}
```

`src/features/ai-assistant/ChatWidget.tsx` — send locale in the request body:

```typescript
const locale = useLocale();
const { messages, input, handleSubmit } = useChat({
  api: "/api/chat",
  body: { locale },
});
```

**9q — RTL CSS additions** (`src/app/globals.css`)

```css
/* Arabic typography smoothing */
[dir="rtl"] {
  font-feature-settings: "kern";
  -webkit-font-smoothing: antialiased;
}
```

> ⚠️ All Step 8 NavBar work already uses logical properties (`ms-`, `me-`, `ps-`, `pe-`, `start-`, `end-`). No further left/right overrides should be needed.

### Acceptance Criteria

- [ ] `/` redirects to `/en` (or `/ar` if the browser's `Accept-Language` is Arabic).
- [ ] `/ar/skills` renders in Arabic, `<html lang="ar" dir="rtl">` confirmed in DevTools.
- [ ] In RTL mode, the sidebar appears on the **right** side of the viewport.
- [ ] Locale switcher in the portfolio NavBar switches `/en ↔ /ar` while preserving the current path.
- [ ] Admin at `/en/admin` and `/ar/admin` both work; UI chrome remains English regardless of locale.
- [ ] Typing an English skill label and clicking **✨ Generate** fills the Arabic field within 3 seconds.
- [ ] Arabic field is editable after auto-generation; the edited value is saved (not overwritten on save).
- [ ] Auto-translate fires on blur of an English field only if the Arabic field is currently empty.
- [ ] AI chat on `/ar/*` responds in Arabic; AI chat on `/en/*` responds in English.
- [ ] Contact form on `/ar/contact` shows all labels in Arabic; success toast shows Arabic text.
- [ ] `en.json` and `ar.json` have identical key sets (CI Vitest parity test passes).
- [ ] Lighthouse Accessibility score = 100 on both `/en` and `/ar` (correct `lang` attribute).
- [ ] `axe-core` reports zero new violations introduced by i18n changes.
- [ ] `pnpm build` compiles both locales without errors.
- [ ] Canvas ribbon renders correctly in both RTL and LTR (it is `position: fixed` and unaffected by `dir`).

### Agent Prompt

```
Act as a senior full-stack engineer. Add English + Arabic internationalisation to the portfolio.

Dependencies:
  pnpm add next-intl
  (shadcn/ui sidebar already installed in Step 8)

1. Create i18n config files:
   - src/i18n/routing.ts — defineRouting({ locales: ['en', 'ar'], defaultLocale: 'en' })
   - src/i18n/request.ts — getRequestConfig loading messages from src/messages/{locale}.json
   - src/i18n/navigation.ts — createNavigation(routing) typed wrappers

2. Create translation files:
   - src/messages/en.json — English strings for all nav, page, contact, theme, locale, sidebar keys
   - src/messages/ar.json — Arabic equivalents (identical key structure)
   Write a Vitest test that asserts both files have exactly the same set of keys (deep key equality).

3. Create src/app/[locale]/layout.tsx:
   - Read locale from params, load messages via getMessages().
   - Set <html lang={locale} dir={locale === 'ar' ? 'rtl' : 'ltr'}>.
   - Apply Cairo font (next/font) when locale === 'ar', Inter otherwise.
   - Wrap body with <NextIntlClientProvider messages={messages}>.

4. Move existing route groups into src/app/[locale]/:
   - (portfolio)/ and (admin)/ become src/app/[locale]/(portfolio)/ and src/app/[locale]/(admin)/.
   - Update all relative imports.

5. Update src/proxy.ts (middleware):
   - Add next-intl createMiddleware(routing) for locale detection + redirect.
   - Keep the existing admin auth guard; update the admin route pattern to match /(en|ar)/admin.
   - Update matcher to ['/((?!api|_next|.*\\..*).*)'].

6. Update next.config.ts: wrap with createNextIntlPlugin('./src/i18n/request.ts').

7. Create src/shared/ui/LocaleSwitcher.tsx:
   - "use client". Uses useLocale(), usePathname() from i18n/navigation.
   - Renders a SidebarMenuButton wrapping a locale-switching Link.
   - Place in NavBar SidebarFooter (below ThemeToggle).

8. Update all 8 public page RSCs to use getTranslations() for headings.
   Update all Link usages to import from @/i18n/navigation (not next/link).

9. Database — add AR locale columns to skills, experiences, projects tables:
   - skills: labelAr TEXT
   - experiences: roleAr TEXT, descriptionAr TEXT
   - projects: titleAr TEXT, descriptionAr TEXT
   Run: pnpm drizzle-kit generate && pnpm drizzle-kit push

10. Update getSkills(), getExperiences(), getProjects() to accept locale param.
    Return AR column value when locale === 'ar' and AR value is non-null, else fall back to EN.

11. Update CACHE_TAGS to add -ar variants (skills-ar, experiences-ar, projects-ar).
    Admin mutation Server Actions must revalidateTag both EN and AR tags on write.

12. Create src/features/admin/actions/translate.ts:
    - "use server". Re-authenticate session before calling Gemini.
    - Call generateText(google('gemini-3.5-flash')) to translate English text to Modern Standard Arabic.
    - Return the trimmed Arabic string.

13. Update each admin CRUD dialog (skills, experiences, projects):
    - Add an Arabic field below each English field (dir="rtl").
    - On blur of English field: if Arabic field is empty, auto-call translateToArabic() and setValue.
    - Add a ✨ Generate button that re-calls translateToArabic() on demand.
    - Arabic fields remain manually editable; edited values are preserved on save.

14. Update buildPortfolioContext(locale) in src/features/ai-assistant/buildContext.ts:
    - locale === 'ar': prepend Arabic-only instruction to system prompt.
    - locale === 'en': prepend English instruction.

15. Update src/app/api/chat/route.ts to read { messages, locale } from req.json().
    Update ChatWidget.tsx to send body: { locale } via useLocale().

Constraints: TypeScript strict, no `any`, pnpm, RSC by default, FSD imports.
Read Stitch MCP design before building any UI layout. Use logical Tailwind properties only (ms-, me-, ps-, pe-, start-, end-) — never left/right variants.
```

---

## Appendix A — Environment Variables Reference

| Variable                       | Required | Used In                                                       |
| ------------------------------ | -------- | ------------------------------------------------------------- |
| `TURSO_DATABASE_URL`           | ✅       | Drizzle DB client                                             |
| `TURSO_AUTH_TOKEN`             | ✅       | Drizzle DB client                                             |
| `AUTH_SECRET`                  | ✅       | Auth.js v5 session signing                                    |
| `AUTH_GITHUB_ID`               | ✅       | Auth.js v5 GitHub OAuth                                       |
| `AUTH_GITHUB_SECRET`           | ✅       | Auth.js v5 GitHub OAuth                                       |
| `ALLOWED_ADMIN_EMAILS`         | ✅       | Admin route guard + Ably token auth                           |
| `BLOB_READ_WRITE_TOKEN`        | ✅       | Vercel Blob uploads                                           |
| `EDGE_CONFIG`                  | ✅       | Vercel Edge Config (site meta)                                |
| `ABLY_API_KEY`                 | ✅       | Ably REST publisher (server-side only — never `NEXT_PUBLIC_`) |
| `GOOGLE_GENERATIVE_AI_API_KEY` | ✅       | Gemini 3.5 Flash API Key for the AI assistant                 |
| `SENTRY_DSN`                   | ✅       | Sentry server-side                                            |
| `NEXT_PUBLIC_SENTRY_DSN`       | ✅       | Sentry client-side                                            |

---

## Appendix B — Cache Tag Reference

| Tag Constant               | Tag String         | Invalidated By                                             |
| -------------------------- | ------------------ | ---------------------------------------------------------- |
| `CACHE_TAGS.skills`        | `'skills'`         | `createSkill`, `updateSkill`, `deleteSkill`                |
| `CACHE_TAGS.skillsAr`      | `'skills-ar'`      | `createSkill`, `updateSkill`, `deleteSkill`                |
| `CACHE_TAGS.experiences`   | `'experiences'`    | `createExperience`, `updateExperience`, `deleteExperience` |
| `CACHE_TAGS.experiencesAr` | `'experiences-ar'` | `createExperience`, `updateExperience`, `deleteExperience` |
| `CACHE_TAGS.projects`      | `'projects'`       | `createProject`, `updateProject`, `deleteProject`          |
| `CACHE_TAGS.projectsAr`    | `'projects-ar'`    | `createProject`, `updateProject`, `deleteProject`          |

> ⚠️ Admin mutation Server Actions must call `revalidateTag` for **both** the EN and AR cache tag on every write (e.g. `revalidateTag(CACHE_TAGS.skills)` **and** `revalidateTag(CACHE_TAGS.skillsAr)`).

---

## Appendix C — Dependencies Quick Reference

```bash
# Production
pnpm add next react react-dom
pnpm add shadcn lucide-react next-themes
pnpm add drizzle-orm @libsql/client
pnpm add next-auth@beta
pnpm add zod react-hook-form @hookform/resolvers
pnpm add @vercel/blob @vercel/edge-config
pnpm add @sentry/nextjs
pnpm add ably                        # Real-time WebSocket pub/sub
pnpm add ai @ai-sdk/google           # AI Portfolio Assistant + auto-translate
pnpm add next-intl                   # i18n — EN + AR (Step 9)

# shadcn/ui components (installed via CLI, not pnpm add)
pnpm dlx shadcn@latest add sidebar tooltip   # Collapsible sidebar (Step 8)

# Development
pnpm add -D typescript @types/react @types/node
pnpm add -D drizzle-kit
pnpm add -D tw-animate-css tailwindcss-motion
pnpm add -D eslint eslint-config-next @typescript-eslint/eslint-plugin
pnpm add -D prettier eslint-config-prettier
pnpm add -D vitest @vitejs/plugin-react @testing-library/react
pnpm add -D @next/bundle-analyzer
```
