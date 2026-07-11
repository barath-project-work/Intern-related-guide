# 09 - Next.js with Supabase Integration

## What is Supabase?

**Supabase** is an open-source Firebase alternative that provides:
- **PostgreSQL database** (with Row Level Security)
- **Authentication** (email, social, magic link)
- **Realtime subscriptions** (websockets)
- **Storage** (file uploads)
- **Edge Functions** (serverless TypeScript)

## Setup

### 1. Create a Supabase Project
Go to [supabase.com](https://supabase.com) → New Project → Copy the URL and anon key.

### 2. Install the Client
```bash
npm install @supabase/supabase-js @supabase/ssr
```

### 3. Environment Variables
```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key  # server-only (secret!)
```

### 4. Create the Client

```typescript
// src/lib/supabase/client.ts
import { createBrowserClient } from "@supabase/ssr";

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

```typescript
// src/lib/supabase/server.ts
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

export async function createClient() {
  const cookieStore = await cookies();

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    }
  );
}
```

## Authentication

### Sign Up
```tsx
// src/app/auth/signup/page.tsx
"use client";
import { createClient } from "@/lib/supabase/client";
import { useState } from "react";

export default function SignUpPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState<string | null>(null);
  const supabase = createClient();

  async function handleSignUp(e: React.FormEvent) {
    e.preventDefault();
    setError(null);

    const { error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        emailRedirectTo: `${location.origin}/auth/callback`,
      },
    });

    if (error) {
      setError(error.message);
    } else {
      alert("Check your email for the confirmation link!");
    }
  }

  return (
    <form onSubmit={handleSignUp} className="max-w-md mx-auto space-y-4">
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        className="w-full px-4 py-2 border rounded"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        className="w-full px-4 py-2 border rounded"
        required
      />
      {error && <p className="text-red-600 text-sm">{error}</p>}
      <button type="submit" className="w-full bg-blue-600 text-white py-2 rounded">
        Sign Up
      </button>
    </form>
  );
}
```

### Sign In (with Google OAuth)
```tsx
async function handleGoogleSignIn() {
  const supabase = createClient();
  const { error } = await supabase.auth.signInWithOAuth({
    provider: "google",
    options: {
      redirectTo: `${location.origin}/auth/callback`,
    },
  });
}
```

### Auth Callback Route
```typescript
// src/app/auth/callback/route.ts
import { createClient } from "@/lib/supabase/server";
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get("code");
  const next = searchParams.get("next") ?? "/dashboard";

  if (code) {
    const supabase = await createClient();
    const { error } = await supabase.auth.exchangeCodeForSession(code);
    if (!error) {
      return NextResponse.redirect(`${origin}${next}`);
    }
  }

  return NextResponse.redirect(`${origin}/auth-error`);
}
```

### Protected Middleware
```typescript
// src/middleware.ts
import { type NextRequest } from "next/server";
import { updateSession } from "@/lib/supabase/middleware";

export async function middleware(request: NextRequest) {
  return await updateSession(request);
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)",
  ],
};
```

### Getting the Current User
```tsx
// In a Server Component
import { createClient } from "@/lib/supabase/server";

export default async function DashboardPage() {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return <div>Please sign in</div>;
  }

  return <div>Welcome, {user.email}!</div>;
}
```

### Sign Out
```tsx
"use client";
import { createClient } from "@/lib/supabase/client";
import { useRouter } from "next/navigation";

export default function SignOutButton() {
  const supabase = createClient();
  const router = useRouter();

  async function handleSignOut() {
    await supabase.auth.signOut();
    router.push("/");
    router.refresh();
  }

  return (
    <button onClick={handleSignOut} className="text-red-600 hover:underline">
      Sign Out
    </button>
  );
}
```

## Database Queries

### Database Schema (SQL)
```sql
-- Create a table via Supabase SQL Editor
CREATE TABLE posts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  published BOOLEAN DEFAULT false
);

-- Enable Row Level Security
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policies (who can do what)
CREATE POLICY "Users can view published posts"
  ON posts FOR SELECT
  USING (published = true OR user_id = auth.uid());

CREATE POLICY "Users can insert their own posts"
  ON posts FOR INSERT
  WITH CHECK (user_id = auth.uid());

CREATE POLICY "Users can update their own posts"
  ON posts FOR UPDATE
  USING (user_id = auth.uid());
```

### Fetching Data (Server Component)
```tsx
import { createClient } from "@/lib/supabase/server";

export default async function PostsPage() {
  const supabase = await createClient();

  const { data: posts, error } = await supabase
    .from("posts")
    .select("*, profiles(username, avatar_url)")
    .eq("published", true)
    .order("created_at", { ascending: false })
    .limit(20);

  if (error) {
    return <div className="text-red-500">Error: {error.message}</div>;
  }

  return (
    <div className="grid gap-6 grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
      {posts.map((post) => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

### Mutating Data (Server Action)
```tsx
// src/app/actions/posts.ts
"use server";

import { createClient } from "@/lib/supabase/server";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { z } from "zod";

const PostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10),
});

export async function createPost(formData: FormData) {
  const supabase = await createClient();

  // Check auth
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) throw new Error("Not authenticated");

  // Validate
  const validated = PostSchema.safeParse({
    title: formData.get("title"),
    content: formData.get("content"),
  });
  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors };
  }

  // Insert
  const { error } = await supabase.from("posts").insert({
    ...validated.data,
    user_id: user.id,
  });

  if (error) return { error: error.message };

  revalidatePath("/posts");
  redirect("/posts");
}
```

### Client-Side Mutation with Optimistic UI
```tsx
"use client";
import { createClient } from "@/lib/supabase/client";

export function LikeButton({ postId, initialLikes }: { postId: string; initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes);
  const [isLiked, setIsLiked] = useState(false);
  const supabase = createClient();

  async function handleLike() {
    // Optimistic update
    setLikes((prev) => prev + 1);
    setIsLiked(true);

    // Actual DB call
    const { error } = await supabase
      .from("likes")
      .insert({ post_id: postId });

    if (error) {
      // Rollback on error
      setLikes((prev) => prev - 1);
      setIsLiked(false);
    }
  }

  return (
    <button onClick={handleLike} className="flex items-center gap-2">
      {isLiked ? "❤️" : "🤍"} {likes}
    </button>
  );
}
```

## Realtime Subscriptions

```tsx
"use client";
import { createClient } from "@/lib/supabase/client";
import { useEffect, useState } from "react";

export function RealtimePosts() {
  const [posts, setPosts] = useState<Post[]>([]);
  const supabase = createClient();

  useEffect(() => {
    // Fetch initial data
    supabase
      .from("posts")
      .select("*")
      .order("created_at", { ascending: false })
      .then(({ data }) => {
        if (data) setPosts(data);
      });

    // Subscribe to changes
    const channel = supabase
      .channel("posts-channel")
      .on(
        "postgres_changes",
        { event: "INSERT", schema: "public", table: "posts" },
        (payload) => {
          setPosts((prev) => [payload.new as Post, ...prev]);
        }
      )
      .on(
        "postgres_changes",
        { event: "DELETE", schema: "public", table: "posts" },
        (payload) => {
          setPosts((prev) => prev.filter((p) => p.id !== payload.old.id));
        }
      )
      .subscribe();

    // Cleanup on unmount
    return () => {
      supabase.removeChannel(channel);
    };
  }, []);

  return (
    <div>
      {posts.map((post) => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

## Storage (File Uploads)

### Upload
```typescript
async function uploadAvatar(file: File, userId: string) {
  const supabase = createClient();

  const fileExt = file.name.split(".").pop();
  const filePath = `avatars/${userId}.${fileExt}`;

  const { error } = await supabase.storage
    .from("avatars")
    .upload(filePath, file, { upsert: true });

  if (error) throw error;

  // Get public URL
  const { data: { publicUrl } } = supabase.storage
    .from("avatars")
    .getPublicUrl(filePath);

  return publicUrl;
}
```

### File Upload UI
```tsx
"use client";
import { useState } from "react";

export function FileUpload() {
  const [uploading, setUploading] = useState(false);
  const [url, setUrl] = useState<string | null>(null);

  async function handleUpload(e: React.ChangeEvent<HTMLInputElement>) {
    const file = e.target.files?.[0];
    if (!file) return;

    setUploading(true);
    const publicUrl = await uploadAvatar(file, userId);
    setUrl(publicUrl);
    setUploading(false);
  }

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={handleUpload}
        disabled={uploading}
        className="block"
      />
      {uploading && <p>Uploading...</p>}
      {url && <img src={url} alt="Avatar" className="w-20 h-20 rounded-full" />}
    </div>
  );
}
```

## Row Level Security (RLS) Best Practices

RLS is **the most important Supabase concept** for security.

```sql
-- User can only see their own data
CREATE POLICY "Users can view own todos"
  ON todos FOR SELECT
  USING (auth.uid() = user_id);

-- But admins can see everything
CREATE POLICY "Admins can view all"
  ON todos FOR SELECT
  USING (
    auth.uid() IN (
      SELECT user_id FROM user_roles WHERE role = 'admin'
    )
  );

-- User can insert todos with their own user_id
CREATE POLICY "Users can create todos"
  ON todos FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- User can only update their own completed todos
CREATE POLICY "Users can complete their todos"
  ON todos FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id AND NEW.completed = true);
```

> ⚠️ **Always enable RLS on every table!** Without it, anyone with your anon key can read/write everything.

## 🧠 Key Takeaway
- Use **`@supabase/ssr`** package for Next.js App Router compatibility
- **Server Components** → use `createClient()` from `server.ts`
- **Client Components** → use `createClient()` from `client.ts`
- **RLS (Row Level Security)** is your primary security layer — always enable it
- Use **Server Actions** for data mutations (insert, update, delete)
- **Realtime** subscriptions enable live updates without polling
- **Storage** for file uploads with public/private buckets
