# 02 - Next.js Routing, Pages & Layouts

## How Routing Works in App Router

In the App Router, **your folder structure IS your URL structure**.

```
src/app/
├── page.tsx           →  /                    (homepage)
├── about/
│   └── page.tsx       →  /about
├── blog/
│   ├── page.tsx       →  /blog
│   └── [slug]/
│       └── page.tsx   →  /blog/hello-world  (dynamic route)
└── dashboard/
    ├── layout.tsx     →  wraps all dashboard pages
    ├── page.tsx       →  /dashboard
    └── settings/
        └── page.tsx   →  /dashboard/settings
```

## Basic Page Component

```tsx
// src/app/page.tsx
export default function HomePage() {
  return (
    <main>
      <h1>Welcome to my app</h1>
    </main>
  );
}
```

## Layout System

**Layouts** wrap pages and preserve state across navigations (no full page reload).

### Root Layout (`src/app/layout.tsx`)
```tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "My App",
  description: "Built with Next.js",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className="antialiased">{children}</body>
    </html>
  );
}
```

### Nested Layout Example
```tsx
// src/app/dashboard/layout.tsx
import Sidebar from "@/components/Sidebar";

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex min-h-screen">
      <Sidebar />
      <main className="flex-1 p-6">{children}</main>
    </div>
  );
}
```

### Layout Rules
- ✅ You can nest layouts infinitely
- ✅ Layouts **preserve state** across navigations (unlike pages)
- ❌ Layouts do **NOT** receive `searchParams` (only pages do)
- ❌ Do NOT fetch data in layouts unless it's truly shared (it blocks navigation)

## Dynamic Routes

```tsx
// src/app/blog/[slug]/page.tsx
interface Props {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

export default async function BlogPostPage({ params, searchParams }: Props) {
  const { slug } = await params;
  const { preview } = await searchParams;

  const post = await getPost(slug);

  return (
    <article>
      <h1>{post.title}</h1>
      {preview && <p className="text-yellow-600">🔍 Preview mode</p>}
      <div>{post.content}</div>
    </article>
  );
}
```

### Dynamic Route Patterns

| Folder Pattern | Matches | Example URL |
|---------------|---------|-------------|
| `[slug]` | Single segment | `/blog/my-post` |
| `[...slug]` | Catch-all segments | `/blog/2024/jan/hello` |
| `[[...slug]]` | Optional catch-all | `/blog`, `/blog/a`, `/blog/a/b` |
| `(group)` | Route group (no URL change) | groups related routes |
| `_folder` | Private folder (excluded from routing) | — |

## Route Groups

Group pages without affecting the URL. Use `(folderName)`.

```
src/app/
├── (marketing)/
│   ├── layout.tsx      ← marketing layout
│   ├── page.tsx        →  /
│   └── pricing/
│       └── page.tsx    →  /pricing
├── (dashboard)/
│   ├── layout.tsx      ← dashboard layout (different header)
│   └── dashboard/
│       └── page.tsx    →  /dashboard
```

## Loading & Error States

### Loading UI
```tsx
// src/app/blog/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse space-y-4">
      <div className="h-8 w-64 bg-gray-200 rounded" />
      <div className="h-4 w-full bg-gray-200 rounded" />
      <div className="h-4 w-3/4 bg-gray-200 rounded" />
    </div>
  );
}
```

### Error UI
```tsx
// src/app/blog/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="text-center py-20">
      <h2 className="text-2xl font-bold text-red-600">Something went wrong!</h2>
      <button
        onClick={reset}
        className="mt-4 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  );
}
```

### Not Found UI
```tsx
// src/app/not-found.tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <div className="text-center py-20">
      <h2 className="text-4xl font-bold text-gray-800">404</h2>
      <p className="mt-2 text-gray-600">Page not found</p>
      <Link
        href="/"
        className="mt-4 inline-block px-4 py-2 bg-blue-600 text-white rounded"
      >
        Go home
      </Link>
    </div>
  );
}
```

## Linking Between Pages

### `Link` Component (client-side navigation)
```tsx
import Link from "next/link";

export default function Navigation() {
  return (
    <nav className="flex gap-4">
      <Link href="/" className="hover:underline">Home</Link>
      <Link href="/about">About</Link>
      <Link href={`/blog/${post.slug}`}>{post.title}</Link>

      {/* Replace current history entry */}
      <Link href="/dashboard" replace>Dashboard</Link>

      {/* Prefetch on hover (default for visible links) */}
      <Link href="/pricing" prefetch={true}>Pricing</Link>
    </nav>
  );
}
```

### `useRouter` Hook (programmatic navigation)
```tsx
"use client";
import { useRouter } from "next/navigation";

export default function SubmitButton() {
  const router = useRouter();

  async function handleSubmit() {
    await saveData();
    router.push("/success");   // navigate to /success
    router.refresh();          // re-fetch current route data
    router.back();             // go back in history
    router.forward();          // go forward
    router.replace("/login");  // replace history (no back button)
  }

  return <button onClick={handleSubmit}>Submit</button>;
}
```

## Parallel Routes (Advanced)

Render multiple pages in the same view — great for dashboards:

```tsx
// src/app/dashboard/layout.tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div className="grid grid-cols-2 gap-4">
      {children}       {/* main dashboard content */}
      {team}           {/* @team/page.tsx */}
      {analytics}      {/* @analytics/page.tsx */}
    </div>
  );
}
```

With folder structure:
```
src/app/dashboard/
├── layout.tsx
├── page.tsx
├── @team/
│   └── page.tsx
└── @analytics/
    └── page.tsx
```

## 🧠 Key Takeaway
- **Folders = routes**, files = UI for those routes
- Use **layouts** for shared UI (navbars, sidebars)
- **Dynamic routes** use `[param]` syntax
- **Loading, error, and not-found** files handle edge cases gracefully
- `Link` component for navigation, `useRouter()` for programmatic navigation
