---
id: nextjs.data-fetching
type: concept
title: Next.js Data Fetching
description: Async data access in Server Components with fetch caching, revalidation, and streaming patterns
tags: [nextjs, data-fetching, fetch, revalidation, streaming]
prerequisites:
  - nextjs.server-components
  - nextjs.client-components
related:
  - nextjs.caching
  - nextjs.server-components
resource: https://nextjs.org/docs/app/building-your-application/data-fetching
timestamp: 2026-01-01
---

# Summary

In the App Router, data fetching happens primarily in Server Components using `async/await`. Components can directly `fetch()` HTTP resources, query databases, or read files—no separate data-fetching API like `getServerSideProps` is required.

The `fetch` API is extended in Next.js with caching and revalidation options. Data fetching integrates with React `Suspense` for streaming: slow data sources do not block the entire page from rendering.

# Mental model

```text
Data fetching locations (App Router):

┌─ Server Component (preferred) ─────────────────┐
│  async function Page() {                     │
│    const data = await fetch(url, { cache }); │
│    return <UI data={data} />;                │
│  }                                           │
└──────────────────────────────────────────────┘

┌─ Route Handler ──────────────────────────────┐
│  export async function GET() { ... }         │
│  Used for external API consumers / mutations │
└──────────────────────────────────────────────┘

┌─ Client Component (avoid for initial data) ──┐
│  useEffect + fetch → loading states, waterfalls│
│  Acceptable for user-triggered refetches     │
└──────────────────────────────────────────────┘
```

**Fetch caching options:**

| Option | Behavior |
|--------|----------|
| Default (`cache: 'force-cache'`) | Cached indefinitely (static) |
| `cache: 'no-store'` | Fresh fetch every request (dynamic) |
| `next: { revalidate: N }` | Cache with time-based revalidation (ISR) |
| `next: { tags: ['posts'] }` | Tag-based cache for on-demand revalidation |

**Revalidation strategies:**

* **Time-based** — `revalidate: 3600` refetches after one hour.
* **On-demand** — `revalidateTag('posts')` or `revalidatePath('/blog')` invalidates cached data (typically called from Route Handlers after mutations).
* **No cache** — `cache: 'no-store'` for always-fresh data.

**Streaming with Suspense:**

```text
Request → Layout renders immediately
       → Suspense boundary shows fallback
       → Slow fetch completes → content streams in
       → Next Suspense boundary resolves → more content streams
```

# Example usage

Fetching blog posts in a Server Component:

```tsx
// Conceptual — app/blog/page.tsx
export default async function BlogPage() {
  const posts = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60, tags: ["posts"] },
  }).then((res) => res.json());

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

After creating a post via a Route Handler, trigger on-demand revalidation:

```tsx
// Conceptual — in a Route Handler after mutation
import { revalidateTag } from "next/cache";
revalidateTag("posts");
```

For database access (non-fetch), use `unstable_cache` or configure route segment `dynamic`/`revalidate` exports to control caching behavior.

# Common mistakes

* **Using `useEffect` + `fetch` for page-level data.** Creates client-side waterfalls and loading spinners. Use Server Component `async/await` instead.
* **Calling internal API routes from Server Components.** An anti-pattern called "fetching your own API." Query the database or service directly in the Server Component—it avoids an unnecessary HTTP round trip.
* **Ignoring cache defaults.** `fetch` caches by default in Server Components. Unexpected stale data often means you forgot `cache: 'no-store'` or a revalidation strategy.
* **Not using Suspense for independent data sources.** Sequential `await` calls block each other. Wrap independent fetches in separate async Server Components inside Suspense boundaries for parallel streaming.
* **Confusing Pages Router data methods with App Router.** `getStaticProps`, `getServerSideProps`, and `getInitialProps` do not exist in `app/`. Data fetching is native to Server Components.
* **Fetching in Client Components when the data is not user-triggered.** Initial page data should always originate from the server.

# Related concepts

* [Caching](/concepts/caching.md) — deep dive into the Data Cache and rendering modes
* [Server Components](/concepts/server-components.md) — where fetching executes
* [API Routes](/concepts/api-routes.md) — endpoints for mutations and external consumers
* [Routing](/concepts/routing.md) — `generateStaticParams` for build-time data
