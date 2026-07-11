# 06 - Tailwind CSS Fundamentals & Setup

## What is Tailwind CSS?

**Tailwind CSS** is a **utility-first** CSS framework. Instead of writing custom CSS, you compose pre-built utility classes directly in your HTML/JSX.

### The Old Way vs Tailwind Way

```css
/* ❌ Traditional CSS */
.card {
  background-color: white;
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.card-title {
  font-size: 20px;
  font-weight: 700;
  color: #1a1a1a;
}
```

```tsx
// ✅ Tailwind Way — everything inline
<div className="bg-white rounded-lg p-6 shadow-md">
  <h2 className="text-xl font-bold text-gray-900">Card Title</h2>
</div>
```

**No context switching between HTML and CSS files!**

## Setting Up Tailwind with Next.js

Tailwind comes pre-configured when you create a project with `--tailwind`:

```bash
npx create-next-app@latest my-app --typescript --tailwind --app --src-dir
```

### What Gets Created

**`src/app/globals.css`**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

**`tailwind.config.ts`**
```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
export default config;
```

## The Core Concepts

### 1. Utility-First Philosophy

Each utility class does ONE thing:

| Class | What it does |
|-------|-------------|
| `p-4` | padding: 1rem (16px) |
| `m-2` | margin: 0.5rem (8px) |
| `flex` | display: flex |
| `text-center` | text-align: center |
| `bg-blue-500` | background-color: #3b82f6 |
| `rounded-lg` | border-radius: 0.5rem (8px) |
| `shadow-md` | box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1) |

### 2. Responsive Design

Use **breakpoint prefixes** to apply styles at different screen sizes:

```tsx
<div className="
  w-full          /* mobile: full width */
  md:w-1/2        /* tablet+: half width */
  lg:w-1/3        /* desktop+: third width */
  xl:w-1/4        /* large desktop+: quarter width */
">
```

Default breakpoints:

| Prefix | Min Width | Device |
|--------|-----------|--------|
| `sm:` | 640px | Large phone |
| `md:` | 768px | Tablet |
| `lg:` | 1024px | Laptop |
| `xl:` | 1280px | Desktop |
| `2xl:` | 1536px | Large desktop |

### 3. Hover, Focus & Other States

```tsx
<button className="
  bg-blue-500        /* normal state */
  hover:bg-blue-600  /* hover state */
  focus:ring-2       /* focus state */
  focus:ring-blue-300
  active:bg-blue-700 /* on click */
  disabled:opacity-50 disabled:cursor-not-allowed
">
  Click me
</button>
```

### 4. Dark Mode

Enable dark mode in `tailwind.config.ts`:

```typescript
const config: Config = {
  darkMode: "class",  // or "media" for system preference
  // ...
};
```

Then use `dark:` variants:

```tsx
<div className="
  bg-white            /* light mode */
  dark:bg-gray-800    /* dark mode */
  text-gray-900       /* light text */
  dark:text-white     /* dark text */
">
  <h1 className="text-2xl font-bold">Hello</h1>
</div>
```

## The Spacing Scale

Tailwind uses a **consistent 4px scale** for spacing:

| Class | Value | Pixels |
|-------|-------|--------|
| `p-0` | 0 | 0 |
| `p-1` | 0.25rem | 4px |
| `p-2` | 0.5rem | 8px |
| `p-3` | 0.75rem | 12px |
| `p-4` | 1rem | 16px |
| `p-5` | 1.25rem | 20px |
| `p-6` | 1.5rem | 24px |
| `p-8` | 2rem | 32px |
| `p-10` | 2.5rem | 40px |
| `p-12` | 3rem | 48px |
| `p-16` | 4rem | 64px |
| `p-20` | 5rem | 80px |
| `p-24` | 6rem | 96px |

> 💡 Apply to any side: `pt-` (top), `pr-` (right), `pb-` (bottom), `pl-` (left), `px-` (x-axis), `py-` (y-axis)

## The Color System

Tailwind has a well-designed color palette. The number = intensity (50–950):

```
blue-50   → lightest    #eff6ff
blue-100  → lighter     #dbeafe
blue-200  → light       #bfdbfe
blue-300  → ...         #93c5fd
blue-400                #60a5fa
blue-500  → base        #3b82f6
blue-600                #2563eb
blue-700                #1d4ed8
blue-800                #1e40af
blue-900  → dark        #1e3a8a
blue-950  → darkest     #172554
```

Available colors: `slate`, `gray`, `zinc`, `neutral`, `stone`, `red`, `orange`, `amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`, `violet`, `purple`, `fuchsia`, `pink`, `rose`

## Typography

```tsx
<h1 className="text-4xl font-bold tracking-tight">Heading 1</h1>
<h2 className="text-3xl font-semibold">Heading 2</h2>
<h3 className="text-2xl font-medium">Heading 3</h3>
<p className="text-base leading-relaxed text-gray-600">Body text</p>
<p className="text-sm text-gray-500">Small text / captions</p>
<p className="text-xs text-gray-400">Tiny / legal text</p>
```

## Layout Basics

```tsx
/* Flexbox */
<div className="flex items-center justify-between gap-4">
  <div>Left</div>
  <div>Right</div>
</div>

/* Grid */
<div className="grid grid-cols-1 md:grid-cols-3 gap-6">
  <div>Card 1</div>
  <div>Card 2</div>
  <div>Card 3</div>
</div>

/* Full height layout */
<div className="min-h-screen flex flex-col">
  <header className="h-16">Header</header>
  <main className="flex-1">Content</main>
  <footer>Footer</footer>
</div>
```

## Working with Class Names (Conditional)

### With `clsx` or `cn` utility
```tsx
import { cn } from "@/lib/utils";  // shadcn/ui convention

export function Button({ variant = "primary", className }: Props) {
  return (
    <button className={cn(
      "px-4 py-2 rounded font-medium transition-colors",
      "focus:outline-none focus:ring-2",
      variant === "primary" && "bg-blue-600 text-white hover:bg-blue-700",
      variant === "secondary" && "bg-gray-200 text-gray-800 hover:bg-gray-300",
      variant === "danger" && "bg-red-600 text-white hover:bg-red-700",
      className
    )}>
      Click me
    </button>
  );
}
```

### Creating the `cn` Utility
```tsx
// src/lib/utils.ts — if using shadcn/ui, this comes pre-installed
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## 🧠 Key Takeaway
- **Utility-first** = compose small classes instead of writing custom CSS
- **Responsive prefixes** `sm:` `md:` `lg:` `xl:` `2xl:` handle breakpoints
- **State variants** `hover:` `focus:` `active:` `disabled:` handle interactions
- **Consistent scale** — everything is 4px-based spacing
- Use `cn()` utility to conditionally combine classes without conflicts
- Tailwind + Next.js = built-in support, zero config needed
