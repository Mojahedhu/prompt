The final pillar of a robust React architecture is how you organize your files. If you get this wrong, your project will eventually collapse under its own weight, making it impossible for new developers to find where things live.

Historically, developers used a "Type-based" architecture (grouping all hooks in one folder, all components in another). The modern standard for scalable apps is **Feature-based architecture**.

Here is how to design it.

---

### The Problem with Type-Based Architecture

In a standard type-based setup, your code is organized by what the file _is_, not what it _does_:

```text
src/
  components/
    Button.tsx
    LoginForm.tsx
    ProductCard.tsx
  hooks/
    useAuth.ts
    useCart.ts
  api/
    auth.ts
    products.ts

```

The issue? If you need to fix a bug in the Login flow, you have to open `components/LoginForm.tsx`, `hooks/useAuth.ts`, and `api/auth.ts`. Your brain has to jump across three completely different directories just to understand one feature.

### The Solution: Feature-Based Architecture

Feature-based architecture treats your application as a collection of mini-applications (domains). If code is only used by the authentication system, it lives in the `auth` folder.

Here is the blueprint for a highly scalable `src` directory:

```text
src/
  app/              # Global setup (providers, router config, global styles)
  assets/           # Global static files (fonts, logos)
  components/       # ONLY generic, globally shared UI (Button, Modal, Input)
  hooks/            # ONLY generic, globally shared hooks (useClickOutside, useTheme)
  utils/            # ONLY generic helpers (formatDate, currencyMath)
  features/         # ⬅️ The heart of your application
    auth/
    cart/
    products/
    users/

```

### Deep Dive: Inside a Feature Folder

Every feature folder should act like an isolated package. It contains its own domain-specific components, hooks, API calls, and types.

Let's look inside `src/features/auth/`:

```text
src/features/auth/
  api/              # Fetch calls (login, logout, getProfile)
    login.ts
  components/       # UI specifically for auth (LoginForm, PasswordReset)
    LoginForm.tsx
  hooks/            # Logic specifically for auth (useLogin, useSession)
    useLogin.ts
  types/            # TypeScript interfaces (User, AuthResponse)
    index.ts
  index.ts          # ⬅️ The Public API of the feature

```

### The Golden Rule: The Barrel File (`index.ts`)

The most critical part of this architecture is the `index.ts` file at the root of every feature. This acts as the **public API** for that feature.

If `src/features/cart` needs to know if a user is logged in, it is **strictly forbidden** from importing a file deep inside the auth folder like this:
`import { useSession } from '@/features/auth/hooks/useSession'`

Instead, the `auth/index.ts` file exports only what the rest of the app is allowed to see:

```typescript
// src/features/auth/index.ts
export * from "./components/LoginForm";
export * from "./hooks/useSession";
export type { User } from "./types";
```

And other features import it cleanly:
`import { useSession } from '@/features/auth'`

> **Why this matters:** This creates strict boundaries. If you decide to completely rewrite how `useLogin.ts` works internally, as long as you don't change the `index.ts` export, you guarantee you won't break the `cart` feature or the `products` feature.

### When does a Component go in the Global Folder?

A common point of confusion is knowing when a component belongs in `src/components` versus `src/features/domain/components`.

**The Rule of Thumb:**

- If the component knows about your business logic (e.g., `ProductCard` needs to format a price and link to a specific product ID), it belongs in the `features` folder.
- If the component is purely visual and could be published to NPM as an open-source library (e.g., a generic `Button`, `Dropdown`, or `TextInput`), it belongs in the global `src/components` folder.
