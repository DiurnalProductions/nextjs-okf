---
id: nextjs.routing
type: concept
title: Next.js Routing
description: URL segments, dynamic routes, route groups, parallel routes, and client-side navigation in the App Router
tags: [nextjs, routing, navigation, urls]
prerequisites:
  - nextjs.app-router
related:
  - nextjs.middleware
  - nextjs.data-fetching
resource: https://nextjs.org/docs/app/building-your-application/routing
timestamp: 2026-01-01
---

# Summary

Routing in the App Router maps URL paths to nested React component trees. Each folder in `app/` corresponds to a URL segment. Dynamic segments (`[slug]`), catch-all segments (`[...slug]`), and optional catch-all (`[[...slug]]`) enable parameterized routes. Route groups `(folder)` organize code without affecting the URL.

Navigation can be server-initiated (full page load or server-rendered transition) or client-initiated via the `Link` component and `useRouter` hook, which trigger soft navigations that fetch new Server Component payloads without full page reloads.

# Mental model

**URL-to-component mapping:**

```text
URL: /shop/electronics/phones

app/
└── shop/
    └── electronics/
        └── phones/
            └── page.tsx   → renders this component
```

**Segment types:**

| Pattern | Example URL | Folder |
|---------|-------------|--------|
| Static | `/about` | `app/about/page.tsx` |
| Dynamic | `/blog/hello` | `app/blog/[slug]/page.tsx` |
| Catch-all | `/docs/a/b/c` | `app/docs/[...slug]/page.tsx` |
| Optional catch-all | `/docs` or `/docs/a` | `app/docs/[[...slug]]/page.tsx` |
| Route group | `/dashboard` (no group in URL) | `app/(admin)/dashboard/page.tsx` |

**Navigation models:**

* **`<Link href="...">`** — Client-side soft navigation. Fetches the next route's React Server Component payload from the server. Layouts persist.
* **`redirect()`** — Server-side redirect during rendering (Server Components, Route Handlers).
* **`useRouter().push()`** — Imperative client navigation (requires Client Component).
* **`middleware`** — Redirect or rewrite before routing resolves (see [Middleware](/concepts/middleware.md)).

The server/client boundary during navigation: soft navigations still fetch server-rendered RSC payloads. The client does not re-execute Server Component logic locally—it requests fresh server output.

# Example usage

An e-commerce route structure:

* `app/(shop)/products/page.tsx` — product listing at `/products` (route group `(shop)` omitted from URL).
* `app/(shop)/products/[id]/page.tsx` — product detail; `params.id` available in the page component.
* `app/(shop)/cart/page.tsx` — cart page.

In `[id]/page.tsx`, a Server Component receives `params` as a prop:

```tsx
// Conceptual — not a project scaffold
export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  return <ProductDetail product={product} />;
}
```

`generateStaticParams()` can pre-render dynamic routes at build time for static generation.

# Common mistakes

* **Confusing route groups with URL prefixes.** Folders wrapped in parentheses `(name)` do not appear in the URL—they only organize files.
* **Using `useRouter` from `next/router` in App Router.** App Router Client Components must use `useRouter` from `next/navigation`, not the Pages Router import.
* **Hardcoding URLs instead of dynamic segments.** Leads to unmaintainable route trees. Use `[param]` segments and `params` props.
* **Fetching in Client Components when `params` are available server-side.** Dynamic route parameters should drive server-side data fetching in the `page.tsx` Server Component.
* **Ignoring `generateStaticParams` for static dynamic routes.** Without it, dynamic routes default to dynamic rendering unless configured otherwise.

# Related concepts

* [App Router](/concepts/app-router.md) — special files and layout system
* [Middleware](/concepts/middleware.md) — intercepts requests before route matching
* [Data Fetching](/concepts/data-fetching.md) — fetching data based on route parameters
