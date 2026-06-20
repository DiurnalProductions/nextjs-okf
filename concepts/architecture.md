---
id: nextjs.architecture
type: concept
title: Next.js Architecture
description: Full-stack React framework model with integrated server runtime and blended server/client execution
tags: [nextjs, architecture, fullstack, react, node]
prerequisites: []
related:
  - nextjs.rendering-model
  - nextjs.app-router
resource: https://nextjs.org/docs
timestamp: 2026-01-01
---

# Summary

Next.js is a full-stack React framework that unifies UI rendering, data access, routing, and backend endpoints in a single application. In the App Router era, every request passes through a Node.js (or edge) server runtime that can render React on the server, stream HTML to the client, and selectively ship JavaScript only where interactivity is required.

Next.js is not merely a client-side SPA toolkit. It is a coordinated system: a file-based router maps URLs to React trees, a rendering engine decides what runs on the server versus the browser, and built-in caching layers control how often content is regenerated.

# Mental model

Think of Next.js as three cooperating layers:

1. **Router** — Maps incoming HTTP requests to React component trees via the `app/` directory.
2. **Renderer** — Executes React components on the server (default) or prepares client bundles for hydration.
3. **Runtime services** — Provides data fetching, caching, middleware, and Route Handlers alongside rendering.

**Request lifecycle (simplified):**

```text
Browser request
    → Middleware (optional, edge)
    → Route matching (app/ file system)
    → Server Component rendering + data fetching
    → HTML stream to browser
    → Client Component hydration (interactive islands)
    → Subsequent navigations (client-side routing with server fetches)
```

The critical boundary: **server code never ships to the browser**. Server Components, direct database access, and secret environment variables stay on the server. Only Client Components and their dependencies are bundled for the client.

This is fundamentally different from a Create React App or Vite SPA where all component code downloads and executes in the browser.

# Example usage

A product listing page illustrates the blended model:

* A **Server Component** at `app/products/page.tsx` fetches product data from a database during the request.
* It renders a static header and product grid on the server—no JavaScript needed for that markup.
* A **Client Component** child handles an "Add to cart" button with `onClick` handlers.
* A **Route Handler** at `app/api/cart/route.ts` processes cart mutations as a backend API.

The page is one React tree, but execution is split: server for data and static UI, client for interactivity, Route Handlers for mutations.

# Common mistakes

* **Treating Next.js as a pure SPA.** Client-side routing exists, but the default rendering path is server-first. Assuming all data fetching happens in `useEffect` misses the primary App Router pattern.
* **Putting secrets in Client Components.** Anything imported into a `"use client"` file can end up in the browser bundle. Database credentials and API keys belong in Server Components or Route Handlers only.
* **Ignoring the server runtime.** Features like middleware, streaming, and the Data Cache only exist because a server processes each request. Static export mode removes many of these capabilities.
* **Confusing Next.js with React itself.** Next.js adds routing, rendering modes, caching, and deployment conventions on top of React. React knowledge is necessary but not sufficient.

# Related concepts

* [Rendering Model](/concepts/rendering-model.md) — how server and client execution is decided
* [App Router](/concepts/app-router.md) — the file-system routing layer
* [Deployment](/concepts/deployment.md) — how the architecture maps to production hosting
