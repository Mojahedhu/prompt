# React and Next.js Folder Structure Architectural Patterns

In the React and Next.js ecosystems, folder structure isn't strictly enforced by the framework outside of routing in Next.js. Choosing the right architectural pattern depends on your project's scale and team size.

---

## 1. Core React Architectural Patterns

### Type-Based Grouping (Standard Starter)

Files are grouped by their technical role. This is the default approach for most tutorials and new Vite applications.

- **Best for:** Small to medium projects, MVPs, and static sites.
- **Drawback:** As the app grows, developers must jump between multiple folders to build or update a single feature.

```plaintext
src/
├── components/   # Reusable UI (Button, Modal, Navbar)
├── hooks/        # Custom React hooks (useAuth, useWindowSize)
├── utils/        # Helper functions (formatDate, calculateTax)
├── services/     # API calls and external integrations
├── types/        # Global TypeScript interfaces
└── pages/        # Route entry points (Vite)

```

### Feature-Based Grouping (Domain-Driven)

Files are grouped by domain or feature (e.g., "Authentication" or "Shopping Cart") rather than technical type.

- **Best for:** Medium to large applications with independent feature sets.
- **Benefit:** Highly scalable. Features can be refactored or removed with minimal risk to the rest of the application.

```plaintext
src/
├── core/                  # App-wide setup (store provider, theme)
├── shared/                # Universal UI components, utils, and hooks
└── features/
    ├── auth/
    │   ├── api/           # Login/logout requests
    │   ├── components/    # LoginForm, UserProfile
    │   ├── hooks/         # useSession
    │   └── types/         # AuthUser type
    └── checkout/
        ├── api/
        ├── components/    # CartList, PaymentForm
        └── store/         # Cart state slice

```

### Feature-Sliced Design (FSD)

A highly formalized methodology for frontend projects enforcing strict one-way dependency rules: lower layers cannot import from higher layers.

- **Best for:** Enterprise-scale applications built by large teams.
- **The Golden Rule:** A layer can only import from layers below it. Features can import Entities and Shared, but Entities cannot import Features.

#### Layer Hierarchy (Top to Bottom):

1. **`app/`**: Global routing, store setup, styles.
2. **`pages/`**: Composition of widgets to build full screens.
3. **`widgets/`**: Complex blocks composed of features and entities (e.g., `Header`, `UserProfileCard`).
4. **`features/`**: User interactions that bring business value (e.g., `SendComment`, `AddToCart`).
5. **`entities/`**: Business entities (e.g., `User`, `Product`, `Order`).
6. **`shared/`**: Reusable UI, infrastructure, and tools (e.g., `UIKit`, `apiClient`).

---

## 2. Next.js App Router Specific Routing Conventions

The Next.js 13+ App Router dictates routing structure through the `app/` directory using specific folder conventions:

- **Colocation:** Because only files named `page.tsx` or `route.ts` are publicly routable, components (`Button.tsx`) or helpers (`api-helpers.ts`) can safely live directly inside route folders.
- **Route Groups `(folderName)`:** Used to organize routes logically without affecting the URL path (e.g., `app/(marketing)/about/page.tsx` resolves to `[yoursite.com/about](https://yoursite.com/about)`).
- **Private Folders `_folderName`:** Opts a folder out of routing entirely. Used to keep `_components` or `_lib` close to page routes without exposing them to the web.
- **Parallel Routes `@folderName`:** Renders multiple pages simultaneously within the same layout (e.g., `@modal`, `@analytics`).

---

## 3. Global Next.js Organizational Patterns

Because Next.js dictates that routing lives in the `app/` (or `pages/`) directory, the primary design decision is placing the remaining application logic.

### 1. The "App is Just a Router" Pattern (External `src/`)

Treats the `app/` directory purely as a routing layer containing minimal logic. Route files act as thin wrappers importing features and layouts from a separate directory inside `src/`.

- **Best for:** Large codebases, teams migrating from standard React, or applications sharing code with React Native.
- **Philosophy:** Next.js serves as the delivery mechanism while the core application lives independently.

```plaintext
src/
├── app/                  # STRICTLY routing (page.tsx, layout.tsx, loading.tsx)
│   ├── dashboard/page.tsx
│   └── page.tsx
├── components/           # Global/shared UI (Button, Modal, Nav)
├── features/             # Domain logic (auth, billing, user-profile)
│   └── dashboard/
│       ├── DashboardView.tsx
│       ├── dashboard-utils.ts
│       └── useDashboard.ts
└── lib/                  # Database clients, API configurations

```

### 2. Strict Colocation Pattern (Feature-in-Route)

Groups a route's components, styles, and logic within the exact folder where the route is defined.

- **Best for:** Highly modular apps with distinct route layouts, or solo developers minimizing cross-folder navigation.
- **Philosophy:** Everything required to understand and run a page resides in one place.

```plaintext
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx          # Homepage route
│   └── dashboard/
│       ├── page.tsx      # Dashboard route
│       ├── components/   # UI ONLY used on the dashboard
│       │   ├── StatCard.tsx
│       │   └── DataGrid.tsx
│       ├── utils.ts      # Logic ONLY used on the dashboard
│       └── styles.css
├── components/           # Truly global components
└── lib/                  # Truly global utilities

```

### 3. The Vercel Standard (`ui/`, `lib/`, `app/`)

Flattening the distinction between features and components into a simple three-pillar approach, commonly seen in official Next.js documentation and Vercel templates.

- **Best for:** Small to medium apps, fast prototyping, and pragmatic setups.
- **Philosophy:** Fast, flat, and simplified structure.

```plaintext
src/
├── app/                  # Routing and page assembly
├── ui/                   # ALL visual elements (basic and complex)
│   ├── global.css
│   ├── buttons/
│   ├── dashboard/        # Dashboard-specific visuals
│   └── marketing/
└── lib/                  # ALL non-visual logic
    ├── definitions.ts    # Types and interfaces
    ├── data.ts           # Data fetching functions
    ├── db.ts             # Database connection
    └── utils.ts          # Formatting, math, etc.

```

---

## 4. Placing Server Actions and Data Fetching Logic

### Core Execution Rules

- **Data Fetching (`data.ts` / `api.ts`):** Imported into **Server Components** (`page.tsx` or `layout.tsx`) so data is resolved on the server before HTML generation.
- **Server Actions (`actions.ts`):** Extracted into dedicated files marked with the `"use server"` directive. Imported into **Client Components** (e.g., forms or buttons) to handle server-side mutations.

### Placement Across Architectural Patterns

#### 1. "App is Just a Router" (Feature-Based)

Logic lives within domain folders inside `src/features/`.

```plaintext
src/
├── app/
│   └── invoices/page.tsx         # Awaits getInvoices() and passes to UI
└── features/
    └── invoices/
        ├── actions.ts            # "use server" - createInvoice, deleteInvoice
        ├── api.ts                # Data fetching - getInvoices, getInvoiceById
        ├── InvoicesList.tsx      # UI consuming fetched data
        └── CreateInvoiceForm.tsx # UI importing createInvoice from actions.ts

```

#### 2. Strict Colocation (Feature-in-Route)

Actions and data fetchers reside directly alongside `page.tsx`. High volumes of actions can be isolated inside private `_actions/` subdirectories.

```plaintext
src/
└── app/
    └── dashboard/
        └── invoices/
            ├── page.tsx          # Awaits data and renders page
            ├── actions.ts        # Server actions for invoices
            ├── data.ts           # Data fetching for invoices
            ├── InvoicesList.tsx
            └── _actions/         # (Optional) Subfolder for scaled actions
                ├── create.ts
                └── update.ts

```

#### 3. The Vercel Standard

Groups all logic by technical purpose inside `src/lib/`.

```plaintext
src/
├── app/
│   └── dashboard/page.tsx
├── ui/
│   └── dashboard/
│       └── CreateForm.tsx        # Imports actions from lib/actions/invoices.ts
└── lib/
    ├── data/                     # Data fetching modules
    │   ├── users.ts
    │   └── invoices.ts
    ├── actions/                  # Server action modules
    │   ├── users.ts
    │   └── invoices.ts
    └── db.ts                     # Database client setup

```

---

## 5. Zod Schemas & TypeScript Types Colocation

### The Golden Rule

Always colocate Zod schemas and their inferred TypeScript types in the same file to keep them synchronized.

```typescript
// schemas.ts (or definitions.ts / types.ts)
import { z } from "zod";

export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["ADMIN", "USER"]),
});

// Export inferred type directly alongside the schema
export type User = z.infer<typeof UserSchema>;
```

### Placement Strategies

#### 1. Feature-Based Structure

Types and schemas are bound to their respective domain directories. Truly global types live in `src/types/`.

```plaintext
src/
├── types/                        # Global system types (e.g., BaseApiResponse)
├── features/
│   └── invoices/
│       ├── schemas.ts            # InvoiceSchema, CreateInvoiceSchema
│       ├── actions.ts            # Validates input using schemas.ts
│       └── InvoicesList.tsx      # Consumes types from schemas.ts

```

#### 2. Strict Colocation

Schemas live alongside the route owning the logic, with shared entities defined in `src/lib/`.

```plaintext
src/
├── lib/
│   └── shared-schemas.ts         # UserSchema, cross-route definitions
└── app/
    └── dashboard/
        └── settings/
            ├── page.tsx
            ├── actions.ts
            └── schemas.ts        # Route-specific SettingsFormSchema

```

#### 3. The Vercel Standard

Centralizes schemas in `src/lib/definitions.ts` or a dedicated `src/lib/schemas/` folder.

```plaintext
src/
├── ui/
│   └── invoices/
│       └── CreateForm.tsx        # Imports CreateInvoiceSchema from lib
└── lib/
    ├── definitions.ts            # Single file for small projects
    └── schemas/                  # Expanded directory for large projects
        ├── user.ts
        ├── invoice.ts
        └── auth.ts

```

### Database Schemas (ORMs)

- **Prisma:** Automatically generates types in `node_modules`. Libraries like `zod-prisma-types` can infer Zod schemas directly from `schema.prisma`.
- **Drizzle:** Define tables directly in TypeScript (e.g., `src/db/` or `src/lib/db/`). Use `drizzle-zod` to infer Zod schemas directly from table definitions in that same file.
