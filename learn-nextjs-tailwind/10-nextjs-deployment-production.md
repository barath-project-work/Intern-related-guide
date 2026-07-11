# 10 - Next.js Deployment & Production Best Practices

## Deployment Options

### Option 1: Vercel (Recommended by Next.js Team)
```bash
# 1. Push to GitHub
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/your-repo.git
git push -u origin main

# 2. Go to vercel.com → Import GitHub repo
# 3. Vercel auto-detects Next.js → Zero config
# 4. Set environment variables in Vercel dashboard
```

**Vercel benefits:**
- Automatic preview deployments for every PR
- Edge Functions / ISR / Image Optimization work out of box
- Global CDN (100+ edge locations)
- Analytics, Speed Insights, Logs included

### Option 2: Docker (Self-hosted)
```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci

# Build the app
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"
CMD ["node", "server.js"]
```

### Option 3: Supabase + Railway / Fly.io / AWS
Deploy your Next.js app on Railway or Fly.io, connect to Supabase for database/auth.

## Environment Variables

```bash
# .env.local (local development — never commit this!)
DATABASE_URL=postgresql://localhost:5432/mydb
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx  # secret — never exposed to client

# .env.example (commit this — template for other devs)
DATABASE_URL=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

### Naming Convention
| Prefix | Access |
|--------|--------|
| `NEXT_PUBLIC_*` | Available on client AND server |
| No prefix | Server-side ONLY (never exposed to browser) |

## Performance Optimization

### 1. Image Optimization
```tsx
import Image from "next/image";

// ✅ Always use Next.js Image component
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority          // for above-the-fold images
  className="rounded-lg"
/>

// ✅ For user-uploaded images
<Image
  src={user.avatar_url}
  alt={user.name}
  width={64}
  height={64}
  className="rounded-full"
/>

// Configure remote images in next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "**.supabase.co" },
      { protocol: "https", hostname: "images.unsplash.com" },
    ],
  },
};
```

### 2. Font Optimization
```tsx
// src/app/layout.tsx
import { Inter } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  display: "swap",      // show fallback font immediately
  variable: "--font-inter",  // CSS variable for custom use
});

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

### 3. Metadata & SEO
```tsx
// src/app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: {
    default: "My App",         // fallback title
    template: "%s | My App",   // %s gets replaced by child page title
  },
  description: "Built with Next.js",
  openGraph: {
    title: "My App",
    description: "Built with Next.js",
    images: ["/og-image.png"],
  },
  robots: {
    index: true,
    follow: true,
  },
};

// Per-page metadata (overrides root)
// src/app/blog/[slug]/page.tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.cover_image],
    },
  };
}
```

### 4. Bundle Optimization

```tsx
// ✅ Dynamic imports — split into separate chunk
import dynamic from "next/dynamic";

const Map = dynamic(() => import("@/components/Map"), {
  loading: () => <div className="h-96 bg-gray-100 animate-pulse rounded" />,
  ssr: false,  // if component uses browser APIs
});

// ✅ Lazy load heavy libraries only when needed
const HeavyChart = dynamic(() => import("@/components/Chart"), {
  ssr: false,
});
```

### 5. Streaming & Suspense
```tsx
// ✅ Stream non-critical content
import { Suspense } from "react";

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        <AnalyticsChart />   {/* Loads and streams in */}
      </Suspense>
    </div>
  );
}
```

## Build & Analyze

```bash
# Analyze bundle size
npm install @next/bundle-analyzer

# next.config.ts
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});
module.exports = withBundleAnalyzer({});

# Run analyzer
ANALYZE=true npm run build

# Check build output for warnings
npm run build
# Look for:
# - Large chunks highlighted in terminal
# - Routes using "use client" unnecessarily
# - Large dependencies
```

## Error Monitoring

### Error Boundary
```tsx
// src/app/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  // Log to Sentry or similar
  console.error(error);

  return (
    <div className="flex flex-col items-center justify-center min-h-[50vh] gap-4">
      <h2 className="text-2xl font-bold text-red-600">Something went wrong</h2>
      <p className="text-gray-600">{error.message}</p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  );
}
```

### Global Error (layout-level)
```tsx
// src/app/global-error.tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <div className="text-center py-20">
          <h1 className="text-4xl font-bold">500</h1>
          <p className="mt-2 text-gray-600">Server error</p>
          <button onClick={reset} className="mt-4 px-4 py-2 bg-blue-600 text-white rounded">
            Reload
          </button>
        </div>
      </body>
    </html>
  );
}
```

## Production Checks

### Before Deploy Checklist

- [ ] **Enable RLS** on all Supabase tables
- [ ] **Remove console.log** statements
- [ ] **Add proper metadata** for SEO
- [ ] **Configure CSP headers** (Content Security Policy)
- [ ] **Set up error monitoring** (Sentry, LogRocket, etc.)
- [ ] **Add analytics** (Vercel Analytics, PostHog, Plausible)
- [ ] **Test on mobile/tablet/desktop** responsive breakpoints
- [ ] **Test loading states** (slow network in DevTools)
- [ ] **Test error states** (404, 500, form validation)
- [ ] **Check Lighthouse scores** (Performance, Accessibility, SEO)
- [ ] **Set up environment variables** in production
- [ ] **Configure custom domain** + SSL

### Security Headers (next.config.ts)
```typescript
const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          { key: "X-Frame-Options", value: "DENY" },
          { key: "X-Content-Type-Options", value: "nosniff" },
          { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
          { key: "X-XSS-Protection", value: "1; mode=block" },
        ],
      },
      {
        source: "/api/:path*",
        headers: [
          { key: "Access-Control-Allow-Origin", value: "https://yourapp.com" },
          { key: "Access-Control-Allow-Methods", value: "GET, POST, PUT, DELETE" },
        ],
      },
    ];
  },
};
```

## Common Production Issues & Fixes

### Issue 1: "Module not found" on build
```bash
# Check imports are correct and case-sensitive
# Restart dev server after installing packages
rm -rf .next && npm run build
```

### Issue 2: Hydration mismatch
```tsx
// ❌ Server renders "Hello", client renders "Hi" → error
// Fix: ensure deterministic rendering

// ✅ Good: consistent on both server and client
<div>Hello</div>

// ✅ For dynamic content, use useEffect or suppressHydrationWarning
<div suppressHydrationWarning>{typeof window !== "undefined" ? theme : ""}</div>
```

### Issue 3: Images not showing
```tsx
// ❌ Need to configure remotePatterns in next.config.ts for external images
// ❌ Did you import Image from "next/image", not <img>?

// ✅ Check the URL works in the browser
// ✅ Check remotePatterns config
```

### Issue 4: Large JS bundles
```bash
# Use @next/bundle-analyzer to see what's large
# Move heavy components to dynamic imports
# Minimize "use client" usage
# Check if you're importing the right version (e.g., lodash-es vs lodash)
```

## CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - run: npm ci
      - run: npm run lint
      - run: npm run type-check   # if you have this script
      - run: npm run build
      # - run: npm test           # if you have tests
```

## 🧠 Key Takeaway
- **Vercel** is the recommended deployment platform with zero config
- Environment variables: `NEXT_PUBLIC_*` for client, no prefix for secrets
- Optimize with `<Image>`, dynamic imports, streaming, and fonts
- Always test: mobile responsiveness, loading/error states, Lighthouse
- Set up **CI/CD** early — automate linting, type-checking, and builds
- Security: enable RLS, set security headers, validate input on server
