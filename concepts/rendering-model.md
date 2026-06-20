---
id: nextjs.rendering-model
type: concept
title: Next.js Rendering Model
description: How Next.js decides what renders on the server, what ships to the client, and when hydration occurs
tags: [nextjs, rendering, ssr, rsc, hydration]
prerequisites:
  - nextjs.architecture
related:
  - nextjs.server-components
  - nextjs.client-components
  - nextjs.caching
resource: https://nextjs.org/docs/app/building-your-application/rendering
timestamp: 2026-01-01
---

# Summary

The Next.js rendering model defines where React components execute and what reaches the browser. In the App Router, **Server Components are the default**: they render on the server, produce HTML (and a serialized component payload), and send zero component JavaScript to the client unless a Client Component is nested within them.

Rendering is not a single mode but a spectrum: static pages pre-rendered at build time, dynamic pages rendered per request, and streaming responses that progressively send HTML as data resolves.

# Mental model

Imagine the React tree as two territories separated by a hard border:

```text
┌─────────────────────────────────────────┐
│  SERVER TERRITORY                       │
│  • Server Components (default)          │
│  • async/await data fetching            │
│  • Direct DB / filesystem access        │
│  • Secrets and env vars                 │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  CLIENT TERRITORY ("use client")  │  │
│  │  • useState, useEffect, hooks     │  │
│  │  • Event handlers (onClick, etc.) │  │
│  │  • Browser APIs (window, localStorage) │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
         │
         ▼  HTML + selective JS bundles
    ┌─────────┐
    │ Browser │  ← hydration attaches event listeners
    └─────────┘
```

**Server rendering** produces HTML on the server for the initial request. The browser displays it immediately—even before JavaScript loads.

**Hydration** is the process where React on the client attaches interactivity to Server Component HTML by activating Client Component islands. Hydration is not re-rendering from scratch; it reconciles the server HTML with client React.

**Static vs dynamic rendering:**

| Mode | When it runs | Typical use |
|------|-------------|-------------|
| Static | Build time or on revalidation | Marketing pages, docs |
| Dynamic | Every request (or per-cache-miss) | Personalized dashboards, real-time data |
| Streaming | During the request, chunk by chunk | Slow data sources, improved TTFB |

The rendering model is the foundation for understanding [Server Components](/concepts/server-components.md), [Client Components](/concepts/client-components.md), and [Caching](/concepts/caching.md).

# Example usage

A dashboard page demonstrates the rendering split:

* The page component is a Server Component. It awaits `getUser()` and `getMetrics()` on the server.
* The server streams the page shell immediately, then streams metric cards as each query resolves.
* A `<DateRangePicker />` Client Component child provides calendar interactivity.
* The browser receives HTML for the metrics (readable without JS) plus a JavaScript bundle only for the date picker and its dependencies.

No `useEffect` fetch runs in the browser for initial data—the server already embedded it in the HTML stream.

# Common mistakes

* **Assuming SSR means "everything runs on the server."** Client Components still download and execute in the browser. SSR sends their initial HTML from the server, but their JavaScript still hydrates client-side.
* **Confusing SSR with CSR.** In pure client-side rendering (CSR), the browser receives an empty shell and fetches all data after JavaScript loads. Next.js Server Components fetch data on the server before HTML is sent—fundamentally different timing.
* **Marking entire pages `"use client"` by default.** This opts out of Server Component benefits (smaller bundles, direct data access) and recreates SPA patterns unnecessarily.
* **Expecting Server Components to re-render on the client.** Server Components run once per request on the server. State changes require Client Components or a new navigation/request.
* **Ignoring streaming.** Blocking the entire page until all data resolves hurts time-to-first-byte. React `Suspense` boundaries enable progressive streaming.

# Related concepts

* [Server Components](/concepts/server-components.md) — the default rendering unit
* [Client Components](/concepts/client-components.md) — the interactivity boundary
* [Caching](/concepts/caching.md) — controls static vs dynamic rendering decisions
* [Data Fetching](/concepts/data-fetching.md) — how data enters the render pipeline
