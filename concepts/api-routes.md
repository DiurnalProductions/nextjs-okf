---
id: nextjs.api-routes
type: concept
title: Next.js API Routes
description: Route Handlers for HTTP endpoints in the app directory supporting REST, webhooks, and server-side mutations
tags: [nextjs, api-routes, route-handlers, backend, rest]
prerequisites:
  - nextjs.app-router
related:
  - nextjs.server-components
  - nextjs.middleware
  - nextjs.deployment
resource: https://nextjs.org/docs/app/building-your-application/routing/route-handlers
timestamp: 2026-01-01
---

# Summary

API Routes in the App Router are called **Route Handlers**, defined in `route.ts` files within the `app/` directory. They export HTTP method functions (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`) that receive a `Request` and return a `Response`.

Route Handlers run on the server (Node.js runtime by default, or edge runtime if configured). They serve as backend endpoints for external clients, webhooks, form submissions, and mutations—complementing Server Components which handle read-heavy UI rendering.

# Mental model

```text
app/api/
└── users/
    └── route.ts    →  /api/users

Route Handler execution context:

┌─ Server (like Server Components) ──────────────┐
│  • Runs on Node.js or Edge runtime              │
│  • Access to databases, env vars, file system   │
│  • Never ships to browser                       │
│  • Returns Response (JSON, HTML, streams)         │
└─────────────────────────────────────────────────┘

Server Components vs Route Handlers:

Server Component          Route Handler
────────────────          ─────────────
Renders React UI          Returns HTTP Response
Embedded in page tree     Standalone endpoint
For browser consumption   For any HTTP client
Read-heavy display        Mutations, webhooks, public APIs
```

**When to use Route Handlers vs Server Components:**

| Use case | Prefer |
|----------|--------|
| Display data in a page | Server Component (direct fetch/DB) |
| Form mutation from same app | Server Actions (or Route Handler) |
| Webhook receiver | Route Handler |
| Public REST API | Route Handler |
| Mobile app backend | Route Handler |
| Third-party integration callback | Route Handler |

Server Components and Route Handlers share the server execution environment but serve different purposes. Avoid fetching your own Route Handlers from Server Components—call the data layer directly instead.

# Example usage

A webhook endpoint:

```tsx
// Conceptual — app/api/webhooks/stripe/route.ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get("stripe-signature");

  const event = verifyStripeWebhook(body, signature);
  await processPaymentEvent(event);

  return NextResponse.json({ received: true });
}
```

A CRUD endpoint with on-demand revalidation:

```tsx
// Conceptual — app/api/posts/route.ts
import { revalidateTag } from "next/cache";
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const data = await request.json();
  const post = await db.post.create(data);

  revalidateTag("posts");
  return NextResponse.json(post, { status: 201 });
}
```

Route Handlers can configure runtime and caching via segment config exports (`export const runtime = 'edge'`, `export const dynamic = 'force-dynamic'`).

# Common mistakes

* **Creating internal API routes for Server Component data fetching.** Adds latency and complexity. Server Components should query data sources directly.
* **Putting UI rendering logic in Route Handlers.** Route Handlers return `Response` objects, not React components. Use Server Components for HTML rendering.
* **Exposing Route Handlers without authentication.** Public endpoints need explicit auth checks. Middleware can enforce auth before handlers run (see [Middleware](/concepts/middleware.md)).
* **Ignoring HTTP method exports.** Only exported methods are served. A `route.ts` with only `GET` returns 405 for POST requests.
* **Confusing Pages Router API routes with App Router Route Handlers.** Pages Router used `pages/api/*.ts` with `NextApiRequest`/`NextApiResponse`. App Router uses `app/**/route.ts` with Web standard `Request`/`Response`.
* **Not handling errors with appropriate status codes.** Route Handlers should return structured error responses (`NextResponse.json({ error }, { status: 400 })`), not throw unhandled exceptions.

# Related concepts

* [Server Components](/concepts/server-components.md) — shared server runtime, different output format
* [Middleware](/concepts/middleware.md) — runs before Route Handlers on matching paths
* [Data Fetching](/concepts/data-fetching.md) — `revalidateTag` and `revalidatePath` after mutations
* [Deployment](/concepts/deployment.md) — serverless function limits for Route Handlers
