# 🎨 Design Systems - Complete Guide

> A comprehensive guide to building and maintaining design systems - component libraries, design tokens, Storybook, documentation, and scaling design across teams.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "A design system is a collection of reusable components, design tokens, patterns, and guidelines that enable teams to build consistent, accessible, and maintainable user interfaces at scale."

### The 7 Key Concepts (Remember These!)
```
1. DESIGN TOKENS     → Variables for colors, spacing, typography
2. COMPONENT LIBRARY → Reusable UI building blocks
3. PATTERNS          → Common UI patterns and compositions
4. DOCUMENTATION     → Usage guidelines and examples
5. STORYBOOK         → Component development and showcase
6. THEMING           → Customizable design variations
7. GOVERNANCE        → Contribution and maintenance process
```

### Design System Layers
```
┌─────────────────────────────────────────────────────────────────┐
│              DESIGN SYSTEM ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    PATTERNS                              │   │
│  │     Login Form, Search, Navigation, Data Table          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   COMPONENTS                             │   │
│  │    Button, Input, Modal, Card, Dropdown, Toast          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 DESIGN TOKENS                            │   │
│  │   Colors, Typography, Spacing, Shadows, Animations      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   FOUNDATION                             │   │
│  │      Principles, Guidelines, Accessibility, Voice       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Component Maturity Model
```
┌─────────────────────────────────────────────────────────────────┐
│              COMPONENT MATURITY LEVELS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LEVEL 1: BASIC                                                │
│  ─────────────────                                              │
│  • Functional component                                        │
│  • Basic props                                                 │
│  • Works in isolation                                          │
│                                                                 │
│  LEVEL 2: DOCUMENTED                                           │
│  ─────────────────────                                          │
│  • Storybook stories                                           │
│  • Props documentation                                         │
│  • Usage examples                                              │
│                                                                 │
│  LEVEL 3: ACCESSIBLE                                           │
│  ─────────────────────                                          │
│  • ARIA attributes                                             │
│  • Keyboard navigation                                         │
│  • Screen reader tested                                        │
│                                                                 │
│  LEVEL 4: TESTED                                               │
│  ────────────────                                               │
│  • Unit tests                                                  │
│  • Visual regression                                           │
│  • Integration tests                                           │
│                                                                 │
│  LEVEL 5: PRODUCTION                                           │
│  ─────────────────────                                          │
│  • Performance optimized                                       │
│  • Edge cases handled                                          │
│  • Themeable                                                   │
│  • Stable API                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Design tokens"** | "Our design tokens define the visual language" |
| **"Atomic design"** | "We follow atomic design - atoms, molecules, organisms" |
| **"Component API"** | "The component API is carefully designed for flexibility" |
| **"Compound components"** | "We use compound components for complex UI" |
| **"Visual regression"** | "Chromatic catches visual regression before deploy" |
| **"Variant"** | "Button has primary, secondary, and ghost variants" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Components | **30-50** | Typical mature design system |
| Coverage | **80%+** | UI built with design system |
| Bundle size | **< 50KB** | Tree-shakeable imports |
| Adoption time | **6-12 months** | Full organization adoption |

### The "Wow" Statement (Memorize This!)
> "Our design system has three layers: tokens for visual language, components for building blocks, and patterns for common compositions. Tokens are platform-agnostic JSON transformed to CSS variables, iOS, Android. Components are React with TypeScript - each has Storybook stories, accessibility tests, and visual regression via Chromatic. We use compound components for flexibility - Select.Root, Select.Trigger, Select.Content. Variants follow consistent naming (primary, secondary, destructive). Everything is tree-shakeable - teams import only what they need. We track adoption via component analytics. Contributions follow RFC process - propose, review, implement, document. The system saves 40% development time and ensures brand consistency across 12 products."

---

## 📚 Table of Contents

1. [Design Tokens](#1-design-tokens)
2. [Component Architecture](#2-component-architecture)
3. [Storybook](#3-storybook)
4. [Theming](#4-theming)
5. [Documentation](#5-documentation)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Design Tokens

```typescript
// ════════════════════════════════════════════════════════════════
// DESIGN TOKENS STRUCTURE
// ════════════════════════════════════════════════════════════════

// tokens/colors.json
{
  "color": {
    "primitive": {
      "blue": {
        "50": { "value": "#eff6ff" },
        "100": { "value": "#dbeafe" },
        "500": { "value": "#3b82f6" },
        "600": { "value": "#2563eb" },
        "700": { "value": "#1d4ed8" }
      },
      "gray": {
        "50": { "value": "#f9fafb" },
        "100": { "value": "#f3f4f6" },
        "900": { "value": "#111827" }
      }
    },
    "semantic": {
      "brand": {
        "primary": { "value": "{color.primitive.blue.600}" },
        "secondary": { "value": "{color.primitive.gray.600}" }
      },
      "text": {
        "primary": { "value": "{color.primitive.gray.900}" },
        "secondary": { "value": "{color.primitive.gray.600}" },
        "muted": { "value": "{color.primitive.gray.400}" }
      },
      "background": {
        "default": { "value": "#ffffff" },
        "muted": { "value": "{color.primitive.gray.50}" },
        "accent": { "value": "{color.primitive.blue.50}" }
      },
      "border": {
        "default": { "value": "{color.primitive.gray.200}" },
        "focus": { "value": "{color.primitive.blue.500}" }
      },
      "status": {
        "success": { "value": "#10b981" },
        "warning": { "value": "#f59e0b" },
        "error": { "value": "#ef4444" },
        "info": { "value": "{color.primitive.blue.500}" }
      }
    }
  }
}

// tokens/typography.json
{
  "font": {
    "family": {
      "sans": { "value": "Inter, system-ui, sans-serif" },
      "mono": { "value": "JetBrains Mono, monospace" }
    },
    "size": {
      "xs": { "value": "0.75rem" },
      "sm": { "value": "0.875rem" },
      "base": { "value": "1rem" },
      "lg": { "value": "1.125rem" },
      "xl": { "value": "1.25rem" },
      "2xl": { "value": "1.5rem" },
      "3xl": { "value": "1.875rem" }
    },
    "weight": {
      "normal": { "value": "400" },
      "medium": { "value": "500" },
      "semibold": { "value": "600" },
      "bold": { "value": "700" }
    },
    "lineHeight": {
      "tight": { "value": "1.25" },
      "normal": { "value": "1.5" },
      "relaxed": { "value": "1.75" }
    }
  }
}

// tokens/spacing.json
{
  "spacing": {
    "0": { "value": "0" },
    "1": { "value": "0.25rem" },
    "2": { "value": "0.5rem" },
    "3": { "value": "0.75rem" },
    "4": { "value": "1rem" },
    "5": { "value": "1.25rem" },
    "6": { "value": "1.5rem" },
    "8": { "value": "2rem" },
    "10": { "value": "2.5rem" },
    "12": { "value": "3rem" },
    "16": { "value": "4rem" }
  }
}

// ════════════════════════════════════════════════════════════════
// TOKEN TRANSFORMATION (Style Dictionary)
// ════════════════════════════════════════════════════════════════

// style-dictionary.config.js
const StyleDictionary = require('style-dictionary');

module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    // CSS Custom Properties
    css: {
      transformGroup: 'css',
      buildPath: 'dist/css/',
      files: [{
        destination: 'tokens.css',
        format: 'css/variables',
        options: {
          outputReferences: true,
        },
      }],
    },
    // JavaScript/TypeScript
    js: {
      transformGroup: 'js',
      buildPath: 'dist/js/',
      files: [{
        destination: 'tokens.js',
        format: 'javascript/es6',
      }, {
        destination: 'tokens.d.ts',
        format: 'typescript/es6-declarations',
      }],
    },
    // iOS Swift
    ios: {
      transformGroup: 'ios-swift',
      buildPath: 'dist/ios/',
      files: [{
        destination: 'Tokens.swift',
        format: 'ios-swift/class.swift',
        className: 'DesignTokens',
      }],
    },
    // Android
    android: {
      transformGroup: 'android',
      buildPath: 'dist/android/',
      files: [{
        destination: 'tokens.xml',
        format: 'android/resources',
      }],
    },
  },
};

// Generated CSS output
:root {
  /* Primitive Colors */
  --color-primitive-blue-500: #3b82f6;
  --color-primitive-blue-600: #2563eb;
  
  /* Semantic Colors */
  --color-brand-primary: var(--color-primitive-blue-600);
  --color-text-primary: var(--color-primitive-gray-900);
  --color-background-default: #ffffff;
  
  /* Typography */
  --font-family-sans: Inter, system-ui, sans-serif;
  --font-size-base: 1rem;
  --font-weight-medium: 500;
  
  /* Spacing */
  --spacing-4: 1rem;
  --spacing-6: 1.5rem;
}
```

---

## 2. Component Architecture

```tsx
// ════════════════════════════════════════════════════════════════
// BUTTON COMPONENT (Full Example)
// ════════════════════════════════════════════════════════════════

// components/Button/Button.types.ts
import { ButtonHTMLAttributes, ReactNode } from 'react';

export type ButtonVariant = 'primary' | 'secondary' | 'ghost' | 'destructive';
export type ButtonSize = 'sm' | 'md' | 'lg';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  /** Visual variant of the button */
  variant?: ButtonVariant;
  /** Size of the button */
  size?: ButtonSize;
  /** Show loading spinner */
  isLoading?: boolean;
  /** Icon to show before text */
  leftIcon?: ReactNode;
  /** Icon to show after text */
  rightIcon?: ReactNode;
  /** Full width button */
  fullWidth?: boolean;
  /** Content */
  children: ReactNode;
}

// components/Button/Button.tsx
import React, { forwardRef } from 'react';
import { clsx } from 'clsx';
import { Spinner } from '../Spinner';
import type { ButtonProps } from './Button.types';
import styles from './Button.module.css';

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      variant = 'primary',
      size = 'md',
      isLoading = false,
      leftIcon,
      rightIcon,
      fullWidth = false,
      disabled,
      className,
      children,
      ...props
    },
    ref
  ) => {
    const isDisabled = disabled || isLoading;

    return (
      <button
        ref={ref}
        className={clsx(
          styles.button,
          styles[variant],
          styles[size],
          fullWidth && styles.fullWidth,
          isLoading && styles.loading,
          className
        )}
        disabled={isDisabled}
        aria-disabled={isDisabled}
        aria-busy={isLoading}
        {...props}
      >
        {isLoading && (
          <span className={styles.spinnerWrapper}>
            <Spinner size="sm" />
          </span>
        )}
        <span className={clsx(styles.content, isLoading && styles.invisible)}>
          {leftIcon && <span className={styles.leftIcon}>{leftIcon}</span>}
          {children}
          {rightIcon && <span className={styles.rightIcon}>{rightIcon}</span>}
        </span>
      </button>
    );
  }
);

Button.displayName = 'Button';

// components/Button/Button.module.css
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);
  border-radius: var(--radius-md);
  font-family: var(--font-family-sans);
  font-weight: var(--font-weight-medium);
  transition: all 150ms ease;
  cursor: pointer;
  border: none;
  position: relative;
}

.button:focus-visible {
  outline: 2px solid var(--color-border-focus);
  outline-offset: 2px;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background: var(--color-brand-primary);
  color: white;
}

.primary:hover:not(:disabled) {
  background: var(--color-primitive-blue-700);
}

.secondary {
  background: transparent;
  color: var(--color-text-primary);
  border: 1px solid var(--color-border-default);
}

.secondary:hover:not(:disabled) {
  background: var(--color-background-muted);
}

.ghost {
  background: transparent;
  color: var(--color-text-primary);
}

.ghost:hover:not(:disabled) {
  background: var(--color-background-muted);
}

.destructive {
  background: var(--color-status-error);
  color: white;
}

/* Sizes */
.sm {
  height: 32px;
  padding: 0 var(--spacing-3);
  font-size: var(--font-size-sm);
}

.md {
  height: 40px;
  padding: 0 var(--spacing-4);
  font-size: var(--font-size-base);
}

.lg {
  height: 48px;
  padding: 0 var(--spacing-6);
  font-size: var(--font-size-lg);
}

.fullWidth {
  width: 100%;
}

/* Loading state */
.loading {
  pointer-events: none;
}

.spinnerWrapper {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.invisible {
  visibility: hidden;
}

// ════════════════════════════════════════════════════════════════
// COMPOUND COMPONENT PATTERN (Select)
// ════════════════════════════════════════════════════════════════

// components/Select/Select.tsx
import React, { createContext, useContext, useState, useRef } from 'react';

interface SelectContextValue {
  open: boolean;
  setOpen: (open: boolean) => void;
  value: string;
  setValue: (value: string) => void;
}

const SelectContext = createContext<SelectContextValue | null>(null);

function useSelectContext() {
  const context = useContext(SelectContext);
  if (!context) {
    throw new Error('Select components must be used within Select.Root');
  }
  return context;
}

// Root component
interface SelectRootProps {
  children: React.ReactNode;
  defaultValue?: string;
  value?: string;
  onValueChange?: (value: string) => void;
}

function SelectRoot({ 
  children, 
  defaultValue = '', 
  value: controlledValue,
  onValueChange 
}: SelectRootProps) {
  const [open, setOpen] = useState(false);
  const [internalValue, setInternalValue] = useState(defaultValue);
  
  const isControlled = controlledValue !== undefined;
  const value = isControlled ? controlledValue : internalValue;
  
  const setValue = (newValue: string) => {
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onValueChange?.(newValue);
    setOpen(false);
  };

  return (
    <SelectContext.Provider value={{ open, setOpen, value, setValue }}>
      <div className="select-root">
        {children}
      </div>
    </SelectContext.Provider>
  );
}

// Trigger component
function SelectTrigger({ children, placeholder }: { 
  children?: React.ReactNode; 
  placeholder?: string;
}) {
  const { open, setOpen, value } = useSelectContext();
  
  return (
    <button
      type="button"
      className="select-trigger"
      onClick={() => setOpen(!open)}
      aria-expanded={open}
      aria-haspopup="listbox"
    >
      <span>{value || placeholder || 'Select...'}</span>
      <ChevronIcon className={open ? 'rotate-180' : ''} />
    </button>
  );
}

// Content component
function SelectContent({ children }: { children: React.ReactNode }) {
  const { open } = useSelectContext();
  
  if (!open) return null;
  
  return (
    <div className="select-content" role="listbox">
      {children}
    </div>
  );
}

// Item component
function SelectItem({ value, children }: { 
  value: string; 
  children: React.ReactNode;
}) {
  const { value: selectedValue, setValue } = useSelectContext();
  const isSelected = value === selectedValue;
  
  return (
    <div
      className={`select-item ${isSelected ? 'selected' : ''}`}
      role="option"
      aria-selected={isSelected}
      onClick={() => setValue(value)}
    >
      {children}
      {isSelected && <CheckIcon />}
    </div>
  );
}

// Export compound component
export const Select = {
  Root: SelectRoot,
  Trigger: SelectTrigger,
  Content: SelectContent,
  Item: SelectItem,
};

// Usage
<Select.Root value={country} onValueChange={setCountry}>
  <Select.Trigger placeholder="Select country" />
  <Select.Content>
    <Select.Item value="us">United States</Select.Item>
    <Select.Item value="uk">United Kingdom</Select.Item>
    <Select.Item value="ca">Canada</Select.Item>
  </Select.Content>
</Select.Root>
```

---

## 3. Storybook

```tsx
// ════════════════════════════════════════════════════════════════
// BUTTON STORIES
// ════════════════════════════════════════════════════════════════

// components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { PlusIcon, ArrowRightIcon } from '../icons';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'destructive'],
      description: 'Visual style variant',
      table: {
        defaultValue: { summary: 'primary' },
      },
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
      description: 'Button size',
      table: {
        defaultValue: { summary: 'md' },
      },
    },
    isLoading: {
      control: 'boolean',
      description: 'Show loading state',
    },
    disabled: {
      control: 'boolean',
      description: 'Disable the button',
    },
    fullWidth: {
      control: 'boolean',
      description: 'Full width button',
    },
  },
  args: {
    children: 'Button',
    variant: 'primary',
    size: 'md',
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

// Basic story
export const Default: Story = {};

// Variants
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Ghost: Story = {
  args: {
    variant: 'ghost',
    children: 'Ghost Button',
  },
};

export const Destructive: Story = {
  args: {
    variant: 'destructive',
    children: 'Delete',
  },
};

// Sizes
export const Sizes: Story = {
  render: () => (
    <div style={{ display: 'flex', alignItems: 'center', gap: '1rem' }}>
      <Button size="sm">Small</Button>
      <Button size="md">Medium</Button>
      <Button size="lg">Large</Button>
    </div>
  ),
};

// With icons
export const WithIcons: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem' }}>
      <Button leftIcon={<PlusIcon />}>Add Item</Button>
      <Button rightIcon={<ArrowRightIcon />}>Continue</Button>
      <Button leftIcon={<PlusIcon />} rightIcon={<ArrowRightIcon />}>
        Both Icons
      </Button>
    </div>
  ),
};

// States
export const Loading: Story = {
  args: {
    isLoading: true,
    children: 'Saving...',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled',
  },
};

// All variants showcase
export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', flexDirection: 'column', gap: '1rem' }}>
      {(['primary', 'secondary', 'ghost', 'destructive'] as const).map(
        (variant) => (
          <div key={variant} style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
            <span style={{ width: '100px' }}>{variant}:</span>
            <Button variant={variant} size="sm">Small</Button>
            <Button variant={variant} size="md">Medium</Button>
            <Button variant={variant} size="lg">Large</Button>
            <Button variant={variant} isLoading>Loading</Button>
            <Button variant={variant} disabled>Disabled</Button>
          </div>
        )
      )}
    </div>
  ),
};

// ════════════════════════════════════════════════════════════════
// STORYBOOK CONFIGURATION
// ════════════════════════════════════════════════════════════════

// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;

// .storybook/preview.tsx
import type { Preview } from '@storybook/react';
import '../src/styles/tokens.css';
import '../src/styles/global.css';

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#1a1a1a' },
      ],
    },
  },
  decorators: [
    (Story) => (
      <div style={{ padding: '2rem' }}>
        <Story />
      </div>
    ),
  ],
};

export default preview;

// ════════════════════════════════════════════════════════════════
// VISUAL REGRESSION (Chromatic)
// ════════════════════════════════════════════════════════════════

// .github/workflows/chromatic.yml
name: Chromatic

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  chromatic:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Publish to Chromatic
        uses: chromaui/action@v1
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          buildScriptName: 'build-storybook'
```

---

## 4. Theming

```tsx
// ════════════════════════════════════════════════════════════════
// THEME SYSTEM
// ════════════════════════════════════════════════════════════════

// themes/tokens.ts
export const lightTheme = {
  colors: {
    background: {
      default: '#ffffff',
      muted: '#f9fafb',
      accent: '#eff6ff',
    },
    text: {
      primary: '#111827',
      secondary: '#6b7280',
      muted: '#9ca3af',
    },
    border: {
      default: '#e5e7eb',
      focus: '#3b82f6',
    },
    brand: {
      primary: '#2563eb',
      primaryHover: '#1d4ed8',
    },
  },
};

export const darkTheme = {
  colors: {
    background: {
      default: '#0f172a',
      muted: '#1e293b',
      accent: '#1e3a5f',
    },
    text: {
      primary: '#f1f5f9',
      secondary: '#94a3b8',
      muted: '#64748b',
    },
    border: {
      default: '#334155',
      focus: '#60a5fa',
    },
    brand: {
      primary: '#3b82f6',
      primaryHover: '#60a5fa',
    },
  },
};

// ════════════════════════════════════════════════════════════════
// THEME PROVIDER
// ════════════════════════════════════════════════════════════════

// providers/ThemeProvider.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { lightTheme, darkTheme } from '../themes/tokens';

type Theme = 'light' | 'dark' | 'system';
type ResolvedTheme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  resolvedTheme: ResolvedTheme;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

function getSystemTheme(): ResolvedTheme {
  if (typeof window === 'undefined') return 'light';
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  const [resolvedTheme, setResolvedTheme] = useState<ResolvedTheme>('light');

  useEffect(() => {
    const stored = localStorage.getItem('theme') as Theme | null;
    if (stored) setTheme(stored);
  }, []);

  useEffect(() => {
    const resolved = theme === 'system' ? getSystemTheme() : theme;
    setResolvedTheme(resolved);
    
    // Apply theme
    const root = document.documentElement;
    root.setAttribute('data-theme', resolved);
    
    // Apply CSS variables
    const tokens = resolved === 'dark' ? darkTheme : lightTheme;
    Object.entries(tokens.colors).forEach(([category, values]) => {
      Object.entries(values).forEach(([name, value]) => {
        root.style.setProperty(`--color-${category}-${name}`, value);
      });
    });
    
    localStorage.setItem('theme', theme);
  }, [theme]);

  // Listen for system theme changes
  useEffect(() => {
    if (theme !== 'system') return;
    
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const handler = () => setResolvedTheme(getSystemTheme());
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, resolvedTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// ════════════════════════════════════════════════════════════════
// THEME TOGGLE COMPONENT
// ════════════════════════════════════════════════════════════════

import { useTheme } from '../providers/ThemeProvider';
import { SunIcon, MoonIcon, ComputerIcon } from '../icons';

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <div className="theme-toggle" role="radiogroup" aria-label="Theme">
      <button
        role="radio"
        aria-checked={theme === 'light'}
        onClick={() => setTheme('light')}
        className={theme === 'light' ? 'active' : ''}
      >
        <SunIcon />
        <span className="sr-only">Light</span>
      </button>
      <button
        role="radio"
        aria-checked={theme === 'dark'}
        onClick={() => setTheme('dark')}
        className={theme === 'dark' ? 'active' : ''}
      >
        <MoonIcon />
        <span className="sr-only">Dark</span>
      </button>
      <button
        role="radio"
        aria-checked={theme === 'system'}
        onClick={() => setTheme('system')}
        className={theme === 'system' ? 'active' : ''}
      >
        <ComputerIcon />
        <span className="sr-only">System</span>
      </button>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// CSS WITH THEME SUPPORT
// ════════════════════════════════════════════════════════════════

/* styles/themes.css */
:root {
  /* Light theme (default) */
  --color-background-default: #ffffff;
  --color-text-primary: #111827;
}

[data-theme="dark"] {
  --color-background-default: #0f172a;
  --color-text-primary: #f1f5f9;
}

/* Component uses CSS variables */
.card {
  background: var(--color-background-default);
  color: var(--color-text-primary);
  border: 1px solid var(--color-border-default);
}
```

---

## 5. Documentation

```tsx
// ════════════════════════════════════════════════════════════════
// COMPONENT DOCUMENTATION (MDX)
// ════════════════════════════════════════════════════════════════

// components/Button/Button.mdx
import { Canvas, Meta, Story, Controls, Description } from '@storybook/blocks';
import * as ButtonStories from './Button.stories';

<Meta of={ButtonStories} />

# Button

<Description of={ButtonStories} />

Buttons trigger actions or events when clicked. Use the appropriate variant
based on the action's importance and context.

## Usage

```tsx
import { Button } from '@mycompany/design-system';

function Example() {
  return (
    <Button variant="primary" onClick={handleClick}>
      Click me
    </Button>
  );
}
```

## Variants

Use different variants to communicate the importance of actions:

- **Primary**: Main call-to-action, use once per section
- **Secondary**: Alternative actions
- **Ghost**: Tertiary actions, minimal visual weight
- **Destructive**: Dangerous or irreversible actions

<Canvas of={ButtonStories.AllVariants} />

## Props

<Controls />

## Accessibility

- Uses native `<button>` element
- Supports keyboard navigation (Enter, Space)
- `aria-disabled` when disabled (allows focus for screen readers)
- `aria-busy` during loading state
- Minimum touch target size: 44x44px

## Do's and Don'ts

### ✅ Do
- Use clear, action-oriented labels ("Save", "Submit", "Delete")
- Use primary variant for main action
- Provide loading state for async actions
- Include icon + text for better clarity

### ❌ Don't
- Don't use multiple primary buttons in one area
- Don't use vague labels ("Click here", "Submit")
- Don't disable without explanation
- Don't use button for navigation (use Link)

// ════════════════════════════════════════════════════════════════
// DESIGN SYSTEM DOCUMENTATION SITE
// ════════════════════════════════════════════════════════════════

// pages/components/button.tsx (Documentation site)
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/Tabs';
import { CodeBlock } from '@/components/CodeBlock';
import { PropsTable } from '@/components/PropsTable';
import { Button } from '@mycompany/design-system';

export default function ButtonDocs() {
  return (
    <article className="docs-page">
      <header>
        <h1>Button</h1>
        <p className="lead">
          Interactive element that triggers actions when clicked.
        </p>
      </header>

      <section className="preview">
        <div className="component-preview">
          <Button variant="primary">Primary</Button>
          <Button variant="secondary">Secondary</Button>
          <Button variant="ghost">Ghost</Button>
        </div>
      </section>

      <Tabs defaultValue="usage">
        <TabsList>
          <TabsTrigger value="usage">Usage</TabsTrigger>
          <TabsTrigger value="props">Props</TabsTrigger>
          <TabsTrigger value="accessibility">Accessibility</TabsTrigger>
        </TabsList>

        <TabsContent value="usage">
          <h2>Installation</h2>
          <CodeBlock language="bash">
            npm install @mycompany/design-system
          </CodeBlock>

          <h2>Import</h2>
          <CodeBlock language="tsx">
            {`import { Button } from '@mycompany/design-system';`}
          </CodeBlock>

          <h2>Examples</h2>
          <h3>Basic</h3>
          <CodeBlock language="tsx">
            {`<Button onClick={handleClick}>Click me</Button>`}
          </CodeBlock>

          <h3>With Loading</h3>
          <CodeBlock language="tsx">
            {`<Button isLoading={isSaving}>
  {isSaving ? 'Saving...' : 'Save'}
</Button>`}
          </CodeBlock>

          <h3>With Icons</h3>
          <CodeBlock language="tsx">
            {`<Button leftIcon={<PlusIcon />}>
  Add Item
</Button>`}
          </CodeBlock>
        </TabsContent>

        <TabsContent value="props">
          <PropsTable
            props={[
              {
                name: 'variant',
                type: "'primary' | 'secondary' | 'ghost' | 'destructive'",
                default: "'primary'",
                description: 'Visual style variant',
              },
              {
                name: 'size',
                type: "'sm' | 'md' | 'lg'",
                default: "'md'",
                description: 'Button size',
              },
              {
                name: 'isLoading',
                type: 'boolean',
                default: 'false',
                description: 'Show loading spinner',
              },
              {
                name: 'leftIcon',
                type: 'ReactNode',
                default: 'undefined',
                description: 'Icon before text',
              },
              {
                name: 'rightIcon',
                type: 'ReactNode',
                default: 'undefined',
                description: 'Icon after text',
              },
            ]}
          />
        </TabsContent>

        <TabsContent value="accessibility">
          <h2>Keyboard Interactions</h2>
          <table className="keyboard-table">
            <thead>
              <tr>
                <th>Key</th>
                <th>Action</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><kbd>Enter</kbd></td>
                <td>Activates the button</td>
              </tr>
              <tr>
                <td><kbd>Space</kbd></td>
                <td>Activates the button</td>
              </tr>
            </tbody>
          </table>

          <h2>ARIA</h2>
          <ul>
            <li><code>aria-disabled</code> when disabled</li>
            <li><code>aria-busy</code> when loading</li>
          </ul>
        </TabsContent>
      </Tabs>
    </article>
  );
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# DESIGN SYSTEM PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Building components no one uses
# Bad
# Build 50 components upfront
# Teams build their own anyway

# Good
# Start with most-used components
# Add based on actual needs
# Track component usage analytics

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Too rigid / Too flexible
# Bad - Too rigid
<Button type="submit" />  # Only one way to use

# Bad - Too flexible
<Button 
  borderRadius={8}
  bgColor="#custom"
  fontSize={14}
/>  # Every prop customizable

# Good - Constrained flexibility
<Button variant="primary" size="md" />
# Predefined variants, limited customization

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: No versioning strategy
# Bad
# Push breaking changes to main
# All consumers break

# Good
# Semantic versioning
# Deprecation warnings
# Migration guides
# Breaking changes in major versions only

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Forgetting accessibility
# Bad
<div onClick={handleClick}>Click me</div>

# Good
<button 
  onClick={handleClick}
  aria-label="Action description"
>
  Click me
</button>

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Tight coupling to one framework
# Bad
# All tokens only in styled-components
const Button = styled.button`
  color: ${props => props.theme.colors.primary};
`;

# Good
# Tokens as CSS variables (framework-agnostic)
.button {
  color: var(--color-brand-primary);
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: No contribution process
# Bad
# Core team only maintains
# Backlog grows, teams wait

# Good
# RFC process for new components
# Contribution guidelines
# Review checklist
# Shared ownership
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is a design system?"**
> "A design system is reusable components, design tokens, patterns, and guidelines that enable consistent UI development at scale. It includes visual language (tokens), building blocks (components), and documentation. Benefits: consistency, faster development, easier maintenance, better accessibility."

**Q: "What are design tokens?"**
> "Design tokens are the atomic values of a design system - colors, spacing, typography, shadows. They're platform-agnostic (JSON) and transform to CSS variables, iOS Swift, Android XML. Using tokens instead of hardcoded values enables theming and ensures consistency. Semantic tokens (brand-primary) reference primitive tokens (blue-600)."

**Q: "What is Storybook?"**
> "Storybook is a tool for developing and documenting UI components in isolation. Each component has 'stories' showing different states. Benefits: develop without running full app, visual documentation, design review, accessibility testing, visual regression with Chromatic. Stories serve as living documentation."

### Intermediate Questions

**Q: "How do you handle theming?"**
> "CSS custom properties for tokens, updated at runtime. ThemeProvider sets `data-theme` attribute, CSS selectors apply different values. Support light/dark/system. Store preference in localStorage. Listen for system preference changes. Components use tokens (--color-text-primary), not hardcoded values."

**Q: "What is the compound component pattern?"**
> "Breaking complex components into composable pieces that share implicit state. Instead of one component with many props, multiple components work together. Example: Select.Root, Select.Trigger, Select.Content, Select.Item. Benefits: flexibility, cleaner API, explicit composition. Used by Radix, Headless UI."

**Q: "How do you ensure accessibility in a design system?"**
> "Every component: native HTML elements, ARIA attributes, keyboard navigation, focus management. Process: use axe-core in Storybook, manual testing with screen readers, WCAG checklist in review. Include accessibility docs per component. Test in multiple assistive technologies."

### Advanced Questions

**Q: "How do you measure design system adoption?"**
> "Component analytics: track imports, render counts. Code analysis: grep for component usage vs. custom implementations. Survey teams: satisfaction, missing features. Metrics: coverage (% UI using system), time saved, consistency (design audits). Dashboard showing adoption over time."

**Q: "How do you handle breaking changes?"**
> "Semantic versioning: breaking changes only in major versions. Deprecation warnings in minor versions (console.warn). Migration codemods where possible. Detailed migration guides. Allow gradual migration with compatibility period. Communicate timeline to teams."

**Q: "How do you scale a design system across multiple products?"**
> "Core tokens and base components shared. Product-specific themes extend base. Publish to npm with tree-shaking. Mono-repo with packages or federated approach. Per-product customization via theming, not forking. Governance: RFC process, cross-team reviews. Balance consistency with flexibility."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              DESIGN SYSTEM CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TOKENS:                                                        │
│  □ Colors (primitive + semantic)                               │
│  □ Typography (family, size, weight)                           │
│  □ Spacing scale                                               │
│  □ Shadows, borders, radii                                     │
│  □ Multi-platform output (CSS, iOS, Android)                   │
│                                                                 │
│  COMPONENTS:                                                    │
│  □ TypeScript + proper types                                   │
│  □ Storybook stories                                           │
│  □ Accessibility (ARIA, keyboard)                              │
│  □ Tests (unit, visual regression)                             │
│  □ Documentation                                               │
│                                                                 │
│  INFRASTRUCTURE:                                                │
│  □ npm package with tree-shaking                               │
│  □ Semantic versioning                                         │
│  □ Chromatic for visual regression                             │
│  □ CI/CD pipeline                                              │
│                                                                 │
│  GOVERNANCE:                                                    │
│  □ Contribution guidelines                                     │
│  □ RFC process                                                 │
│  □ Review checklist                                            │
│  □ Adoption tracking                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

