---
name: css-first
description: >
  Build and enforce a vanilla CSS-first architecture where all design decisions — colors, fonts, sizes, spacing, shadows, radius — live in a single centralized variables.css file as CSS custom properties. Use this skill whenever starting a new frontend project, setting up a design system, asking about theming or CSS architecture, or reviewing components for style fragmentation. Trigger when the user mentions "design tokens," "tokens," "theming," "style guide," "CSS variables," "centralize styles," "global CSS," "consistent typography," "brand colors," or when components have hex colors, font families, or sizes hardcoded inline or scattered across files. Also trigger when a user asks whether they need Tailwind, how to handle dark mode, or how to structure CSS. Opinionated: vanilla CSS with custom properties is the default. Tailwind is opt-in for layout utilities only — never for brand identity. All visual identity lives in variables.css.
---

# CSS-First Skill

**The rule:** all design decisions live in `variables.css` as CSS custom properties. Components consume them — they don't define them.

> "variables.css is the product. Everything else is plumbing."

**Vocabulary:** If the user says "design tokens" or "tokens" (Figma language), they mean CSS custom properties. See `references/design-tokens.md` for the full mapping.

---

## File Structure

```
styles/
├── variables.css       ← Design decisions only. No selectors, no rules.
├── globals.css         ← Applies variables to base HTML. Cascade does the work.
└── [name].module.css   ← Component-scoped styles, always referencing variables.
```

**When scaffolding a new project:** read `references/variables-template.css` and `references/globals-template.css` before generating these files.

**Import order:**
```js
import '../styles/variables.css';  // 1. variables first
import '../styles/globals.css';    // 2. base styles
// Tailwind (if present): import last so it doesn't override base styles
```

---

## What Goes Where

| Decision | File |
|---|---|
| Brand color, font, size, spacing value | `variables.css` |
| Base `body`, `h1`–`h6`, `a`, `code` styles | `globals.css` |
| Product-wide component classes (`.btn`) | `globals.css` |
| Component-specific styles | `[name].module.css` |
| Truly dynamic/computed values | inline `style={{}}` only |

**Every `.module.css` value must reference a variable. No bare hex, no bare px font sizes.**

---

## Inline Style Rules

Acceptable only for:
1. Truly dynamic/computed values (`height: \`${dynamicHeight}px\``)
2. CSS variable injection (`style={{ '--avatar-size': size + 'px' }}`)

Never for brand decisions. Never for values that appear in more than one place.

---

## Tailwind: Opt-In for Layout Only

**Permitted:** `flex`, `grid`, `gap-*`, `items-*`, `justify-*`, `col-span-*`, `relative`, `absolute`, `hidden`, `overflow-hidden`, `w-full`, `max-w-*`, `md:`, `lg:`, `sr-only`, `cursor-pointer`

**Forbidden:** any color (`text-blue-600`, `bg-gray-100`), typography (`font-sans`, `text-lg`, `font-bold`), radius (`rounded-lg`), shadows (`shadow-md`), `dark:` prefix, arbitrary values (`text-[#hex]`)

**Bridge tailwind.config.js to your variables (relay only — never source of truth):**
```js
module.exports = {
  theme: {
    extend: {
      colors: {
        interactive: 'var(--color-interactive)',
        surface: 'var(--color-surface)',
        border: 'var(--color-border)',
        text: { primary: 'var(--color-text-primary)', secondary: 'var(--color-text-secondary)' },
      },
      fontFamily: { sans: ['var(--font-sans)'], mono: ['var(--font-mono)'] },
    },
  },
}
```

---

## Legacy Tailwind Migration

For projects where Tailwind already owns colors, fonts, and spacing everywhere. Migrate progressively — never a big-bang rewrite.

**Step 1 — Establish the foundation without touching components.**
Create `variables.css` and `globals.css`, import before Tailwind. Nothing breaks.

**Step 2 — Bridge `tailwind.config.js` using `replace` not `extend`.**
Existing Tailwind color classes now resolve to your variables without touching a single component.

```js
module.exports = {
  theme: {
    colors: {
      transparent: 'transparent', current: 'currentColor',
      brand:   { primary: 'var(--color-brand-primary)' },
      bg:      { DEFAULT: 'var(--color-bg)', subtle: 'var(--color-bg-subtle)' },
      surface: { DEFAULT: 'var(--color-surface)' },
      border:  { DEFAULT: 'var(--color-border)', strong: 'var(--color-border-strong)' },
      text: {
        primary: 'var(--color-text-primary)', secondary: 'var(--color-text-secondary)',
        muted: 'var(--color-text-muted)', inverse: 'var(--color-text-inverse)',
      },
      interactive: { DEFAULT: 'var(--color-interactive)', hover: 'var(--color-interactive-hover)' },
      success: 'var(--color-success)', warning: 'var(--color-warning)', error: 'var(--color-error)',
    },
    fontFamily: { sans: ['var(--font-sans)'], mono: ['var(--font-mono)'] },
    borderRadius: {
      sm: 'var(--radius-sm)', DEFAULT: 'var(--radius-md)', md: 'var(--radius-md)',
      lg: 'var(--radius-lg)', full: 'var(--radius-full)',
    },
  },
}
```

**Step 3 — Find and prioritize violations:**
```bash
grep -r "text-\[#\|bg-\[#\|border-\[#" src/   # arbitrary hex — fix first
grep -r "font-sans\|font-bold\|text-lg" src/    # typography classes
```

Priority order: arbitrary values → colors → typography → brand spacing

**Step 4 — Migrate components one at a time.**
Replace Tailwind brand classes with a CSS Module referencing variables. Layout classes stay. Mixed state is fine — don't stop shipping while migrating.

**Step 5 — Harden against regression:**
```js
// .eslintrc
rules: { 'tailwindcss/no-arbitrary-value': 'error' }
```
Add `// MIGRATION IN PROGRESS — no new color/font/radius classes` to `tailwind.config.js`.

---

## Component Library Coexistence

CSS custom properties at `:root` don't conflict with libraries that ignore them. The challenge is when a library has its own style injection or token system.

**No component library** — full vanilla CSS, no caveats.

**Headless libraries (Radix, shadcn/ui, Headless UI)** — designed for CSS variables. Bridge their expected names to yours in `globals.css`:
```css
:root {
  --background: var(--color-bg);
  --foreground: var(--color-text-primary);
  --primary: var(--color-interactive);
  --primary-foreground: var(--color-text-inverse);
  --muted: var(--color-bg-subtle);
  --muted-foreground: var(--color-text-muted);
  --border: var(--color-border);
  --radius: var(--radius-md);
}
```
Never set their variables to raw hex — always point at your variables.

**Opinionated libraries (MUI, Chakra, Mantine, Ant Design)** — inject styles at runtime. Don't fight their cascade. Use their theme API as the bridge:
```js
createTheme({
  palette: {
    primary: { main: 'var(--color-interactive)', dark: 'var(--color-interactive-hover)' },
    background: { default: 'var(--color-bg)', paper: 'var(--color-surface)' },
    text: { primary: 'var(--color-text-primary)', secondary: 'var(--color-text-secondary)' },
  },
  typography: { fontFamily: 'var(--font-sans)' },
})
```
- ✅ Pass `var(--your-variable)` into their theme config — `variables.css` stays the source of truth
- ❌ Never override their injected classes (`.MuiButton-root`) in `globals.css`
- ❌ Never hardcode values in the theme file

**Runtime libraries (Framer Motion, GSAP, Stripe, Intercom)** — inject transforms/opacity or run in iframes. No conflict with your variables. No action needed.

---

## Audit Checklist

```
[ ] Hex/rgb literals outside variables.css
[ ] Font family declared outside variables.css
[ ] Font sizes as bare px/rem (not --text-* variables)
[ ] Inline style with a brand color or font
[ ] Same value appearing in 2+ files
[ ] Tailwind arbitrary values: text-[#hex], text-[16px]
[ ] Tailwind color/typography classes: text-blue-600, font-bold, text-lg
[ ] Dark mode per-component instead of variable override
[ ] .module.css file with bare values instead of variable references
[ ] Component redeclaring font-family or color already set in globals
```

---

> If you're deciding what the product looks like → `variables.css`.
> If you're applying that to elements → `globals.css` or a CSS module.
> Components consume the system. They don't define it.

References: `design-tokens.md` · `rationale.md` · `variables-template.css` · `globals-template.css`
