# Framer Motion

> Production-ready animation library for React.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Animation library for React |
| **Why** | Declarative animations, gestures, layout animations |
| **Version** | 10.x |
| **Location** | `apps/web/src/components/` |

## Basic Usage

### Simple Animation
```tsx
import { motion } from 'framer-motion';

function FadeInCard() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3 }}
      className="bg-white rounded-xl p-6 shadow-lg"
    >
      Content fades in
    </motion.div>
  );
}
```

### Exit Animations
```tsx
import { motion, AnimatePresence } from 'framer-motion';

function Modal({ isOpen, onClose, children }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <>
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="fixed inset-0 bg-black/50"
            onClick={onClose}
          />
          <motion.div
            initial={{ opacity: 0, scale: 0.95 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.95 }}
            className="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
          >
            {children}
          </motion.div>
        </>
      )}
    </AnimatePresence>
  );
}
```

## Variants

### Defining Variants
```tsx
const cardVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
  hover: { scale: 1.02, boxShadow: '0 10px 30px rgba(0,0,0,0.1)' },
};

function MedicationCard({ medication }) {
  return (
    <motion.div
      variants={cardVariants}
      initial="hidden"
      animate="visible"
      whileHover="hover"
      transition={{ duration: 0.2 }}
      className="bg-white rounded-xl p-4"
    >
      <h3>{medication.name}</h3>
    </motion.div>
  );
}
```

### Staggered Children
```tsx
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};

function MedicationsList({ medications }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      {medications.map((med) => (
        <motion.li key={med.id} variants={itemVariants}>
          <MedicationCard medication={med} />
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

## Gestures

### Hover & Tap
```tsx
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  className="bg-primary-600 text-white px-4 py-2 rounded-lg"
>
  Click me
</motion.button>
```

### Drag
```tsx
<motion.div
  drag
  dragConstraints={{ left: 0, right: 300, top: 0, bottom: 300 }}
  dragElastic={0.1}
  className="w-20 h-20 bg-blue-500 rounded-lg cursor-grab"
/>
```

## Page Transitions

```tsx
// app/layout.tsx
'use client';

import { motion, AnimatePresence } from 'framer-motion';
import { usePathname } from 'next/navigation';

export default function Layout({ children }) {
  const pathname = usePathname();

  return (
    <AnimatePresence mode="wait">
      <motion.main
        key={pathname}
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3 }}
      >
        {children}
      </motion.main>
    </AnimatePresence>
  );
}
```

## Layout Animations

### Auto Layout
```tsx
<motion.div layout className="grid gap-4">
  {items.map((item) => (
    <motion.div key={item.id} layout className="bg-white p-4 rounded-lg">
      {item.content}
    </motion.div>
  ))}
</motion.div>
```

### Shared Layout
```tsx
function Tabs() {
  const [selected, setSelected] = useState(0);

  return (
    <div className="flex gap-4">
      {tabs.map((tab, i) => (
        <button
          key={tab.id}
          onClick={() => setSelected(i)}
          className="relative px-4 py-2"
        >
          {selected === i && (
            <motion.div
              layoutId="activeTab"
              className="absolute inset-0 bg-primary-100 rounded-lg"
            />
          )}
          <span className="relative z-10">{tab.label}</span>
        </button>
      ))}
    </div>
  );
}
```

## Common Animation Patterns

### Fade In Up
```tsx
const fadeInUp = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.4, ease: 'easeOut' },
};

<motion.div {...fadeInUp}>Content</motion.div>
```

### Scale In
```tsx
const scaleIn = {
  initial: { opacity: 0, scale: 0.9 },
  animate: { opacity: 1, scale: 1 },
  transition: { duration: 0.3 },
};
```

### Slide In From Right
```tsx
const slideInRight = {
  initial: { opacity: 0, x: 100 },
  animate: { opacity: 1, x: 0 },
  exit: { opacity: 0, x: 100 },
  transition: { type: 'spring', damping: 25 },
};
```

### Notification Toast
```tsx
const toastVariants = {
  initial: { opacity: 0, x: 50, scale: 0.9 },
  animate: { opacity: 1, x: 0, scale: 1 },
  exit: { opacity: 0, x: 50, scale: 0.9 },
};

function Toast({ message }) {
  return (
    <motion.div
      variants={toastVariants}
      initial="initial"
      animate="animate"
      exit="exit"
      className="bg-white shadow-lg rounded-lg p-4"
    >
      {message}
    </motion.div>
  );
}
```

## Scroll Animations

### Scroll-Triggered
```tsx
import { motion, useInView } from 'framer-motion';
import { useRef } from 'react';

function ScrollReveal({ children }) {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true, margin: '-100px' });

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 50 }}
      animate={isInView ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.6 }}
    >
      {children}
    </motion.div>
  );
}
```

### Scroll Progress
```tsx
import { motion, useScroll, useSpring } from 'framer-motion';

function ScrollProgress() {
  const { scrollYProgress } = useScroll();
  const scaleX = useSpring(scrollYProgress, { stiffness: 100 });

  return (
    <motion.div
      style={{ scaleX }}
      className="fixed top-0 left-0 right-0 h-1 bg-primary-500 origin-left"
    />
  );
}
```

## Performance Tips

### Reduce Re-renders
```tsx
// Use MotionConfig for shared settings
import { MotionConfig } from 'framer-motion';

<MotionConfig reducedMotion="user">
  <App />
</MotionConfig>
```

### GPU-Accelerated Properties
```tsx
// Prefer transform and opacity (GPU-accelerated)
// ✅ Good
animate={{ x: 100, opacity: 0.5 }}

// ❌ Avoid (triggers layout)
animate={{ left: 100, width: 200 }}
```

### Lazy Motion
```tsx
import { LazyMotion, domAnimation, m } from 'framer-motion';

// Smaller bundle with limited features
<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
```

## Troubleshooting

### Exit Animation Not Working
- Wrap with `AnimatePresence`
- Use unique `key` prop
- Set `mode="wait"` for sequential exits

### Layout Animation Glitches
- Use `layoutId` for shared elements
- Add `layout` prop to parent containers
- Check for CSS transforms conflicting

### Performance Issues
- Use `will-change: transform` sparingly
- Avoid animating width/height (use scale)
- Use `useReducedMotion` hook for accessibility

---

*See also: [Tailwind CSS](tailwindcss.md), [React](react.md)*


