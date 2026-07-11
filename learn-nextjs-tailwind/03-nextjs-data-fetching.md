# 03 - Next.js Data Fetching Patterns

## The Shift in Thinking

**Old way (Pages Router):**
```tsx
export async function getServerSideProps() { ... }
export async function getStaticProps() { ... }
```

**✅ New way (App Router):**
```tsx
// Just fetch directly in your Server Component — it's async!
export default async function Page() {
  const data = await fetch('https://api.example.com/data');
  const posts = await data.json();
  return <div>...</div>;
}
```

## Server Component Data Fetching

Server Components can use `async/await` directly.

### Fetching on Every Request (SSR equivalent)
```tsx
// src/app/dashboard/page.tsx
export default async function DashboardPage() {
  // Data is fetched on every request (no cache by default)
  const res = await fetch("https://api.example.com/stats");
  const stats = await res.json();

  // OR use a database client directly (server-side only)
  const recentOrders = await db.query("SELECT * FROM orders LIMIT 10");

  return (
    <div>
      <h1>Dashboard</h1>
      <pre>{JSON.stringify(stats, null, 2)}</pre>
    </div>
  );
}
```

### Caching & Revalidation

```tsx
// 1. STATIC - fetched once at build time (default for no options)
const res = await fetch("https://api.example.com/posts");
// → cached forever (like old getStaticProps)

// 2. TIME-BASED REVALIDATION - re-fetch every 60 seconds
const res = await fetch("https://api.example.com/posts", {
  next: { revalidate: 60 },  // ISR — Incremental Static Regeneration
});

// 3. ON-DEMAND REVALIDATION - re-fetch when you trigger it
const res = await fetch("https://api.example.com/posts", {
  next: { tags: ["posts"] },  // tag it!
});
// Then call: revalidateTag("posts") from a Server Action or API route

// 4. NO CACHE - always fresh
const res = await fetch("https://api.example.com/user", {
  cache: "no-store",  // don't cache at all
});
```

### Request Memoization (Automatic!)
Next.js **deduplicates** the same `fetch` call within a single request:

```tsx
// Even if you call it 3 times, it only fetches ONCE
async function getPosts() {
  const res = await fetch("https://api.example.com/posts");
  return res.json();
}

export default async function Page() {
  const posts = await getPosts();                  // 1st call → actual fetch
  const morePosts = await getPosts();              // 2nd call → returns cached
  const { data } = await getPosts();               // 3rd call → returns cached
  return <div>...</div>;
}
```

## Parallel vs Sequential Data Fetching

### ✅ Parallel (faster — do this)
```tsx
export default async function Page() {
  // Start both requests at the same time
  const postsPromise = fetch("https://api.example.com/posts");
  const usersPromise = fetch("https://api.example.com/users");

  // Wait for both
  const [postsRes, usersRes] = await Promise.all([
    postsPromise,
    usersPromise,
  ]);

  const [posts, users] = await Promise.all([
    postsRes.json(),
    usersRes.json(),
  ]);

  return <div>...</div>;
}
```

### ❌ Sequential (slower — avoid if independent)
```tsx
export default async function Page() {
  const postsRes = await fetch("https://api.example.com/posts");
  const posts = await postsRes.json();
  // ↑ this must finish before the next line starts ↓

  const usersRes = await fetch("https://api.example.com/users");
  const users = await usersRes.json();

  return <div>...</div>;
}
```

### Sequential When You HAVE TO (data depends on previous)
```tsx
export default async function Page({ params }: { params: { id: string } }) {
  const userRes = await fetch(`https://api.example.com/users/${params.id}`);
  const user = await userRes.json();

  // Now fetch user's posts — depends on userId
  const postsRes = await fetch(`https://api.example.com/users/${user.id}/posts`);
  const posts = await postsRes.json();

  return <div>...</div>;
}
```

## Client-Side Data Fetching (useEffect / SWR / TanStack Query)

Use a **data-fetching library** instead of raw `useEffect`:

### Option 1: SWR (recommended by Vercel)
```tsx
"use client";
import useSWR from "swr";

const fetcher = (url: string) => fetch(url).then((r) => r.json());

export default function PostsList() {
  const { data, error, isLoading, mutate } = useSWR("/api/posts", fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading posts</div>;

  return (
    <div>
      <button onClick={() => mutate()}>Refresh</button>
      {data.map((post) => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

### Option 2: TanStack Query (React Query)
```tsx
"use client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

export default function PostsList() {
  const queryClient = useQueryClient();

  const { data, isLoading } = useQuery({
    queryKey: ["posts"],
    queryFn: () => fetch("/api/posts").then((r) => r.json()),
  });

  const mutation = useMutation({
    mutationFn: (newPost) =>
      fetch("/api/posts", {
        method: "POST",
        body: JSON.stringify(newPost),
      }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });

  return <div>...</div>;
}
```

### Option 3: Simple useEffect (for simple cases)
```tsx
"use client";
import { useState, useEffect } from "react";

export default function Profile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/me")
      .then((r) => r.json())
      .then(setUser)
      .finally(() => setLoading(false));
  }, []); // empty deps = runs once on mount

  if (loading) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

## Server Actions (Form Handling)

```tsx
// src/app/posts/create/page.tsx
async function createPost(formData: FormData) {
  "use server";  // ← THIS makes it a Server Action

  const title = formData.get("title");
  const content = formData.get("content");

  // Server-side validation
  if (!title || typeof title !== "string") {
    throw new Error("Title is required");
  }

  // Save to database
  await db.insert({ title, content });

  // Revalidate the posts list
  revalidatePath("/posts");
  redirect("/posts");
}

export default function CreatePostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Post title" required />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### With Server-Side Validation (Zod)
```tsx
import { z } from "zod";

const PostSchema = z.object({
  title: z.string().min(1, "Title is required").max(100),
  content: z.string().min(10, "Content must be at least 10 chars"),
});

async function createPost(formData: FormData) {
  "use server";

  const validated = PostSchema.safeParse({
    title: formData.get("title"),
    content: formData.get("content"),
  });

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors };
  }

  await db.insert({ ...validated.data });
  revalidatePath("/posts");
  redirect("/posts");
}
```

## Streaming with Suspense

Stream parts of the page as they load:

```tsx
import { Suspense } from "react";

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Each section streams in independently! */}
      <Suspense fallback={<div>Loading posts...</div>}>
        <PostsSection />
      </Suspense>

      <Suspense fallback={<div>Loading analytics...</div>}>
        <AnalyticsSection />
      </Suspense>
    </div>
  );
}

// This component is async — it fetches its own data
async function PostsSection() {
  const posts = await fetch("https://api.example.com/posts").then(r => r.json());
  return <div>{posts.map(p => <div key={p.id}>{p.title}</div>)}</div>;
}

async function AnalyticsSection() {
  const stats = await fetch("https://api.example.com/analytics").then(r => r.json());
  return <div>{stats.visitors} visitors</div>;
}
```

## 🧠 Key Takeaway
- **Server Components** = fetch data directly with `async/await`
- Use `cache: "no-store"` for fresh data, `next: { revalidate }` for timed updates
- **Parallel fetching** with `Promise.all()` is faster
- Use **SWR or TanStack Query** for client-side fetching with caching
- **Server Actions** replace API routes for form submissions
- **Suspense** enables streaming UI (parts load independently)
