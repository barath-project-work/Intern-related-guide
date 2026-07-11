# 04 - Next.js Server & Client Components

## The Fundamental Concept

**Every component in Next.js is a Server Component by default.**

This is the biggest change from traditional React. Understanding when to use each is critical.

```
┌──────────────────────────────┐
│       Server Component        │ ← DEFAULT — NO "use client"
│                              │
│  • Runs on the server        │
│  • Can be async              │
│  • Can access DB/filesystem  │
│  • No state/effects          │
│  • No browser APIs           │
│  • Smaller JS bundle         │
└──────────────┬───────────────┘
               │
    ┌──────────┴──────────┐
    ▼                     ▼
┌─────────────────┐ ┌─────────────────┐
│   Client         │ │   Server        │
│   Component      │ │   Component     │
│   ("use client") │ │   (default)     │
│                  │ │                 │
│ • useState       │ │ • async/await   │
│ • useEffect      │ │ • fetch()       │
│ • onClick/onSubmit│ │ • db.query()   │
│ • Browser APIs   │ │ • file I/O     │
│ • Hooks          │ │ • env secrets  │
└─────────────────┘ └─────────────────┘
```

## Server Components ✅

### What You CAN Do
```tsx
// src/app/posts/page.tsx — NO "use client" needed
import { db } from "@/lib/db";

export default async function PostsPage() {
  // ✅ Direct database access
  const posts = await db.query("SELECT * FROM posts");

  // ✅ Fetch from external API
  const res = await fetch("https://api.example.com/comments");
  const comments = await res.json();

  // ✅ Access environment variables (even secrets!)
  const apiKey = process.env.STRIPE_SECRET_KEY;

  // ✅ Read files
  const markdown = await fs.readFile("content.md", "utf-8");

  // ✅ Render a client component inside
  return (
    <div>
      <h1>Posts ({posts.length})</h1>
      {posts.map((post) => (
        <PostCard key={post.id} post={post} />  // ← This can be a Client Component
      ))}
    </div>
  );
}
```

### What You CANNOT Do
```tsx
// ❌ Server Components CANNOT:
// "use client" is NOT present, so these won't work:

// ❌ No state
// const [count, setCount] = useState(0);

// ❌ No effects
// useEffect(() => { ... }, []);

// ❌ No event handlers
// <button onClick={() => setCount(count + 1)}>Click</button>

// ❌ No browser-only APIs
// localStorage.getItem("token");
// window.innerWidth;
// document.querySelector(".header");
```

## Client Components ✅

Add `"use client"` at the top of the file.

```tsx
// src/app/components/Counter.tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div className="p-4 border rounded">
      <p>Count: {count}</p>
      <button
        onClick={() => setCount(count + 1)}
        className="px-4 py-2 bg-blue-600 text-white rounded"
      >
        Increment
      </button>
    </div>
  );
}
```

### When MUST You Use "use client"?

| Scenario | Example |
|----------|---------|
| useState / useReducer | `const [open, setOpen] = useState(false)` |
| useEffect | `useEffect(() => { fetchData() }, [])` |
| Event handlers | `onClick`, `onSubmit`, `onChange` |
| React context | `useContext`, `createContext` providers |
| Custom hooks with state | `useLocalStorage`, `useMediaQuery` |
| Browser-only APIs | `window`, `document`, `localStorage` |
| Animation libraries | Framer Motion, GSAP |
| Third-party UI libs | Material UI, shadcn/ui, Ant Design |

## The "use client" Boundary

When a file has `"use client"`, **all components in that file** AND **all imported components** run on the client:

```tsx
// components/Header.tsx
"use client";
// EVERYTHING in this file is a Client Component

// Even this simple component runs on the client
export function Logo() {
  return <img src="/logo.png" alt="Logo" />;
}

export function NavLink({ href, children }: { href: string; children: React.ReactNode }) {
  return <a href={href}>{children}</a>;
}
```

## Composition Pattern (Best Practice)

**Pass Server Components as children to Client Components:**

```tsx
// ✅ GOOD: Server Component wrapper, Client Component inside

// src/app/layout.tsx (Server Component)
import ClientLayout from "@/components/ClientLayout";
import Sidebar from "@/components/Sidebar";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ClientLayout>       {/* ← client component wrapper */}
          <Sidebar />        {/* ← server component passed as children! */}
          {children}         {/* ← page content (can be server components!) */}
        </ClientLayout>
      </body>
    </html>
  );
}

// components/ClientLayout.tsx
"use client";
import { useState } from "react";

export default function ClientLayout({ children }: { children: React.ReactNode }) {
  const [sidebarOpen, setSidebarOpen] = useState(true);

  return (
    <div className="flex">
      {sidebarOpen && <aside>{children}</aside>}
      {/* sidebarOpen controls the sidebar, but the Sidebar component
          inside children is still a Server Component — best of both worlds! */}
      <button onClick={() => setSidebarOpen(!sidebarOpen)}>Toggle</button>
    </div>
  );
}
```

### Why This Pattern Matters

```tsx
// ❌ BAD: This forces Sidebar to be a Client Component
// components/Dashboard.tsx
"use client";
import Sidebar from "./Sidebar";  // ← Sidebar now runs on client too!

export default function Dashboard() {
  return (
    <div>
      <Sidebar />
    </div>
  );
}

// ✅ GOOD: Sidebar stays a Server Component
// components/Dashboard.tsx
"use client";
export default function Dashboard({ sidebar }: { sidebar: React.ReactNode }) {
  return (
    <div>
      {sidebar}  {/* ← Sidebar is passed in, so it stays server-side */}
    </div>
  );
}

// Using it — sidebar is a server component passed in
// app/page.tsx
import Dashboard from "@/components/Dashboard";
import Sidebar from "@/components/Sidebar";

export default function Page() {
  return <Dashboard sidebar={<Sidebar />} />;
}
```

## Data Flow Between Components

```
Server Component
    │
    │  Props (data, serializable only)
    ▼
Client Component
    │
    │  Props (data, or ReactNode for server children)
    ▼
Another Client Component
```

### Serializable Props Only
Server Components can only pass **serializable** props to Client Components:

```tsx
// ✅ OK — these are serializable
// strings, numbers, booleans, objects, arrays, dates

// ❌ NOT OK — these are NOT serializable
// functions, React components, JSX (unless as children prop)
```

## Loading Patterns

### Approach 1: Fetch on server, pass to client
```tsx
// app/page.tsx (Server Component)
export default async function Page() {
  const data = await fetchData();
  return <InteractiveTable data={data} />;
}

// components/InteractiveTable.tsx (Client Component)
"use client";
export default function InteractiveTable({ data }: { data: Data[] }) {
  const [sortColumn, setSortColumn] = useState("name");
  // ... client-side sorting, filtering, etc.
}
```

### Approach 2: Hybrid — server fetches initial, client fetches more
```tsx
// app/page.tsx
export default async function Page() {
  const initialPosts = await fetch(".../posts?limit=10").then(r => r.json());
  return <PostList initialPosts={initialPosts} />;
}

// components/PostList.tsx
"use client";
import { useState, useEffect } from "react";
import useSWR from "swr";

export default function PostList({ initialPosts }: { initialPosts: Post[] }) {
  const [page, setPage] = useState(1);
  const { data, mutate } = useSWR(`/api/posts?page=${page}`, fetcher, {
    fallbackData: page === 1 ? initialPosts : undefined,
  });

  return <div>...</div>;
}
```

## 🧠 Key Takeaway
- **All components are Server Components by default**
- Add `"use client"` only when you need interactivity (state, effects, events)
- Use the **composition pattern**: pass Server Components as `children` to Client Components
- Keep your data fetching in Server Components, sprinkle interactivity in Client Components
- Less `"use client"` = smaller JS bundle = faster page loads
