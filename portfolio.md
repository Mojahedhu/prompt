**Your Optimized Build Prompt:**

> "Act as an expert frontend engineer. I am building my personal developer portfolio using Next.js (App Router), TypeScript, Tailwind CSS, shadcn/ui, and the pnpm package manager.
> The app must include these pages/sections: Home, About, Skills, Projects, Experience, Education, Recommendations, and Contact. Implement form validation for the Contact section using Zod and React Hook Form. Integrate Sentry for error tracking and performance monitoring.
> For animations, strictly prioritize Tailwind CSS utility classes and `animate-tailwind` for entrance and interaction states before reaching for heavier JS libraries.
> Finally, build a custom HTML5 `<canvas>` background component for the layout. It must create a fluid ribbon effect tracking the cursor and behave like smoke disappear gradually, utilizing a requestAnimationFrame loop with dynamic HSL color shifting (e.g., pink to blue) for specific interval (e.g., per each focus on window).
> Please output the scaffolding steps, the Sentry configuration, and the code for the custom Canvas ribbon component first."
> Finally I will deploy the app on Vercel, so I will use the following for the database:

- Vercel Global Config (for project info)
- Blob of images and my downloadable file, like a CV
- Turso (SQLite) for client messages
  so please generate a step for designing the DB for the project

> there will be a ready design for the app from google stitch and I'll hand it to you via mcp

---

## Step 1: Architecture & Foundation

**Tasks**

- Initialize the Next.js App Router project using pnpm and TypeScript.
- Configure Tailwind CSS, `animate-tailwind`, and integrate shadcn/ui for accessible base components.
- Initialize and configure Sentry via the `@sentry/nextjs` wizard to capture client and edge runtime errors.
- Set up a dynamic data layer to manage all content and assets so you aren't hardcoding data into your components:
  - Vercel Edge Config: Fetch text content and project info (Experience, Skills, Projects).

  - Vercel Blob: Store and serve images and downloadable files (like your CV).

  - Turso (SQLite): Handle and store incoming client messages from your contact form.

**Questions to Consider**

- Are you deploying to Vercel? (This affects how you configure Sentry environment variables and source maps).
- This app will be distinct route pages (e.g. `/about`, `/projects`)?

**Tech Stack**

- Next.js (App Router), TypeScript, pnpm, Tailwind CSS, `animate-tailwind`, shadcn/ui, `@sentry/nextjs`, react-form-hook, zod.

**AI Agent Prompt**

> "Provide the exact CLI commands using pnpm to initialize a Next.js App Router project with TypeScript, Tailwind, and shadcn/ui. Then, show me the `sentry.client.config.ts` and `next.config.mjs` setup required to correctly monitor errors without bloating the client bundle. Finally, write a `tailwind.config.ts` that includes the `animate-tailwind` plugin."

## Step 2: The Fluid Ribbon Canvas Engine

**Tasks**

- Create a client-side only `<CanvasBackground/>` component using `"use client"`.
- Implement a `requestAnimationFrame` loop that calculates physics for trailing points based on mouse coordinates.
- Write an HSL color incrementer to create the dynamic color-to-color transition for every window focus.
- Handle window resize events to rebuild the canvas dimensions without clearing the animation abruptly.

**Questions to Consider**

- How long should the ribbon trail be, and what is the friction/decay rate of the particles? the ribbon behave like smoke , disappear gradually until hidden completely.
- To save CPU performance, should the animation pause when the tab loses focus or when the user stops moving the mouse for a certain duration? when the the tab lose focus

**Tech Stack**

- React (`useRef`, `useEffect`), HTML5 `<canvas>`, native JavaScript Math/Physics.

**AI Agent Prompt**

> "Write a React component named `<RibbonCanvas/>` using HTML5 Canvas. It needs a `requestAnimationFrame` loop that draws a fluid ribbon following the user's mouse cursor. The ribbon should consist of multiple connected lines with a dynamic HSL color loop (transitioning through pinks and blues). Include logic to handle window resizing and ensure it unmounts cleanly to prevent memory leaks."

## Step 3: Core Layout & Routing

**Tasks**

- from the ready design.
- Construct the root `layout.tsx` incorporating the `<RibbonCanvas/>` as a fixed background layer with `z-index: ..` must be above all the other content.
- Build a side navigation bar utilizing shadcn/ui that supports active state highlighting (slide out in close like a sheet or dialog ).
- Implement a mobile-responsive navigation drawer (using a shadcn/ui Sheet or Dialog).

**Questions to Consider**

- Does the canvas ribbon interfere with the readability of your text? (You may need a slight semi-transparent overlay or strong text shadows). the canvas must has a transparent bg, and the ribbon is come over the text
- Will you support a system-preference dark mode, and how does the ribbon color interact with light/dark backgrounds? the app will has light / dark mode, and the user switch between them

**Tech Stack**

- Next.js (`layout.tsx`), shadcn/ui (Navigation Menu, Sheet), `lucide-react` for icons.

**AI Agent Prompt**

. from the ready design.

> "Create a root `layout.tsx` for my Next.js app. The layout should include a side `<NavBar/>` component and place a `<RibbonCanvas/>` component in the background so it persists across page navigations. Use Tailwind to ensure the canvas sits above the contents (`z-index`) and with a readable contrast."

## Step 4: Content Pages & Tailwind Animations

**Tasks**

- from the ready design.
- Develop the 8 distinct sections (Home, About, Skills, Project, Experience, Education, Recommendation, Contact).
- Apply `animate-tailwind` classes (e.g., `animate-zoom-in`, `animate-fade-in-up`, `animate-delay-100`,...) to stagger the rendering of lists like Skills and Projects.
- Build the Contact form with strict input sanitization and schema validation to prevent spam, for the contact form make integration between RHF and React 19+ features ( e.g. useActionState, ...).

**Questions to Consider**

- Should the entrance animations trigger on initial load, or when the user scrolls them into view (using an `IntersectionObserver`)? on the initial load
- How will you handle state and submission loading for the Contact form?
  by integrate RFH with React 19+ (useActionState, useTransition...)
  **Tech Stack**

- React, Tailwind CSS, `animate-tailwind`, Zod, React Hook Form.

**AI Agent Prompt**

> "Generate the `<Skills/>` and `<Projects/>` components. Map over a data array for each. Use `animate-tailwind` classes to stagger the entrance of each skill pill and project card. Then, generate a `<ContactForm/>` component using React Hook Form and Zod for validation, including a loading state for the submit button."

## Step 5: Optimization & Deployment Preparation

**Tasks**

- Verify all interactive elements (buttons, links, inputs) have `cursor-pointer` so the custom canvas tracking feels intentional alongside standard web behavior.
- Test Sentry by forcing a deliberate error to ensure source maps are correctly demangling your TypeScript code in the dashboard.
- Run Lighthouse to ensure the `requestAnimationFrame` canvas loop isn't causing severe main-thread blocking.

**Questions to Consider**

- Is the bundle size optimized? (Check if any large libraries sneaked in during the build).

**Tech Stack**

- Vercel, Sentry Dashboard, Chrome DevTools.

**AI Agent Prompt**

> "Provide a checklist and code snippets for optimizing a Next.js App Router application before deploying to Vercel. Include a method to intentionally trigger a test error to verify my Sentry integration, and suggest ways to ensure the HTML5 canvas component doesn't ruin my Lighthouse performance score."
