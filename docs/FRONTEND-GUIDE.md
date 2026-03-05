# Frontend Development Guide

### concorde_bs Theme · Concorde Career Colleges

---

## Theme Structure

```
concorde_bs/
│
├── src/                        ← SOURCE (edit here)
│   ├── scss/
│   │   ├── styles.scss              Main SCSS entry point
│   │   ├── _variables.scss          Theme variables
│   │   ├── _components.scss         Imports all component partials
│   │   ├── _a11y-fixes.scss          Accessibility overrides (top-level)
│   │   └── components/
│   │       ├── _navbar.scss
│   │       ├── _buttons.scss
│   │       └── ...
│   └── js/
│       ├── scripts.js               Main JS entry point
│       ├── a11y-fixes.js            Accessibility fixes (top-level)
│       └── components/
│           ├── ada.js
│           ├── dropdown-megamenu.js
│           └── ...
│
├── assets/                     ← OUTPUT (auto-generated, do not edit)
│   ├── css/styles.css
│   └── js/scripts.min.js
│
├── build/                      ← Build scripts (Node.js)
└── package.json                ← npm scripts
```

> **Source** goes in `src/`. The build compiles it to `assets/`. Drupal loads from `assets/`.

---

<br>

## Where to Put What

| Type of Change | Where It Goes | Imported In |
|---------------|---------------|-------------|
| Component styles (navbar, buttons, forms, etc.) | `src/scss/components/_name.scss` | `src/scss/_components.scss` |
| Accessibility / cross-cutting overrides | `src/scss/_a11y-fixes.scss` | `src/scss/styles.scss` |
| Component JS (dropdown, carousel, etc.) | `src/js/components/name.js` | `src/js/scripts.js` |
| Accessibility / cross-cutting JS | `src/js/a11y-fixes.js` | `src/js/scripts.js` |

**Component styles** are scoped to a single UI element (navbar, card, accordion).

**Accessibility overrides** touch multiple unrelated elements across the site (chat widget, search form, sticky footer, focus indicators). These go at the top level of `src/scss/` and `src/js/`, not inside `components/`.

---

<br>

## How to Add Component CSS

**Step 1** — Create a new SCSS partial inside `src/scss/components/`

```
src/scss/components/_my-component.scss
```

You have access to all Bootstrap variables (`$gray-700`, `$spacer`, `$border-width`, etc.) and mixins (`@include media-breakpoint-down(sm)`).

<br>

**Step 2** — Open `src/scss/_components.scss` and add your import at the bottom

```scss
@import "components/my-component";
```

<br>

**Step 3** — Build

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles"
```

<br>

**Step 4** — Clear Drupal cache

```bash
lando drush cr
```

<br>

**Step 5** — Open https://concorde-www.lndo.site/ and hard refresh (`Cmd + Shift + R`)

---

<br>

## How to Add Accessibility / Cross-Cutting CSS

**Step 1** — Open `src/scss/_a11y-fixes.scss` (create it if it doesn't exist yet)

Add your rules there. This file is for overrides that don't belong to any single component — focus indicators, chat widget tweaks, screen reader fixes, etc.

<br>

**Step 2** — If the file is new, import it in `src/scss/styles.scss` after `components`

```scss
@import "root";
@import "reboot";
@import "components";
@import "a11y-fixes";
```

<br>

**Step 3** — Build

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles"
```

<br>

**Step 4** — Clear Drupal cache

```bash
lando drush cr
```

<br>

**Step 5** — Open https://concorde-www.lndo.site/ and hard refresh (`Cmd + Shift + R`)

---

<br>

## How to Add Component JS

**Step 1** — Create a new JS module inside `src/js/components/`

```
src/js/components/my-feature.js
```

Use the `export default (() => { ... })()` pattern to match existing components.

<br>

**Step 2** — Open `src/js/scripts.js` and add your import at the bottom

```javascript
import "./components/my-feature";
```

<br>

**Step 3** — Build

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run scripts"
```

<br>

**Step 4** — Clear Drupal cache

```bash
lando drush cr
```

<br>

**Step 5** — Open https://concorde-www.lndo.site/ and hard refresh (`Cmd + Shift + R`)

---

<br>

## How to Add Accessibility / Cross-Cutting JS

**Step 1** — Open `src/js/a11y-fixes.js` (create it if it doesn't exist yet)

Use the same `export default (() => { ... })()` pattern. This file is for JS fixes that span multiple unrelated elements.

<br>

**Step 2** — If the file is new, import it in `src/js/scripts.js` at the bottom

```javascript
import "./a11y-fixes";
```

<br>

**Step 3** — Build

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run scripts"
```

<br>

**Step 4** — Clear Drupal cache

```bash
lando drush cr
```

<br>

**Step 5** — Open https://concorde-www.lndo.site/ and hard refresh (`Cmd + Shift + R`)

---

<br>

## Build Commands

| Command | Scope | When to Use |
|---------|-------|-------------|
| `npm run styles` | CSS only | After changing any `.scss` file |
| `npm run scripts` | JS only | After changing any `.js` file |
| `npm run build` | Full build — CSS + JS + icons + vendor | Only when needed (adds icons, vendor assets) |
| `npm run dev` | Full build + watch mode | During active development |

All commands must be run through Lando:

```bash
# After CSS changes
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles"

# After JS changes
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run scripts"

# After both CSS and JS changes
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles && npm run scripts"
```

> Do **not** run `npm run build` for routine CSS/JS changes — it rebuilds icons and vendor assets unnecessarily. Use `npm run styles` and `npm run scripts` instead.

> Always **build before pushing**. Always **clear cache after building**.

---

<br>

## Verifying Your Changes

After building and clearing cache, your changes may not show up in the browser. Follow these steps to confirm the full pipeline is working.

<br>

### 1. Check the Compiled Output

After running `npm run styles` or `npm run scripts`, verify your code made it into the compiled files:

```bash
# CSS — search for a unique string from your SCSS
grep "your-selector-or-keyword" web/themes/custom/concorde_bs/assets/css/styles.css

# JS — search for a unique string from your JS
grep "your-selector-or-keyword" web/themes/custom/concorde_bs/assets/js/scripts.js
```

If your code is **not found** in the compiled output, the build did not pick up your source file. Check:
- Is the file saved?
- Is the `@import` (SCSS) or `import` (JS) line present in the entry point?
- Run the build command again.

<br>

### 2. Disable CSS/JS Aggregation (Development Only)

Drupal aggregates CSS and JS by default, which means it may serve a cached bundle that does not include your latest build. Disable aggregation during development:

```bash
lando drush config:set system.performance css.preprocess 0 -y
lando drush config:set system.performance js.preprocess 0 -y
lando drush cr
```

> **Re-enable before pushing to production:**
> ```bash
> lando drush config:set system.performance css.preprocess 1 -y
> lando drush config:set system.performance js.preprocess 1 -y
> lando drush cr
> ```

<br>

### 3. Hard Refresh the Browser

Always use **Cmd + Shift + R** (Mac) or **Ctrl + Shift + R** (Windows) to bypass the browser cache.

<br>

### 4. Inspect in DevTools

Open the browser DevTools (`Cmd + Option + I`) and go to the **Network** tab:

- Filter by **CSS** → look for `styles.css` and confirm it loaded (200 status)
- Filter by **JS** → look for `scripts.min.js` and confirm it loaded (200 status)
- Click the file → **Preview** tab → search for your code

---

<br>

## Quick Test: Confirm the CSS Pipeline

Use this to confirm your local setup compiles and serves CSS correctly.

<br>

**1.** Add a temporary test rule at the top of `src/scss/_a11y-fixes.scss` (or any SCSS file you're working in)

```scss
// TEST — remove after confirming
body::before {
  content: "CSS PIPELINE WORKING";
  display: block;
  background: red;
  color: white;
  text-align: center;
  padding: 8px;
  font-weight: bold;
  font-size: 14px;
  z-index: 99999;
  position: relative;
}
```

<br>

**2.** Build, clear cache, and hard refresh

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles"
lando drush cr
```

<br>

**3.** What you should see

A **bright red bar at the very top of the page** saying **"CSS PIPELINE WORKING"**.

<br>

**4.** Verify in compiled output

```bash
grep "CSS PIPELINE WORKING" web/themes/custom/concorde_bs/assets/css/styles.css
```

<br>

**5.** Clean up — remove the test rule when done

---

<br>

## Quick Test: Confirm the JS Pipeline

Use this to confirm your local setup compiles and serves JS correctly.

<br>

**1.** Add a temporary test block at the top of the IIFE in `src/js/a11y-fixes.js` (or any JS file you're working in)

```javascript
// TEST — remove after confirming
const testBanner = document.createElement('div');
testBanner.textContent = 'JS PIPELINE WORKING';
testBanner.style.cssText = 'position:fixed;bottom:0;left:0;right:0;background:blue;color:white;text-align:center;padding:8px;font-weight:bold;font-size:14px;z-index:99999;';
document.body.appendChild(testBanner);
```

<br>

**2.** Build, clear cache, and hard refresh

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run scripts"
lando drush cr
```

<br>

**3.** What you should see

A **blue bar fixed to the bottom of the page** saying **"JS PIPELINE WORKING"**.

<br>

**4.** Verify in compiled output

```bash
grep "JS PIPELINE WORKING" web/themes/custom/concorde_bs/assets/js/scripts.js
```

<br>

**5.** Clean up — remove the test block when done

---

<br>

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Code not in compiled `styles.css` or `scripts.js` | Missing import in entry point | Add `@import` in `styles.scss` or `import` in `scripts.js` |
| Build succeeds but site shows old styles/JS | Drupal CSS/JS aggregation is on | Disable aggregation (see above) and `lando drush cr` |
| Hard refresh doesn't help | Browser is serving a stale cached version | Open a new incognito/private window, or clear browser cache entirely |
| Changes appear inside Lando but not on host | Lando file sync delay | Wait a few seconds, or restart Lando (`lando restart`) |
| `npm run styles` fails with Sass error | Syntax error in your SCSS | Check the error message for the file and line number |
