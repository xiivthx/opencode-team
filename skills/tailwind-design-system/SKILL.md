---
name: tailwind-design-system
description: Tailwind CSS v4 design-system playbook: define tokens with @theme, eliminate magic numbers, fully specify component states, and enforce an accessibility baseline. Use when adding/adjusting tokens, theming, or handing off UI classes to Lyra.
compatibility: opencode
metadata:
  owner: luna
  domain: design-system
  freedom: low
---

# Tailwind Design System (Luna)

This skill defines the project's Tailwind v4 design system rules:
- **Tokens in CSS** using `@theme` (CSS-first Tailwind v4)
- **No magic numbers** (promote to tokens)
- **State exhaustion** for components (default/hover/focus/disabled/loading/error)
- **A11y baseline** (keyboard + focus + semantics + contrast)
- **Handoff format** for Lyra (HTML structure + class strings + token usage)

## When to use me
Use this skill when:
- Adding or changing brand colors, spacing, radii, typography, shadows
- Implementing a new UI component or redesigning an existing one
- Cleaning up inconsistent styles or hardcoded values
- Ensuring accessibility compliance and predictable UI states
- Preparing a design-to-frontend handoff (Luna → Lyra)

## Related skills (routing)
- For UI async states and error UX mapping logic: use skill({ name: "rust-ui-patterns" }).
- For broader maintainability heuristics: use skill({ name: "engineering-principles" }).

---

# 1) Tailwind v4 CSS-first rules (non-negotiable)

## Use `@theme` for design tokens that should map to utilities
Theme variables defined via `@theme` influence which utilities exist (e.g., define `--color-mint-500` → you can use `bg-mint-500`).  
Define theme variables at the top-level (not nested under selectors).

### Minimal entrypoint pattern (example)
```css
@import "tailwindcss";

@theme {
  --color-brand-500: oklch(0.72 0.11 178);
  --radius-md: 0.75rem;
  --shadow-card: 0 10px 30px rgba(0,0,0,0.10);
}
````

## Use `:root` (or selectors) for variables NOT intended to create utilities

If a variable should not create utility classes, define it as a normal CSS variable.

## Avoid legacy JS config unless explicitly required

* Do not create or expand `tailwind.config.js` unless the project explicitly requests v3-style configuration.
* If legacy is required, use the v4 compatibility directives intentionally (not by habit).

---

# 2) Token taxonomy & naming (project convention)

## Goals

* Tokens are **semantic**, consistent, and reusable.
* Names communicate meaning, not implementation.

## Recommended namespaces

Use Tailwind’s namespaces where possible:

* Colors: `--color-*` (creates `bg-*`, `text-*`, `fill-*`, etc.)
* Spacing/sizing: `--spacing-*`
* Radius: `--radius-*`
* Shadows: `--shadow-*`
* Fonts: `--font-*`

## Naming rules

* Prefer semantic roles over raw palette when used in components:

  * Good: `--color-surface`, `--color-content`, `--color-brand`, `--color-danger`
  * Also acceptable: `--color-brand-500` (palette tokens) if you need multiple steps
* No raw hex/pixels in component classes if it can be a token.
* If Lyra sees `#123456` or `p-[13px]` in markup, treat it as a refactor task:

  * promote to `@theme` token (or `:root` if not utility-backed).

---

# 3) State exhaustion (required for every component)

Every component spec/hand-off MUST define classes for:

* `DEFAULT`
* `HOVER`
* `FOCUS / FOCUS-VISIBLE` (keyboard-visible, not hidden)
* `DISABLED`
* `LOADING` (skeleton/spinner integration)
* `ERROR` (validation/server failure visuals)

## Required focus behavior

* Keyboard users must see a clear focus indicator.
* Prefer `focus-visible:` variants over `focus:` for most interactive controls.
* Do not remove focus styles unless you add an accessible replacement.

### Example focus pattern (adjust tokens as needed)

```html
<button class="
  rounded-md
  bg-brand-500 text-white
  hover:bg-brand-600
  focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2
  focus-visible:outline-brand-500
  disabled:opacity-50 disabled:cursor-not-allowed
">
  Save
</button>
```

---

# 4) Accessibility baseline (A11y is non-negotiable)

Minimum bar for every interactive UI:

* Keyboard navigation works (tab order makes sense)
* Visible focus indication (not color-only)
* Inputs have labels (or `aria-label` for icon-only)
* Error messages are perceivable and connected to fields where applicable
* Semantic HTML preferred (`button`, `nav`, `main`, `label`) over div soup
* Contrast should meet WCAG AA where applicable (treat as required unless explicitly exempted)

If the UI depends on color to convey meaning (e.g., error), add text/icon/aria cues.

---

# 5) Dark mode / theming strategy (practical guidance)

## Prefer token overrides rather than new classes

Define tokens in `@theme`, then override the same CSS variables under a theme selector:

* `.dark { --color-surface: ... }` or
* `:root[data-theme="dark"] { ... }`

Use utilities (`dark:*`) for structural layout differences only when necessary.

---

# 6) Handoff format to Lyra (required output)

When handing a component to Lyra, provide:

## Component spec block

* **Component name**
* **HTML structure** (minimal, semantic)
* **Class strings** for each element
* **State classes** (DEFAULT/HOVER/FOCUS/DISABLED/LOADING/ERROR)
* **Token list used** (which tokens must exist in `@theme`)
* **A11y notes** (labels, aria attributes, keyboard behavior)

### Example handoff snippet

```md
## Component: PrimaryButton

### Structure
<button type="button">Label</button>

### Classes
DEFAULT:
- bg-brand-500 text-white rounded-md px-4 py-2 shadow-card

HOVER:
- hover:bg-brand-600

FOCUS:
- focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-brand-500

DISABLED:
- disabled:opacity-50 disabled:cursor-not-allowed

LOADING:
- aria-busy=true + show spinner (layout preserved)

ERROR (if applicable):
- ring-2 ring-danger-500 (or token-based equivalent)

### Tokens required
- --color-brand-500, --color-brand-600
- --shadow-card
```

## Maintainability escape hatch (important)

If Lyra reports the design is “too complex to maintain”:

* Provide a **simplified V2** variant with fewer tokens/states and less bespoke styling.
* Prefer consistency over cleverness.

---

# 7) Verification checklist (before merge)

* [ ] Tokens added/updated in `@theme` (not scattered magic numbers)
* [ ] Component states fully defined (state exhaustion)
* [ ] Focus-visible behavior present and visible
* [ ] Semantic HTML used; labels/aria present
* [ ] No sensitive information exposed in CSS (e.g., embedding data)
* [ ] Handoff includes HTML + classes + token list + a11y notes
