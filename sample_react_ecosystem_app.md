Ah, I understand! You're asking how to give me the right instructions so that I automatically apply all of those best practices (Next.js, Zustand, Tailwind, Shadcn, Framer Motion) when generating code for you.

You don't need to do anything complicated. Because my instructions (my "skills") are built into my system, you just need to be explicit in your initial prompt about the stack you want to use.

Here is the best way to notify me to use all of them. You can use a prompt template like this:

---

**"I want to build a [describe your feature/app]. Please use my preferred modern React stack:**

1. **Framework:** Next.js App Router (Server Components where possible).
2. **Styling:** Tailwind CSS and Shadcn UI components.
3. **State Management:** Zustand for global client state.
4. **Data Fetching:** TanStack Query for server state.
5. **Animations:** Framer Motion for interactive UI (shared layouts, presence, gestures).

**Please act as a senior frontend engineer and ensure you are applying the best practices for this specific stack."**

---

### Why this works:

1.  **It triggers my specific "skills"**: When you explicitly mention "Next.js App Router", "Zustand", or "Framer Motion", it signals my internal systems to pull from those specific knowledge bases rather than generic React knowledge.
2.  **It sets constraints**: If you don't mention Shadcn or Tailwind, I might try to write custom CSS or inline styles. Telling me your exact stack forces me to write code that fits your architecture.
3.  **It defines the architecture**: By separating "Client State (Zustand)" from "Server State (TanStack Query)", I know exactly how you want your data flow structured before I write a single line of code.

If you ever want to start a brand new project with me using this exact setup, you can just copy-paste that prompt above!
