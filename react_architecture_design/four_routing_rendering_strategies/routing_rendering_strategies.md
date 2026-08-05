Routing and rendering dictate exactly _where_ and _when_ your React code turns into HTML. Historically, React only did this in the user's browser. Today, thanks to frameworks like Next.js and Remix, you have four distinct strategies.

Choosing the right one is the most consequential architectural decision you will make.

---

### 1. Client-Side Rendering (CSR)

This is the "traditional" React Single Page Application (SPA). The server sends a nearly empty HTML file containing just a `<div id="root"></div>` and a massive JavaScript bundle.

- **How it works:** The browser downloads the JavaScript, parses it, and React builds the entire UI on the fly in the browser.
- **The Pros:** Once the app is loaded, navigating between pages is instant. It feels like a desktop application. It's incredibly cheap to host (you just serve static JS files).
- **The Cons:** Terrible for SEO (search engines just see a blank `div` initially). It suffers from the "waterfall effect"—the user sees a blank screen while the JS downloads, then a spinner while the JS fetches data, and _then_ the UI.
- **Best for:** Highly interactive dashboards behind a login screen (like Jira or Spotify) where SEO doesn't matter.

### 2. Server-Side Rendering (SSR)

Instead of forcing the user's phone to do all the work, the server builds the HTML on demand _every time a user requests a page_.

- **How it works:** A user requests `/profile`. The server fetches the data from the database, renders the React components into HTML string, and sends a fully formed webpage to the browser. React then boots up in the background and attaches event listeners (onClick, onChange) in a process called **Hydration**.
- **The Pros:** Excellent SEO. Users see content immediately without waiting for JavaScript to download.
- **The Cons:** Slower Time to First Byte (TTFB). The server has to do work on every single request. If your database is slow, the user is stuck looking at a white loading screen in their browser tab.
- **Best for:** Dynamic, personalized sites where data changes constantly and SEO matters (like Twitter or an e-commerce shopping cart).

### 3. Static Site Generation (SSG)

SSG takes SSR a step further. Instead of building the HTML on every request, it builds the HTML _once_ during the build process (when you run `npm run build`).

- **How it works:** The server fetches all necessary data at build time, generates static HTML files for every page, and uploads them to a global CDN.
- **The Pros:** Blazing fast. Because the HTML is already built and sitting on a server near the user, page loads are nearly instantaneous. Perfect SEO. Incredibly cheap and secure.
- **The Cons:** If the data changes (e.g., you fix a typo on a blog post), you have to rebuild and redeploy the entire site. It cannot handle highly personalized data (like a logged-in user's profile).
- **Best for:** Blogs, documentation sites, marketing pages, and portfolios.

### 4. React Server Components (RSC)

RSC is the bleeding-edge paradigm shift introduced in React 18 and popularized by the Next.js App Router. It fundamentally changes how we think about React.

Historically, whether you used CSR, SSR, or SSG, _all_ of your React component code was eventually downloaded by the user's browser so it could be hydrated.

- **How it works:** Server Components run **exclusively on the server**. Their code is _never_ sent to the browser. They can securely access databases and file systems directly. They output a special JSON-like format that the browser uses to paint the UI.
- **The Paradigm Shift:** By default, all components are Server Components. You only opt-in to Client Components (using the `"use client"` directive) when you specifically need browser interactivity (like `useState`, `onClick`, or `useEffect`).
- **The Pros:** Massive reduction in JavaScript bundle sizes. You can write `async/await` directly inside your React components to fetch data. You no longer expose sensitive API keys or complex logic to the client.

> **Key Takeaway:** Modern React architecture is no longer a binary choice. In a Next.js app, you will likely use RSC for your page layouts and data fetching, Client Components for your interactive buttons, and SSG for your static marketing pages—all within the exact same application.
