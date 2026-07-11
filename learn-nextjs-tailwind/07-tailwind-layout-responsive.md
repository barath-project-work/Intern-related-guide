# 07 - Tailwind Layout & Responsive Design

## The Flexbox System

### Basic Flex
```tsx
{/* Horizontal layout */}
<div className="flex gap-4">
  <div className="bg-blue-100 p-4 rounded">Item 1</div>
  <div className="bg-blue-200 p-4 rounded">Item 2</div>
  <div className="bg-blue-300 p-4 rounded">Item 3</div>
</div>

{/* Vertical layout */}
<div className="flex flex-col gap-2">
  <span>Row 1</span>
  <span>Row 2</span>
  <span>Row 3</span>
</div>
```

### Alignment & Justification
```tsx
{/* Center everything (both axes) */}
<div className="flex items-center justify-center h-40 bg-gray-100">
  <span>Centered</span>
</div>

{/* Space between items */}
<div className="flex justify-between items-center px-4">
  <h2 className="text-xl font-bold">Title</h2>
  <nav className="flex gap-4">
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
  <button className="bg-blue-600 text-white px-4 py-2 rounded">
    Sign In
  </button>
</div>

{/* Items stretch to same height */}
<div className="flex items-stretch gap-4">
  <div className="bg-red-100 p-4">Short</div>
  <div className="bg-blue-100 p-4">Taller content that fills space</div>
  <div className="bg-green-100 p-4">Also stretches</div>
</div>
```

### Common Flex Patterns

| Class | What it does |
|-------|-------------|
| `flex` | display: flex |
| `flex-col` | flex-direction: column |
| `flex-row` | flex-direction: row (default) |
| `flex-wrap` | flex-wrap: wrap |
| `flex-1` | flex: 1 (grow to fill space) |
| `flex-shrink-0` | won't shrink |
| `flex-grow` | flex-grow: 1 |
| `gap-{size}` | gap between items |
| `items-center` | align-items: center |
| `items-start` | align-items: flex-start |
| `items-end` | align-items: flex-end |
| `justify-center` | justify-content: center |
| `justify-between` | justify-content: space-between |
| `justify-around` | justify-content: space-around |
| `justify-end` | justify-content: flex-end |

## The Grid System

### Basic Grid
```tsx
{/* 3 equal columns by default */}
<div className="grid grid-cols-3 gap-4">
  <div className="bg-purple-100 p-4 rounded">1</div>
  <div className="bg-purple-200 p-4 rounded">2</div>
  <div className="bg-purple-300 p-4 rounded">3</div>
</div>

{/* Responsive grid — changes columns based on screen size */}
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  {posts.map(post => (
    <PostCard key={post.id} post={post} />
  ))}
</div>

{/* Auto-fit grid (no media queries needed!) */}
<div className="grid grid-cols-[repeat(auto-fit,minmax(300px,1fr))] gap-6">
  {posts.map(post => (
    <PostCard key={post.id} post={post} />
  ))}
  {/* Automatically wraps to new rows when < 300px */}
</div>
```

### Grid Spanning
```tsx
{/* Hero layout */}
<div className="grid grid-cols-12 gap-4">
  {/* Sidebar spans 3 columns */}
  <aside className="col-span-12 md:col-span-3 bg-gray-100 p-4">
    Sidebar
  </aside>

  {/* Main content spans 9 columns */}
  <main className="col-span-12 md:col-span-9 bg-white p-4">
    <h1 className="text-2xl font-bold">Main Content</h1>
    <p>This layout adapts from stacked (mobile) to sidebar (desktop).</p>
  </main>
</div>
```

## Responsive Design Patterns

### The Mobile-First Approach

**Always write the mobile layout first, then add `sm:` `md:` `lg:` overrides:**

```tsx
<div className="
  /* Mobile (default) */
  flex flex-col gap-4

  /* Tablet */
  md:flex-row md:gap-6

  /* Desktop */
  lg:gap-8 lg:p-8
">
  <div className="
    /* Mobile: full width */
    w-full

    /* Tablet: half width */
    md:w-1/2

    /* Desktop: third width */
    lg:w-1/3
  ">
    Responsive card
  </div>
</div>
```

### Common Responsive Patterns

#### 1. Stacked on Mobile, Side-by-Side on Desktop
```tsx
<div className="flex flex-col md:flex-row gap-6">
  <div className="flex-1">Left content</div>
  <div className="flex-1">Right content</div>
</div>
```

#### 2. Responsive Navigation
```tsx
<nav className="flex flex-col md:flex-row md:items-center gap-4 p-4">
  <a href="/" className="text-lg font-bold">Logo</a>

  {/* Mobile: full width links, Desktop: horizontal */}
  <div className="flex flex-col md:flex-row gap-2 md:gap-6 md:ml-auto">
    <a href="/" className="hover:text-blue-600">Home</a>
    <a href="/about" className="hover:text-blue-600">About</a>
    <a href="/contact" className="hover:text-blue-600">Contact</a>
  </div>

  {/* Hide button on mobile, show on desktop */}
  <button className="hidden md:inline-block bg-blue-600 text-white px-4 py-2 rounded">
    Get Started
  </button>
</nav>
```

#### 3. Responsive Cards Grid
```tsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 p-6">
  {products.map(product => (
    <div key={product.id} className="bg-white rounded-lg shadow overflow-hidden">
      <img
        src={product.image}
        alt={product.name}
        className="w-full h-48 object-cover"
      />
      <div className="p-4">
        <h3 className="font-semibold text-lg">{product.name}</h3>
        <p className="text-gray-600 text-sm">{product.description}</p>
        <p className="text-lg font-bold mt-2">${product.price}</p>
        <button className="w-full mt-4 bg-blue-600 text-white py-2 rounded hover:bg-blue-700">
          Add to Cart
        </button>
      </div>
    </div>
  ))}
</div>
```

#### 4. Responsive Table (card layout on mobile)
```tsx
<div className="overflow-x-auto">
  {/* Table: horizontal scroll on small screens */}
  <table className="min-w-full divide-y divide-gray-200">
    <thead className="bg-gray-50">
      <tr>
        <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Name</th>
        <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Role</th>
        <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Status</th>
        <th className="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase">Actions</th>
      </tr>
    </thead>
    <tbody className="bg-white divide-y divide-gray-200">
      {users.map(user => (
        <tr key={user.id} className="hover:bg-gray-50">
          <td className="px-6 py-4 whitespace-nowrap">{user.name}</td>
          <td className="px-6 py-4 whitespace-nowrap text-gray-600">{user.role}</td>
          <td className="px-6 py-4 whitespace-nowrap">
            <span className="px-2 py-1 text-xs font-medium bg-green-100 text-green-800 rounded-full">
              {user.status}
            </span>
          </td>
          <td className="px-6 py-4 whitespace-nowrap text-right">
            <button className="text-blue-600 hover:text-blue-800">Edit</button>
          </td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

### Image Responsiveness
```tsx
{/* Responsive image */}
<img
  src={image.url}
  alt={image.alt}
  className="w-full h-auto max-w-full"
/>

{/* With object-fit */}
<img
  src={image.url}
  alt={image.alt}
  className="w-full h-64 object-cover rounded-lg"
/>

{/* Next.js Image component (optimized) */}
import Image from "next/image";

<Image
  src={product.image}
  alt={product.name}
  width={400}
  height={300}
  className="w-full h-48 object-cover rounded-t-lg"
/>
```

## Responsive Text

```tsx
<h1 className="
  text-2xl          /* mobile */
  sm:text-3xl       /* tablet */
  md:text-4xl       /* desktop */
  lg:text-5xl       /* large desktop */
  font-bold
">
  Responsive Heading
</h1>

<p className="
  text-sm           /* mobile */
  md:text-base      /* tablet+ */
  text-gray-600
  leading-relaxed
">
  This paragraph text gets larger on bigger screens
  for better readability.
</p>
```

## Responsive Spacing

```tsx
<div className="
  p-4               /* mobile */
  md:p-8            /* tablet */
  lg:p-12           /* desktop */
  space-y-4
">
  <section className="
    py-4 md:py-8   /* vertical padding changes */
    px-4 md:px-0   /* horizontal padding disappears on tablet+ */
  ">
    Content section
  </section>
</div>
```

## Hide / Show Elements

```tsx
{/* Visible on mobile only */}
<div className="block md:hidden">Mobile only</div>

{/* Visible on desktop only */}
<div className="hidden md:block">Desktop only</div>

{/* Visible on specific ranges */}
<div className="hidden lg:block">Large screens only (1024px+)</div>
<div className="block lg:hidden">Everything except large screens</div>
<div className="hidden sm:block md:hidden">Only tablet-sized (640-768px)</div>
```

## Container & Max-Width

```tsx
{/* Centered content with max-width */}
<div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
  {/* Content stays centered and never exceeds 1280px */}
</div>

{/* Common container sizes */}
<div className="max-w-xs">     {/* 320px - mobile form */}</div>
<div className="max-w-sm">     {/* 384px - card */}</div>
<div className="max-w-md">     {/* 448px - single column */}</div>
<div className="max-w-lg">     {/* 512px - form */}</div>
<div className="max-w-xl">     {/* 576px - article */}</div>
<div className="max-w-2xl">    {/* 672px - blog post */}</div>
<div className="max-w-4xl">    {/* 896px - dashboard content */}</div>
<div className="max-w-7xl">    {/* 1280px - full page */}</div>
```

## 🧠 Key Takeaway
- **Mobile-first**: write mobile styles first, override with breakpoint prefixes
- Use `flex` for 1D layouts, `grid` for 2D layouts
- `grid-cols-[repeat(auto-fit,minmax(300px,1fr))]` = auto-responsive grid
- `hidden md:block` to show/hide elements at different breakpoints
- Always wrap content in `max-w-7xl mx-auto px-4` for a clean centered layout
- Test on real device sizes — don't just resize the browser!
