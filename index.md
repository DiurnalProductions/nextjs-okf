# Next.js Knowledge Pack

A self-contained OKF knowledge bundle for Next.js (App Router era). Install this pack into any OKF-compatible client to enable structured learning, concept traversal, and adaptive course generation.

## Start here

If you are new to Next.js or migrating from the Pages Router, begin with [Next.js Architecture](/concepts/architecture.md). It establishes the full-stack mental model—how requests flow, where React runs, and how server and client code coexist in one framework.

After architecture, read [Rendering Model](/concepts/rendering-model.md) before touching routing or components. The server/client boundary is the single most important concept in modern Next.js; everything else builds on it.

## Recommended learning progression

Follow this order for a coherent path from fundamentals to production deployment:

1. [Architecture](/concepts/architecture.md) — full-stack framework model and request lifecycle
2. [Rendering Model](/concepts/rendering-model.md) — server vs client execution boundaries
3. [App Router](/concepts/app-router.md) — file-based routing system, layouts, and special files
4. [Routing](/concepts/routing.md) — URLs, dynamic segments, navigation, and route groups
5. [Server Components](/concepts/server-components.md) — default rendering mode and server-side data access
6. [Client Components](/concepts/client-components.md) — interactivity, hydration, and the `"use client"` boundary
7. [Data Fetching](/concepts/data-fetching.md) — fetching in server components, revalidation, and streaming
8. [Caching](/concepts/caching.md) — request memoization, static vs dynamic rendering, ISR
9. [API Routes](/concepts/api-routes.md) — Route Handlers and backend endpoints
10. [Middleware](/concepts/middleware.md) — edge execution and request interception
11. [Deployment](/concepts/deployment.md) — build output, hosting models, and production concerns

## Fundamentals

Core concepts that explain what Next.js is and how it fits into a full-stack application:

* [Architecture](/concepts/architecture.md) — React framework with integrated server runtime
* [Rendering Model](/concepts/rendering-model.md) — where code executes and what ships to the browser

## Rendering model

How React components are rendered, split, and delivered:

* [Server Components](/concepts/server-components.md) — zero-bundle server rendering by default
* [Client Components](/concepts/client-components.md) — interactive islands with hydration
* [Data Fetching](/concepts/data-fetching.md) — async server data access patterns
* [Caching](/concepts/caching.md) — Next.js Data Cache, static generation, and revalidation

## Routing system

File-system routing and navigation in the App Router:

* [App Router](/concepts/app-router.md) — `app/` directory conventions and special files
* [Routing](/concepts/routing.md) — segments, dynamic routes, parallel routes, and navigation APIs

## Backend features

Server-side capabilities beyond page rendering:

* [API Routes](/concepts/api-routes.md) — Route Handlers for REST and webhooks
* [Middleware](/concepts/middleware.md) — edge middleware for auth, redirects, and headers
* [Deployment](/concepts/deployment.md) — production builds, adapters, and platform integration

## Concept graph overview

```text
architecture
    └── rendering-model
            ├── server-components ── client-components
            │         └── data-fetching ↔ caching
            └── app-router
                    ├── routing ↔ middleware
                    ├── api-routes
                    └── deployment
```
