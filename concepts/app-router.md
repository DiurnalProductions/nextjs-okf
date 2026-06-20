---
id: nextjs.app-router
type: concept
title: Next.js App Router
description: File-based routing system in the app directory with layouts, pages, loading, and error boundaries
tags: [nextjs, app-router, routing, layouts]
prerequisites:
  - nextjs.architecture
  - nextjs.rendering-model
related:
  - nextjs.routing
  - nextjs.api-routes
  - nextjs.server-components
resource: https://nextjs.org/docs/app
timestamp: 2026-01-01
---

# Summary

The App Router is Next.js's modern routing system, introduced alongside React Server Components. All routes live under the `app/` directory. Folders define URL segments; special files (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`) define UI behavior for those segments.

The App Router replaces the Pages Router (`pages/` directory) as the recommended approach. It supports nested layouts that persist across navigations, built-in loading and error states, and Server Components by default.

# Mental model

The `app/` directory is a tree that mirrors URL structure:

```text
app/
├── layout.tsx          → root layout (wraps entire app)
├── page.tsx            → /
├── dashboard/
│   ├── layout.tsx      → layout for /dashboard/*
│   ├── page.tsx        → /dashboard
│   └── settings/
│       └── page.tsx    → /dashboard/settings
└── api/
    └── health/
        └── route.ts    → /api/health (Route Handler)
```

**Special files and their roles:**

| File | Purpose |
|------|---------|
| `page.tsx` | Unique UI for a route segment; makes the segment publicly accessible |
| `layout.tsx` | Shared UI that wraps child segments; persists state across navigations |
| `loading.tsx` | Instant loading UI shown while segment content streams |
| `error.tsx` | Error boundary for the segment; must be a Client Component |
| `not-found.tsx` | UI for 404 within the segment |
| `route.ts` | API endpoint (Route Handler), not a page |
| `template.tsx` | Like layout but re-mounts on navigation |

**Server/client boundary in routing:** Route files (`page.tsx`, `layout.tsx`) are Server Components by default. They can import Client Components as children but cannot use hooks or browser APIs directly. The router itself runs on the server—it resolves the URL, assembles the layout tree, and initiates rendering.

Layouts do not re-render on client-side navigation between sibling pages sharing the same layout. This is a key UX advantage over the Pages Router.

# Example usage

A blog structure:

* `app/layout.tsx` — root HTML shell, fonts, global providers (if needed).
* `app/blog/layout.tsx` — blog sidebar and header, shared across all blog posts.
* `app/blog/page.tsx` — blog index listing posts (Server Component fetching posts).
* `app/blog/[slug]/page.tsx` — individual post page with dynamic segment.
* `app/blog/[slug]/loading.tsx` — skeleton shown while post content streams.
* `app/blog/[slug]/error.tsx` — fallback if post fetch fails.

Navigating from `/blog` to `/blog/my-post` keeps the blog layout mounted while only the page segment updates.

# Common mistakes

* **Forgetting that only `page.tsx` creates a public route.** A folder without `page.tsx` is a route group or organizational segment, not a visitable URL (unless it has a `route.ts`).
* **Putting too much logic in layouts.** Layouts wrap all child routes. Heavy data fetching in a layout runs for every child navigation. Prefer fetching in `page.tsx` when data is page-specific.
* **Using Pages Router conventions in App Router.** `_app.tsx`, `getServerSideProps`, and `getStaticProps` do not exist in `app/`. Data fetching moves into Server Components directly.
* **Making `error.tsx` a Server Component.** Error boundaries require client-side React error boundary APIs; `error.tsx` must include `"use client"`.
* **Nested layouts without understanding persistence.** State in a layout survives navigations among its children. Accidentally storing page-specific state in layouts causes stale UI.

# Related concepts

* [Routing](/concepts/routing.md) — dynamic segments, route groups, and navigation APIs
* [Server Components](/concepts/server-components.md) — default component type for route files
* [API Routes](/concepts/api-routes.md) — `route.ts` files for backend endpoints
* [Middleware](/concepts/middleware.md) — runs before the App Router resolves routes
