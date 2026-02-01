# Tailwind CSS

> Utility-first CSS framework for rapid UI development.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Utility-first CSS framework |
| **Why** | Rapid development, consistency, small bundle |
| **Version** | 3.x |
| **Location** | `apps/web/tailwind.config.ts` |

## Configuration

```typescript
// apps/web/tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          500: '#0ea5e9',
          600: '#0284c7',
          // ...
        },
        care: {
          green: '#10B981',
          blue: '#3B82F6',
          // ...
        },
      },
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui'],
        heading: ['var(--font-poppins)', 'system-ui'],
      },
      animation: {
        'fade-in': 'fadeIn 0.3s ease-out',
        'slide-up': 'slideUp 0.3s ease-out',
        'pulse-slow': 'pulse 3s infinite',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
};

export default config;
```

## Global Styles

```css
/* apps/web/src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    /* ... */
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ... */
  }
}

@layer components {
  .btn-primary {
    @apply bg-primary-600 text-white px-4 py-2 rounded-lg
           hover:bg-primary-700 transition-colors
           disabled:opacity-50 disabled:cursor-not-allowed;
  }

  .card {
    @apply bg-white dark:bg-gray-800 rounded-xl shadow-sm
           border border-gray-200 dark:border-gray-700 p-6;
  }

  .input-field {
    @apply w-full px-4 py-2 border rounded-lg
           focus:ring-2 focus:ring-primary-500 focus:border-primary-500
           dark:bg-gray-800 dark:border-gray-700;
  }
}
```

## Common Patterns

### Responsive Design
```tsx
<div className="
  w-full           // Mobile: full width
  md:w-1/2         // Tablet: half width
  lg:w-1/3         // Desktop: third width
  xl:w-1/4         // Large: quarter width
">
```

### Dark Mode
```tsx
<div className="
  bg-white dark:bg-gray-800
  text-gray-900 dark:text-white
  border-gray-200 dark:border-gray-700
">
```

### Flexbox Layout
```tsx
<div className="flex items-center justify-between gap-4">
  <span>Left</span>
  <span>Right</span>
</div>
```

### Grid Layout
```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {items.map(item => <Card key={item.id} />)}
</div>
```

### States
```tsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  focus:ring-2 focus:ring-blue-500
  active:bg-blue-700
  disabled:opacity-50 disabled:cursor-not-allowed
">
```

### Groups & Peers
```tsx
<div className="group">
  <span className="group-hover:text-blue-500">Hover parent to change</span>
</div>

<input className="peer" />
<span className="peer-invalid:text-red-500">Error message</span>
```

## Component Examples

### Button
```tsx
function Button({ variant = 'primary', children, ...props }) {
  const variants = {
    primary: 'bg-primary-600 text-white hover:bg-primary-700',
    secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200',
    danger: 'bg-red-600 text-white hover:bg-red-700',
  };

  return (
    <button
      className={`
        px-4 py-2 rounded-lg font-medium
        transition-colors duration-200
        disabled:opacity-50 disabled:cursor-not-allowed
        ${variants[variant]}
      `}
      {...props}
    >
      {children}
    </button>
  );
}
```

### Card
```tsx
function Card({ children, className }) {
  return (
    <div className={`
      bg-white dark:bg-gray-800
      rounded-xl shadow-sm
      border border-gray-200 dark:border-gray-700
      p-6
      ${className}
    `}>
      {children}
    </div>
  );
}
```

### Form Input
```tsx
function Input({ label, error, ...props }) {
  return (
    <div className="space-y-1">
      {label && (
        <label className="block text-sm font-medium text-gray-700 dark:text-gray-300">
          {label}
        </label>
      )}
      <input
        className={`
          w-full px-4 py-2 rounded-lg border
          focus:ring-2 focus:ring-primary-500 focus:border-primary-500
          dark:bg-gray-800 dark:border-gray-700
          ${error ? 'border-red-500' : 'border-gray-300'}
        `}
        {...props}
      />
      {error && (
        <p className="text-sm text-red-500">{error}</p>
      )}
    </div>
  );
}
```

## Animations

```css
/* In globals.css */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(10px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

```tsx
// Usage
<div className="animate-fade-in">Fade in</div>
<div className="animate-slide-up">Slide up</div>
<div className="animate-spin">Spinner</div>
```

## Utility Functions

### cn() - Class Name Merger
```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div className={cn(
  'base-classes',
  isActive && 'active-classes',
  className
)} />
```

## Common Commands

```bash
# Build CSS (happens automatically with Next.js)
npx tailwindcss -i ./src/input.css -o ./dist/output.css

# Watch mode
npx tailwindcss -i ./src/input.css -o ./dist/output.css --watch
```

## Troubleshooting

### Classes Not Applied
- Check `content` paths in `tailwind.config.ts`
- Ensure file extensions are included
- Restart dev server after config changes

### Conflicting Classes
Use `tailwind-merge`:
```tsx
// Bad: conflicting classes
<div className="p-4 p-8"> // Both applied, unpredictable

// Good: use cn() with tailwind-merge
<div className={cn('p-4', 'p-8')}> // p-8 wins
```

### Dark Mode Not Working
```tsx
// Ensure parent has dark class
<html className={isDark ? 'dark' : ''}>
  <body className="bg-white dark:bg-gray-900">
```

---

*See also: [Framer Motion](framer-motion.md), [Radix UI](radix-ui.md)*


