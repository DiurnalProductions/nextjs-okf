---
id: nextjs.deployment
type: concept
title: Next.js Deployment
description: Production builds, hosting models, static export, and platform integration for App Router applications
tags: [nextjs, deployment, production, hosting, vercel, docker]
prerequisites:
  - nextjs.caching
  - nextjs.api-routes
  - nextjs.middleware
related:
  - nextjs.caching
  - nextjs.architecture
resource: https://nextjs.org/docs/app/building-your-application/deploying
timestamp: 2026-01-01
---

# Summary

Deploying a Next.js App Router application involves building an optimized production bundle and choosing a hosting model that supports the framework's server features. The `next build` command analyzes routes, pre-renders static pages, and prepares serverless functions for dynamic routes, Route Handlers, and middleware.

Next.js is platform-agnostic but integrates deeply with Vercel (its creator). Self-hosting via Docker or Node.js is fully supported, as is static export for sites that require no server runtime.

# Mental model

```text
next build output:

┌─ Static assets ─────────────────────────────┐
│  JS/CSS bundles, images, fonts              │
│  Served from CDN                            │
└─────────────────────────────────────────────┘

┌─ Pre-rendered pages (Static/ISR) ───────────┐
│  HTML + RSC payload generated at build      │
│  Revalidated on timer or on-demand          │
│  Served from CDN / Full Route Cache         │
└─────────────────────────────────────────────┘

┌─ Server Functions (Dynamic) ────────────────┐
│  Server Components rendered per request     │
│  Route Handlers (API endpoints)             │
│  Executed as serverless or long-running Node│
└─────────────────────────────────────────────┘

┌─ Edge Functions ────────────────────────────┐
│  Middleware (always edge)                   │
│  Route Handlers with runtime = 'edge'       │
│  Deployed to CDN edge locations             │
└─────────────────────────────────────────────┘
```

**Hosting models:**

| Model | Server features | Best for |
|-------|----------------|----------|
| Vercel / managed platform | Full (default) | Production apps with minimal ops |
| Docker + Node.js (`next start`) | Full | Self-hosted, Kubernetes, VPS |
| Static export (`output: 'export'`) | None (no SSR, no API) | Pure static sites |
| Serverless adapters | Full (with cold starts) | AWS Lambda, Cloudflare Workers |

**Production checklist concepts:**

* Environment variables: `NEXT_PUBLIC_*` for client-exposed values; all others server-only.
* `next build` fails on TypeScript and ESLint errors by default.
* Image optimization requires a compatible image loader or `unoptimized: true` for static export.
* Middleware and edge Route Handlers deploy to edge networks on supported platforms.

# Example usage

Standard production build and start:

```bash
# Conceptual commands — not project scaffolding
next build    # analyzes routes, generates static pages, bundles server code
next start    # serves production build on Node.js
```

Docker self-hosting pattern:

```dockerfile
# Conceptual — multi-stage Dockerfile
# Stage 1: install dependencies
# Stage 2: next build
# Stage 3: copy .next/standalone output, run node server.js
```

Static export for CDN-only hosting:

```js
// Conceptual — next.config.js
module.exports = { output: "export" };
// Disables: Server Components SSR, Route Handlers, middleware, ISR, image optimization
```

Environment variable usage:

```text
DATABASE_URL=...          → server-only (Server Components, Route Handlers)
API_SECRET=...            → server-only
NEXT_PUBLIC_API_URL=...   → exposed to browser bundle
```

# Common mistakes

* **Deploying to a static host without understanding limitations.** Static export (`output: 'export'`) removes SSR, API routes, middleware, and ISR. Many App Router features require a server.
* **Exposing secrets via `NEXT_PUBLIC_` prefix.** Any variable prefixed with `NEXT_PUBLIC_` is embedded in the client JavaScript bundle. Database URLs and API keys must never use this prefix.
* **Ignoring build output analysis.** `next build` prints which routes are static, dynamic, or ISR. Unexpected dynamic routes often indicate missing cache configuration.
* **Not configuring image domains.** `next/image` requires explicit `remotePatterns` or `domains` for external images in production.
* **Assuming development mode behavior matches production.** Caching is aggressive in production; features like Fast Refresh and verbose errors are dev-only.
* **Undersizing serverless functions.** Dynamic routes and Route Handlers run as serverless functions on many platforms. Cold starts and execution time limits affect UX—consider `runtime = 'edge'` for lightweight handlers or a long-running Node server for heavy workloads.
* **Skipping `standalone` output for Docker.** `output: 'standalone'` in `next.config.js` produces a minimal self-contained server bundle, significantly reducing Docker image size.
* **Forgetting to set production environment variables on the hosting platform.** `.env.local` is not deployed. Secrets must be configured in the hosting provider's dashboard or CI/CD pipeline.

# Related concepts

* [Caching](/concepts/caching.md) — static vs dynamic routes affect deployment architecture
* [Architecture](/concepts/architecture.md) — overall system model mapped to infrastructure
* [Middleware](/concepts/middleware.md) — deploys as edge functions
* [API Routes](/concepts/api-routes.md) — deploy as serverless or Node.js endpoints
