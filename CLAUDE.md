# Tools — conventions

Collection of small, single-file front-end tools. No build process.

## File structure

Each tool lives in its own directory as a self-contained HTML file:

```
tool-slug/index.html
```

Root `index.html` is the home page. Add each new tool there under the appropriate category heading.

## Every tool file must

- Link the shared token file: `<link rel="stylesheet" href="../tokens.css">`
- Include the theme-init script (see below) as the **first** `<script>` in `<head>` — it must run before paint to avoid flash
- Include the standard footer (Home link, View source link, Updated date)
- Set `<meta name="last-modified" content="YYYY-MM-DD">` and update it on each edit
- Put all styles in an inline `<style>` block — no external stylesheets beyond `tokens.css`
- Put all logic in an inline `<script>` block — no external scripts, no frameworks

## Theme

Themes are applied via `data-theme="light" | "dark"` on `<html>`. Always init before paint:

```html
<script>
  (function () {
    const stored = localStorage.getItem('theme');
    const preferred = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    document.documentElement.dataset.theme = stored || preferred;
  })();
</script>
```

Toggle wiring (JS, runs after DOM):

```js
const themeToggle = document.getElementById('theme-toggle');
function syncThemeLabel() {
  themeToggle.textContent = document.documentElement.dataset.theme === 'dark' ? '☀ Light' : '☾ Dark';
}
syncThemeLabel();
themeToggle.addEventListener('click', () => {
  const next = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
  document.documentElement.dataset.theme = next;
  localStorage.setItem('theme', next);
  syncThemeLabel();
});
```

## Standard page shell

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="last-modified" content="YYYY-MM-DD">
  <title>Tool Name — Tools</title>
  <link rel="stylesheet" href="../tokens.css">
  <script>/* theme init — see above */</script>
  <style>/* tool styles */</style>
</head>
<body>
  <div class="container">
    <header class="page-header">
      <h1 class="page-title">Tool Name</h1>
      <p class="page-subtitle">One-line description.</p>
    </header>
    <!-- tool UI -->
  </div>

  <footer class="site-footer">
    <div class="footer-links">
      <a href="../">← Home</a>
      <span class="footer-sep">|</span>
      <a href="https://github.com/jamboid/tools/blob/main/tool-slug/index.html"
         target="_blank" rel="noopener">View source</a>
      <span class="footer-sep">|</span>
      <span>Updated: <span id="last-updated"></span></span>
    </div>
    <button class="theme-toggle" id="theme-toggle" aria-label="Toggle colour theme"></button>
  </footer>

  <script>/* tool logic + theme toggle wiring + last-updated */</script>
</body>
</html>
```

Last-updated wiring:

```js
const lastMod = document.querySelector('meta[name="last-modified"]')?.content;
if (lastMod) document.getElementById('last-updated').textContent = lastMod;
```

## Standard CSS

Copy this block into every tool's `<style>` and adjust as needed:

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: var(--font-ui);
  font-size: var(--font-size-base);
  line-height: var(--line-height-base);
  color: var(--color-text);
  background: var(--color-bg);
  min-height: 100vh;
  display: grid;
  grid-template-rows: 1fr auto;  /* container grows, footer sticks to bottom */
}

.container {
  max-width: 640px;   /* adjust per tool — converter uses 600px, index 720px */
  margin: 0 auto;
  padding: var(--space-10) var(--space-6);
  width: 100%;
}

.page-header  { margin-bottom: var(--space-8); }
.page-title   { font-size: var(--font-size-2xl); font-weight: var(--font-weight-semibold); margin-bottom: var(--space-1); }
.page-subtitle { color: var(--color-muted); font-size: var(--font-size-sm); }

.site-footer {
  border-top: 1px solid var(--color-border);
  padding: var(--space-3) var(--space-6);
  display: grid;
  grid-template-columns: 1fr auto;
  align-items: center;
  gap: var(--space-4);
}

.footer-links {
  display: grid;
  grid-auto-flow: column;
  grid-auto-columns: max-content;
  align-items: center;
  gap: var(--space-3);
  font-size: var(--font-size-xs);
  color: var(--color-muted);
}

.footer-links a { color: var(--color-muted); text-decoration: none; }
.footer-links a:hover { color: var(--color-accent); text-decoration: underline; }
.footer-sep { opacity: 0.4; }

.theme-toggle {
  background: none;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  padding: var(--space-1) var(--space-3);
  cursor: pointer;
  font-family: var(--font-ui);
  font-size: var(--font-size-xs);
  color: var(--color-muted);
  transition: background var(--transition-base), color var(--transition-base);
}
.theme-toggle:hover { background: var(--color-surface); color: var(--color-text); }
```

## Design rules

**Aesthetic:** GitHub UI-inspired. Utilitarian, not decorative.

- Structure with **1px borders** (`var(--color-border)`), not shadows
- Flat components — no gradients, no heavy rounding on structural elements
- `--radius-md` (6px) for interactive controls; `--radius-sm` (3px) for small chips/badges
- Shadows (`--shadow-sm`, `--shadow-md`) only for floating/overlapping elements

**Layout:** Grid throughout — no `display: flex` for page-level layout.

**Colour:** Use tokens only — never hardcode colours. Accent (`--color-accent`) for links and primary actions. Muted (`--color-muted`) for labels, captions, secondary text.

**Typography:**
- Body copy: `var(--font-ui)` at `var(--font-size-base)`
- Code/values: `var(--font-mono)` at `var(--font-size-sm)`
- Section labels: `var(--font-size-xs)`, `var(--font-weight-semibold)`, uppercase, `letter-spacing: 0.05em`, `color: var(--color-muted)`

**Spacing:** Use spacing tokens (`--space-*`). 4px base grid. Don't mix token spacing with raw `px` values.

**Interactive states:**
- Focus: `box-shadow: 0 0 0 3px color-mix(in srgb, var(--color-accent) 20%, transparent)` + `border-color: var(--color-accent)`
- Hover on links/buttons: `--color-accent` text or `--color-surface` background
- Error state: `border-color: var(--color-danger)`, same focus ring pattern with danger colour
- Disabled: `opacity: 0.35; cursor: default`

**Transitions:** `var(--transition-base)` (80ms ease) on `background`, `color`, `border-color`. Not on layout properties.

## Adding a tool to the index

In `index.html`, add a `<li>` under the appropriate `<ul>`:

```html
<li class="tool-item">
  <a class="tool-link" href="tool-slug/">
    <span class="tool-name">Tool Name</span>
    <span class="tool-desc">One-line description</span>
  </a>
</li>
```

If a new category is needed, copy the `<section class="category">` block and give it a new heading.

## GitHub URL

Source links currently use `https://github.com/placeholder/tools` — update to the real repo URL once known.
