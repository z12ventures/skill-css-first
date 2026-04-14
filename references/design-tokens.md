# Design Tokens → CSS Variables

"Design tokens" is the term used by Figma, Style Dictionary, and the broader design tool ecosystem. On the web, design tokens are implemented as CSS custom properties. These are the same concept — different vocabulary.

Use this reference when a user comes from a design background and uses token language.

---

## Vocabulary Map

| Design tool term | Web equivalent |
|---|---|
| Design token | CSS custom property (`--variable-name`) |
| Token file / token set | `variables.css` |
| Primitive / global token | Raw value: `--color-blue-600: #2563eb` |
| Semantic / alias token | Semantic reference: `--color-interactive: var(--color-blue-600)` |
| Token value | The CSS value after the colon |
| Token name | The CSS property name (e.g. `--color-interactive`) |
| Theme / token mode | `:root` overrides in a `[data-theme="dark"]` block |
| Token group | CSS comment section (`/* ─── Colors ─── */`) |
| Token export | The `variables.css` file itself |

---

## The Two-Layer Pattern

Design tools distinguish between **primitive tokens** (raw values) and **semantic tokens** (named purposes). This maps directly to how variables.css should be structured:

```css
/* ─── Primitive (raw values, rarely used directly in components) ─── */
:root {
  --blue-500: #3b82f6;
  --blue-600: #2563eb;
  --blue-700: #1d4ed8;
  --gray-100: #f1f5f9;
  --gray-900: #0f172a;
}

/* ─── Semantic (named purposes, what components actually use) ─── */
:root {
  --color-interactive:       var(--blue-600);
  --color-interactive-hover: var(--blue-700);
  --color-text-primary:      var(--gray-900);
  --color-bg-subtle:         var(--gray-100);
}
```

Components only ever reference semantic variables. This means swapping a brand color means changing one primitive value and every semantic reference updates automatically.

---

## Figma Token Plugins

If the user is working with Figma token plugins (Tokens Studio, Variables, etc.), they typically export a JSON file that maps to CSS variables. The output looks like:

```json
{
  "color": {
    "brand": {
      "primary": { "value": "#2563eb", "type": "color" }
    },
    "interactive": { "value": "{color.brand.primary}", "type": "color" }
  }
}
```

This maps to:
```css
--color-brand-primary: #2563eb;
--color-interactive: var(--color-brand-primary);
```

If a user is exporting tokens from Figma and wants to use them in a project, the translation is direct. Style Dictionary can automate this export → CSS variables pipeline if the project warrants it, but for most projects hand-writing variables.css is faster and clearer.

---

## When "Tokens" Language Is Actually Appropriate

Use the word "token" (not "variable") only when:
1. The project uses a multi-platform design system (web + iOS + Android) where the same values need to be shared across platforms via a tool like Style Dictionary
2. The user is explicitly working with a token management tool like Tokens Studio for Figma
3. Communicating with designers who use Figma and think in token terms

In all other cases, say "CSS variable" or "CSS custom property." It's more precise, searchable, and what a developer will look for in documentation.
