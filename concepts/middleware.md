---
id: nextjs.middleware
type: concept
title: Next.js Middleware
description: Edge-executed request interception for authentication, redirects, rewrites, and header manipulation
tags: [nextjs, middleware, edge, auth, redirects]
prerequisites:
  - nextjs.routing
  - nextjs.api-routes
related:
  - nextjs.routing
  - nextjs.deployment
resource: https://nextjs.org/docs/app/building-your-application/routing/middleware
timestamp: 2026-01-01
---

# Summary

Middleware is a function that runs before a request is completed, intercepting every matching route. It executes on the **Edge Runtime**—a lightweight V8 isolate close to the user—not the full Node.js server. Middleware can rewrite URLs, redirect requests, modify headers, set cookies, and perform authentication checks before the App Router or Route Handlers process the request.

Middleware is defined in a single `middleware.ts` file at the project root (or `src/middleware.ts`). A `matcher` config controls which paths it applies to.

# Mental model

```text
Request flow with middleware:

Incoming request
    │
    ▼
┌─ middleware.ts (Edge Runtime) ─────────────┐
│  • Auth check                                │
│  • Redirect / rewrite                        │
│  • Header/cookie manipulation                │
│  • A/B testing, geolocation routing          │
│  • Bot detection                             │
└─────────────────────────────────────────────┘
    │
    ├── NextResponse.redirect()  → browser redirect (stops)
    ├── NextResponse.rewrite()   → internal URL change (continues)
    └── NextResponse.next()      → pass through (continues)
    │
    ▼
App Router / Route Handler / Static file

Edge vs Node.js runtime:

EDGE (middleware)              NODE (default server)
─────                          ────
Lightweight, fast cold start   Full Node.js APIs
Limited APIs (no fs, no heavy  Database drivers, file system
  native modules)              Longer cold starts
Runs globally distributed      Runs in deployment region
```

**Middleware ↔ Routing relationship:** Middleware sees the request before route matching resolves. It can rewrite `/dashboard` to `/dashboard/en` internally (routing still applies to the rewritten URL) or redirect unauthenticated users away from protected routes before any Server Component renders.

Middleware cannot render React components or return HTML directly—it only manipulates the request/response pipeline.

# Example usage

Authentication gate for protected routes:

```tsx
// Conceptual — middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("session")?.value;

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

Locale-based rewrite:

```tsx
// Conceptual — middleware.ts
export function middleware(request: NextRequest) {
  const locale = request.headers.get("accept-language")?.split(",")[0] ?? "en";
  const url = request.nextUrl.clone();
  url.pathname = `/${locale}${url.pathname}`;
  return NextResponse.rewrite(url);
}
```

# Common mistakes

* **Performing heavy computation or database queries in middleware.** Edge Runtime has limited APIs and short execution timeouts. Keep middleware fast; defer heavy work to Server Components or Route Handlers.
* **Using Node.js-only APIs in middleware.** `fs`, native database drivers, and many npm packages are incompatible with Edge Runtime. Use edge-compatible libraries or move logic server-side.
* **Applying middleware to all routes without a matcher.** Unrestricted middleware runs on every request including static assets and images, adding latency unnecessarily. Use `config.matcher` to scope it.
* **Trying to render UI from middleware.** Middleware returns `NextResponse` objects, not React components. Redirect or rewrite to a page that renders the UI.
* **Confusing middleware with Server Component auth checks.** Middleware provides a first gate (fast redirect), but Server Components should still verify authorization for data access. Middleware can be bypassed in some edge cases; defense in depth matters.
* **Setting cookies without understanding downstream effects.** Cookies set in middleware are available to Server Components via `cookies()` but affect caching—reading cookies opts routes into dynamic rendering.
* **Assuming middleware runs on the same runtime as Route Handlers.** Middleware is always edge; Route Handlers default to Node.js unless `export const runtime = 'edge'` is set.

# Related concepts

* [Routing](/concepts/routing.md) — middleware intercepts before route resolution
* [API Routes](/concepts/api-routes.md) — middleware can protect API endpoints
* [Deployment](/concepts/deployment.md) — edge middleware deploys to CDN edge locations
* [Caching](/concepts/caching.md) — cookie/header reads in pages affect cache behavior
