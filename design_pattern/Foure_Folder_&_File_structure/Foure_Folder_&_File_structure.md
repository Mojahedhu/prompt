Modern React and Next.js applications rely on three primary folder structure patterns, ranging from simple component grouping to strict enterprise architecture.

## 1. Feature-Driven Architecture (Recommended Standard)

Grouping code by business functionality (`auth`, `checkout`, `dashboard`) rather than file type (`components`, `hooks`, `services`) keeps related logic colocated and prevents top-level directories from turning into messy "global buckets".

### Standard Vite / Client-Side React Structure

```text
src/
├── assets/             # Static assets (images, fonts, SVGs)
├── components/         # Shared, domain-agnostic UI elements
│   └── ui/             # Design system primitives (Button, Input, Modal)
├── features/           # Self-contained business domains
│   ├── auth/           # Feature module
│   │   ├── api/        # Endpoint queries, mutations, React Query hooks
│   │   ├── components/ # Feature-specific UI (LoginForm.tsx, AuthModal.tsx)
│   │   ├── hooks/      # Feature-specific custom hooks
│   │   ├── types/      # Domain TypeScript definitions
│   │   └── index.ts    # Public API export (barrel file)
│   └── analytics/
├── hooks/              # Reusable app-wide custom React hooks
├── lib/                # Pre-configured third-party clients (Axios, Firebase)
├── routes/             # Router setup (React Router definitions)
├── stores/             # Global client state (Zustand, Jotai)
├── utils/              # Pure utility functions (formatting, validation)
├── App.tsx             # Root application component
└── main.tsx            # Entry point

```

**Key Rule:** Feature modules export their public API via `index.ts`. Outside code imports from `@/features/auth`, never directly reaching into internal implementation files like `@/features/auth/components/LoginForm`.

---

## 2. Next.js App Router Structure

Next.js App Router uses file-system routing inside the `app/` directory. The modern pattern is to keep `app/` strictly for routing and layouts while housing domain logic inside a parallel `src/features/` folder or colocated private directories.

```text
src/
├── app/                        # Next.js Routing Layer ONLY
│   ├── (auth)/                 # Route Group (organizes routes without URL impact)
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   │   ├── page.tsx        # Thin page wrapper
│   │   │   └── _components/    # Colocated private components (excluded from routing)
│   │   └── layout.tsx
│   ├── api/                    # Route Handlers
│   ├── global-error.tsx
│   ├── layout.tsx              # Root Layout
│   └── page.tsx
├── components/                 # Shared UI primitives (Shadcn UI, Radix)
├── features/                   # Core application business logic
│   └── billing/
│       ├── actions/            # Server Actions
│       ├── components/         # Feature UI components
│       ├── services/           # DB queries and domain business logic
│       └── types.ts
├── lib/                        # Database clients (Prisma, Drizzle) & singletons
└── styles/                     # Tailwind CSS and global style imports

```

### Next.js Best Practices:

- **Keep Pages Thin:** Route files (`page.tsx`) should act as simple wrappers that call feature components or trigger initial data fetches.
- **Private Folders (`_folder`):** Folders prefixed with an underscore (e.g., `_components/`) are ignored by the router, allowing route-specific files to live right next to `page.tsx`.
- **Thin Server Actions:** Move business logic and database access into dedicated service functions inside `features/` or `lib/` so they remain easy to unit test independently.

---

## 3. Feature-Sliced Design (FSD) for Enterprise Scale

For large projects managed by multiple teams, **Feature-Sliced Design (FSD)** introduces a strict 6-layer architecture with explicit dependency rules.

```text
src/
├── app/        # Provider setup, global styles, router initialization
├── pages/      # Route composition pages
├── widgets/    # Complex standalone UI blocks (Header, Sidebar, Feed)
├── features/   # User interactions (SendMoneyForm, AddToCart, LikeButton)
├── entities/   # Domain entities (User, Product, Cart, Order)
└── shared/     # Domain-agnostic assets, UI kits, API wrappers, helpers

```

**Unidirectional Rule:** A module can only import code from layers positioned strictly _below_ it. For example, `features/` can import from `entities/` and `shared/`, but `entities/` can never import from `features/`.

---

## Pattern Comparison

| Strategy                        | Ideal For                               | Strengths                                                        | Trade-offs                                                  |
| ------------------------------- | --------------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------- |
| **Layered / Technical**         | Small apps, MVPs, quick prototypes      | Simple setup, shallow directory nesting                          | Difficult to maintain beyond ~50 components                 |
| **Feature-Driven**              | Medium to large React & Next.js apps    | High cohesion, isolated domain refactoring                       | Requires strict discipline with barrel exports (`index.ts`) |
| **Feature-Sliced Design (FSD)** | Large enterprise apps, multi-team repos | Eliminates circular dependencies and cross-module tight coupling | Steeper learning curve and higher upfront overhead          |

---

## Essential Setup Conventions

- **Path Aliases:** Configure path aliases in `tsconfig.json` (e.g., `@/components/*`, `@/features/*`) to avoid long relative imports like `../../../../components/Button`.
- **Colocate Assets:** Store unit tests (`Button.test.tsx`), styles (`Button.module.css`), and sub-components directly alongside the file they belong to.
- **Keep `shared/` Lean:** Avoid treating `utils/` or `components/` as a dumping ground. If a component is only used within one feature, keep it inside that feature.
