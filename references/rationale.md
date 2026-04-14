# Why Centralize Styles? (Stakeholder Rationale)

Use this when team members push back on the CSS custom properties approach.

---

## "We already have Tailwind, why add more CSS?"

Tailwind is a utility framework — it gives you a vocabulary for layout and composition. It is not a design system. The difference:

- **Tailwind**: "arrange these boxes, space them out, make this flex"
- **Design system**: "this is what our brand looks like — these are our colors, our fonts, our voice"

Using Tailwind for brand decisions (`text-blue-600` for your primary CTA) means your brand lives inside a third-party config file's color palette. When Tailwind releases v4 and renames classes, or when you want to white-label the product, you're doing find-replace across hundreds of files.

With CSS custom properties: you change `--color-brand-primary` in one place.

---

## "CSS variables have less tooling support"

This was true in 2017. Today:
- All modern browsers support CSS custom properties natively (99.5%+ coverage)
- TypeScript can type CSS variable names
- Design tools like Figma export tokens as CSS variables
- CSS custom properties cascade, are inspectable in DevTools, and work with `:hover`, media queries, and container queries natively

---

## "Tailwind's JIT handles theming fine"

JIT handles *generating* classes from config. It doesn't handle:
- Runtime theming (dark/light toggle without page reload)
- Dynamic values from user preferences
- Component libraries that don't use Tailwind
- Semantic aliasing (why `bg-slate-100` is your "subtle background" requires reading code, not config)

CSS custom properties are runtime values. Tailwind classes are build-time strings. This is a fundamental difference.

---

## "Inline styles are easier to read in JSX"

Inline styles in JSX are fine for truly dynamic values. They are a maintenance problem when they carry brand meaning:

```jsx
// This is a policy decision masquerading as a style
<button style={{ backgroundColor: '#2563eb', borderRadius: '6px' }}>

// Six months later, in a different component:
<div style={{ color: '#2563eb' }}>  ← is this the same blue? who knows?
```

Tokens make implicit conventions explicit. When you see `var(--color-interactive)`, you know it's the interactive blue, wherever it appears.

---

## "We can just extend tailwind.config.js"

Yes, and you should — but the *values* should live in CSS vars, not in the config. The config should bridge Tailwind's class system to your tokens:

```js
// ✅ Tailwind config as a bridge (values in tokens.css)
colors: { primary: 'var(--color-brand-primary)' }

// ❌ Tailwind config as the source of truth
colors: { primary: '#2563eb' }
```

The difference: in the first case, `tokens.css` is the style guide. In the second case, the style guide is... a JavaScript object that only applies to Tailwind classes.

---

## "This adds overhead at project start"

Setting up `tokens.css` takes ~30 minutes. Not setting it up costs hours of:
- Tracking down which shade of gray is "our border gray"
- Updating 40 components when the brand refreshes
- Debugging why dark mode broke on 3 components
- Explaining to a new dev why there are 7 slightly different blues

This is a decision that pays for itself on day 2.
