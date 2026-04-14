# skill-css-first

A Claude Code skill that enforces a vanilla CSS-first architecture — all design decisions centralized in a single `variables.css` file using CSS custom properties, with components that consume the system rather than define it.

Built and maintained by [Z12 Ventures](https://z12ventures.com).

---

## The Problem

CSS fragmentation is one of the most common causes of frontend maintenance debt. It looks like this:

- Brand colors as hex literals scattered across 40 component files
- Font families declared in `tailwind.config.js`, two component `style={}` props, and a globals file
- Dark mode handled differently in every component that needed it
- A "rebrand" that takes two weeks because there's no single source of truth

The fix isn't a new framework. It's a discipline: **one file owns what the product looks like. Everything else references it.**

---

## What This Skill Does

When installed, this skill guides Claude Code to:

- Create and maintain a `variables.css` file as the single source of truth for all design decisions
- Apply those variables through a `globals.css` base layer so components inherit styles via cascade
- Use CSS Modules for component-scoped styles that still reference the central variables
- Constrain Tailwind (if present) to layout utilities only — never colors, fonts, or brand decisions
- Flag style violations during code review: hardcoded hex values, inline font declarations, repeated magic numbers
- Translate Figma/design token vocabulary into the correct CSS custom property equivalents

### The default file structure it produces

```
styles/
├── variables.css       ← The style guide. Every design decision lives here.
├── globals.css         ← Applies variables to base HTML. Cascade does the work.
└── [component].module.css  ← Scoped styles that reference variables, never hardcode values.
```

---

## Install

The install method depends on which Claude product you're using.

---

### Claude Code

Skills live in a folder — no packaging needed. Pick the location based on scope.

**Personal install (all projects):**

```bash
git clone https://github.com/z12ventures/skill-css-first ~/.claude/skills/css-first
```

**Project install (one project, shareable with your team):**

```bash
git clone https://github.com/z12ventures/skill-css-first .claude/skills/css-first
```

Commit `.claude/skills/css-first/` to version control to share it with your team.

Verify it's active by asking Claude: `What skills are available?` — you should see `css-first` in the list. You can also invoke it directly with `/css-first`.

---

### Claude.ai and Claude Desktop

Skills are uploaded as a ZIP file through the settings UI.

**Prerequisites:** Code execution must be enabled first.
- **Free, Pro, Max:** Settings → Capabilities → enable **Code execution and file creation**
- **Team, Enterprise:** Organization settings → Skills → enable both **Code execution** and **Skills**

**Install steps:**

1. [Download the ZIP](https://github.com/z12ventures/skill-css-first/archive/refs/heads/main.zip)
2. Go to [Customize → Skills](https://claude.ai/customize/skills)
3. Click **+** → **Create skill** → **Upload a skill**
4. Upload the ZIP file
5. Toggle the skill on

Once enabled, the skill triggers automatically when you're working on CSS architecture. No slash command needed — Claude applies it when relevant.

> **Note:** Skills on claude.ai require the code execution environment to be enabled. If you don't see the Skills section, check that code execution is turned on in your settings.

---

## Usage

Once installed, the skill triggers automatically. You don't need to invoke it explicitly. Examples of prompts that will activate it:

```
Set up the CSS architecture for my new Next.js project
```
```
We have colors scattered across a bunch of components — help me centralize them
```
```
I'm coming from Figma and need to implement our design tokens
```
```
Review this component for style issues
```
```
Should I use Tailwind for this project?
```
```
How should I handle dark mode?
```

Claude will apply the opinionated architecture from the skill: `variables.css` first, `globals.css` as the base layer, CSS Modules for components.

---

## What's Included

```
skill-css-first/
├── SKILL.md                        ← Core skill instructions and architecture rules
└── references/
    ├── variables-template.css      ← Full starter variables.css to copy into a new project
    ├── globals-template.css        ← Full starter globals.css to copy into a new project
    ├── design-tokens.md            ← Figma/token vocabulary → CSS variables mapping
    └── rationale.md                ← The argument for this approach (for team discussions)
```

### On Tailwind

This skill is opinionated: vanilla CSS is the default. Tailwind is not assumed to be present.

If your project uses Tailwind, the skill constrains it to layout utilities only (`flex`, `grid`, `gap`, positioning, display, overflow) and forbids it from touching colors, typography, radius, shadows, or anything carrying brand meaning. It also shows you how to wire `tailwind.config.js` as a relay to your CSS variables so the token layer remains the source of truth.

### On design tokens

"Design tokens" is the vocabulary used by Figma, Tokens Studio, and Style Dictionary. On the web, design tokens are CSS custom properties. The skill includes a reference that maps the full design token vocabulary to its CSS equivalent — useful if you're working with a designer who thinks in token terms, or if you're importing a Figma variable export.

---

## Philosophy

> If you have to change a brand color in more than one file, your architecture is broken.

Modern CSS — custom properties, cascade, CSS Modules, `clamp()`, container queries — is capable enough that a framework is rarely the right answer for design system concerns. The gap that made Tailwind compelling in 2019 has largely closed.

The primitives:

- **`variables.css`** declares what the product looks like. No selectors. No rules. Only decisions.
- **`globals.css`** applies those decisions to HTML elements. The cascade propagates them to components.
- **CSS Modules** scope component styles without creating islands that ignore the system.
- **Inline styles** are reserved for genuinely dynamic values. Never for brand decisions.

Dark mode, theming, and white-labeling all reduce to: override the right variables in the right selector. No framework needed.

---

## Contributing

Issues and PRs welcome. If you have a pattern that belongs in the skill — a common violation, a migration example, a framework-specific coexistence rule — open an issue describing it.

---

## License

[MIT](./LICENSE) © [Z12 Ventures LLC](https://z12ventures.com)
