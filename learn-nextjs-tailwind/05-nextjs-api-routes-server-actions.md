# 05 - Next.js API Routes & Server Actions

## Two Ways to Build APIs in Next.js

| Feature | API Routes (`route.ts`) | Server Actions |
|---------|------------------------|----------------|
| Purpose | RESTful API endpoints | Form submissions & mutations |
| Called via | `fetch("/api/...")` | `action={handler}` or `startTransition` |
| Type-safe | Manual validation | Automatic (form data) |
| Best for | Public APIs, webhooks, mobile apps | Server mutations, forms |

## API Routes with Route Handlers

Create a `route.ts` file inside the `app` directory:

```
src/app/
└── api/
    └── posts/
        ├── route.ts       →  GET, POST /api/posts
        └── [id]/
            └── route.ts   →  GET, PUT, DELETE /api/posts/123
```

### Basic CRUD Example

```tsx
// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from "next/server";

// GET /api/posts
export async function GET(request: NextRequest) {
  // Access query params
  const { searchParams } = new URL(request.url);
  const limit = searchParams.get("limit") ?? "10";
  const page = searchParams.get("page") ?? "1";

  const posts = await db.query(
    "SELECT * FROM posts ORDER BY created_at DESC LIMIT $1 OFFSET $2",
    [limit, (Number(page) - 1) * Number(limit)]
  );

  return NextResponse.json({ posts, page: Number(page) });
}

// POST /api/posts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { title, content } = body;

    // Validation
    if (!title || typeof title !== "string") {
      return NextResponse.json(
        { error: "Title is required" },
        { status: 400 }
      );
    }

    const post = await db.query(
      "INSERT INTO posts (title, content) VALUES ($1, $2) RETURNING *",
      [title, content]
    );

    return NextResponse.json(post[0], { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

### Dynamic Route Handler

```tsx
// src/app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

// To access params, the function receives a context object
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;  // params is a Promise — must await

  const post = await db.query("SELECT * FROM posts WHERE id = $1", [id]);

  if (!post.length) {
    return NextResponse.json({ error: "Post not found" }, { status: 404 });
  }

  return NextResponse.json(post[0]);
}

// PUT /api/posts/123
export async function PUT(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const body = await request.json();

  const updated = await db.query(
    "UPDATE posts SET title = $1, content = $2 WHERE id = $3 RETURNING *",
    [body.title, body.content, id]
  );

  return NextResponse.json(updated[0]);
}

// DELETE /api/posts/123
export async function DELETE(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  await db.query("DELETE FROM posts WHERE id = $1", [id]);
  return NextResponse.json({ success: true });
}
```

### Response Helpers

```tsx
import { NextResponse } from "next/server";

// JSON responses
NextResponse.json({ data: "hello" });

// With status code
NextResponse.json({ error: "Not found" }, { status: 404 });

// Redirect
NextResponse.redirect(new URL("/login", request.url));

// Rewrite (show different content at same URL)
NextResponse.rewrite(new URL("/fallback", request.url));
```

### Middleware (run before every request)

```tsx
// src/middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const session = request.cookies.get("session");

  // Protect dashboard routes
  if (request.nextUrl.pathname.startsWith("/dashboard")) {
    if (!session) {
      return NextResponse.redirect(new URL("/login", request.url));
    }
  }

  // Add custom headers
  const response = NextResponse.next();
  response.headers.set("X-Custom-Header", "hello");

  return response;
}

// Only run on specific paths
export const config = {
  matcher: [
    "/dashboard/:path*",
    "/api/:path*",
    "/((?!_next/static|_next/image|favicon.ico).*)",
  ],
};
```

## Server Actions (Modern Way)

Server Actions are **async functions** that run on the server but can be called from the client.

### Basic Server Action

```tsx
// src/app/posts/page.tsx
async function addPost(formData: FormData) {
  "use server";  // ← Directive that makes it a Server Action

  const title = formData.get("title") as string;
  const content = formData.get("content") as string;

  await db.query(
    "INSERT INTO posts (title, content) VALUES ($1, $2)",
    [title, content]
  );

  revalidatePath("/posts");  // Re-fetch the posts page
}

export default function PostsPage() {
  return (
    <form action={addPost}>  {/* ← call server action directly */}
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Add Post</button>
    </form>
  );
}
```

### Server Action in a Separate File

```tsx
// src/app/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { z } from "zod";

const PostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(10),
});

export async function createPost(formData: FormData) {
  const validated = PostSchema.safeParse({
    title: formData.get("title"),
    content: formData.get("content"),
  });

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors };
  }

  const post = await db.insert(validated.data);
  revalidatePath("/posts");
  redirect(`/posts/${post.id}`);
}

export async function deletePost(id: string) {
  await db.delete(id);
  revalidatePath("/posts");
}
```

### Using Actions from Client Components

```tsx
"use client";

import { createPost } from "@/app/actions";
import { useActionState } from "react";

export default function PostForm() {
  const [state, formAction, isPending] = useActionState(createPost, null);

  return (
    <form action={formAction}>
      {state?.errors?.title && (
        <p className="text-red-500">{state.errors.title[0]}</p>
      )}
      <input name="title" placeholder="Title" required />
      {state?.errors?.content && (
        <p className="text-red-500">{state.errors.content[0]}</p>
      )}
      <textarea name="content" placeholder="Content" />
      <button type="submit" disabled={isPending}>
        {isPending ? "Creating..." : "Create Post"}
      </button>
    </form>
  );
}
```

### Without a Form (using startTransition)

```tsx
"use client";

import { deletePost } from "@/app/actions";
import { useTransition } from "react";

export default function DeleteButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      onClick={() => startTransition(() => deletePost(postId))}
      disabled={isPending}
      className="px-3 py-1 bg-red-500 text-white rounded disabled:opacity-50"
    >
      {isPending ? "Deleting..." : "Delete"}
    </button>
  );
}
```

## When to Use Which?

```
┌──────────────────────────────────────────────────────┐
│                  Which one to use?                    │
├────────────────────┬─────────────────────────────────┤
│  Use API Routes    │  Use Server Actions              │
├────────────────────┼─────────────────────────────────┤
│ • Public REST API  │ • Form submissions               │
│ • Mobile app API   │ • Button clicks / mutations      │
│ • Webhook handlers │ • Revalidating cached data       │
│ • Third-party      │ • Server-side validation         │
│   integrations     │ • Database mutations             │
│ • CORS endpoints   │ • Within Server Components       │
└────────────────────┴─────────────────────────────────┘
```

## 🧠 Key Takeaway
- **API Routes** (`route.ts`) = traditional REST endpoints
- **Server Actions** = modern way for mutations & form handling
- Server Actions with `useActionState` give you loading states + validation
- Server Actions auto-revalidate with `revalidatePath()` / `revalidateTag()`
- Use **middleware.ts** for auth checks, redirects, and header manipulation
