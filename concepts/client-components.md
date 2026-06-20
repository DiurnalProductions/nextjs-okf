---
id: nextjs.client-components
type: concept
title: Next.js Client Components
description: Interactive React components marked with use client that hydrate in the browser
tags: [nextjs, client-components, react, hydration, interactivity]
prerequisites:
  - nextjs.rendering-model
  - nextjs.server-components
related:
  - nextjs.server-components
  - nextjs.data-fetching
resource: https://nextjs.org/docs/app/building-your-application/rendering/client-components
timestamp: 2026-01-01
---

# Summary

Client Components are React components that execute in the browser. They are created by adding the `"use client"` directive at the top of a file. Client Components enable interactivity—state, effects, event handlers, and browser APIs—that Server Components cannot provide.

In the App Router, Client Components are not an alternative to Server Components but a complement. The recommended pattern is a Server Component shell with small Client Component "islands" for interactive elements.

# Mental model

```text
"use client" directive → entire module + its imports become client bundle

Hydration flow:
1. Server renders Client Component to HTML (for fast first paint)
2. HTML sent to browser (visible immediately)
3. Client JS bundle downloads
4. React hydrates — attaches event listeners and state to existing DOM
5. Component becomes fully interactive

┌──────────────── Server ────────────────┐
│  Page (Server Component)               │
│    └── <Counter />  ←── "use client"   │
│         Server renders initial HTML    │
└────────────────────────────────────────┘
                  │
                  ▼ HTML + JS bundle
┌──────────────── Browser ───────────────┐
│  React hydrates <Counter />            │
│  useState(0) now active                │
│  onClick handlers attached             │
└────────────────────────────────────────┘
```

**The interactivity boundary:** Once you cross `"use client"`, everything imported by that file (unless it's another already-client module) joins the client bundle. Keep Client Components small and leaf-level in the component tree.

**Composition pattern (recommended):**

```text
ServerPage
  ├── ServerHeader
  ├── ServerProductList        ← data fetched on server
  │     └── ClientAddToCart    ← only this button is client-side
  └── ServerFooter
```

# Example usage

A search input with live filtering:

```tsx
// Conceptual — components/SearchBox.tsx
"use client";

import { useState } from "react";

export function SearchBox({ items }: { items: string[] }) {
  const [query, setQuery] = useState("");
  const filtered = items.filter((item) =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{filtered.map((item) => <li key={item}>{item}</li>)}</ul>
    </div>
  );
}
```

The parent Server Component fetches `items` and passes them as serializable props:

```tsx
// Conceptual — app/search/page.tsx (Server Component)
export default async function SearchPage() {
  const items = await getItems();
  return <SearchBox items={items} />;
}
```

# Common mistakes

* **Marking the root layout `"use client"`**. This forces the entire application into the client bundle, eliminating Server Component benefits. Only mark components that need interactivity.
* **Fetching initial data in `useEffect`.** This causes loading spinners and waterfalls. Pass server-fetched data as props from a parent Server Component.
* **Importing Server Components into Client Components.** This is a build error. Pass Server Components as `children` instead.
* **Confusing hydration with CSR.** Hydration attaches to server-rendered HTML. Pure CSR (no server HTML) is not the App Router default. The server still renders Client Components to HTML first.
* **Large Client Component trees.** Every `"use client"` dependency adds to the bundle. Split interactive parts into small leaf components.
* **Using browser APIs during server render in Client Components.** `window` and `document` are undefined during server-side pre-rendering of Client Components. Guard with `useEffect` or check `typeof window`.

# Related concepts

* [Server Components](/concepts/server-components.md) — default rendering mode and parent composition
* [Rendering Model](/concepts/rendering-model.md) — hydration and streaming context
* [Data Fetching](/concepts/data-fetching.md) — server-side fetching to pass props to Client Components
