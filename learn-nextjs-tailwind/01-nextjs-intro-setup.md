# 01 - Next.js Introduction & Project Setup

## What is Next.js?

**Next.js** is a React framework for building full-stack web applications. It provides:

- **Server-Side Rendering (SSR)** & **Static Site Generation (SSG)**
- **File-based routing** (App Router & Pages Router)
- **API Routes** — build your backend in the same project
- **Server Components** — render logic on the server by default
- **Automatic code splitting** & optimized bundling
- **Built-in CSS/Sass/PostCSS** support (including Tailwind CSS)

## App Router vs Pages Router

| Feature | Pages Router (legacy) | App Router (current ✅) |
|---------|-----------------------|------------------------|
| Folder | `pages/` | `app/` |
| Components | Client by default | Server by default |
| Layouts | Manual (shared via `_app.tsx`) | Built-in `layout.js` |
| Data Fetching | `getServerSideProps`, `getStaticProps` | `fetch()` in Server Components |
| React Version | React 17/18 | React 18+ with RSC |

> **🚀 For your role, learn the App Router — that's what modern Next.js uses.**

## Creating a New Next.js Project

```bash
# Recommended: create with Tailwind CSS included
npx create-next-app@latest my-app --typescript --tailwind --app --src-dir

# If you want ESLint too
npx create-next-app@latest my-app --typescript --tailwind --app --src-dir --eslint

# Answer the prompts:
# ✔ Would you like to use TypeScript? → Yes
# ✔ Would you like to use ESLint? → Yes
# ✔ Would you like to use Tailwind CSS? → Yes
# ✔ Would you like to use `src/` directory? → Yes
# ✔ Would you like to use App Router? → Yes
# ✔ Would you like to customize the default import alias (@/*)? → Yes
```

## Project Structure

```
my-app/
├── src/
│   ├── app/               ← App Router pages
│   │   ├── layout.tsx     ← Root layout (required)
│   │   ├── page.tsx       ← Home page (/)
│   │   ├── globals.css    ← Global CSS
│   │   └── ...
│   └── lib/               ← Shared utilities
├── public/                ← Static assets
├── tailwind.config.ts     ← Tailwind configuration
├── next.config.ts         ← Next.js configuration
├── package.json
└── tsconfig.json
```

## Key Configuration Files

### `next.config.ts`
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "images.unsplash.com" },
      { protocol: "https", hostname: "*.supabase.co" },  // for Supabase images
    ],
  },
  // Enable React strict mode (recommended)
  reactStrictMode: true,
};

export default nextConfig;
```

### `tailwind.config.ts`
```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",  // scan all files in src/
    "./pages/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        brand: "#3b82f6",    // custom brand color
        "brand-dark": "#2563eb",
      },
    },
  },
  plugins: [],
};

export default config;
```

## Running the Project

```bash
cd my-app

# development server (with hot reload)
npm run dev

# build for production
npm run build

# start production server
npm start

# run linter
npm run lint
```

## Important Next.js Conventions

### File-based Routing (App Router)

| File | Purpose |
|------|---------|
| `page.tsx` | The public page content |
| `layout.tsx` | Shared layout wrapping all child pages |
| `loading.tsx` | Loading UI (suspense boundary) |
| `error.tsx` | Error UI (error boundary) |
| `not-found.tsx` | 404 page |
| `route.ts` | API route |

### Naming Conventions
- Files are **`page.tsx`**, **`layout.tsx`**, etc. — these are **reserved filenames**
- Components go in a separate `components/` folder
- Use `kebab-case` for folders: `user-profile/`
- Use `PascalCase` for components: `UserProfile.tsx`

## 🧠 Key Takeaway
- Next.js gives you the **power of React + a built-in backend**
- The **App Router** is the modern way — learn it
- **TypeScript** + **Tailwind CSS** comes pre-configured
- File-based routing means **your folder structure = your routes**
