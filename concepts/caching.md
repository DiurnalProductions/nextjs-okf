---
id: nextjs.caching
type: concept
title: Next.js Caching
description: Request memoization, Data Cache, static vs dynamic rendering, and incremental static regeneration
tags: [nextjs, caching, isr, static, dynamic, revalidation]
prerequisites:
  - nextjs.data-fetching
  - nextjs.rendering-model
related:
  - nextjs.data-fetching
  - nextjs.deployment
resource: https://nextjs.org/docs/app/building-your-application/caching
timestamp: 2026-01-01
---

# Summary

Next.js implements a multi-layer caching system that controls how often pages and data are regenerated. Understanding caching is essential for balancing performance (fast responses from cache) against freshness (up-to-date content).

The four caching mechanisms are: **Request Memoization** (per-request deduplication), **Data Cache** (persistent cross-request `fetch` cache), **Full Route Cache** (pre-rendered HTML and RSC payload), and **Router Cache** (client-side session cache for soft navigations).

# Mental model

```text
Caching layers (simplified):

Request → Request Memoization (same fetch in one render = one call)
       → Data Cache (fetch results persisted across requests)
       → Full Route Cache (pre-rendered page output at build/revalidate)
       → Response to browser
       → Router Cache (client caches RSC payloads for back/forward)

Static vs Dynamic rendering:

STATIC                          DYNAMIC
──────                          ───────
Rendered at build time          Rendered per request
OR revalidated on timer         OR when cache miss
Served from Full Route Cache    Skips Full Route Cache
Fast CDN delivery               Always fresh (slower)

A route becomes DYNAMIC when it uses:
  • fetch(..., { cache: 'no-store' })
  • cookies() or headers()
  • searchParams in page component
  • export const dynamic = 'force-dynamic'
```

**Incremental Static Regeneration (ISR):**

ISR combines static performance with periodic freshness. A page is statically generated, then regenerated in the background after a revalidation interval:

```text
t=0:    Build generates /blog/post-1 (cached)
t=60s:  Revalidation window expires
t=61s:  Next request serves stale page AND triggers background regeneration
t=62s:  Subsequent requests serve freshly regenerated page
```

**On-demand revalidation** bypasses the timer: `revalidatePath('/blog')` or `revalidateTag('posts')` immediately marks cached content stale.

# Example usage

Static page with hourly revalidation:

```tsx
// Conceptual — app/products/page.tsx
export const revalidate = 3600; // segment-level revalidation

export default async function ProductsPage() {
  const products = await fetch("https://api.example.com/products", {
    next: { tags: ["products"] },
  }).then((res) => res.json());

  return <ProductGrid products={products} />;
}
```

Force dynamic rendering for a personalized dashboard:

```tsx
// Conceptual — app/dashboard/page.tsx
export const dynamic = "force-dynamic";

export default async function DashboardPage() {
  const user = await getCurrentUser(); // uses cookies()
  return <Dashboard user={user} />;
}
```

Request memoization (automatic):

```tsx
// Conceptual — both calls deduplicated within the same request
async function getUser() {
  return fetch("/api/user").then((r) => r.json());
}

export default async function Page() {
  const [userA, userB] = await Promise.all([getUser(), getUser()]);
  // fetch executed only once — Request Memoization
}
```

# Common mistakes

* **Assuming `fetch` is always uncached.** In Server Components, `fetch` defaults to `force-cache`. Unexpected stale content usually means you need explicit `no-store` or revalidation.
* **Using `cache: 'no-store'` everywhere.** Forces dynamic rendering for all routes, losing static generation and CDN benefits. Apply selectively to truly dynamic data.
* **Confusing Request Memoization with Data Cache.** Request Memoization deduplicates within a single render pass. Data Cache persists across requests. They solve different problems.
* **Ignoring that `cookies()` and `headers()` opt into dynamic rendering.** Reading request-specific data automatically makes the route dynamic, even if all fetches are cached.
* **Expecting the Router Cache to persist across hard reloads.** The client Router Cache clears on full page refresh. It only optimizes soft navigations within a session.
* **Not tagging cached data for on-demand revalidation.** Without `next: { tags: [...] }`, you can only revalidate by path or time interval, not by logical data group.
* **Treating ISR as "SSR with a timer."** ISR serves cached static pages instantly; regeneration happens in the background. SSR renders on every request.

# Related concepts

* [Data Fetching](/concepts/data-fetching.md) — fetch options that control cache behavior
* [Rendering Model](/concepts/rendering-model.md) — static vs dynamic rendering spectrum
* [Deployment](/concepts/deployment.md) — how caching interacts with CDN and hosting platforms
