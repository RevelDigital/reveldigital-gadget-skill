# Signage Best Practices Reference (gadgets)

Framework-agnostic guidance for **theme**, **accessibility** (Section 508 / WCAG 2.1 AA), and
**readability**. Apply this to every scaffolded gadget regardless of framework — create `theme.css`
(below), import it globally, and follow the accessibility and readability rules.

Gadgets differ from full-screen apps: they render inside a **zone** placed by the template, usually
over a **transparent background**, and the zone's size is set by the designer. So this reference
keeps the theme **transparent-friendly** and **zone-relative** — there is intentionally **no
full-screen / overscan padding** (that's the template's job, not the gadget's).

---

## 1. Theme system — `src/theme.css`

A single source of truth for color and type, exposed as CSS custom properties. Recolor by changing
`--brand`. The body background stays **transparent** so the gadget layers cleanly over player
content. Default tokens are contrast-checked for WCAG AA against both light and dark backdrops by
keeping text high-contrast and adding a subtle text shadow option for unknown backdrops.

```css
/* =========================================================================
   Revel Digital gadget theme tokens — transparent-zone friendly
   ========================================================================= */
:root {
  --brand: #2f6fed;            /* primary/brand color — recolor here */
  --brand-contrast: #ffffff;   /* text/icon color ON --brand (AA: >= 4.5:1) */

  --fg: #ffffff;               /* primary text — gadgets often sit on dark/photo zones */
  --fg-muted: rgba(255,255,255,0.75);
  --accent: var(--brand);

  --space-1: 0.4em;
  --space-2: 0.8em;
  --space-3: 1.4em;
  --radius: 0.5em;

  /* Zone-relative type: scales with the gadget's own box, not the whole screen. */
  --font-sm:   clamp(0.9rem, 2cqi, 2rem);
  --font-body: clamp(1.1rem, 3cqi, 3rem);
  --font-lead: clamp(1.5rem, 5cqi, 5rem);
  --font-xl:   clamp(2rem, 9cqi, 9rem);

  --font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  --font-weight-normal: 600;   /* avoid thin weights on signage */
  --font-weight-bold: 800;
}

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* Transparent background is the gadget convention — never paint an opaque page bg. */
html, body { height: 100%; background: transparent; }

body {
  color: var(--fg);
  font-family: var(--font-family);
  font-weight: var(--font-weight-normal);
  font-size: var(--font-body);
  line-height: 1.3;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

/* The root container establishes a container-query context so `cqi` type scales to the zone. */
.gadget-root {
  container-type: size;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Optional legibility aid when the gadget may sit over an unknown/photographic backdrop. */
.on-photo { text-shadow: 0 1px 4px rgba(0,0,0,0.6); }

.accent { background: var(--brand); color: var(--brand-contrast); border-radius: var(--radius); }

:where(a, button, [tabindex]):focus-visible { outline: 3px solid var(--brand); outline-offset: 3px; }

@media (prefers-reduced-motion: no-preference) {
  .animate-fade { animation: fade-in 500ms ease both; }
}
@keyframes fade-in { from { opacity: 0 } to { opacity: 1 } }

.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; border: 0;
}
```

### Let designers override the theme via gadget preferences

Gadgets already expose a `style` / `color` preference type. Honor them at runtime so designers can
recolor without code changes:

```ts
const prefs = client.getPrefs()
const brand = prefs.getString('brandColor')        // datatype: color
if (brand) document.documentElement.style.setProperty('--brand', `#${brand}`)
// a `style` pref returns a CSS string (font-family/color/size) you can apply to a container
```

---

## 2. Accessibility — Section 508 / WCAG 2.1 AA

- **Language** — set `document.documentElement.lang` from `getLanguageCode()` (fall back to `en`).
- **Semantic structure** — use real headings and landmarks where the gadget has structure; don't
  build everything from bare `<div>`s.
- **Live regions** — content that updates on its own (clocks, tickers, rotating messages) belongs in
  an element with `aria-live`. Use `aria-live="off"` (or `aria-atomic`) for a once-a-second clock to
  avoid spamming assistive tech; use `aria-live="polite"` for rotating announcements.
- **Visible focus** — provided by the `:focus-visible` rule above; never remove outlines without a
  replacement.
- **Reduced motion** — wrap every animation/transition in `@media (prefers-reduced-motion: no-preference)`.
- **Color contrast** — text and essential UI must meet **AA 4.5:1** (3:1 for large/bold ≥ ~24px).
  Because a gadget may sit over varied backdrops, prefer high-contrast text plus the `.on-photo`
  shadow when the backdrop is unknown.
- **Don't rely on color alone** — pair color-coded status with text or an icon + label.
- **Images** — every `<img>` needs `alt`; decorative images use `alt=""`.

---

## 3. Readability for distance viewing

- **Zone-relative type** — the `--font-*` scale uses container query units (`cqi`) so text sizes to
  the gadget's own box. Establish the context with `.gadget-root { container-type: size }`.
- **Avoid thin weights** — `--font-weight-normal: 600`; hairline fonts disappear at distance.
- **High contrast, generous spacing** — use `--fg` on the backdrop; reserve `--fg-muted` for
  secondary labels; use the `--space-*` scale.
- **No scrollbars** — a signage zone can't scroll. Design for the fixed zone; if content can
  overflow, rotate/paginate on a timer instead of scrolling.
- **Adapt to the zone if needed** — `getWidth()` / `getHeight()` expose the gadget's pixel box for
  layout decisions beyond what container queries cover.
