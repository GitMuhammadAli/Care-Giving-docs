# 🌍 Internationalization (i18n) - Complete Guide

> A comprehensive guide to internationalization - RTL support, pluralization, date/number formatting, translations, and building apps for global audiences.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Internationalization (i18n) is the process of designing applications to support multiple languages, locales, and cultural conventions without code changes, while localization (l10n) is the actual adaptation for specific regions."

### The 7 Key Concepts (Remember These!)
```
1. LOCALE          → Language + region (en-US, de-DE, ar-SA)
2. TRANSLATIONS    → Text strings in different languages
3. PLURALIZATION   → "1 item" vs "2 items" (complex in some languages)
4. RTL             → Right-to-left layout (Arabic, Hebrew)
5. DATE FORMATTING → mm/dd/yyyy vs dd/mm/yyyy
6. NUMBER FORMAT   → 1,234.56 vs 1.234,56
7. CURRENCY        → $100.00 vs 100,00 €
```

### i18n vs l10n
```
┌─────────────────────────────────────────────────────────────────┐
│              i18n vs l10n                                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INTERNATIONALIZATION (i18n)                                   │
│  ───────────────────────────                                    │
│  • Design/architecture to support localization                 │
│  • Extract strings into resource files                         │
│  • Support for different text directions                       │
│  • Locale-aware formatting APIs                                │
│  • Done by developers                                          │
│  • Done once, enables many locales                             │
│                                                                 │
│  LOCALIZATION (l10n)                                           │
│  ─────────────────────                                          │
│  • Actual translation of content                               │
│  • Adapting for cultural conventions                           │
│  • Currency, dates, numbers for locale                         │
│  • Images/icons appropriate for region                         │
│  • Done by translators                                         │
│  • Done per locale                                             │
│                                                                 │
│  EXAMPLE FLOW:                                                 │
│  ─────────────                                                  │
│  Developer: {t('welcome')} → i18n                              │
│  en.json: "welcome": "Hello" → l10n                            │
│  de.json: "welcome": "Hallo" → l10n                            │
│  ja.json: "welcome": "こんにちは" → l10n                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complexity by Feature
```
┌─────────────────────────────────────────────────────────────────┐
│              i18n COMPLEXITY LEVELS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EASY:                                                         │
│  • Static text translations                                    │
│  • Simple string replacement                                   │
│                                                                 │
│  MEDIUM:                                                       │
│  • Pluralization (1 item vs N items)                           │
│  • Date/time formatting                                        │
│  • Number/currency formatting                                  │
│  • Interpolation (Hello, {name}!)                              │
│                                                                 │
│  COMPLEX:                                                      │
│  • RTL layout (Arabic, Hebrew)                                 │
│  • Gender-based translations                                   │
│  • Complex plural rules (Slavic, Arabic)                       │
│  • Text expansion (German 30% longer)                          │
│  • Character sets (CJK, emoji)                                 │
│                                                                 │
│  PLURAL RULE COMPLEXITY:                                       │
│  ─────────────────────────                                      │
│  English: 1 = singular, else plural (2 forms)                  │
│  Russian: 1, 2-4, 5-20, 21, 22-24... (3 forms)                 │
│  Arabic: 0, 1, 2, 3-10, 11-99, 100+ (6 forms!)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Locale"** | "We detect the user's locale from Accept-Language header" |
| **"ICU MessageFormat"** | "We use ICU MessageFormat for complex pluralization" |
| **"Text expansion"** | "UI accounts for text expansion - German is 30% longer" |
| **"Fallback chain"** | "Translations fallback: de-AT → de → en" |
| **"Logical properties"** | "We use CSS logical properties for RTL support" |
| **"Pseudo-localization"** | "Pseudo-localization helps catch hardcoded strings" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Text expansion | **30-40%** | German longer than English |
| Text contraction | **20-30%** | Chinese shorter than English |
| RTL languages | **12+** | Arabic, Hebrew, Persian, Urdu |
| Plural forms | **1-6** | Depends on language |

### The "Wow" Statement (Memorize This!)
> "Our i18n architecture uses react-intl with ICU MessageFormat for pluralization and gender. Translations are JSON files loaded lazily per locale. We extract strings at build time to catch missing translations. For RTL, we use CSS logical properties (margin-inline-start not margin-left) and dir='rtl' on html. Date/currency use Intl API with user's locale - no libraries needed. We handle text expansion by avoiding fixed widths. The fallback chain goes de-AT → de → en. Pseudo-localization in dev catches hardcoded strings - we see Ħêļļö instead of Hello. Translation management is via Crowdin with CI sync. Context and screenshots help translators understand usage."

---

## 📚 Table of Contents

1. [Translation Setup](#1-translation-setup)
2. [Pluralization & ICU](#2-pluralization--icu)
3. [Date, Time & Numbers](#3-date-time--numbers)
4. [RTL Support](#4-rtl-support)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Translation Setup

```tsx
// ════════════════════════════════════════════════════════════════
// REACT-INTL SETUP
// ════════════════════════════════════════════════════════════════

// locales/en.json
{
  "common.welcome": "Welcome to our app",
  "common.greeting": "Hello, {name}!",
  "nav.home": "Home",
  "nav.products": "Products",
  "nav.cart": "Cart",
  "product.addToCart": "Add to Cart",
  "product.price": "Price: {price}",
  "cart.itemCount": "{count, plural, one {# item} other {# items}} in cart"
}

// locales/de.json
{
  "common.welcome": "Willkommen in unserer App",
  "common.greeting": "Hallo, {name}!",
  "nav.home": "Startseite",
  "nav.products": "Produkte",
  "nav.cart": "Warenkorb",
  "product.addToCart": "In den Warenkorb",
  "product.price": "Preis: {price}",
  "cart.itemCount": "{count, plural, one {# Artikel} other {# Artikel}} im Warenkorb"
}

// locales/ar.json
{
  "common.welcome": "مرحبًا بك في تطبيقنا",
  "common.greeting": "مرحبًا، {name}!",
  "nav.home": "الرئيسية",
  "nav.products": "المنتجات",
  "nav.cart": "سلة التسوق"
}

// ════════════════════════════════════════════════════════════════
// PROVIDER SETUP
// ════════════════════════════════════════════════════════════════

// providers/IntlProvider.tsx
import { IntlProvider as ReactIntlProvider } from 'react-intl';
import { useState, useEffect, createContext, useContext } from 'react';

type Locale = 'en' | 'de' | 'ar' | 'ja';

interface LocaleContextValue {
  locale: Locale;
  setLocale: (locale: Locale) => void;
  direction: 'ltr' | 'rtl';
}

const LocaleContext = createContext<LocaleContextValue | null>(null);

// Lazy load translations
async function loadMessages(locale: Locale): Promise<Record<string, string>> {
  switch (locale) {
    case 'de':
      return (await import('../locales/de.json')).default;
    case 'ar':
      return (await import('../locales/ar.json')).default;
    case 'ja':
      return (await import('../locales/ja.json')).default;
    default:
      return (await import('../locales/en.json')).default;
  }
}

const RTL_LOCALES: Locale[] = ['ar'];

export function LocaleProvider({ children }: { children: React.ReactNode }) {
  const [locale, setLocale] = useState<Locale>('en');
  const [messages, setMessages] = useState<Record<string, string>>({});
  const [isLoading, setIsLoading] = useState(true);

  // Detect initial locale
  useEffect(() => {
    const stored = localStorage.getItem('locale') as Locale;
    const browserLocale = navigator.language.split('-')[0] as Locale;
    const initial = stored || browserLocale || 'en';
    setLocale(initial);
  }, []);

  // Load messages when locale changes
  useEffect(() => {
    setIsLoading(true);
    loadMessages(locale).then((msgs) => {
      setMessages(msgs);
      setIsLoading(false);
      localStorage.setItem('locale', locale);
      
      // Update HTML attributes
      document.documentElement.lang = locale;
      document.documentElement.dir = RTL_LOCALES.includes(locale) ? 'rtl' : 'ltr';
    });
  }, [locale]);

  const direction = RTL_LOCALES.includes(locale) ? 'rtl' : 'ltr';

  if (isLoading) return <LoadingScreen />;

  return (
    <LocaleContext.Provider value={{ locale, setLocale, direction }}>
      <ReactIntlProvider 
        locale={locale} 
        messages={messages}
        defaultLocale="en"
        onError={(err) => {
          // Ignore missing translation warnings in dev
          if (err.code === 'MISSING_TRANSLATION') {
            console.warn(`Missing translation: ${err.message}`);
            return;
          }
          console.error(err);
        }}
      >
        {children}
      </ReactIntlProvider>
    </LocaleContext.Provider>
  );
}

export function useLocale() {
  const context = useContext(LocaleContext);
  if (!context) throw new Error('useLocale must be used within LocaleProvider');
  return context;
}

// ════════════════════════════════════════════════════════════════
// COMPONENT USAGE
// ════════════════════════════════════════════════════════════════

import { FormattedMessage, useIntl } from 'react-intl';

// Declarative component approach
function WelcomeMessage({ name }: { name: string }) {
  return (
    <h1>
      <FormattedMessage
        id="common.greeting"
        defaultMessage="Hello, {name}!"
        values={{ name }}
      />
    </h1>
  );
}

// Imperative hook approach (for attributes, etc.)
function ProductCard({ product }: { product: Product }) {
  const intl = useIntl();
  
  const addToCartLabel = intl.formatMessage({
    id: 'product.addToCart',
    defaultMessage: 'Add to Cart',
  });

  return (
    <article>
      <h2>{product.name}</h2>
      <p>
        <FormattedMessage
          id="product.price"
          values={{
            price: intl.formatNumber(product.price, {
              style: 'currency',
              currency: 'USD',
            }),
          }}
        />
      </p>
      <button aria-label={addToCartLabel}>
        {addToCartLabel}
      </button>
    </article>
  );
}

// Language selector
function LanguageSelector() {
  const { locale, setLocale } = useLocale();
  
  const languages = [
    { code: 'en', name: 'English', nativeName: 'English' },
    { code: 'de', name: 'German', nativeName: 'Deutsch' },
    { code: 'ar', name: 'Arabic', nativeName: 'العربية' },
    { code: 'ja', name: 'Japanese', nativeName: '日本語' },
  ];

  return (
    <select
      value={locale}
      onChange={(e) => setLocale(e.target.value as Locale)}
      aria-label="Select language"
    >
      {languages.map((lang) => (
        <option key={lang.code} value={lang.code}>
          {lang.nativeName}
        </option>
      ))}
    </select>
  );
}
```

---

## 2. Pluralization & ICU

```tsx
// ════════════════════════════════════════════════════════════════
// ICU MESSAGE FORMAT
// ════════════════════════════════════════════════════════════════

// Basic pluralization
{
  "cart.items": "{count, plural, one {# item} other {# items}}"
}

// With zero case
{
  "cart.items": "{count, plural, =0 {No items} one {# item} other {# items}}"
}

// Complex plural (English)
{
  "notifications": "{count, plural, =0 {No notifications} one {# notification} other {# notifications}}"
}

// Complex plural (Russian - 3 forms)
{
  "notifications": "{count, plural, one {# уведомление} few {# уведомления} many {# уведомлений} other {# уведомления}}"
}

// Complex plural (Arabic - 6 forms!)
{
  "notifications": "{count, plural, zero {لا إشعارات} one {إشعار واحد} two {إشعاران} few {# إشعارات} many {# إشعارًا} other {# إشعار}}"
}

// ════════════════════════════════════════════════════════════════
// SELECT (Gender/Variants)
// ════════════════════════════════════════════════════════════════

// Gender-based message
{
  "user.welcome": "{gender, select, male {He joined} female {She joined} other {They joined}} on {date}"
}

// Variant selection
{
  "notification.type": "{type, select, message {New message} friend {Friend request} comment {New comment} other {Notification}}"
}

// Combined plural + select
{
  "user.followers": "{gender, select, male {He has} female {She has} other {They have}} {count, plural, one {# follower} other {# followers}}"
}

// ════════════════════════════════════════════════════════════════
// USAGE IN COMPONENTS
// ════════════════════════════════════════════════════════════════

import { FormattedMessage, FormattedPlural } from 'react-intl';

// Using FormattedMessage with ICU
function CartCount({ count }: { count: number }) {
  return (
    <span>
      <FormattedMessage
        id="cart.items"
        defaultMessage="{count, plural, one {# item} other {# items}}"
        values={{ count }}
      />
    </span>
  );
}

// Using FormattedPlural (simpler for basic cases)
function ItemCount({ count }: { count: number }) {
  return (
    <FormattedPlural
      value={count}
      one="# item"
      other="# items"
    />
  );
}

// Gender-based greeting
function UserWelcome({ user }: { user: User }) {
  const intl = useIntl();
  
  return (
    <p>
      {intl.formatMessage(
        {
          id: 'user.welcome',
          defaultMessage: '{gender, select, male {He joined} female {She joined} other {They joined}} on {date}',
        },
        {
          gender: user.gender,
          date: intl.formatDate(user.joinedAt, { dateStyle: 'long' }),
        }
      )}
    </p>
  );
}

// ════════════════════════════════════════════════════════════════
// RICH TEXT FORMATTING
// ════════════════════════════════════════════════════════════════

// Messages with tags
{
  "terms.agreement": "I agree to the <terms>Terms of Service</terms> and <privacy>Privacy Policy</privacy>"
}

function TermsAgreement() {
  return (
    <FormattedMessage
      id="terms.agreement"
      values={{
        terms: (chunks) => <a href="/terms">{chunks}</a>,
        privacy: (chunks) => <a href="/privacy">{chunks}</a>,
      }}
    />
  );
}

// Bold and emphasis
{
  "promo.message": "Get <b>50% off</b> your first order with code <code>WELCOME</code>"
}

function PromoMessage() {
  return (
    <FormattedMessage
      id="promo.message"
      values={{
        b: (chunks) => <strong>{chunks}</strong>,
        code: (chunks) => <code className="promo-code">{chunks}</code>,
      }}
    />
  );
}
```

---

## 3. Date, Time & Numbers

```tsx
// ════════════════════════════════════════════════════════════════
// DATE AND TIME FORMATTING
// ════════════════════════════════════════════════════════════════

import { FormattedDate, FormattedTime, FormattedRelativeTime, useIntl } from 'react-intl';

// Date formats vary by locale
// en-US: 12/25/2024
// en-GB: 25/12/2024
// de-DE: 25.12.2024
// ja-JP: 2024年12月25日

function EventDate({ date }: { date: Date }) {
  return (
    <time dateTime={date.toISOString()}>
      <FormattedDate
        value={date}
        year="numeric"
        month="long"
        day="numeric"
      />
    </time>
  );
}

// Different date styles
function DateFormats({ date }: { date: Date }) {
  return (
    <>
      {/* Full: Wednesday, December 25, 2024 */}
      <FormattedDate value={date} dateStyle="full" />
      
      {/* Long: December 25, 2024 */}
      <FormattedDate value={date} dateStyle="long" />
      
      {/* Medium: Dec 25, 2024 */}
      <FormattedDate value={date} dateStyle="medium" />
      
      {/* Short: 12/25/24 */}
      <FormattedDate value={date} dateStyle="short" />
    </>
  );
}

// Time formatting
function EventTime({ time }: { time: Date }) {
  return (
    <FormattedTime
      value={time}
      hour="numeric"
      minute="numeric"
      timeZoneName="short"
    />
  );
}

// Relative time
function useRelativeTime(date: Date) {
  const [unit, value] = getRelativeTimeArgs(date);
  
  return (
    <FormattedRelativeTime
      value={value}
      unit={unit}
      updateIntervalInSeconds={60}
    />
  );
}

function getRelativeTimeArgs(date: Date): [Intl.RelativeTimeFormatUnit, number] {
  const now = Date.now();
  const diff = date.getTime() - now;
  const absDiff = Math.abs(diff);
  
  if (absDiff < 60000) return ['second', Math.round(diff / 1000)];
  if (absDiff < 3600000) return ['minute', Math.round(diff / 60000)];
  if (absDiff < 86400000) return ['hour', Math.round(diff / 3600000)];
  if (absDiff < 2592000000) return ['day', Math.round(diff / 86400000)];
  if (absDiff < 31536000000) return ['month', Math.round(diff / 2592000000)];
  return ['year', Math.round(diff / 31536000000)];
}

// ════════════════════════════════════════════════════════════════
// NUMBER AND CURRENCY FORMATTING
// ════════════════════════════════════════════════════════════════

import { FormattedNumber } from 'react-intl';

// Number formats vary by locale
// en-US: 1,234.56
// de-DE: 1.234,56
// fr-FR: 1 234,56

function ProductPrice({ price, currency }: { price: number; currency: string }) {
  return (
    <FormattedNumber
      value={price}
      style="currency"
      currency={currency}
      minimumFractionDigits={2}
    />
  );
}

// Different number formats
function NumberFormats({ value }: { value: number }) {
  return (
    <>
      {/* Decimal: 1,234.567 */}
      <FormattedNumber value={value} style="decimal" />
      
      {/* Percent: 75% */}
      <FormattedNumber value={0.75} style="percent" />
      
      {/* Currency: $1,234.56 */}
      <FormattedNumber value={value} style="currency" currency="USD" />
      
      {/* Compact: 1.2K, 1.2M */}
      <FormattedNumber value={1234567} notation="compact" />
      
      {/* Unit: 5 kilograms */}
      <FormattedNumber value={5} style="unit" unit="kilogram" unitDisplay="long" />
    </>
  );
}

// Using Intl API directly
function useFormatters() {
  const intl = useIntl();
  
  return {
    formatPrice: (price: number, currency = 'USD') =>
      intl.formatNumber(price, {
        style: 'currency',
        currency,
      }),
    
    formatPercent: (value: number) =>
      intl.formatNumber(value, {
        style: 'percent',
        minimumFractionDigits: 1,
      }),
    
    formatCompact: (value: number) =>
      intl.formatNumber(value, {
        notation: 'compact',
        compactDisplay: 'short',
      }),
    
    formatDate: (date: Date) =>
      intl.formatDate(date, {
        dateStyle: 'medium',
      }),
    
    formatRelative: (date: Date) => {
      const [unit, value] = getRelativeTimeArgs(date);
      return intl.formatRelativeTime(value, unit);
    },
  };
}

// ════════════════════════════════════════════════════════════════
// LIST FORMATTING
// ════════════════════════════════════════════════════════════════

import { FormattedList } from 'react-intl';

// List formatting varies by locale
// en: "A, B, and C"
// de: "A, B und C"
// ja: "A、B、C"

function AuthorList({ authors }: { authors: string[] }) {
  return (
    <FormattedList
      type="conjunction"
      value={authors}
    />
  );
}

// Different list types
function ListFormats({ items }: { items: string[] }) {
  return (
    <>
      {/* Conjunction: A, B, and C */}
      <FormattedList type="conjunction" value={items} />
      
      {/* Disjunction: A, B, or C */}
      <FormattedList type="disjunction" value={items} />
      
      {/* Unit: A, B, C */}
      <FormattedList type="unit" value={items} />
    </>
  );
}
```

---

## 4. RTL Support

```tsx
// ════════════════════════════════════════════════════════════════
// RTL SETUP
// ════════════════════════════════════════════════════════════════

// Set direction on HTML element
useEffect(() => {
  const isRTL = ['ar', 'he', 'fa', 'ur'].includes(locale);
  document.documentElement.dir = isRTL ? 'rtl' : 'ltr';
  document.documentElement.lang = locale;
}, [locale]);

// ════════════════════════════════════════════════════════════════
// CSS LOGICAL PROPERTIES
// ════════════════════════════════════════════════════════════════

/* ❌ BAD: Physical properties break in RTL */
.sidebar {
  margin-left: 20px;
  padding-right: 10px;
  text-align: left;
  float: left;
}

/* ✅ GOOD: Logical properties adapt to direction */
.sidebar {
  margin-inline-start: 20px;  /* left in LTR, right in RTL */
  padding-inline-end: 10px;   /* right in LTR, left in RTL */
  text-align: start;          /* left in LTR, right in RTL */
  float: inline-start;        /* left in LTR, right in RTL */
}

/* Logical property mapping */
/*
  Physical  →  Logical
  ─────────────────────────────────────
  left      →  inline-start
  right     →  inline-end
  top       →  block-start
  bottom    →  block-end
  
  margin-left    →  margin-inline-start
  margin-right   →  margin-inline-end
  padding-left   →  padding-inline-start
  padding-right  →  padding-inline-end
  
  border-left    →  border-inline-start
  border-right   →  border-inline-end
  
  text-align: left  →  text-align: start
  text-align: right →  text-align: end
*/

/* Complete example */
.card {
  /* Spacing */
  margin-inline: auto;           /* horizontal centering */
  padding-inline: 1rem;          /* left and right */
  padding-block: 0.5rem;         /* top and bottom */
  
  /* Borders */
  border-inline-start: 3px solid blue;
  border-start-start-radius: 8px;  /* top-left in LTR */
  border-end-start-radius: 8px;    /* bottom-left in LTR */
  
  /* Positioning */
  inset-inline-start: 0;         /* left: 0 in LTR */
}

.icon-button {
  /* Gap between icon and text */
  gap: 0.5rem;
  
  /* Icon on the "start" side */
  flex-direction: row;
}

[dir="rtl"] .icon-button {
  /* Flex handles direction automatically */
  /* No changes needed if using gap */
}

// ════════════════════════════════════════════════════════════════
// RTL-AWARE COMPONENTS
// ════════════════════════════════════════════════════════════════

import { useLocale } from '../providers/LocaleProvider';

// Icon that flips in RTL
function DirectionalIcon({ icon: Icon, flip = true, ...props }) {
  const { direction } = useLocale();
  
  return (
    <Icon
      {...props}
      style={{
        transform: flip && direction === 'rtl' ? 'scaleX(-1)' : undefined,
      }}
    />
  );
}

// Arrows and chevrons should flip
<DirectionalIcon icon={ArrowRightIcon} />  // → in LTR, ← in RTL
<DirectionalIcon icon={ChevronLeftIcon} /> // ← in LTR, → in RTL

// But some icons shouldn't flip
<DirectionalIcon icon={CheckIcon} flip={false} />  // ✓ always
<DirectionalIcon icon={ClockIcon} flip={false} />  // 🕐 always

// Progress bar
function ProgressBar({ value, max = 100 }) {
  const { direction } = useLocale();
  
  return (
    <div className="progress-track">
      <div
        className="progress-fill"
        style={{
          width: `${(value / max) * 100}%`,
          // Grows from start side
          marginInlineEnd: 'auto',
        }}
        role="progressbar"
        aria-valuenow={value}
        aria-valuemin={0}
        aria-valuemax={max}
      />
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// TAILWIND RTL
// ════════════════════════════════════════════════════════════════

// tailwind.config.js
module.exports = {
  plugins: [
    require('tailwindcss-rtl'),
  ],
};

// Usage with RTL plugin
<div className="ms-4 ps-2 text-start">
  {/* ms = margin-start, ps = padding-start */}
  {/* text-start = text-align: start */}
</div>

<div className="ltr:ml-4 rtl:mr-4">
  {/* Explicit LTR/RTL variants */}
</div>
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# i18n BEST PRACTICES
# ════════════════════════════════════════════════════════════════

translation_keys:
  # Good: Descriptive, hierarchical
  good:
    - "nav.home"
    - "product.addToCart"
    - "cart.checkout.button"
    - "error.validation.email"
  
  # Bad: Cryptic, flat
  bad:
    - "btn1"
    - "msg_23"
    - "Add to Cart"  # Using source as key

content_guidelines:
  # Don't split sentences
  bad: |
    t('welcome') + ' ' + t('user') + '!'  # Breaks in languages with different word order
  
  good: |
    t('welcome.user', { name })  # "Welcome, {name}!" as single message

  # Don't concatenate translated strings
  bad: |
    t('item') + count + t('inCart')  # "item" + "5" + "in cart"
  
  good: |
    t('cart.items', { count })  # "{count, plural, ...}"

  # Include context for translators
  example: |
    {
      "button.submit": "Submit",
      "button.submit_description": "Submit button on contact form",
      "header.menu": "Menu",
      "header.menu_description": "Mobile navigation menu toggle"
    }

text_expansion:
  # Account for text expansion in UI
  german: "30-40% longer than English"
  finnish: "30-40% longer than English"
  chinese: "20-30% shorter than English"
  
  # Solutions:
  - Avoid fixed widths where possible
  - Use min-width instead of width
  - Test with longest translations
  - Use pseudo-localization

# ════════════════════════════════════════════════════════════════
# PSEUDO-LOCALIZATION
# ════════════════════════════════════════════════════════════════

pseudo_localization:
  purpose:
    - Find hardcoded strings
    - Test text expansion
    - Test character support
    - Find truncation issues
  
  example:
    original: "Welcome to our app"
    pseudo: "[Ŵêļĉöɱê ţö öûŕ àþþ!!! !!!]"
    # - Accented characters test encoding
    # - Extra characters test expansion
    # - Brackets make i18n strings obvious

# Implementation
import { pseudoLocalize } from '@formatjs/cli';

function usePseudoLocalization() {
  if (process.env.NODE_ENV === 'development') {
    // Transform all messages
    return Object.fromEntries(
      Object.entries(messages).map(([key, value]) => [
        key,
        pseudoLocalize(value),
      ])
    );
  }
  return messages;
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# i18n PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Hardcoded strings
# Bad
<button>Submit</button>
<p>Error: Invalid email</p>

# Good
<button><FormattedMessage id="form.submit" /></button>
<p><FormattedMessage id="error.email.invalid" /></p>

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: String concatenation
# Bad
const message = t('you_have') + count + t('items');
# Breaks in languages with different word order

# Good
const message = t('cart.itemCount', { count });
# "You have {count} items" as single translatable unit

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Assuming English plural rules
# Bad
const label = count === 1 ? 'item' : 'items';
# Breaks for Russian (3 forms), Arabic (6 forms)

# Good
<FormattedPlural value={count} one="# item" other="# items" />
# Or ICU: "{count, plural, one {# item} other {# items}}"

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Hardcoded date/number formats
# Bad
const formatted = `${date.getMonth()}/${date.getDate()}/${date.getFullYear()}`;
const price = `$${amount.toFixed(2)}`;

# Good
intl.formatDate(date, { dateStyle: 'short' });
intl.formatNumber(amount, { style: 'currency', currency: 'USD' });

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Physical CSS properties
# Bad
.sidebar {
  margin-left: 20px;
  text-align: left;
}
# Breaks in RTL

# Good
.sidebar {
  margin-inline-start: 20px;
  text-align: start;
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Fixed-width containers
# Bad
.button {
  width: 100px;  # German "Absenden" won't fit
}

# Good
.button {
  min-width: 100px;
  padding-inline: 16px;
}
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What's the difference between i18n and l10n?"**
> "i18n (internationalization) is designing your app to support multiple locales - extracting strings, using locale-aware APIs. Done by developers, done once. l10n (localization) is actually translating and adapting for specific locales - done by translators, per locale. i18n enables l10n."

**Q: "How do you handle pluralization?"**
> "Using ICU MessageFormat: '{count, plural, one {# item} other {# items}}'. Different languages have different plural rules - English has 2 forms, Russian has 3, Arabic has 6. Libraries like react-intl handle this using CLDR plural rules. Never hardcode 'count === 1'."

**Q: "How do you handle RTL languages?"**
> "Set dir='rtl' on HTML element. Use CSS logical properties: margin-inline-start instead of margin-left, text-align: start instead of left. Flip directional icons. Test with actual RTL content. Modern CSS and flexbox handle most layout automatically."

### Intermediate Questions

**Q: "How do you organize translation files?"**
> "JSON files per locale (en.json, de.json). Keys are hierarchical: 'nav.home', 'product.addToCart'. Lazy-load translations per locale. Store in source with code or external TMS (Crowdin, Lokalise). CI extracts new strings, flags missing translations."

**Q: "How do you handle dates and numbers?"**
> "Use Intl API (built-in) or react-intl. Never format manually. formatDate with locale-appropriate options. formatNumber for currencies, percentages. Formats differ: US is 12/25/2024 and $1,234.56, Germany is 25.12.2024 and 1.234,56 €."

**Q: "What is pseudo-localization?"**
> "Transforming text to test i18n without real translations. 'Hello' becomes '[Ħêļļö!!!]'. Benefits: find hardcoded strings (non-pseudo text is hardcoded), test text expansion (extra characters), test character encoding (accented chars). Run in development mode."

### Advanced Questions

**Q: "How do you handle gender in translations?"**
> "ICU select: '{gender, select, male {He} female {She} other {They}} joined'. Pass gender as variable. Some languages have more complex agreement - adjectives change based on gender. Provide context to translators. Consider inclusive alternatives."

**Q: "How do you manage translations at scale?"**
> "Translation Management System (Crowdin, Lokalise, Phrase). CI integration: extract strings on push, sync translations on PR. Context for translators: screenshots, descriptions. Review workflow: translator → reviewer → deploy. Track coverage metrics."

**Q: "How do you handle dynamic content like user-generated text?"**
> "User content isn't translated - stored in original language. But: format dates/numbers according to viewer's locale. Display language indicator if needed. For multi-language content (product descriptions), store translations in DB, select by user locale."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              i18n CHECKLIST                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRANSLATIONS:                                                  │
│  □ All UI text in translation files                            │
│  □ No string concatenation                                     │
│  □ ICU MessageFormat for plurals/select                        │
│  □ Context for translators                                     │
│                                                                 │
│  FORMATTING:                                                    │
│  □ Intl API for dates, numbers, currencies                     │
│  □ No manual date/number formatting                            │
│  □ Locale passed to all formatters                             │
│                                                                 │
│  RTL:                                                           │
│  □ CSS logical properties                                      │
│  □ dir attribute on html                                       │
│  □ Flip directional icons                                      │
│  □ Test with RTL content                                       │
│                                                                 │
│  TESTING:                                                       │
│  □ Pseudo-localization in dev                                  │
│  □ Test text expansion                                         │
│  □ Test actual RTL languages                                   │
│  □ CI checks for missing translations                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

LOCALE FORMATS:
┌────────────────────────────────────────────────────────────────┐
│ Date: en-US (12/25/24) vs de-DE (25.12.24) vs ja (2024年12月)  │
│ Number: en (1,234.56) vs de (1.234,56) vs fr (1 234,56)       │
│ Currency: en ($100) vs de (100 €) vs ja (¥100)                │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

