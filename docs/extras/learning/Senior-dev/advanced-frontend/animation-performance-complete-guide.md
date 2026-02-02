# ⚡ Animation Performance - Complete Guide

> A comprehensive guide to animation performance - GPU acceleration, FLIP technique, requestAnimationFrame, CSS transitions, and building smooth 60fps animations.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "High-performance animations leverage GPU-accelerated properties (transform, opacity), avoid layout thrashing, use the FLIP technique for layout animations, and target 60fps (16.67ms per frame) by minimizing main thread work."

### The 7 Key Concepts (Remember These!)
```
1. GPU ACCELERATION  → transform, opacity (composite only)
2. LAYOUT THRASHING  → Reading then writing DOM repeatedly
3. FLIP              → First, Last, Invert, Play technique
4. RAF               → requestAnimationFrame for smooth timing
5. WILL-CHANGE       → Hints browser to optimize property
6. COMPOSITE LAYERS  → Elements promoted to own GPU layer
7. JANK              → Dropped frames, stuttering animation
```

### Rendering Pipeline
```
┌─────────────────────────────────────────────────────────────────┐
│              BROWSER RENDERING PIPELINE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JavaScript → Style → Layout → Paint → Composite               │
│      │          │        │        │         │                  │
│      │          │        │        │         └─ CHEAPEST        │
│      │          │        │        │            transform       │
│      │          │        │        │            opacity         │
│      │          │        │        │                            │
│      │          │        │        └─ EXPENSIVE                 │
│      │          │        │           color, background         │
│      │          │        │           box-shadow                │
│      │          │        │                                     │
│      │          │        └─ VERY EXPENSIVE                     │
│      │          │           width, height, top, left           │
│      │          │           margin, padding, font-size         │
│      │          │                                              │
│      │          └─ Recalculate styles                          │
│      │                                                         │
│      └─ Trigger any of the above                               │
│                                                                 │
│  GOAL: Skip as many steps as possible                          │
│  BEST: Only composite (transform/opacity)                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Animatable Properties
```
┌─────────────────────────────────────────────────────────────────┐
│              PROPERTIES BY PERFORMANCE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ COMPOSITE ONLY (Best - GPU accelerated)                    │
│  ─────────────────────────────────────────                      │
│  • transform (translate, scale, rotate)                        │
│  • opacity                                                     │
│  • filter (blur, brightness, etc.)                             │
│                                                                 │
│  ⚠️ PAINT ONLY (Okay - no layout)                              │
│  ────────────────────────────────                               │
│  • color                                                       │
│  • background                                                  │
│  • box-shadow                                                  │
│  • text-shadow                                                 │
│  • border-color                                                │
│                                                                 │
│  ❌ TRIGGER LAYOUT (Avoid - expensive)                         │
│  ─────────────────────────────────                              │
│  • width, height                                               │
│  • top, right, bottom, left                                    │
│  • margin, padding                                             │
│  • border-width                                                │
│  • font-size                                                   │
│  • position                                                    │
│                                                                 │
│  INSTEAD OF:           USE:                                    │
│  top/left              transform: translate()                  │
│  width/height          transform: scale()                      │
│  visibility            opacity: 0                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Compositor"** | "Transform and opacity only hit the compositor, skipping paint" |
| **"Layout thrashing"** | "We batch DOM reads and writes to avoid layout thrashing" |
| **"FLIP"** | "We use FLIP for animating layout changes smoothly" |
| **"Jank"** | "Dropped frames cause jank - we aim for consistent 60fps" |
| **"Layer promotion"** | "will-change promotes elements to their own compositing layer" |
| **"Frame budget"** | "We have 16ms frame budget to stay at 60fps" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Target FPS | **60fps** | Standard smooth animation |
| Frame budget | **16.67ms** | 1000ms / 60fps |
| Script budget | **~10ms** | Leave time for render |
| will-change | **Sparingly** | Memory cost per layer |

### The "Wow" Statement (Memorize This!)
> "We only animate transform and opacity - they're compositor-only properties that skip layout and paint. For layout animations, we use FLIP: measure first position, apply final state, invert with transform, then animate to zero transform. This turns expensive layout animations into cheap transform animations. We avoid layout thrashing by batching DOM reads before writes. requestAnimationFrame syncs with browser refresh rate. For complex sequences, we use Framer Motion or GSAP which handle these optimizations automatically. will-change hints the browser but we use it sparingly - each promoted layer uses GPU memory. We profile with Chrome DevTools Performance panel, targeting under 10ms script time per frame."

---

## 📚 Table of Contents

1. [CSS Animations & Transitions](#1-css-animations--transitions)
2. [FLIP Technique](#2-flip-technique)
3. [requestAnimationFrame](#3-requestanimationframe)
4. [React Animation Libraries](#4-react-animation-libraries)
5. [Performance Optimization](#5-performance-optimization)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. CSS Animations & Transitions

```css
/* ════════════════════════════════════════════════════════════════
   CSS TRANSITIONS
   ════════════════════════════════════════════════════════════════ */

/* Basic transition */
.button {
  background-color: #3b82f6;
  transform: translateY(0);
  opacity: 1;
  transition: 
    background-color 200ms ease,
    transform 200ms ease,
    opacity 200ms ease;
}

.button:hover {
  background-color: #2563eb;
  transform: translateY(-2px);
}

.button:active {
  transform: translateY(0);
}

/* ✅ GOOD: Only animate transform and opacity */
.card {
  transform: scale(1);
  opacity: 1;
  transition: transform 300ms ease, opacity 300ms ease;
}

.card:hover {
  transform: scale(1.02);
}

/* ❌ BAD: Animating layout properties */
.card-bad {
  width: 200px;
  transition: width 300ms ease; /* Triggers layout! */
}

.card-bad:hover {
  width: 220px;
}

/* ════════════════════════════════════════════════════════════════
   CSS KEYFRAME ANIMATIONS
   ════════════════════════════════════════════════════════════════ */

/* Spin animation */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.spinner {
  animation: spin 1s linear infinite;
}

/* Fade in with slide */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeInUp 300ms ease forwards;
}

/* Staggered animations */
.list-item {
  opacity: 0;
  animation: fadeInUp 300ms ease forwards;
}

.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 50ms; }
.list-item:nth-child(3) { animation-delay: 100ms; }
.list-item:nth-child(4) { animation-delay: 150ms; }

/* Dynamic stagger with CSS custom properties */
.list-item {
  animation: fadeInUp 300ms ease forwards;
  animation-delay: calc(var(--index) * 50ms);
}

/* ════════════════════════════════════════════════════════════════
   PERFORMANCE HINTS
   ════════════════════════════════════════════════════════════════ */

/* will-change - use sparingly */
.animated-element {
  will-change: transform, opacity;
}

/* Remove after animation */
.animated-element.animation-complete {
  will-change: auto;
}

/* Force GPU layer (older technique) */
.gpu-accelerated {
  transform: translateZ(0);
  /* or */
  transform: translate3d(0, 0, 0);
}

/* Contain layout for better performance */
.card {
  contain: layout style paint;
}

/* ════════════════════════════════════════════════════════════════
   REDUCED MOTION
   ════════════════════════════════════════════════════════════════ */

/* Respect user preferences */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Or selectively disable specific animations */
@media (prefers-reduced-motion: reduce) {
  .hero-animation {
    animation: none;
    opacity: 1;
    transform: none;
  }
}
```

---

## 2. FLIP Technique

```tsx
// ════════════════════════════════════════════════════════════════
// FLIP: First, Last, Invert, Play
// ════════════════════════════════════════════════════════════════

/*
  FLIP turns expensive layout animations into cheap transform animations:
  
  1. FIRST:  Record initial position
  2. LAST:   Apply final state, record new position
  3. INVERT: Use transform to move back to initial position
  4. PLAY:   Remove transform to animate to final position
*/

function flipAnimate(element: HTMLElement, options: { duration?: number } = {}) {
  const { duration = 300 } = options;

  // FIRST: Record initial position
  const first = element.getBoundingClientRect();

  // LAST: Apply final state (trigger layout change)
  element.classList.add('expanded');
  
  // Record final position
  const last = element.getBoundingClientRect();

  // INVERT: Calculate the difference
  const deltaX = first.left - last.left;
  const deltaY = first.top - last.top;
  const deltaW = first.width / last.width;
  const deltaH = first.height / last.height;

  // Apply inverted transform (element appears in original position)
  element.style.transform = `translate(${deltaX}px, ${deltaY}px) scale(${deltaW}, ${deltaH})`;
  element.style.transformOrigin = 'top left';

  // PLAY: Animate to final position
  requestAnimationFrame(() => {
    // Enable transition
    element.style.transition = `transform ${duration}ms ease`;
    
    // Remove transform to animate to final position
    element.style.transform = '';
  });

  // Cleanup after animation
  element.addEventListener('transitionend', () => {
    element.style.transition = '';
    element.style.transformOrigin = '';
  }, { once: true });
}

// ════════════════════════════════════════════════════════════════
// FLIP HOOK FOR REACT
// ════════════════════════════════════════════════════════════════

import { useRef, useLayoutEffect, useCallback } from 'react';

interface FlipState {
  rect: DOMRect;
  opacity: number;
}

function useFLIP<T extends HTMLElement>() {
  const elementRef = useRef<T>(null);
  const previousState = useRef<FlipState | null>(null);

  // Capture current state before DOM changes
  const captureState = useCallback(() => {
    if (elementRef.current) {
      previousState.current = {
        rect: elementRef.current.getBoundingClientRect(),
        opacity: parseFloat(getComputedStyle(elementRef.current).opacity),
      };
    }
  }, []);

  // Apply FLIP animation after DOM changes
  useLayoutEffect(() => {
    const element = elementRef.current;
    const previous = previousState.current;

    if (!element || !previous) return;

    const current = element.getBoundingClientRect();

    // Calculate deltas
    const deltaX = previous.rect.left - current.left;
    const deltaY = previous.rect.top - current.top;
    const deltaW = previous.rect.width / current.width;
    const deltaH = previous.rect.height / current.height;

    // Skip if no change
    if (deltaX === 0 && deltaY === 0 && deltaW === 1 && deltaH === 1) {
      return;
    }

    // INVERT
    element.style.transform = `translate(${deltaX}px, ${deltaY}px) scale(${deltaW}, ${deltaH})`;
    element.style.transformOrigin = 'top left';

    // PLAY
    requestAnimationFrame(() => {
      element.style.transition = 'transform 300ms ease';
      element.style.transform = '';
    });

    const cleanup = () => {
      element.style.transition = '';
      element.style.transformOrigin = '';
    };

    element.addEventListener('transitionend', cleanup, { once: true });

    return () => {
      element.removeEventListener('transitionend', cleanup);
    };
  });

  return { ref: elementRef, captureState };
}

// Usage
function ExpandableCard() {
  const [isExpanded, setIsExpanded] = useState(false);
  const { ref, captureState } = useFLIP<HTMLDivElement>();

  const handleToggle = () => {
    captureState(); // Capture before state change
    setIsExpanded(!isExpanded);
  };

  return (
    <div
      ref={ref}
      className={`card ${isExpanded ? 'expanded' : ''}`}
      onClick={handleToggle}
    >
      Content
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// FLIP FOR LIST REORDERING
// ════════════════════════════════════════════════════════════════

function useFLIPList<T extends { id: string }>(items: T[]) {
  const positionsRef = useRef<Map<string, DOMRect>>(new Map());
  const elementsRef = useRef<Map<string, HTMLElement>>(new Map());

  // Register element
  const registerElement = useCallback((id: string, element: HTMLElement | null) => {
    if (element) {
      elementsRef.current.set(id, element);
    } else {
      elementsRef.current.delete(id);
    }
  }, []);

  // Capture positions before update
  const capturePositions = useCallback(() => {
    elementsRef.current.forEach((element, id) => {
      positionsRef.current.set(id, element.getBoundingClientRect());
    });
  }, []);

  // Apply FLIP after update
  useLayoutEffect(() => {
    const animations: Animation[] = [];

    elementsRef.current.forEach((element, id) => {
      const previousRect = positionsRef.current.get(id);
      if (!previousRect) return;

      const currentRect = element.getBoundingClientRect();
      const deltaX = previousRect.left - currentRect.left;
      const deltaY = previousRect.top - currentRect.top;

      if (deltaX === 0 && deltaY === 0) return;

      const animation = element.animate(
        [
          { transform: `translate(${deltaX}px, ${deltaY}px)` },
          { transform: 'translate(0, 0)' },
        ],
        {
          duration: 300,
          easing: 'ease',
        }
      );

      animations.push(animation);
    });

    return () => {
      animations.forEach(a => a.cancel());
    };
  }, [items]);

  return { registerElement, capturePositions };
}
```

---

## 3. requestAnimationFrame

```tsx
// ════════════════════════════════════════════════════════════════
// REQUESTANIMATIONFRAME BASICS
// ════════════════════════════════════════════════════════════════

// Basic RAF loop
function animate() {
  // Update animation state
  element.style.transform = `translateX(${position}px)`;
  position += speed;

  // Continue animation
  if (position < targetPosition) {
    requestAnimationFrame(animate);
  }
}

// Start animation
requestAnimationFrame(animate);

// ════════════════════════════════════════════════════════════════
// ANIMATION WITH TIMING
// ════════════════════════════════════════════════════════════════

interface AnimationOptions {
  duration: number;
  easing?: (t: number) => number;
  onUpdate: (progress: number) => void;
  onComplete?: () => void;
}

function animate({
  duration,
  easing = (t) => t, // Linear by default
  onUpdate,
  onComplete,
}: AnimationOptions) {
  const startTime = performance.now();
  let animationId: number;

  function tick(currentTime: number) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const easedProgress = easing(progress);

    onUpdate(easedProgress);

    if (progress < 1) {
      animationId = requestAnimationFrame(tick);
    } else {
      onComplete?.();
    }
  }

  animationId = requestAnimationFrame(tick);

  // Return cancel function
  return () => cancelAnimationFrame(animationId);
}

// Easing functions
const easings = {
  linear: (t: number) => t,
  easeInQuad: (t: number) => t * t,
  easeOutQuad: (t: number) => t * (2 - t),
  easeInOutQuad: (t: number) => (t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t),
  easeOutCubic: (t: number) => 1 - Math.pow(1 - t, 3),
  easeInOutCubic: (t: number) =>
    t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2,
  easeOutElastic: (t: number) => {
    const c4 = (2 * Math.PI) / 3;
    return t === 0
      ? 0
      : t === 1
      ? 1
      : Math.pow(2, -10 * t) * Math.sin((t * 10 - 0.75) * c4) + 1;
  },
};

// Usage
const cancel = animate({
  duration: 500,
  easing: easings.easeOutCubic,
  onUpdate: (progress) => {
    element.style.transform = `translateX(${progress * 200}px)`;
    element.style.opacity = String(progress);
  },
  onComplete: () => console.log('Animation complete!'),
});

// ════════════════════════════════════════════════════════════════
// REACT HOOKS FOR RAF
// ════════════════════════════════════════════════════════════════

function useAnimationFrame(callback: (deltaTime: number) => void, isActive = true) {
  const requestRef = useRef<number>();
  const previousTimeRef = useRef<number>();

  useEffect(() => {
    if (!isActive) return;

    const animate = (time: number) => {
      if (previousTimeRef.current !== undefined) {
        const deltaTime = time - previousTimeRef.current;
        callback(deltaTime);
      }
      previousTimeRef.current = time;
      requestRef.current = requestAnimationFrame(animate);
    };

    requestRef.current = requestAnimationFrame(animate);

    return () => {
      if (requestRef.current) {
        cancelAnimationFrame(requestRef.current);
      }
    };
  }, [callback, isActive]);
}

// Usage: Animated counter
function AnimatedCounter({ target }: { target: number }) {
  const [current, setCurrent] = useState(0);

  useAnimationFrame(
    (deltaTime) => {
      setCurrent((prev) => {
        const increment = (target - prev) * 0.1;
        if (Math.abs(increment) < 0.5) return target;
        return prev + increment;
      });
    },
    current !== target
  );

  return <span>{Math.round(current)}</span>;
}

// ════════════════════════════════════════════════════════════════
// SPRING ANIMATIONS
// ════════════════════════════════════════════════════════════════

interface SpringConfig {
  stiffness: number; // Higher = snappier
  damping: number; // Higher = less bouncy
  mass: number;
}

function useSpring(
  target: number,
  config: SpringConfig = { stiffness: 170, damping: 26, mass: 1 }
) {
  const [current, setCurrent] = useState(target);
  const velocityRef = useRef(0);

  useAnimationFrame(
    () => {
      const { stiffness, damping, mass } = config;
      
      // Spring physics
      const springForce = stiffness * (target - current);
      const dampingForce = damping * velocityRef.current;
      const acceleration = (springForce - dampingForce) / mass;
      
      velocityRef.current += acceleration * 0.016; // Assuming 60fps
      const newValue = current + velocityRef.current * 0.016;
      
      // Stop when settled
      if (
        Math.abs(velocityRef.current) < 0.01 &&
        Math.abs(target - newValue) < 0.01
      ) {
        setCurrent(target);
        velocityRef.current = 0;
      } else {
        setCurrent(newValue);
      }
    },
    current !== target || velocityRef.current !== 0
  );

  return current;
}

// Usage
function SpringBox() {
  const [isOpen, setIsOpen] = useState(false);
  const scale = useSpring(isOpen ? 1.2 : 1);

  return (
    <div
      style={{ transform: `scale(${scale})` }}
      onClick={() => setIsOpen(!isOpen)}
    >
      Click me
    </div>
  );
}
```

---

## 4. React Animation Libraries

```tsx
// ════════════════════════════════════════════════════════════════
// FRAMER MOTION
// ════════════════════════════════════════════════════════════════

import { motion, AnimatePresence, useAnimation } from 'framer-motion';

// Basic animation
function FadeIn({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}

// Hover and tap animations
function InteractiveCard() {
  return (
    <motion.div
      className="card"
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
      transition={{ type: 'spring', stiffness: 400, damping: 17 }}
    >
      Card content
    </motion.div>
  );
}

// Exit animations with AnimatePresence
function Modal({ isOpen, onClose }: { isOpen: boolean; onClose: () => void }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <>
          {/* Backdrop */}
          <motion.div
            className="backdrop"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={onClose}
          />
          {/* Modal */}
          <motion.div
            className="modal"
            initial={{ opacity: 0, scale: 0.9, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.9, y: 20 }}
            transition={{ type: 'spring', damping: 25, stiffness: 300 }}
          >
            Modal content
          </motion.div>
        </>
      )}
    </AnimatePresence>
  );
}

// Staggered list animations
function StaggeredList({ items }: { items: string[] }) {
  const container = {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.1,
      },
    },
  };

  const item = {
    hidden: { opacity: 0, x: -20 },
    show: { opacity: 1, x: 0 },
  };

  return (
    <motion.ul variants={container} initial="hidden" animate="show">
      {items.map((text, index) => (
        <motion.li key={index} variants={item}>
          {text}
        </motion.li>
      ))}
    </motion.ul>
  );
}

// Layout animations (FLIP automatically)
function LayoutAnimation() {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <motion.div
      layout // Enables automatic FLIP animations
      className={isExpanded ? 'expanded' : 'collapsed'}
      onClick={() => setIsExpanded(!isExpanded)}
      transition={{ type: 'spring', stiffness: 300, damping: 30 }}
    >
      <motion.h2 layout="position">Title</motion.h2>
      {isExpanded && (
        <motion.p
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          Expanded content
        </motion.p>
      )}
    </motion.div>
  );
}

// ════════════════════════════════════════════════════════════════
// REACT SPRING
// ════════════════════════════════════════════════════════════════

import { useSpring, animated, useTransition, config } from '@react-spring/web';

// Basic spring
function SpringAnimation() {
  const [isOpen, setIsOpen] = useState(false);

  const springs = useSpring({
    opacity: isOpen ? 1 : 0,
    transform: isOpen ? 'translateY(0px)' : 'translateY(20px)',
    config: config.gentle,
  });

  return (
    <animated.div style={springs}>
      Content
    </animated.div>
  );
}

// Transitions for mount/unmount
function TransitionExample({ items }: { items: string[] }) {
  const transitions = useTransition(items, {
    from: { opacity: 0, transform: 'translateX(-20px)' },
    enter: { opacity: 1, transform: 'translateX(0px)' },
    leave: { opacity: 0, transform: 'translateX(20px)' },
    keys: items,
    trail: 100, // Stagger delay
  });

  return (
    <ul>
      {transitions((style, item) => (
        <animated.li style={style}>{item}</animated.li>
      ))}
    </ul>
  );
}

// ════════════════════════════════════════════════════════════════
// CSS TRANSITION GROUPS (Simple)
// ════════════════════════════════════════════════════════════════

import { CSSTransition, TransitionGroup } from 'react-transition-group';

function CSSTransitionExample({ items }: { items: { id: string; text: string }[] }) {
  return (
    <TransitionGroup component="ul">
      {items.map((item) => (
        <CSSTransition
          key={item.id}
          timeout={300}
          classNames="fade"
        >
          <li>{item.text}</li>
        </CSSTransition>
      ))}
    </TransitionGroup>
  );
}

// CSS
/*
.fade-enter {
  opacity: 0;
  transform: translateX(-20px);
}
.fade-enter-active {
  opacity: 1;
  transform: translateX(0);
  transition: opacity 300ms, transform 300ms;
}
.fade-exit {
  opacity: 1;
  transform: translateX(0);
}
.fade-exit-active {
  opacity: 0;
  transform: translateX(20px);
  transition: opacity 300ms, transform 300ms;
}
*/
```

---

## 5. Performance Optimization

```tsx
// ════════════════════════════════════════════════════════════════
// AVOID LAYOUT THRASHING
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Read-write-read-write (forces multiple layouts)
elements.forEach((el) => {
  const height = el.offsetHeight; // READ - forces layout
  el.style.height = height + 10 + 'px'; // WRITE - invalidates layout
});
// Each iteration forces a new layout!

// ✅ GOOD: Batch reads, then batch writes
const heights = elements.map((el) => el.offsetHeight); // All reads

elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
// Only one layout calculation!

// ════════════════════════════════════════════════════════════════
// FASTDOM FOR LAYOUT BATCHING
// ════════════════════════════════════════════════════════════════

import fastdom from 'fastdom';

// Reads and writes are batched automatically
elements.forEach((el) => {
  fastdom.measure(() => {
    const height = el.offsetHeight;
    
    fastdom.mutate(() => {
      el.style.height = height + 10 + 'px';
    });
  });
});

// ════════════════════════════════════════════════════════════════
// VIRTUAL SCROLLING FOR LONG LISTS
// ════════════════════════════════════════════════════════════════

import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: string[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // Estimated row height
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index]}
          </div>
        ))}
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DEBOUNCE SCROLL/RESIZE HANDLERS
// ════════════════════════════════════════════════════════════════

function useAnimatedScroll() {
  const [scrollY, setScrollY] = useState(0);
  const rafId = useRef<number>();

  useEffect(() => {
    const handleScroll = () => {
      // Cancel previous frame
      if (rafId.current) {
        cancelAnimationFrame(rafId.current);
      }

      // Schedule update on next frame
      rafId.current = requestAnimationFrame(() => {
        setScrollY(window.scrollY);
      });
    };

    window.addEventListener('scroll', handleScroll, { passive: true });

    return () => {
      window.removeEventListener('scroll', handleScroll);
      if (rafId.current) {
        cancelAnimationFrame(rafId.current);
      }
    };
  }, []);

  return scrollY;
}

// ════════════════════════════════════════════════════════════════
// OPTIMIZE REACT RENDERS DURING ANIMATION
// ════════════════════════════════════════════════════════════════

// ❌ BAD: State updates cause re-renders
function BadAnimation() {
  const [x, setX] = useState(0);

  useAnimationFrame(() => {
    setX((prev) => prev + 1); // Re-renders every frame!
  });

  return <div style={{ transform: `translateX(${x}px)` }}>Moving</div>;
}

// ✅ GOOD: Use refs and direct DOM manipulation
function GoodAnimation() {
  const elementRef = useRef<HTMLDivElement>(null);
  const xRef = useRef(0);

  useAnimationFrame(() => {
    xRef.current += 1;
    if (elementRef.current) {
      elementRef.current.style.transform = `translateX(${xRef.current}px)`;
    }
  });

  return <div ref={elementRef}>Moving</div>;
}

// ════════════════════════════════════════════════════════════════
// MEASURE WITH DEVTOOLS
// ════════════════════════════════════════════════════════════════

/*
  Chrome DevTools Performance Panel:
  
  1. Record during animation
  2. Look for:
     - Long tasks (> 50ms)
     - Layout thrashing (purple blocks)
     - Paint storms (green blocks)
     - Frame drops (red markers)
  
  3. Aim for:
     - Scripting: < 10ms per frame
     - Rendering: < 4ms per frame
     - Painting: < 2ms per frame
     - Total: < 16ms per frame
  
  4. Use Layers panel to check:
     - Elements with will-change
     - Composited layers
     - Layer count (too many = memory issue)
*/
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# ANIMATION PERFORMANCE PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Animating layout properties
# Bad
.box {
  transition: width 300ms, height 300ms;  # Triggers layout
}

# Good
.box {
  transition: transform 300ms;  # Compositor only
}
.box:hover {
  transform: scale(1.1);
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Layout thrashing
# Bad
elements.forEach(el => {
  el.style.width = el.offsetWidth + 10 + 'px';  # Read-write loop
});

# Good
const widths = elements.map(el => el.offsetWidth);  # Batch reads
elements.forEach((el, i) => {
  el.style.width = widths[i] + 10 + 'px';  # Batch writes
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Overusing will-change
# Bad
* { will-change: transform, opacity; }  # Everything promoted

# Good
.animating { will-change: transform; }
# Remove after animation completes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Not using requestAnimationFrame
# Bad
setInterval(() => {
  element.style.transform = `translateX(${x++}px)`;
}, 16);

# Good
function animate() {
  element.style.transform = `translateX(${x++}px)`;
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Animating too many elements
# Bad
# Animating 1000 list items simultaneously

# Good
# Virtualize the list
# Only animate visible items
# Use CSS contain

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Ignoring reduced motion preference
# Bad
# All users see the same animations

# Good
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; }
}
```

---

## 7. Interview Questions

### Basic Questions

**Q: "Why should you only animate transform and opacity?"**
> "Transform and opacity are compositor-only properties - they can be handled entirely by the GPU without triggering layout or paint. Other properties like width, height, top, left trigger expensive layout recalculations. This is the key to achieving 60fps animations."

**Q: "What is layout thrashing?"**
> "Layout thrashing happens when you alternate between reading and writing to the DOM. Each write invalidates the layout, so the next read forces a synchronous layout calculation. Batch all reads together, then all writes, to avoid this. Use tools like fastdom to automate batching."

**Q: "What is will-change?"**
> "will-change hints to the browser that an element will animate, so it can optimize ahead of time by promoting it to a compositing layer. But use sparingly - each layer uses GPU memory. Apply before animation, remove after. Don't use on everything."

### Intermediate Questions

**Q: "What is the FLIP technique?"**
> "FLIP stands for First, Last, Invert, Play. Record element's first position, apply final state, calculate difference and use transform to invert back to first position, then animate transform to zero. It turns expensive layout animations into cheap transform animations."

**Q: "How do you achieve 60fps animations?"**
> "Only animate transform and opacity. Batch DOM reads and writes. Use requestAnimationFrame for timing. Avoid React state updates during animation - use refs and direct DOM manipulation. Keep JavaScript under 10ms per frame. Profile with DevTools Performance panel."

**Q: "When should you use a library vs CSS animations?"**
> "CSS animations: simple transitions, hover effects, no JavaScript needed. Libraries (Framer Motion, GSAP): complex sequences, physics-based springs, orchestrated animations, interruptible animations, gesture-based interactions. Libraries handle many optimizations automatically."

### Advanced Questions

**Q: "How do you animate layout changes performantly?"**
> "Use FLIP technique: measure before, apply change, invert with transform, animate transform to zero. Framer Motion's layout prop does this automatically. For lists, use layout animations with keys. Never animate actual layout properties directly."

**Q: "How do you handle animations in React without re-renders?"**
> "Use refs for animated values, update DOM directly without setState. Animation libraries like Framer Motion and react-spring handle this internally. For scroll animations, use passive event listeners and requestAnimationFrame throttling."

**Q: "How do you debug animation performance issues?"**
> "Chrome DevTools: Performance panel for frame analysis, Layers panel for compositing, Rendering panel for paint flashing. Look for: long tasks (> 50ms), layout thrashing (purple), excessive paints (green). Use CSS contain to limit scope of style recalculations."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              ANIMATION PERFORMANCE CHECKLIST                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PROPERTIES:                                                    │
│  □ Only animate transform and opacity                          │
│  □ Use scale instead of width/height                           │
│  □ Use translate instead of top/left                           │
│                                                                 │
│  TECHNIQUE:                                                     │
│  □ Use requestAnimationFrame                                   │
│  □ Batch DOM reads and writes                                  │
│  □ Use FLIP for layout animations                              │
│  □ Use refs instead of state in React                          │
│                                                                 │
│  OPTIMIZATION:                                                  │
│  □ will-change on animated elements (sparingly)                │
│  □ Virtualize long lists                                       │
│  □ Debounce scroll/resize handlers                             │
│                                                                 │
│  ACCESSIBILITY:                                                 │
│  □ Respect prefers-reduced-motion                              │
│                                                                 │
│  TARGET:                                                        │
│  □ < 16ms per frame (60fps)                                    │
│  □ < 10ms JavaScript                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

