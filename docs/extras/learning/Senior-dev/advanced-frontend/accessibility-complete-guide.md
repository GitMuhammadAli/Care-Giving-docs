# ♿ Accessibility (a11y) - Complete Guide

> A comprehensive guide to web accessibility - WCAG guidelines, ARIA, keyboard navigation, screen readers, and building inclusive user experiences.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Web accessibility (a11y) ensures websites and applications are usable by everyone, including people with disabilities, by following WCAG guidelines for perceivable, operable, understandable, and robust content."

### The 7 Key Concepts (Remember These!)
```
1. WCAG           → Web Content Accessibility Guidelines (2.1/2.2)
2. POUR           → Perceivable, Operable, Understandable, Robust
3. ARIA           → Accessible Rich Internet Applications attributes
4. SEMANTIC HTML  → Use correct elements (button, not div)
5. KEYBOARD NAV   → All functionality via keyboard
6. FOCUS MGMT     → Visible focus, logical order
7. SCREEN READERS → Alternative content for visual elements
```

### WCAG POUR Principles
```
┌─────────────────────────────────────────────────────────────────┐
│              WCAG POUR PRINCIPLES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PERCEIVABLE - Can users perceive the content?                 │
│  ────────────────────────────────────────────                   │
│  • Text alternatives for images (alt text)                     │
│  • Captions for video/audio                                    │
│  • Sufficient color contrast (4.5:1 text, 3:1 large)           │
│  • Don't rely on color alone                                   │
│  • Resizable text up to 200%                                   │
│                                                                 │
│  OPERABLE - Can users operate the interface?                   │
│  ─────────────────────────────────────────────                  │
│  • Keyboard accessible (no mouse required)                     │
│  • Enough time to read/interact                                │
│  • No seizure-inducing flashing (< 3/second)                   │
│  • Skip navigation links                                       │
│  • Focus visible                                               │
│                                                                 │
│  UNDERSTANDABLE - Can users understand content?                │
│  ────────────────────────────────────────────                   │
│  • Language of page declared                                   │
│  • Consistent navigation                                       │
│  • Input errors identified and described                       │
│  • Labels and instructions provided                            │
│                                                                 │
│  ROBUST - Works with assistive technologies?                   │
│  ────────────────────────────────────────────                   │
│  • Valid HTML                                                  │
│  • Name, role, value for custom components                     │
│  • Works across browsers and AT                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### WCAG Conformance Levels
```
┌─────────────────────────────────────────────────────────────────┐
│              WCAG CONFORMANCE LEVELS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LEVEL A (Minimum)                                             │
│  ─────────────────                                              │
│  • All images have alt text                                    │
│  • All form inputs have labels                                 │
│  • Keyboard navigation works                                   │
│  • No keyboard traps                                           │
│  "The basics that prevent total barriers"                      │
│                                                                 │
│  LEVEL AA (Standard) ← MOST COMMON TARGET                      │
│  ────────────────────                                           │
│  • All Level A requirements                                    │
│  • 4.5:1 color contrast (text)                                 │
│  • Resize text to 200%                                         │
│  • Skip navigation links                                       │
│  • Focus visible                                               │
│  "Legal requirement in many countries"                         │
│                                                                 │
│  LEVEL AAA (Enhanced)                                          │
│  ────────────────────                                           │
│  • All Level AA requirements                                   │
│  • 7:1 color contrast                                          │
│  • Sign language for video                                     │
│  • Extended audio descriptions                                 │
│  "Highest level, not always achievable"                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"WCAG 2.1 AA"** | "We target WCAG 2.1 AA compliance" |
| **"Semantic markup"** | "Semantic markup provides meaning to assistive tech" |
| **"Focus management"** | "Modals need proper focus management" |
| **"Live region"** | "We use aria-live for dynamic content announcements" |
| **"Landmark roles"** | "Landmark roles help screen reader navigation" |
| **"Accessible name"** | "Every interactive element needs an accessible name" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Text contrast | **4.5:1** | WCAG AA normal text |
| Large text contrast | **3:1** | 18pt+ or 14pt bold |
| Focus visible | **Required** | 2.4.7 WCAG AA |
| Touch target | **44×44px** | Minimum touch size |

### The "Wow" Statement (Memorize This!)
> "We build accessibility in from the start, not bolt it on. Every component uses semantic HTML - buttons are buttons, not clickable divs. We run axe-core in CI to catch issues early. Custom components get proper ARIA: roles, states, properties. Focus management is critical - modals trap focus, return focus on close. We test with VoiceOver and NVDA, not just automated tools. Color contrast is checked in design handoff. All images have meaningful alt text. Forms have associated labels and error messages. We target WCAG 2.1 AA - it's both the right thing to do and a legal requirement. Accessibility benefits everyone: captions help in noisy environments, keyboard nav helps power users."

---

## 📚 Table of Contents

1. [Semantic HTML](#1-semantic-html)
2. [ARIA](#2-aria)
3. [Keyboard Navigation](#3-keyboard-navigation)
4. [Screen Readers](#4-screen-readers)
5. [Forms](#5-forms)
6. [Testing](#6-testing)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Interview Questions](#8-interview-questions)

---

## 1. Semantic HTML

```html
<!-- ════════════════════════════════════════════════════════════════ -->
<!-- SEMANTIC STRUCTURE                                               -->
<!-- ════════════════════════════════════════════════════════════════ -->

<!-- ❌ BAD: Divs for everything -->
<div class="header">
  <div class="nav">
    <div class="nav-item" onclick="navigate()">Home</div>
  </div>
</div>
<div class="main">
  <div class="article">
    <div class="title">Article Title</div>
  </div>
  <div class="sidebar">
    <div class="widget">Related</div>
  </div>
</div>
<div class="footer">© 2024</div>

<!-- ✅ GOOD: Semantic elements -->
<header>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>Article Title</h1>
    <p>Content...</p>
  </article>
  
  <aside aria-label="Related content">
    <h2>Related Articles</h2>
    <ul>
      <li><a href="/related-1">Related 1</a></li>
    </ul>
  </aside>
</main>

<footer>
  <p>© 2024 Company Name</p>
</footer>

<!-- ════════════════════════════════════════════════════════════════ -->
<!-- HEADINGS HIERARCHY                                               -->
<!-- ════════════════════════════════════════════════════════════════ -->

<!-- ❌ BAD: Skipping heading levels -->
<h1>Page Title</h1>
<h3>Section</h3>  <!-- Skipped h2 -->
<h5>Subsection</h5>  <!-- Skipped h4 -->

<!-- ✅ GOOD: Proper hierarchy -->
<h1>Page Title</h1>
  <h2>Section 1</h2>
    <h3>Subsection 1.1</h3>
    <h3>Subsection 1.2</h3>
  <h2>Section 2</h2>
    <h3>Subsection 2.1</h3>

<!-- ════════════════════════════════════════════════════════════════ -->
<!-- INTERACTIVE ELEMENTS                                             -->
<!-- ════════════════════════════════════════════════════════════════ -->

<!-- ❌ BAD: Div as button -->
<div 
  class="button" 
  onclick="handleClick()"
>
  Click me
</div>

<!-- ✅ GOOD: Actual button -->
<button 
  type="button" 
  onclick="handleClick()"
>
  Click me
</button>

<!-- ❌ BAD: Button for navigation -->
<button onclick="window.location='/about'">
  About Us
</button>

<!-- ✅ GOOD: Link for navigation -->
<a href="/about">About Us</a>

<!-- ════════════════════════════════════════════════════════════════ -->
<!-- IMAGES AND MEDIA                                                 -->
<!-- ════════════════════════════════════════════════════════════════ -->

<!-- Informative image -->
<img 
  src="chart.png" 
  alt="Sales increased 25% from Q1 to Q2 2024"
/>

<!-- Decorative image (empty alt) -->
<img src="decorative-border.png" alt="" />

<!-- Complex image with longer description -->
<figure>
  <img 
    src="complex-diagram.png" 
    alt="System architecture diagram"
    aria-describedby="diagram-desc"
  />
  <figcaption id="diagram-desc">
    The system consists of three layers: presentation, business logic, 
    and data access. The presentation layer communicates with...
  </figcaption>
</figure>

<!-- Icon with text (decorative) -->
<button>
  <svg aria-hidden="true" focusable="false">...</svg>
  Save
</button>

<!-- Icon-only button (needs label) -->
<button aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false">
    <path d="M6 6L18 18M6 18L18 6" />
  </svg>
</button>
```

---

## 2. ARIA

```tsx
// ════════════════════════════════════════════════════════════════
// ARIA ROLES, STATES, AND PROPERTIES
// ════════════════════════════════════════════════════════════════

// RULE 1: Don't use ARIA if native HTML works
// ❌ BAD
<div role="button" tabindex="0" onclick={handleClick}>
  Click me
</div>

// ✅ GOOD
<button onClick={handleClick}>Click me</button>

// ════════════════════════════════════════════════════════════════
// COMMON ARIA PATTERNS
// ════════════════════════════════════════════════════════════════

// Expanded/Collapsed (Accordion, Dropdown)
function Accordion({ title, children, isOpen, onToggle }) {
  const contentId = useId();
  
  return (
    <div className="accordion">
      <button
        aria-expanded={isOpen}
        aria-controls={contentId}
        onClick={onToggle}
      >
        {title}
        <ChevronIcon aria-hidden="true" />
      </button>
      <div
        id={contentId}
        role="region"
        aria-labelledby={`${contentId}-header`}
        hidden={!isOpen}
      >
        {children}
      </div>
    </div>
  );
}

// Tabs
function Tabs({ tabs, activeTab, onTabChange }) {
  return (
    <div className="tabs">
      <div role="tablist" aria-label="Settings tabs">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            role="tab"
            id={`tab-${tab.id}`}
            aria-selected={activeTab === index}
            aria-controls={`panel-${tab.id}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => onTabChange(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}

// Modal Dialog
function Modal({ isOpen, onClose, title, children }) {
  const titleId = useId();
  const descId = useId();
  
  if (!isOpen) return null;
  
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
      aria-describedby={descId}
    >
      <h2 id={titleId}>{title}</h2>
      <div id={descId}>{children}</div>
      <button onClick={onClose}>Close</button>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// LIVE REGIONS (Dynamic Content)
// ════════════════════════════════════════════════════════════════

// Polite announcement (doesn't interrupt)
function StatusMessage({ message }) {
  return (
    <div 
      role="status" 
      aria-live="polite"
      aria-atomic="true"
    >
      {message}
    </div>
  );
}

// Alert (interrupts, important)
function AlertMessage({ message }) {
  return (
    <div 
      role="alert" 
      aria-live="assertive"
    >
      {message}
    </div>
  );
}

// Toast notifications
function ToastContainer({ toasts }) {
  return (
    <div 
      aria-live="polite" 
      aria-label="Notifications"
      className="toast-container"
    >
      {toasts.map(toast => (
        <div 
          key={toast.id}
          role="status"
          className={`toast toast-${toast.type}`}
        >
          {toast.message}
        </div>
      ))}
    </div>
  );
}

// Loading states
function LoadingButton({ isLoading, children, ...props }) {
  return (
    <button
      {...props}
      aria-busy={isLoading}
      disabled={isLoading}
    >
      {isLoading ? (
        <>
          <Spinner aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}

// ════════════════════════════════════════════════════════════════
// LANDMARK ROLES
// ════════════════════════════════════════════════════════════════

// Page structure with landmarks
function PageLayout() {
  return (
    <>
      <header role="banner">
        <nav role="navigation" aria-label="Main">
          {/* Primary navigation */}
        </nav>
      </header>
      
      <main role="main">
        {/* Main content */}
        <form role="search" aria-label="Site search">
          <input type="search" aria-label="Search" />
          <button type="submit">Search</button>
        </form>
      </main>
      
      <aside role="complementary" aria-label="Related content">
        {/* Sidebar */}
      </aside>
      
      <footer role="contentinfo">
        {/* Footer */}
      </footer>
    </>
  );
}
```

---

## 3. Keyboard Navigation

```tsx
// ════════════════════════════════════════════════════════════════
// FOCUS MANAGEMENT
// ════════════════════════════════════════════════════════════════

// Focus trap for modals
import { useEffect, useRef } from 'react';

function useFocusTrap(isActive: boolean) {
  const containerRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (!isActive || !containerRef.current) return;

    // Store previous focus
    previousFocusRef.current = document.activeElement as HTMLElement;

    // Get focusable elements
    const focusableElements = containerRef.current.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];

    // Focus first element
    firstElement?.focus();

    // Trap focus
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;

      if (e.shiftKey) {
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement?.focus();
        }
      } else {
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement?.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);

    return () => {
      document.removeEventListener('handleKeyDown', handleKeyDown);
      // Restore focus
      previousFocusRef.current?.focus();
    };
  }, [isActive]);

  return containerRef;
}

// Usage in modal
function Modal({ isOpen, onClose, children }) {
  const containerRef = useFocusTrap(isOpen);

  // Close on Escape
  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };
    
    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      return () => document.removeEventListener('keydown', handleEscape);
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={containerRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        onClick={e => e.stopPropagation()}
      >
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// ROVING TABINDEX (Composite Widgets)
// ════════════════════════════════════════════════════════════════

function useRovingTabIndex<T extends HTMLElement>(
  items: any[],
  options: { direction?: 'horizontal' | 'vertical' | 'both' } = {}
) {
  const [activeIndex, setActiveIndex] = useState(0);
  const itemRefs = useRef<(T | null)[]>([]);

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    const { direction = 'horizontal' } = options;
    let nextIndex = index;

    switch (e.key) {
      case 'ArrowRight':
        if (direction === 'horizontal' || direction === 'both') {
          nextIndex = index < items.length - 1 ? index + 1 : 0;
        }
        break;
      case 'ArrowLeft':
        if (direction === 'horizontal' || direction === 'both') {
          nextIndex = index > 0 ? index - 1 : items.length - 1;
        }
        break;
      case 'ArrowDown':
        if (direction === 'vertical' || direction === 'both') {
          nextIndex = index < items.length - 1 ? index + 1 : 0;
        }
        break;
      case 'ArrowUp':
        if (direction === 'vertical' || direction === 'both') {
          nextIndex = index > 0 ? index - 1 : items.length - 1;
        }
        break;
      case 'Home':
        nextIndex = 0;
        break;
      case 'End':
        nextIndex = items.length - 1;
        break;
      default:
        return;
    }

    e.preventDefault();
    setActiveIndex(nextIndex);
    itemRefs.current[nextIndex]?.focus();
  };

  const getItemProps = (index: number) => ({
    ref: (el: T | null) => (itemRefs.current[index] = el),
    tabIndex: index === activeIndex ? 0 : -1,
    onKeyDown: (e: React.KeyboardEvent) => handleKeyDown(e, index),
    onFocus: () => setActiveIndex(index),
  });

  return { activeIndex, getItemProps };
}

// Usage: Toolbar
function Toolbar({ items }) {
  const { getItemProps } = useRovingTabIndex(items, { direction: 'horizontal' });

  return (
    <div role="toolbar" aria-label="Formatting">
      {items.map((item, index) => (
        <button
          key={item.id}
          {...getItemProps(index)}
          aria-pressed={item.active}
        >
          {item.icon}
          <span className="sr-only">{item.label}</span>
        </button>
      ))}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// SKIP LINKS
// ════════════════════════════════════════════════════════════════

function SkipLinks() {
  return (
    <div className="skip-links">
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <a href="#main-navigation" className="skip-link">
        Skip to navigation
      </a>
    </div>
  );
}

// CSS for skip links
/*
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
*/
```

---

## 4. Screen Readers

```tsx
// ════════════════════════════════════════════════════════════════
// SCREEN READER ONLY CONTENT
// ════════════════════════════════════════════════════════════════

// CSS class for visually hidden content
/*
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
*/

// Component
function ScreenReaderOnly({ children }: { children: React.ReactNode }) {
  return <span className="sr-only">{children}</span>;
}

// Usage examples
function IconButton({ icon, label, onClick }) {
  return (
    <button onClick={onClick} aria-label={label}>
      {icon}
    </button>
  );
}

// Badge with count
function NotificationBadge({ count }) {
  return (
    <button>
      <BellIcon aria-hidden="true" />
      {count > 0 && (
        <>
          <span className="badge">{count}</span>
          <span className="sr-only">
            {count} unread notification{count !== 1 ? 's' : ''}
          </span>
        </>
      )}
    </button>
  );
}

// Progress indicator
function ProgressBar({ value, max = 100 }) {
  const percentage = Math.round((value / max) * 100);
  
  return (
    <div
      role="progressbar"
      aria-valuenow={value}
      aria-valuemin={0}
      aria-valuemax={max}
      aria-label={`Progress: ${percentage}%`}
    >
      <div className="progress-fill" style={{ width: `${percentage}%` }} />
      <span className="sr-only">{percentage}% complete</span>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// ANNOUNCEMENTS FOR DYNAMIC CONTENT
// ════════════════════════════════════════════════════════════════

// Announcement hook
function useAnnouncement() {
  const [announcement, setAnnouncement] = useState('');

  const announce = useCallback((message: string, priority: 'polite' | 'assertive' = 'polite') => {
    // Clear first to ensure re-announcement
    setAnnouncement('');
    setTimeout(() => setAnnouncement(message), 100);
  }, []);

  const Announcer = () => (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    >
      {announcement}
    </div>
  );

  return { announce, Announcer };
}

// Usage
function SearchResults() {
  const { announce, Announcer } = useAnnouncement();
  const [results, setResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  const search = async (query: string) => {
    setIsLoading(true);
    announce('Searching...');
    
    const data = await fetchResults(query);
    setResults(data);
    setIsLoading(false);
    
    announce(`Found ${data.length} result${data.length !== 1 ? 's' : ''}`);
  };

  return (
    <div>
      <Announcer />
      <form onSubmit={e => { e.preventDefault(); search(query); }}>
        <input type="search" value={query} onChange={e => setQuery(e.target.value)} />
        <button type="submit">Search</button>
      </form>
      
      {isLoading ? (
        <p aria-busy="true">Loading...</p>
      ) : (
        <ul aria-label="Search results">
          {results.map(result => (
            <li key={result.id}>{result.title}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DATA TABLES
// ════════════════════════════════════════════════════════════════

function DataTable({ data, columns }) {
  return (
    <table>
      <caption>
        User accounts
        <span className="sr-only">, sorted by name ascending</span>
      </caption>
      <thead>
        <tr>
          {columns.map(col => (
            <th 
              key={col.key} 
              scope="col"
              aria-sort={col.sorted ? col.direction : undefined}
            >
              <button onClick={() => handleSort(col.key)}>
                {col.label}
                {col.sorted && (
                  <span className="sr-only">
                    , sorted {col.direction}
                  </span>
                )}
              </button>
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row, index) => (
          <tr key={row.id}>
            <th scope="row">{row.name}</th>
            <td>{row.email}</td>
            <td>{row.role}</td>
            <td>
              <button aria-label={`Edit ${row.name}`}>Edit</button>
              <button aria-label={`Delete ${row.name}`}>Delete</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

## 5. Forms

```tsx
// ════════════════════════════════════════════════════════════════
// ACCESSIBLE FORMS
// ════════════════════════════════════════════════════════════════

// Form field with proper labeling
function TextField({
  label,
  name,
  error,
  hint,
  required,
  ...props
}: TextFieldProps) {
  const inputId = useId();
  const errorId = useId();
  const hintId = useId();

  const describedBy = [
    hint && hintId,
    error && errorId,
  ].filter(Boolean).join(' ') || undefined;

  return (
    <div className="form-field">
      <label htmlFor={inputId}>
        {label}
        {required && <span aria-hidden="true"> *</span>}
        {required && <span className="sr-only"> (required)</span>}
      </label>
      
      {hint && (
        <p id={hintId} className="hint">
          {hint}
        </p>
      )}
      
      <input
        id={inputId}
        name={name}
        aria-describedby={describedBy}
        aria-invalid={!!error}
        aria-required={required}
        {...props}
      />
      
      {error && (
        <p id={errorId} className="error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

// Form with validation
function RegistrationForm() {
  const [errors, setErrors] = useState({});
  const formRef = useRef<HTMLFormElement>(null);
  const { announce, Announcer } = useAnnouncement();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const formData = new FormData(formRef.current!);
    
    const validationErrors = validate(formData);
    setErrors(validationErrors);

    if (Object.keys(validationErrors).length > 0) {
      // Announce errors
      const errorCount = Object.keys(validationErrors).length;
      announce(
        `Form has ${errorCount} error${errorCount !== 1 ? 's' : ''}. Please review and correct.`,
        'assertive'
      );
      
      // Focus first error field
      const firstErrorField = formRef.current?.querySelector('[aria-invalid="true"]');
      (firstErrorField as HTMLElement)?.focus();
      return;
    }

    // Submit form...
  };

  return (
    <>
      <Announcer />
      <form ref={formRef} onSubmit={handleSubmit} noValidate>
        <TextField
          label="Email"
          name="email"
          type="email"
          required
          error={errors.email}
          hint="We'll never share your email"
        />
        
        <TextField
          label="Password"
          name="password"
          type="password"
          required
          error={errors.password}
          hint="Must be at least 8 characters"
        />

        <fieldset>
          <legend>Notification preferences</legend>
          
          <div className="checkbox-group">
            <input
              type="checkbox"
              id="email-notifications"
              name="notifications"
              value="email"
            />
            <label htmlFor="email-notifications">Email notifications</label>
          </div>
          
          <div className="checkbox-group">
            <input
              type="checkbox"
              id="sms-notifications"
              name="notifications"
              value="sms"
            />
            <label htmlFor="sms-notifications">SMS notifications</label>
          </div>
        </fieldset>

        <button type="submit">Register</button>
      </form>
    </>
  );
}

// ════════════════════════════════════════════════════════════════
// AUTOCOMPLETE / COMBOBOX
// ════════════════════════════════════════════════════════════════

function Autocomplete({ label, options, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const [inputValue, setInputValue] = useState('');
  const [activeIndex, setActiveIndex] = useState(-1);
  
  const inputId = useId();
  const listboxId = useId();
  
  const filteredOptions = options.filter(opt =>
    opt.label.toLowerCase().includes(inputValue.toLowerCase())
  );

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setIsOpen(true);
        setActiveIndex(prev => 
          prev < filteredOptions.length - 1 ? prev + 1 : 0
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(prev => 
          prev > 0 ? prev - 1 : filteredOptions.length - 1
        );
        break;
      case 'Enter':
        if (activeIndex >= 0 && filteredOptions[activeIndex]) {
          onChange(filteredOptions[activeIndex]);
          setInputValue(filteredOptions[activeIndex].label);
          setIsOpen(false);
        }
        break;
      case 'Escape':
        setIsOpen(false);
        break;
    }
  };

  return (
    <div className="autocomplete">
      <label htmlFor={inputId}>{label}</label>
      <input
        id={inputId}
        type="text"
        role="combobox"
        aria-expanded={isOpen}
        aria-haspopup="listbox"
        aria-controls={listboxId}
        aria-activedescendant={
          activeIndex >= 0 ? `option-${activeIndex}` : undefined
        }
        aria-autocomplete="list"
        value={inputValue}
        onChange={e => {
          setInputValue(e.target.value);
          setIsOpen(true);
        }}
        onKeyDown={handleKeyDown}
        onFocus={() => setIsOpen(true)}
        onBlur={() => setTimeout(() => setIsOpen(false), 200)}
      />
      
      {isOpen && filteredOptions.length > 0 && (
        <ul
          id={listboxId}
          role="listbox"
          aria-label={`${label} suggestions`}
        >
          {filteredOptions.map((option, index) => (
            <li
              key={option.value}
              id={`option-${index}`}
              role="option"
              aria-selected={index === activeIndex}
              onClick={() => {
                onChange(option);
                setInputValue(option.label);
                setIsOpen(false);
              }}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
      
      <div aria-live="polite" className="sr-only">
        {isOpen && `${filteredOptions.length} suggestions available`}
      </div>
    </div>
  );
}
```

---

## 6. Testing

```tsx
// ════════════════════════════════════════════════════════════════
// AUTOMATED TESTING (jest-axe)
// ════════════════════════════════════════════════════════════════

import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Button accessibility', () => {
  it('should have no accessibility violations', async () => {
    const { container } = render(
      <Button onClick={() => {}}>Click me</Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('should have accessible name when icon-only', async () => {
    const { container } = render(
      <Button aria-label="Close" onClick={() => {}}>
        <CloseIcon />
      </Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});

describe('Form accessibility', () => {
  it('should associate labels with inputs', async () => {
    const { container, getByLabelText } = render(
      <TextField label="Email" name="email" />
    );
    
    // Label should be associated
    expect(getByLabelText('Email')).toBeInTheDocument();
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('should announce errors', async () => {
    const { container, getByRole } = render(
      <TextField 
        label="Email" 
        name="email" 
        error="Invalid email format"
      />
    );
    
    // Error should be announced
    expect(getByRole('alert')).toHaveTextContent('Invalid email format');
    
    // Input should be marked invalid
    expect(getByRole('textbox')).toHaveAttribute('aria-invalid', 'true');
  });
});

// ════════════════════════════════════════════════════════════════
// KEYBOARD NAVIGATION TESTS
// ════════════════════════════════════════════════════════════════

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Modal keyboard navigation', () => {
  it('should trap focus within modal', async () => {
    const user = userEvent.setup();
    
    render(
      <Modal isOpen onClose={() => {}}>
        <input data-testid="first" />
        <button>Action</button>
        <button data-testid="last">Close</button>
      </Modal>
    );
    
    // First element should be focused
    expect(screen.getByTestId('first')).toHaveFocus();
    
    // Tab to last element
    await user.tab();
    await user.tab();
    expect(screen.getByTestId('last')).toHaveFocus();
    
    // Tab should wrap to first element
    await user.tab();
    expect(screen.getByTestId('first')).toHaveFocus();
  });

  it('should close on Escape', async () => {
    const onClose = jest.fn();
    const user = userEvent.setup();
    
    render(<Modal isOpen onClose={onClose}>Content</Modal>);
    
    await user.keyboard('{Escape}');
    expect(onClose).toHaveBeenCalled();
  });
});

// ════════════════════════════════════════════════════════════════
// STORYBOOK A11Y ADDON
// ════════════════════════════════════════════════════════════════

// .storybook/main.ts
export default {
  addons: ['@storybook/addon-a11y'],
};

// Component story
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Click me',
  },
  parameters: {
    a11y: {
      config: {
        rules: [
          { id: 'color-contrast', enabled: true },
        ],
      },
    },
  },
};

// ════════════════════════════════════════════════════════════════
// CI ACCESSIBILITY TESTING
// ════════════════════════════════════════════════════════════════

// cypress/e2e/a11y.cy.ts
describe('Accessibility', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.injectAxe();
  });

  it('should have no violations on homepage', () => {
    cy.checkA11y();
  });

  it('should have no violations on forms', () => {
    cy.visit('/contact');
    cy.checkA11y('form');
  });

  it('should have no violations after interaction', () => {
    cy.get('[data-testid="open-modal"]').click();
    cy.checkA11y('[role="dialog"]');
  });
});

// GitHub Actions workflow
// .github/workflows/a11y.yml
/*
name: Accessibility Tests
on: [push, pull_request]
jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:a11y
      - run: npm run build-storybook
      - run: npx chromatic --exit-once-uploaded
*/
```

---

## 7. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# ACCESSIBILITY PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Using divs for interactive elements
# Bad
<div onclick="handleClick()">Click me</div>

# Good
<button onclick="handleClick()">Click me</button>
# Buttons provide: keyboard access, focus, role announcement

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Missing alt text (or useless alt)
# Bad
<img src="chart.png" />
<img src="chart.png" alt="chart" />
<img src="chart.png" alt="chart.png" />

# Good
<img src="chart.png" alt="Sales increased 25% in Q2" />
# Or for decorative: alt=""

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Color as only indicator
# Bad
<span class="error-red">Invalid</span>  # Red text only

# Good
<span class="error">
  <ErrorIcon aria-hidden="true" />
  Invalid email format
</span>
# Icon + text, not just color

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No visible focus
# Bad
button:focus { outline: none; }

# Good
button:focus-visible { 
  outline: 2px solid blue; 
  outline-offset: 2px;
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Unlabeled form inputs
# Bad
<input type="text" placeholder="Email" />

# Good
<label for="email">Email</label>
<input id="email" type="text" />

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Inaccessible custom components
# Bad
<div class="custom-select" onclick="toggle()">
  <span>Selected value</span>
  <div class="options">...</div>
</div>

# Good - Use ARIA for custom components
<div 
  role="combobox"
  aria-expanded="true"
  aria-haspopup="listbox"
  aria-controls="options"
  tabindex="0"
>...</div>

# Best - Use headless UI libraries
# Radix, Headless UI, React Aria handle ARIA for you
```

---

## 8. Interview Questions

### Basic Questions

**Q: "What is WCAG?"**
> "Web Content Accessibility Guidelines - the standard for web accessibility. WCAG 2.1 has three conformance levels: A (minimum), AA (standard, legally required in many places), AAA (enhanced). It's organized around POUR principles: Perceivable, Operable, Understandable, Robust."

**Q: "What's the difference between aria-label and aria-labelledby?"**
> "aria-label provides the accessible name directly as a string. aria-labelledby references another element's ID to use as the label - better for visible labels since it avoids duplication. Use aria-label when there's no visible text, aria-labelledby when referencing existing text."

**Q: "Why is semantic HTML important for accessibility?"**
> "Semantic HTML conveys meaning to assistive technologies. A button element announces as 'button', handles keyboard events, appears in accessibility tree correctly. A div needs ARIA roles, tabindex, keyboard handlers manually. Semantic HTML is accessible by default."

### Intermediate Questions

**Q: "How do you make a custom dropdown accessible?"**
> "Use proper ARIA: combobox role on trigger, listbox role on options, aria-expanded state, aria-activedescendant for highlight. Keyboard: arrow keys for navigation, Enter to select, Escape to close, type-ahead search. Focus management: trap focus, return on close. Or use headless libraries like Radix."

**Q: "What is a live region?"**
> "Live regions announce dynamic content changes to screen readers. aria-live='polite' announces when user is idle, 'assertive' interrupts immediately. role='status' is polite by default, role='alert' is assertive. Use for notifications, loading states, form errors, search results count."

**Q: "How do you test accessibility?"**
> "Layered approach: 1) Automated tools (axe-core, Lighthouse) catch ~30% of issues. 2) Keyboard testing - can you use without mouse? 3) Screen reader testing (VoiceOver, NVDA) - is content understandable? 4) Manual checks - color contrast, focus visible. 5) User testing with people with disabilities."

### Advanced Questions

**Q: "How do you handle focus management in SPAs?"**
> "On route change, move focus to main content or h1 - screen readers otherwise miss the change. After modal close, return focus to trigger. After deleting item in list, focus next/previous item. Use aria-live to announce page changes. Libraries like @reach/router handle this."

**Q: "What's the difference between accessibility and usability?"**
> "Accessibility ensures people with disabilities CAN use the interface - it's about access. Usability ensures everyone can use it EFFICIENTLY - it's about ease. Accessible sites aren't automatically usable, but good usability often improves accessibility. Both require user testing."

**Q: "How do you build accessibility into a team's workflow?"**
> "Shift left: accessibility in design reviews, component acceptance criteria, PR checklists. Automated: axe-core in CI, Storybook a11y addon, Chromatic. Manual: keyboard and screen reader testing in QA. Training: team education on WCAG. Governance: accessibility champion, regular audits."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              ACCESSIBILITY CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SEMANTIC HTML:                                                 │
│  □ Use native elements (button, link, input)                   │
│  □ Proper heading hierarchy (h1 → h2 → h3)                     │
│  □ Landmark regions (main, nav, aside)                         │
│  □ Lists for lists (ul, ol)                                    │
│                                                                 │
│  IMAGES & MEDIA:                                                │
│  □ Alt text for informative images                             │
│  □ Empty alt for decorative images                             │
│  □ Captions/transcripts for video/audio                        │
│                                                                 │
│  KEYBOARD:                                                      │
│  □ All interactive elements focusable                          │
│  □ Visible focus indicator                                     │
│  □ Logical focus order                                         │
│  □ No keyboard traps                                           │
│                                                                 │
│  FORMS:                                                         │
│  □ Labels associated with inputs                               │
│  □ Error messages announced                                    │
│  □ Required fields indicated                                   │
│                                                                 │
│  COLOR:                                                         │
│  □ 4.5:1 contrast for text                                     │
│  □ Don't rely on color alone                                   │
│                                                                 │
│  TESTING:                                                       │
│  □ Automated (axe-core)                                        │
│  □ Keyboard navigation                                         │
│  □ Screen reader                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

