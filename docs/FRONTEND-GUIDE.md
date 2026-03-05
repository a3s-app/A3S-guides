# Frontend Development Guide — Accessibility Fixes

### concorde_bs Theme · Concorde Career Colleges

**Theme path:** `web/themes/custom/concorde_bs/`

---

## Theme Structure (Relevant Files)

```
concorde_bs/
│
├── src/                        ← SOURCE (edit here)
│   ├── scss/
│   │   ├── styles.scss              Main SCSS entry point
│   │   ├── _a11y-fixes.scss         Accessibility CSS overrides
│   │   └── ...
│   └── js/
│       ├── scripts.js               Main JS entry point
│       ├── a11y-fixes.js            Accessibility JS fixes
│       └── ...
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
| Accessibility / cross-cutting CSS | `src/scss/_a11y-fixes.scss` | `src/scss/styles.scss` |
| Accessibility / cross-cutting JS | `src/js/a11y-fixes.js` | `src/js/scripts.js` |

Accessibility overrides touch multiple unrelated elements across the site (chat widget, search form, focus indicators, etc.). These go at the top level of `src/scss/` and `src/js/`.

---

<br>

## How to Add Accessibility CSS

**Step 1** — Open `src/scss/_a11y-fixes.scss` (create it if it doesn't exist yet)

The `_` prefix is a Sass convention called a **partial**. It tells the Sass compiler "don't compile this file on its own — it will be imported by another file." Without the `_`, the build would try to compile it as a standalone CSS file. JS files don't use this convention because the JS bundler (Rollup) handles imports differently — every file is resolved through `import` statements, so no naming prefix is needed.

Add your rules in this file. It's for overrides that don't belong to any single component — focus indicators, chat widget tweaks, screen reader fixes, etc.

You have access to all Bootstrap variables (`$gray-700`, `$spacer`, `$border-width`, etc.) and mixins (`@include media-breakpoint-down(sm)`).

<br>

**Step 2** — If the file is new, import it in `src/scss/styles.scss` after `components`

```scss
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

## How to Add Accessibility JS

**Step 1** — Open `src/js/a11y-fixes.js` (create it if it doesn't exist yet)

Use the `export default (() => { ... })()` pattern. This file is for JS fixes that span multiple unrelated elements.

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

### 2. Hard Refresh the Browser

Always use **Cmd + Shift + R** (Mac) or **Ctrl + Shift + R** (Windows) to bypass the browser cache.

<br>

### 3. Inspect in DevTools

Open the browser DevTools (`Cmd + Option + I`) and go to the **Network** tab:

- Filter by **CSS** → look for `styles.css` and confirm it loaded (200 status)
- Filter by **JS** → look for `scripts.min.js` and confirm it loaded (200 status)
- Click the file → **Preview** tab → search for your code

---

<br>

## Quick Test: Confirm CSS Pipeline Is Working

<br>

**1.** Add a temporary test rule at the top of `src/scss/_a11y-fixes.scss`

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

**2.** Build and clear cache

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run styles"
lando drush cr
```

<br>

**3.** Hard refresh the site (`Cmd + Shift + R`)

You should see a **bright red bar at the very top of the page** saying **"CSS PIPELINE WORKING"**.

<br>

**4.** Verify in compiled output

```bash
grep "CSS PIPELINE WORKING" web/themes/custom/concorde_bs/assets/css/styles.css
```

<br>

**5.** Clean up — remove the test rule from `_a11y-fixes.scss`, rebuild, and clear cache

---

<br>

## Quick Test: Confirm JS Pipeline Is Working

<br>

**1.** Add a temporary test block inside the IIFE in `src/js/a11y-fixes.js`

```javascript
// TEST — remove after confirming
const testBanner = document.createElement('div');
testBanner.textContent = 'JS PIPELINE WORKING';
testBanner.style.cssText = 'position:fixed;bottom:0;left:0;right:0;background:blue;color:white;text-align:center;padding:8px;font-weight:bold;font-size:14px;z-index:99999;';
document.body.appendChild(testBanner);
```

<br>

**2.** Build and clear cache

```bash
lando ssh -s node -c "cd /app/web/themes/custom/concorde_bs && npm run scripts"
lando drush cr
```

<br>

**3.** Hard refresh the site (`Cmd + Shift + R`)

You should see a **blue bar fixed to the bottom of the page** saying **"JS PIPELINE WORKING"**.

<br>

**4.** Verify in compiled output

```bash
grep "JS PIPELINE WORKING" web/themes/custom/concorde_bs/assets/js/scripts.js
```

<br>

**5.** Clean up — remove the test block from `a11y-fixes.js`, rebuild, and clear cache

---

<br>

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Code not in compiled `styles.css` or `scripts.js` | Missing import in entry point | Add `@import` in `styles.scss` or `import` in `scripts.js` |
| Build succeeds but site shows old styles/JS | Browser or Drupal cache | Hard refresh (`Cmd + Shift + R`) and `lando drush cr` |
| Hard refresh doesn't help | Browser is serving a stale cached version | Open a new incognito/private window, or clear browser cache entirely |
| Changes appear inside Lando but not on host | Lando file sync delay | Wait a few seconds, or restart Lando (`lando restart`) |
| `npm run styles` fails with Sass error | Syntax error in your SCSS | Check the error message for the file and line number |
