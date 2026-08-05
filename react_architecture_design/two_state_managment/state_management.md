State management is often the most debated topic in React architecture. For years, the default strategy was "put absolutely everything into Redux."

Modern React architecture completely rejects that monolithic approach. Today, the golden rule is **State Categorization**: identifying what _kind_ of state you are dealing with and using a specialized tool for it.

Here is the modern breakdown of the four types of state and how to manage them.

---

### 1. Server State (The Game Changer)

This is the biggest architectural shift in modern React. Server state refers to any data that actually lives on your database but is temporarily displayed in your UI (e.g., a list of users, a user's profile, product inventory).

- **The characteristics:** It is asynchronous, can be changed by other users at any time, and requires handling loading and error states.
- **The tools:** TanStack Query (React Query), SWR, RTK Query, or Apollo Client (for GraphQL).
- **The architecture rule:** **Never put server data into a global client store.**

Instead of writing actions and reducers to fetch data, tools like React Query handle the caching, background refetching, deduplication, and garbage collection automatically.

### 2. URL State (The Forgotten State)

Before you reach for `useState` or a global store, ask yourself: _"If the user refreshes the page, or sends this URL to a friend, should this state persist?"_ If the answer is yes, that state belongs in the URL.

- **The characteristics:** Shareable, bookmarkable, and globally accessible to any component on the page via the router.
- **Examples:** Search queries (`?q=react`), active filters (`?category=shoes`), pagination (`?page=3`), and even open modals (`?modal=login`).
- **The tools:** `useSearchParams` from React Router or Next.js.
- **The architecture rule:** The URL should be the single source of truth for navigation and filtering.

### 3. Local / UI State

This is data that only one component (and perhaps its immediate children) cares about.

- **The characteristics:** Ephemeral (it disappears when the component unmounts), synchronous, and isolated.
- **Examples:** A boolean tracking if an accordion is expanded, the current text in a controlled input field, or hover states.
- **The tools:** `useState` or `useReducer`.
- **The architecture rule:** **Colocation**. State should live as close as possible to where it is used. Don't lift state up to a parent component unless a sibling component absolutely needs to know about it.

### 4. Global / App State (The Leftovers)

If you successfully move your server data to React Query, your shareable data to the URL, and your UI data to local state, you will be shocked by how little "Global State" is actually left.

Global state is strictly for synchronous, client-side data that needs to be accessed by many different, disconnected components across your application.

- **Examples:** The user's UI theme (light/dark mode), a global toast notification system, or a complex multi-step wizard form.
- **The tools:**
- **Context API:** Best for data that changes rarely (like theme or language preference). It is _not_ a state manager; it's a dependency injection tool, and frequent updates will cause performance-killing re-renders.
- **Zustand or Jotai:** The modern standards for React global state. They are incredibly lightweight, require zero boilerplate, and bypass Context rendering issues.
- **Redux Toolkit (RTK):** Still useful for massive enterprise applications with highly complex, event-driven state tracking, but overkill for most modern apps.

---

> **Key takeaway:** A mature React architecture doesn't use one tool for all state. It uses React Query for the backend data, the URL for filters, `useState` for local UI, and a tiny Zustand store for the 5% of data that genuinely needs to be global.
