---
id: nextjs.server-components
type: concept
title: Next.js Server Components
description: React components rendered on the server by default with direct data access and zero client bundle cost
tags: [nextjs, server-components, react, rsc, ssr]
prerequisites:
  - nextjs.rendering-model
  - nextjs.app-router
related:
  - nextjs.client-components
  - nextjs.data-fetching
  - nextjs.api-routes
resource: https://nextjs.org/docs/app/building-your-application/rendering/server-components
timestamp: 2026-01-01
---

# Summary

Server Components (RSC) are React components that render exclusively on the server. In the App Router, **all components are Server Components unless marked with `"use client"`**. They can be `async`, fetch data directly, access databases and filesystems, and read secret environment variables—all without shipping that code to the browser.

Server Components reduce client JavaScript bundle size because their implementation never downloads to the client. The browser receives rendered HTML and a compact serialized format (the RSC payload) that describes the component tree.

# Mental model

```text
Server Component lifecycle (per request):

1. Request arrives
2. Server executes component function (can await data)
3. Server produces:
   a. HTML for initial paint
   b. RSC payload (serialized tree) for client navigations
4. Client receives output — component source code stays on server

What Server Components CAN do:
  ✓ async/await
  ✓ Direct DB queries
  ✓ Read process.env secrets
  ✓ Import server-only modules
  ✓ Render static markup

What Server Components CANNOT do:
  ✗ useState, useEffect, useReducer
  ✗ Event handlers (onClick, onChange)
  ✗ Browser APIs (window, document, localStorage)
  ✗ Custom hooks that use the above
```

**Composition rule:** Server Components can import and render Client Components (passing serializable props). Client Components cannot import Server Components—they can only receive them as `children` or via props slots passed from a parent Server Component.

```text
ServerComponent
  ├── StaticHeader          (Server)
  ├── InteractiveButton     (Client — "use client")
  └── ServerComponent
        └── AnotherServer    (Server — nested under Client via children slot)
```

The boundary is enforced at the `"use client"` directive. Everything in a Client Component's import graph ships to the browser.

# Example usage

A user profile page as a Server Component:

```tsx
// Conceptual — app/profile/page.tsx
export default async function ProfilePage() {
  const session = await getSession();        // server-only auth check
  const user = await db.user.find(session);  // direct database access

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
      <EditProfileButton userId={user.id} />  {/* Client Component child */}
    </div>
  );
}
```

The database query, session logic, and static markup never appear in the client bundle. Only `EditProfileButton` and its dependencies download to the browser.

Server Components relate conceptually to [API Routes](/concepts/api-routes.md): both run server-side code. Server Components render UI with embedded data; Route Handlers expose HTTP endpoints. Often a Server Component fetches data directly instead of calling an internal API route—a pattern called "data fetching where you need it."

# Common mistakes

* **Adding `"use client"` to pages that only display data.** If a component has no interactivity, it should remain a Server Component.
* **Using `useEffect` for initial data loading in pages.** This is the CSR anti-pattern. Fetch in the Server Component with `async/await` instead.
* **Importing server-only modules into Client Components.** Even indirect imports through a shared utility file will fail or leak code to the bundle. Use the `server-only` package to enforce boundaries.
* **Passing non-serializable props to Client Components.** Functions, class instances, and Symbols cannot cross the server/client boundary. Pass plain data (strings, numbers, plain objects, arrays).
* **Expecting Server Components to update after client state changes.** Server Components render per request. Client-side state changes only affect Client Components unless a navigation triggers a new server render.

# Related concepts

* [Client Components](/concepts/client-components.md) — the interactivity counterpart
* [Rendering Model](/concepts/rendering-model.md) — overall server/client execution model
* [Data Fetching](/concepts/data-fetching.md) — async data access in Server Components
* [API Routes](/concepts/api-routes.md) — alternative server-side data and mutation layer
