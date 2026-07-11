# 08 - Tailwind Styling Deep Dive

## Colors & Backgrounds

### Text Colors
```tsx
<p className="text-gray-900">Dark text (headings)</p>
<p className="text-gray-600">Medium text (body)</p>
<p className="text-gray-400">Light text (placeholders)</p>
<p className="text-blue-600">Link / brand</p>
<p className="text-red-600">Error / danger</p>
<p className="text-green-600">Success</p>
<p className="text-yellow-600">Warning</p>
<p className="text-white bg-gray-900">White on dark</p>
```

### Background Colors
```tsx
<div className="bg-white">White background</div>
<div className="bg-gray-50">Light gray (page background)</div>
<div className="bg-gray-100">Subtle gray (card background)</div>
<div className="bg-blue-500 text-white">Primary button</div>
<div className="bg-gradient-to-r from-blue-500 to-purple-600 text-white">
  Gradient background
</div>
<div className="bg-gradient-to-br from-green-400 via-cyan-500 to-blue-600">
  Multi-stop gradient
</div>
```

### Opacity Modifiers
```tsx
<div className="bg-blue-500/20">   {/* 20% opacity bg */}</div>
<div className="bg-blue-500/50">   {/* 50% opacity bg */}</div>
<div className="bg-blue-500/100">  {/* 100% opacity bg */}</div>
<div className="text-gray-900/50"> {/* 50% opacity text */}</div>
<div className="border-black/10">   {/* 10% opacity border */}</div>
```

## Typography Deep Dive

### Font Sizes
```tsx
<p className="text-xs">     {/* 12px - captions */}</p>
<p className="text-sm">     {/* 14px - small text */}</p>
<p className="text-base">   {/* 16px - body (default) */}</p>
<p className="text-lg">     {/* 18px - large body */}</p>
<p className="text-xl">     {/* 20px - subtitle */}</p>
<p className="text-2xl">    {/* 24px - heading 2 */}</p>
<p className="text-3xl">    {/* 30px - heading 1 */}</p>
<p className="text-4xl">    {/* 36px - hero */}</p>
<p className="text-5xl">    {/* 48px - large hero */}</p>
<p className="text-6xl">    {/* 60px - display */}</p>
<p className="text-7xl">    {/* 72px */}</p>
<p className="text-8xl">    {/* 96px */}</p>
<p className="text-9xl">    {/* 128px */}</p>
```

### Font Weight
```tsx
<p className="font-thin">       {/* 100 */}</p>
<p className="font-extralight"> {/* 200 */}</p>
<p className="font-light">      {/* 300 */}</p>
<p className="font-normal">     {/* 400 (default) */}</p>
<p className="font-medium">     {/* 500 */}</p>
<p className="font-semibold">   {/* 600 */}</p>
<p className="font-bold">       {/* 700 */}</p>
<p className="font-extrabold">  {/* 800 */}</p>
<p className="font-black">      {/* 900 */}</p>
```

### Line Height & Letter Spacing
```tsx
<p className="leading-none">      {/* 1 (tight) */}</p>
<p className="leading-tight">     {/* 1.25 */}</p>
<p className="leading-normal">    {/* 1.5 (default) */}</p>
<p className="leading-relaxed">   {/* 1.625 */}</p>
<p className="leading-loose">     {/* 2 (spacious) */}</p>

<p className="tracking-tighter">  {/* -0.05em */}</p>
<p className="tracking-tight">    {/* -0.025em */}</p>
<p className="tracking-normal">   {/* 0 (default) */}</p>
<p className="tracking-wide">     {/* 0.025em */}</p>
<p className="tracking-wider">    {/* 0.05em */}</p>
<p className="tracking-widest">   {/* 0.1em */}</p>
```

## Borders & Rings

### Border Basics
```tsx
{/* All sides */}
<div className="border border-gray-200">Default border</div>
<div className="border-2 border-blue-500">Thicker blue border</div>
<div className="border-4 border-red-500">Thick red border</div>

{/* Individual sides */}
<div className="border-t border-gray-200">Top only</div>
<div className="border-b-2 border-blue-500">Bottom only</div>
<div className="border-l-4 border-green-500">Left accent</div>
<div className="border-r border-gray-200">Right only</div>

{/* Border radius */}
<div className="rounded-sm">     {/* 2px */}</div>
<div className="rounded">        {/* 4px (default) */}</div>
<div className="rounded-md">     {/* 6px */}</div>
<div className="rounded-lg">     {/* 8px */}</div>
<div className="rounded-xl">     {/* 12px */}</div>
<div className="rounded-2xl">    {/* 16px */}</div>
<div className="rounded-3xl">    {/* 24px */}</div>
<div className="rounded-full">   {/* 9999px (pill) */}</div>

{/* Individual corners */}
<div className="rounded-t-lg">   {/* top-left + top-right */}</div>
<div className="rounded-b-lg">   {/* bottom-left + bottom-right */}</div>
<div className="rounded-l-lg">   {/* left */}</div>
<div className="rounded-r-lg">   {/* right */}</div>
```

## Shadows

```tsx
<div className="shadow-sm">    {/* small — subtle card */}</div>
<div className="shadow">       {/* medium — default card */}</div>
<div className="shadow-md">    {/* medium-large */}</div>
<div className="shadow-lg">    {/* large — modal/dropdown */}</div>
<div className="shadow-xl">    {/* extra large */}</div>
<div className="shadow-2xl">   {/* 2x — hero elements */}</div>
<div className="shadow-inner"> {/* inner — inset */}</div>
<div className="shadow-none">  {/* no shadow */}</div>

{/* Custom colored shadows */}
<div className="shadow-lg shadow-blue-500/30">Blue glow</div>
<div className="shadow-xl shadow-green-500/20">Green glow</div>
```

## Transitions & Animations

### Transitions
```tsx
<button className="
  bg-blue-500 hover:bg-blue-700
  transition                /* enable transitions */
  transition-all            /* transition all properties */
  duration-200              /* 200ms duration */
  ease-in-out               /* timing function */
">
  Hover me
</button>

{/* Common transition combos */}
<button className="
  bg-blue-500 hover:bg-blue-700
  text-white
  px-4 py-2 rounded
  transition-colors duration-200 ease-in-out    /* only color changes */
">
  Color Transition
</button>

<button className="
  opacity-80 hover:opacity-100
  scale-100 hover:scale-105
  transition-all duration-200
">
  Scale & Fade
</button>

<div className="
  translate-y-0 hover:-translate-y-1
  shadow-md hover:shadow-lg
  transition-all duration-200
">
  Lift on hover
</div>
```

### Animations
```tsx
// Built-in animations
<div className="animate-pulse">     {/* pulsing opacity */}</div>
<div className="animate-spin">      {/* spinning (good for loading) */}</div>
<div className="animate-bounce">    {/* bouncing */}</div>
<div className="animate-ping">      {/* ping effect */}</div>
<div className="animate-pulse">     {/* pulse (skeleton) */}</div>

{/* Custom animation in tailwind.config.ts */}
// tailwind.config.ts
const config: Config = {
  theme: {
    extend: {
      animation: {
        "fade-in": "fadeIn 0.5s ease-out",
        "slide-up": "slideUp 0.3s ease-out",
      },
      keyframes: {
        fadeIn: {
          "0%": { opacity: "0" },
          "100%": { opacity: "1" },
        },
        slideUp: {
          "0%": { transform: "translateY(20px)", opacity: "0" },
          "100%": { transform: "translateY(0)", opacity: "1" },
        },
      },
    },
  },
};

// Then use:
<div className="animate-fade-in">Fade in</div>
<div className="animate-slide-up">Slide up</div>
```

## Practical Component Examples

### Card
```tsx
<div className="bg-white rounded-xl shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-200">
  <img className="w-full h-48 object-cover" src="/image.jpg" alt="" />
  <div className="p-6">
    <div className="text-xs text-blue-600 font-semibold uppercase tracking-wider">Category</div>
    <h3 className="mt-2 text-xl font-semibold text-gray-900">Card Title</h3>
    <p className="mt-2 text-gray-600 text-sm leading-relaxed">
      Description goes here. Keep it concise and engaging.
    </p>
    <div className="mt-4 flex items-center gap-2">
      <img className="w-8 h-8 rounded-full" src="/avatar.jpg" alt="" />
      <span className="text-sm text-gray-500">John Doe</span>
      <span className="text-sm text-gray-400 ml-auto">3 min read</span>
    </div>
  </div>
</div>
```

### Button Variants
```tsx
{/* Primary */}
<button className="bg-blue-600 text-white px-6 py-2.5 rounded-lg font-medium
  hover:bg-blue-700 active:bg-blue-800
  focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
  transition-colors duration-200">
  Primary Button
</button>

{/* Secondary */}
<button className="bg-gray-100 text-gray-700 px-6 py-2.5 rounded-lg font-medium
  hover:bg-gray-200 active:bg-gray-300
  border border-gray-300
  transition-colors duration-200">
  Secondary
</button>

{/* Ghost */}
<button className="text-gray-600 px-6 py-2.5 rounded-lg font-medium
  hover:bg-gray-100 active:bg-gray-200
  transition-colors duration-200">
  Ghost Button
</button>

{/* Danger */}
<button className="bg-red-600 text-white px-6 py-2.5 rounded-lg font-medium
  hover:bg-red-700 active:bg-red-800
  transition-colors duration-200">
  Delete
</button>

{/* Icon Button */}
<button className="p-2 rounded-lg text-gray-500 hover:bg-gray-100 hover:text-gray-700 transition-colors">
  <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
  </svg>
</button>
```

### Input Fields
```tsx
<div className="space-y-2">
  <label className="block text-sm font-medium text-gray-700">
    Email Address
  </label>
  <input
    type="email"
    placeholder="you@example.com"
    className="w-full px-4 py-2.5 rounded-lg border border-gray-300
      focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent
      placeholder:text-gray-400
      disabled:bg-gray-50 disabled:text-gray-500
      transition-shadow duration-200"
  />
  <p className="text-sm text-red-600 mt-1">Error message here</p>
  <p className="text-sm text-gray-500 mt-1">Helper text goes here</p>
</div>
```

### Badge / Tag
```tsx
<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-blue-100 text-blue-800">
  Active
</span>

<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-green-100 text-green-800">
  Completed
</span>

<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-yellow-100 text-yellow-800">
  Pending
</span>

<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-red-100 text-red-800">
  Error
</span>

<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-800">
  Draft
</span>
```

### Skeleton Loading (Pulse Placeholder)
```tsx
<div className="animate-pulse space-y-4 p-4">
  <div className="w-16 h-16 bg-gray-200 rounded-full mx-auto" /> {/* avatar */}
  <div className="h-4 bg-gray-200 rounded w-3/4 mx-auto" />      {/* title */}
  <div className="h-3 bg-gray-200 rounded w-1/2 mx-auto" />      {/* subtitle */}
  <div className="space-y-2">
    <div className="h-3 bg-gray-200 rounded" />                   {/* line 1 */}
    <div className="h-3 bg-gray-200 rounded w-5/6" />             {/* line 2 */}
    <div className="h-3 bg-gray-200 rounded w-4/6" />             {/* line 3 */}
  </div>
</div>
```

## 🧠 Key Takeaway
- Colors use a 50–950 intensity scale (500 = base, 50 = lightest, 950 = darkest)
- Use **opacity modifiers** with slash notation: `bg-blue-500/50`
- Shadows: `shadow-sm` → `shadow-2xl` plus colored shadows
- Transitions: `transition-colors` for color, `transition-all` for everything
- Combine `hover:` + `transition-*` + `duration-*` for smooth interactions
- Build reusable patterns (cards, buttons, inputs) into components, not repeated classes
