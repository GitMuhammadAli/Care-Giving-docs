# 🎨 CSS Architecture - Complete Guide

> A comprehensive guide to CSS architecture - BEM, CSS Modules, CSS-in-JS, Tailwind patterns, and building scalable, maintainable stylesheets.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "CSS architecture encompasses methodologies and patterns for organizing stylesheets at scale - from naming conventions (BEM) to scoping solutions (CSS Modules, CSS-in-JS) to utility-first approaches (Tailwind) - each with trade-offs in specificity management, bundle size, and developer experience."

### The 7 Key Concepts (Remember These!)
```
1. BEM              → Block Element Modifier naming convention
2. CSS MODULES      → Locally scoped class names, compile-time
3. CSS-IN-JS        → Styles in JavaScript (styled-components, Emotion)
4. TAILWIND         → Utility-first CSS framework
5. SPECIFICITY      → How browsers decide which styles apply
6. SCOPING          → Preventing style conflicts
7. DESIGN TOKENS    → CSS custom properties for theming
```

### CSS Architecture Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              CSS ARCHITECTURE COMPARISON                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BEM (Block Element Modifier)                                  │
│  ────────────────────────────                                   │
│  • Naming convention: .block__element--modifier                │
│  • No tooling required                                         │
│  • Flat specificity                                            │
│  ✅ Simple, works everywhere                                   │
│  ❌ Verbose, discipline required                               │
│  Use: Team standards, global CSS                               │
│                                                                 │
│  CSS MODULES                                                   │
│  ───────────                                                    │
│  • Locally scoped classes                                      │
│  • Compile-time transformation                                 │
│  • .button → .Button_button__abc123                            │
│  ✅ True isolation, tree-shakeable                             │
│  ❌ Dynamic styles harder                                      │
│  Use: Component libraries, React/Vue                           │
│                                                                 │
│  CSS-IN-JS (styled-components, Emotion)                        │
│  ───────────────────────────────────────                        │
│  • Styles defined in JavaScript                                │
│  • Runtime or compile-time                                     │
│  • Dynamic styles with props                                   │
│  ✅ Co-location, dynamic styling                               │
│  ❌ Runtime cost, bundle size                                  │
│  Use: Themeable apps, design systems                           │
│                                                                 │
│  TAILWIND CSS                                                  │
│  ────────────                                                   │
│  • Utility-first classes                                       │
│  • No custom CSS (ideally)                                     │
│  • Compile-time purging                                        │
│  ✅ Fast development, consistent                               │
│  ❌ Verbose HTML, learning curve                               │
│  Use: Rapid development, landing pages                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Specificity Hierarchy
```
┌─────────────────────────────────────────────────────────────────┐
│              CSS SPECIFICITY                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LOWEST TO HIGHEST:                                            │
│  ──────────────────                                             │
│  *                        → 0,0,0,0 (universal)                │
│  element                  → 0,0,0,1 (div, p, span)             │
│  .class                   → 0,0,1,0                            │
│  [attribute]              → 0,0,1,0                            │
│  #id                      → 0,1,0,0                            │
│  inline style             → 1,0,0,0                            │
│  !important               → Overrides all (avoid!)             │
│                                                                 │
│  EXAMPLES:                                                     │
│  ─────────                                                      │
│  div                      → 0,0,0,1                            │
│  div.class                → 0,0,1,1                            │
│  div#id                   → 0,1,0,1                            │
│  div#id.class             → 0,1,1,1                            │
│  #id .class div           → 0,1,1,1                            │
│                                                                 │
│  BEST PRACTICE:                                                │
│  Keep specificity flat with single classes:                    │
│  .button { }           → 0,0,1,0                               │
│  .button--primary { }  → 0,0,1,0                               │
│  Both are equal, order matters                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Specificity wars"** | "BEM avoids specificity wars by using flat selectors" |
| **"Critical CSS"** | "We inline critical CSS for above-the-fold content" |
| **"Dead code elimination"** | "PurgeCSS removes unused Tailwind utilities" |
| **"Co-location"** | "CSS-in-JS enables style co-location with components" |
| **"Zero-runtime"** | "We use vanilla-extract for zero-runtime CSS-in-JS" |
| **"Atomic CSS"** | "Tailwind is an atomic CSS approach" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Tailwind base | **~10KB** | After purging unused |
| styled-components | **~15KB** | Runtime library |
| CSS Modules | **0KB** | No runtime |
| !important | **0** | Target usage |

### The "Wow" Statement (Memorize This!)
> "We use CSS Modules for component isolation with zero runtime cost. Classes become unique hashes, so no naming conflicts. For design tokens, we use CSS custom properties - they cascade naturally and support runtime theming. We compose styles using composes keyword rather than duplicating. Global styles use BEM for predictability. Critical CSS is inlined for fast initial render. We track bundle size - after tree-shaking, CSS is under 30KB. For Tailwind projects, we extend the config with our design tokens and use @apply sparingly. The key is consistency: one approach per project, documented in our style guide."

---

## 📚 Table of Contents

1. [BEM Methodology](#1-bem-methodology)
2. [CSS Modules](#2-css-modules)
3. [CSS-in-JS](#3-css-in-js)
4. [Tailwind Patterns](#4-tailwind-patterns)
5. [CSS Custom Properties](#5-css-custom-properties)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. BEM Methodology

```css
/* ════════════════════════════════════════════════════════════════
   BEM: Block Element Modifier
   ════════════════════════════════════════════════════════════════ */

/* 
   STRUCTURE:
   .block              → Standalone component
   .block__element     → Part of block (child)
   .block--modifier    → Variation of block
   .block__element--modifier → Variation of element
*/

/* ════════════════════════════════════════════════════════════════
   BASIC EXAMPLES
   ════════════════════════════════════════════════════════════════ */

/* Block: Standalone component */
.card {
  padding: 1rem;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Element: Part of card (using __) */
.card__header {
  padding-bottom: 1rem;
  border-bottom: 1px solid #eee;
}

.card__title {
  font-size: 1.25rem;
  font-weight: 600;
}

.card__body {
  padding: 1rem 0;
}

.card__footer {
  padding-top: 1rem;
  border-top: 1px solid #eee;
}

/* Modifier: Variation of block (using --) */
.card--featured {
  border: 2px solid gold;
}

.card--compact {
  padding: 0.5rem;
}

/* Element modifier */
.card__title--large {
  font-size: 1.5rem;
}

/* ════════════════════════════════════════════════════════════════
   BUTTON COMPONENT EXAMPLE
   ════════════════════════════════════════════════════════════════ */

/* Block */
.button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
}

/* Modifiers for variants */
.button--primary {
  background-color: #3b82f6;
  color: white;
}

.button--primary:hover {
  background-color: #2563eb;
}

.button--secondary {
  background-color: transparent;
  color: #374151;
  border: 1px solid #d1d5db;
}

.button--secondary:hover {
  background-color: #f3f4f6;
}

.button--ghost {
  background-color: transparent;
  color: #374151;
}

.button--ghost:hover {
  background-color: #f3f4f6;
}

/* Modifiers for sizes */
.button--sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.button--lg {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

/* Elements */
.button__icon {
  width: 1em;
  height: 1em;
}

.button__icon--left {
  margin-right: 0.25rem;
}

.button__icon--right {
  margin-left: 0.25rem;
}

/* State modifiers */
.button--loading {
  opacity: 0.7;
  pointer-events: none;
}

.button--disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* ════════════════════════════════════════════════════════════════
   HTML USAGE
   ════════════════════════════════════════════════════════════════ */

/*
<button class="button button--primary button--lg">
  <svg class="button__icon button__icon--left">...</svg>
  Submit
</button>

<div class="card card--featured">
  <div class="card__header">
    <h2 class="card__title card__title--large">Featured</h2>
  </div>
  <div class="card__body">Content</div>
  <div class="card__footer">
    <button class="button button--primary">Action</button>
  </div>
</div>
*/

/* ════════════════════════════════════════════════════════════════
   BEM WITH SASS
   ════════════════════════════════════════════════════════════════ */

.card {
  padding: 1rem;
  
  /* Elements with & */
  &__header {
    padding-bottom: 1rem;
  }
  
  &__title {
    font-size: 1.25rem;
    
    /* Element modifier */
    &--large {
      font-size: 1.5rem;
    }
  }
  
  &__body {
    padding: 1rem 0;
  }
  
  /* Modifiers */
  &--featured {
    border: 2px solid gold;
  }
  
  &--compact {
    padding: 0.5rem;
    
    /* Nested element in modifier context */
    .card__header {
      padding-bottom: 0.5rem;
    }
  }
}
```

---

## 2. CSS Modules

```tsx
// ════════════════════════════════════════════════════════════════
// CSS MODULES BASICS
// ════════════════════════════════════════════════════════════════

// Button.module.css
.button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  background-color: #3b82f6;
  color: white;
}

.secondary {
  background-color: transparent;
  border: 1px solid #d1d5db;
}

.small {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.large {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

.icon {
  width: 1em;
  height: 1em;
}

// Button.tsx
import styles from './Button.module.css';
import { clsx } from 'clsx';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  children: React.ReactNode;
}

export function Button({ 
  variant = 'primary', 
  size = 'medium',
  children 
}: ButtonProps) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[variant],
        size !== 'medium' && styles[size]
      )}
    >
      {children}
    </button>
  );
}

// Generated HTML:
// <button class="Button_button__x7f3s Button_primary__k2j4d">

// ════════════════════════════════════════════════════════════════
// COMPOSITION
// ════════════════════════════════════════════════════════════════

// base.module.css
.flexCenter {
  display: flex;
  align-items: center;
  justify-content: center;
}

.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// Button.module.css
.button {
  /* Compose from other modules */
  composes: flexCenter from './base.module.css';
  padding: 0.5rem 1rem;
}

.buttonText {
  composes: truncate from './base.module.css';
  max-width: 200px;
}

// ════════════════════════════════════════════════════════════════
// GLOBAL STYLES IN CSS MODULES
// ════════════════════════════════════════════════════════════════

// styles.module.css
/* Local by default */
.localClass {
  color: blue;
}

/* Escape hatch for global */
:global(.globalClass) {
  color: red;
}

/* Target global elements within local scope */
.container :global(.third-party-class) {
  margin: 0;
}

// ════════════════════════════════════════════════════════════════
// CSS MODULES WITH TYPESCRIPT
// ════════════════════════════════════════════════════════════════

// Button.module.css.d.ts (generated or manual)
declare const styles: {
  readonly button: string;
  readonly primary: string;
  readonly secondary: string;
  readonly small: string;
  readonly large: string;
  readonly icon: string;
};
export default styles;

// For auto-generation, use typed-css-modules or css-modules-typescript-loader

// vite.config.ts (Vite handles this automatically)
export default {
  css: {
    modules: {
      localsConvention: 'camelCase', // .my-class → styles.myClass
      generateScopedName: '[name]__[local]__[hash:base64:5]',
    },
  },
};

// ════════════════════════════════════════════════════════════════
// ADVANCED PATTERNS
// ════════════════════════════════════════════════════════════════

// Card.module.css
.card {
  --card-padding: 1rem;
  padding: var(--card-padding);
  border-radius: 8px;
}

.card[data-variant="compact"] {
  --card-padding: 0.5rem;
}

.header {
  padding-bottom: var(--card-padding);
  border-bottom: 1px solid var(--border-color, #e5e7eb);
}

// Card.tsx
import styles from './Card.module.css';

function Card({ variant, children }: CardProps) {
  return (
    <div className={styles.card} data-variant={variant}>
      {children}
    </div>
  );
}

function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className={styles.header}>{children}</div>;
}
```

---

## 3. CSS-in-JS

```tsx
// ════════════════════════════════════════════════════════════════
// STYLED-COMPONENTS
// ════════════════════════════════════════════════════════════════

import styled, { css, keyframes } from 'styled-components';

// Basic styled component
const Button = styled.button`
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
`;

// With props
interface ButtonProps {
  $variant?: 'primary' | 'secondary' | 'ghost';
  $size?: 'sm' | 'md' | 'lg';
}

const StyledButton = styled.button<ButtonProps>`
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  /* Size variants */
  ${({ $size = 'md' }) => {
    switch ($size) {
      case 'sm':
        return css`
          padding: 0.25rem 0.5rem;
          font-size: 0.875rem;
        `;
      case 'lg':
        return css`
          padding: 0.75rem 1.5rem;
          font-size: 1.125rem;
        `;
      default:
        return css`
          padding: 0.5rem 1rem;
          font-size: 1rem;
        `;
    }
  }}
  
  /* Color variants */
  ${({ $variant = 'primary', theme }) => {
    switch ($variant) {
      case 'secondary':
        return css`
          background: transparent;
          color: ${theme.colors.text};
          border: 1px solid ${theme.colors.border};
          
          &:hover {
            background: ${theme.colors.backgroundHover};
          }
        `;
      case 'ghost':
        return css`
          background: transparent;
          color: ${theme.colors.text};
          
          &:hover {
            background: ${theme.colors.backgroundHover};
          }
        `;
      default:
        return css`
          background: ${theme.colors.primary};
          color: white;
          
          &:hover {
            background: ${theme.colors.primaryHover};
          }
        `;
    }
  }}
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// Extending styles
const PrimaryButton = styled(StyledButton)`
  font-weight: 600;
`;

// Animations
const spin = keyframes`
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
`;

const Spinner = styled.div`
  width: 20px;
  height: 20px;
  border: 2px solid #f3f4f6;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: ${spin} 0.8s linear infinite;
`;

// ════════════════════════════════════════════════════════════════
// THEMING
// ════════════════════════════════════════════════════════════════

import { ThemeProvider, createGlobalStyle } from 'styled-components';

const theme = {
  colors: {
    primary: '#3b82f6',
    primaryHover: '#2563eb',
    text: '#111827',
    textSecondary: '#6b7280',
    background: '#ffffff',
    backgroundHover: '#f3f4f6',
    border: '#e5e7eb',
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
  },
  radii: {
    sm: '4px',
    md: '8px',
    lg: '12px',
    full: '9999px',
  },
};

const darkTheme = {
  ...theme,
  colors: {
    ...theme.colors,
    text: '#f9fafb',
    textSecondary: '#9ca3af',
    background: '#111827',
    backgroundHover: '#1f2937',
    border: '#374151',
  },
};

const GlobalStyle = createGlobalStyle`
  *, *::before, *::after {
    box-sizing: border-box;
  }
  
  body {
    margin: 0;
    font-family: system-ui, sans-serif;
    background: ${({ theme }) => theme.colors.background};
    color: ${({ theme }) => theme.colors.text};
  }
`;

function App() {
  const [isDark, setIsDark] = useState(false);
  
  return (
    <ThemeProvider theme={isDark ? darkTheme : theme}>
      <GlobalStyle />
      <StyledButton $variant="primary">Click me</StyledButton>
    </ThemeProvider>
  );
}

// ════════════════════════════════════════════════════════════════
// EMOTION (Alternative to styled-components)
// ════════════════════════════════════════════════════════════════

import { css } from '@emotion/react';
import styled from '@emotion/styled';

// Object styles (type-safe)
const buttonStyles = css({
  display: 'inline-flex',
  alignItems: 'center',
  gap: '0.5rem',
  padding: '0.5rem 1rem',
  borderRadius: '4px',
});

// Template literal styles
const Button = styled.button`
  ${buttonStyles}
  background: ${props => props.theme.colors.primary};
`;

// ════════════════════════════════════════════════════════════════
// ZERO-RUNTIME: VANILLA-EXTRACT
// ════════════════════════════════════════════════════════════════

// button.css.ts
import { style, styleVariants } from '@vanilla-extract/css';
import { recipe } from '@vanilla-extract/recipes';

// Basic style
export const button = style({
  display: 'inline-flex',
  alignItems: 'center',
  gap: '0.5rem',
  padding: '0.5rem 1rem',
  borderRadius: '4px',
  border: 'none',
  cursor: 'pointer',
});

// Variants
export const buttonVariants = styleVariants({
  primary: {
    backgroundColor: '#3b82f6',
    color: 'white',
  },
  secondary: {
    backgroundColor: 'transparent',
    border: '1px solid #e5e7eb',
  },
});

// Recipe (variant API)
export const buttonRecipe = recipe({
  base: {
    display: 'inline-flex',
    alignItems: 'center',
    borderRadius: '4px',
    border: 'none',
    cursor: 'pointer',
  },
  variants: {
    variant: {
      primary: { backgroundColor: '#3b82f6', color: 'white' },
      secondary: { backgroundColor: 'transparent', border: '1px solid #e5e7eb' },
    },
    size: {
      sm: { padding: '0.25rem 0.5rem', fontSize: '0.875rem' },
      md: { padding: '0.5rem 1rem', fontSize: '1rem' },
      lg: { padding: '0.75rem 1.5rem', fontSize: '1.125rem' },
    },
  },
  defaultVariants: {
    variant: 'primary',
    size: 'md',
  },
});

// Button.tsx
import { buttonRecipe } from './button.css';

function Button({ variant, size, children }) {
  return (
    <button className={buttonRecipe({ variant, size })}>
      {children}
    </button>
  );
}
```

---

## 4. Tailwind Patterns

```tsx
// ════════════════════════════════════════════════════════════════
// TAILWIND BASICS
// ════════════════════════════════════════════════════════════════

// Basic button
function Button({ children }: { children: React.ReactNode }) {
  return (
    <button className="inline-flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
      {children}
    </button>
  );
}

// With variants using clsx/tailwind-merge
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  className?: string;
  children: React.ReactNode;
}

const variantStyles = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-transparent border border-gray-300 hover:bg-gray-100',
  ghost: 'bg-transparent hover:bg-gray-100',
};

const sizeStyles = {
  sm: 'px-2 py-1 text-sm',
  md: 'px-4 py-2',
  lg: 'px-6 py-3 text-lg',
};

function Button({
  variant = 'primary',
  size = 'md',
  className,
  children,
}: ButtonProps) {
  return (
    <button
      className={cn(
        'inline-flex items-center gap-2 rounded-lg transition-colors',
        variantStyles[variant],
        sizeStyles[size],
        className
      )}
    >
      {children}
    </button>
  );
}

// ════════════════════════════════════════════════════════════════
// CLASS VARIANCE AUTHORITY (CVA)
// ════════════════════════════════════════════════════════════════

import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center gap-2 rounded-lg font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:ring-blue-600',
        secondary: 'bg-transparent border border-gray-300 hover:bg-gray-100',
        ghost: 'bg-transparent hover:bg-gray-100',
        destructive: 'bg-red-600 text-white hover:bg-red-700',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants>;

function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  );
}

// ════════════════════════════════════════════════════════════════
// TAILWIND CONFIG CUSTOMIZATION
// ════════════════════════════════════════════════════════════════

// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      // Custom colors
      colors: {
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
      },
      // Custom spacing
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      // Custom font
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      // Custom animations
      animation: {
        'spin-slow': 'spin 3s linear infinite',
        'fade-in': 'fadeIn 0.2s ease-in-out',
      },
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
};

// ════════════════════════════════════════════════════════════════
// RESPONSIVE DESIGN
// ════════════════════════════════════════════════════════════════

function ResponsiveCard() {
  return (
    <div className="
      p-4 
      md:p-6 
      lg:p-8
      
      grid 
      grid-cols-1 
      md:grid-cols-2 
      lg:grid-cols-3 
      gap-4
    ">
      {/* Responsive text */}
      <h1 className="text-xl md:text-2xl lg:text-3xl font-bold">
        Title
      </h1>
      
      {/* Hide/show based on breakpoint */}
      <div className="hidden md:block">
        Desktop only content
      </div>
      <div className="block md:hidden">
        Mobile only content
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DARK MODE
// ════════════════════════════════════════════════════════════════

// tailwind.config.js
module.exports = {
  darkMode: 'class', // or 'media'
};

function Card() {
  return (
    <div className="
      bg-white dark:bg-gray-800
      text-gray-900 dark:text-gray-100
      border border-gray-200 dark:border-gray-700
      rounded-lg p-4
    ">
      <h2 className="text-gray-800 dark:text-gray-200">
        Card Title
      </h2>
      <p className="text-gray-600 dark:text-gray-400">
        Card content
      </p>
    </div>
  );
}
```

---

## 5. CSS Custom Properties

```css
/* ════════════════════════════════════════════════════════════════
   CSS CUSTOM PROPERTIES (Variables)
   ════════════════════════════════════════════════════════════════ */

/* Define on :root for global access */
:root {
  /* Colors */
  --color-primary: #3b82f6;
  --color-primary-hover: #2563eb;
  --color-text: #111827;
  --color-text-secondary: #6b7280;
  --color-background: #ffffff;
  --color-border: #e5e7eb;
  
  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Typography */
  --font-family: system-ui, -apple-system, sans-serif;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  
  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  
  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 200ms ease;
}

/* Dark mode override */
[data-theme="dark"] {
  --color-text: #f9fafb;
  --color-text-secondary: #9ca3af;
  --color-background: #111827;
  --color-border: #374151;
}

/* ════════════════════════════════════════════════════════════════
   USAGE IN COMPONENTS
   ════════════════════════════════════════════════════════════════ */

.button {
  padding: var(--spacing-sm) var(--spacing-md);
  background-color: var(--color-primary);
  color: white;
  border-radius: var(--radius-md);
  font-family: var(--font-family);
  transition: background-color var(--transition-base);
}

.button:hover {
  background-color: var(--color-primary-hover);
}

.card {
  padding: var(--spacing-lg);
  background-color: var(--color-background);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
}

/* ════════════════════════════════════════════════════════════════
   COMPONENT-SCOPED VARIABLES
   ════════════════════════════════════════════════════════════════ */

.button {
  /* Define component-specific defaults */
  --button-bg: var(--color-primary);
  --button-color: white;
  --button-padding-x: var(--spacing-md);
  --button-padding-y: var(--spacing-sm);
  
  padding: var(--button-padding-y) var(--button-padding-x);
  background-color: var(--button-bg);
  color: var(--button-color);
}

/* Variants override the variables */
.button--secondary {
  --button-bg: transparent;
  --button-color: var(--color-text);
  border: 1px solid var(--color-border);
}

.button--sm {
  --button-padding-x: var(--spacing-sm);
  --button-padding-y: var(--spacing-xs);
  font-size: var(--font-size-sm);
}

/* ════════════════════════════════════════════════════════════════
   DYNAMIC THEMING WITH JS
   ════════════════════════════════════════════════════════════════ */

// JavaScript to update CSS variables
function setTheme(theme: Record<string, string>) {
  const root = document.documentElement;
  Object.entries(theme).forEach(([key, value]) => {
    root.style.setProperty(`--${key}`, value);
  });
}

// Usage
setTheme({
  'color-primary': '#8b5cf6',
  'color-primary-hover': '#7c3aed',
});

// React hook for theme
function useTheme() {
  const setColor = (name: string, value: string) => {
    document.documentElement.style.setProperty(`--color-${name}`, value);
  };
  
  const getColor = (name: string) => {
    return getComputedStyle(document.documentElement)
      .getPropertyValue(`--color-${name}`)
      .trim();
  };
  
  return { setColor, getColor };
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CSS ARCHITECTURE PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Overly specific selectors
# Bad
#header .nav ul li a.active { }  # Specificity: 1,2,3

# Good
.nav__link--active { }  # Specificity: 0,1,0

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Using !important
# Bad
.button {
  background: blue !important;
}

# Good
# Increase specificity naturally or refactor
.button.button--primary { }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Global styles leaking
# Bad
button { }  # Affects ALL buttons
.active { }  # Too generic

# Good
.my-component button { }  # Scoped
.my-component--active { }  # Namespaced

# Or use CSS Modules for automatic scoping

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Inconsistent spacing/sizing
# Bad
.card { padding: 17px; }
.modal { padding: 23px; }

# Good - Use design tokens
.card { padding: var(--spacing-md); }
.modal { padding: var(--spacing-lg); }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Mixing approaches
# Bad
# Some files use BEM, some use random names
# Some use CSS Modules, some use global CSS

# Good
# Pick one approach and stick to it
# Document in style guide

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Not handling dark mode
# Bad
.card {
  background: white;  # Hardcoded
  color: black;
}

# Good
.card {
  background: var(--color-background);
  color: var(--color-text);
}
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is BEM?"**
> "BEM stands for Block Element Modifier - a naming convention for CSS. Block is a standalone component (.card), Element is a part of block (.card__title using __), Modifier is a variation (.card--featured using --). Benefits: flat specificity, clear relationships, predictable naming."

**Q: "CSS Modules vs CSS-in-JS - when to use each?"**
> "CSS Modules: zero runtime, better performance, familiar CSS syntax, works with any framework. CSS-in-JS: dynamic styles from props, co-located with components, theming built-in. Use Modules for performance-critical apps. Use CSS-in-JS when you need runtime theming or heavily dynamic styles."

**Q: "How does CSS specificity work?"**
> "Browser calculates specificity as a four-part number: inline, IDs, classes/attributes, elements. Higher specificity wins regardless of order. #id beats .class which beats div. Equal specificity: later rule wins. Avoid !important and high specificity - use flat selectors like single classes."

### Intermediate Questions

**Q: "How do you handle responsive design?"**
> "Mobile-first approach: start with mobile styles, use min-width media queries for larger screens. With Tailwind: responsive prefixes (md:, lg:). Use CSS Grid/Flexbox for layout. Define breakpoints as tokens. Test on actual devices. Consider container queries for component-level responsiveness."

**Q: "How do you organize CSS at scale?"**
> "Layers: tokens (variables), base (reset, typography), components, utilities. One approach per project (BEM, Modules, or Tailwind). Co-locate styles with components. Use design tokens for consistency. Document patterns. Review for duplicated styles. Track bundle size."

**Q: "What are CSS custom properties good for?"**
> "Theming (light/dark mode), design tokens, component variants without duplicating styles. They cascade, so you can override at any level. Can be changed with JavaScript at runtime. Reduce duplication by defining values once. Better than Sass variables because they work at runtime."

### Advanced Questions

**Q: "How do you optimize CSS for performance?"**
> "Critical CSS: inline above-fold styles. Lazy-load rest. Remove unused CSS (PurgeCSS, Tailwind's purge). Avoid expensive selectors (descendant combinators, universal). Use CSS containment. Minimize repaints/reflows. Consider code splitting CSS per route."

**Q: "Compare CSS-in-JS libraries and their trade-offs."**
> "styled-components/Emotion: mature, runtime styling, ~15KB. Linaria/vanilla-extract: zero-runtime, compile-time extraction, better performance. Stitches: near-zero runtime, variant API. Panda CSS: build-time extraction, Tailwind-like but type-safe. Choose based on: performance needs, dynamic styling requirements, DX preferences."

**Q: "How do you prevent CSS architecture from degrading over time?"**
> "Establish conventions in style guide. Use linting (stylelint) to enforce patterns. Code review for CSS. Track bundle size in CI. Regular audits for duplication and unused styles. Team education. Use TypeScript types for CSS Modules/CSS-in-JS. Automated visual regression testing."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              CSS ARCHITECTURE CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  METHODOLOGY:                                                   │
│  □ Pick one approach (BEM, Modules, CSS-in-JS, Tailwind)       │
│  □ Document in style guide                                     │
│  □ Enforce with linting                                        │
│                                                                 │
│  SPECIFICITY:                                                   │
│  □ Keep selectors flat (single class)                          │
│  □ Avoid IDs in selectors                                      │
│  □ No !important (except utilities)                            │
│                                                                 │
│  TOKENS:                                                        │
│  □ Use CSS custom properties                                   │
│  □ Define colors, spacing, typography                          │
│  □ Support theming (dark mode)                                 │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Purge unused CSS                                            │
│  □ Critical CSS inline                                         │
│  □ Monitor bundle size                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMPARISON:
┌────────────────────────────────────────────────────────────────┐
│ BEM:        Convention only, works everywhere, verbose         │
│ CSS Modules: Scoped, zero runtime, compile-time                │
│ CSS-in-JS:   Dynamic, co-located, runtime cost                 │
│ Tailwind:    Utility-first, fast dev, verbose HTML            │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

