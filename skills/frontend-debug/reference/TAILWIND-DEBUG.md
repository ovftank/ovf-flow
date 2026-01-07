# Tailwind CSS v4 Debug Guide

Comprehensive debug guide for Tailwind CSS v4 (Released Jan 2025).

## Quick Reference: v3 vs v4

| Feature           | v3                      | v4                       |
|-------------------|-------------------------|--------------------------|
| Configuration     | `tailwind.config.js`    | `@theme` in CSS          |
| Import            | `@tailwind` directives  | `@import "tailwindcss"`  |
| Variant stacking  | Right-to-left           | Left-to-right            |
| Important         | `!flex`                 | `flex!`                  |
| Custom utilities  | `@layer utilities`      | `@utility`               |
| Container queries | Plugin required         | Built-in `@container`    |
| PostCSS           | Built-in                | `@tailwindcss/postcss`   |

**CRITICAL: NEVER use `tailwind.config.js` in Tailwind CSS v4.**

## 1. Configuration Issues

### Problem: Styles not generating

**Symptoms:**

- Utility classes have no effect
- Build completes but styles don't apply
- No CSS output

**Root Causes:**

1. **Using `tailwind.config.js` (DEPRECATED)**

```bash
# Check if file exists
ls tailwind.config.js
# If yes -> DELETE THIS FILE in v4
```

1. **Wrong import syntax**

```css
/* WRONG - v3 syntax */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* CORRECT - v4 syntax */
@import "tailwindcss";

/* OR explicit layers */
@layer theme, base, components, utilities;
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
```

1. **Missing @theme directive**

```css
/* When customizing theme, MUST use @theme */
@import "tailwindcss";

@theme {
  --color-brand-500: oklch(0.72 0.11 221.19);
  --font-display: "Satoshi", sans-serif;
  --spacing-unit: 4px;
}
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check Tailwind version
console.log('[DEBUG] Tailwind version:', require('tailwindcss/package.json').version);

// TODO:DEBUG - Check if config file exists (should NOT exist in v4)
const fs = require('fs');
const hasConfigFile = fs.existsSync('tailwind.config.js');
console.log('[DEBUG] Has tailwind.config.js (should be false in v4):', hasConfigFile);

// TODO:DEBUG - Check CSS imports
const cssContent = fs.readFileSync('src/input.css', 'utf8');
console.log('[DEBUG] CSS imports:', cssContent.includes('@import "tailwindcss"'));
```

### Problem: Custom theme not working

**Symptoms:**

- Custom colors have no effect
- Custom spacing doesn't apply
- Theme values are ignored

**Solution:**

```css
/* WRONG - Trying to use JS config */
// tailwind.config.js (DELETE THIS FILE)
module.exports = {
  theme: {
    extend: {
      colors: { brand: '#...' }
    }
  }
}

/* CORRECT - Use @theme in CSS */
@import "tailwindcss";

@theme {
  /* Custom colors using oklch (recommended) */
  --color-brand-50: oklch(0.98 0.01 221.19);
  --color-brand-500: oklch(0.72 0.11 221.19);
  --color-brand-900: oklch(0.45 0.08 221.19);

  /* OR hex values (works but not recommended) */
  --color-accent-500: #3b82f6;

  /* Custom fonts */
  --font-display: "Satoshi", sans-serif;
  --font-body: system-ui, sans-serif;

  /* Custom spacing */
  --spacing-unit: 4px;
  --spacing-section: 2rem;

  /* Custom breakpoints */
  --breakpoint-3xl: 120rem;

  /* Custom container queries */
  --container-8xl: 96rem;
}

/* Disable all defaults */
@theme {
  --*: initial; /* Reset ALL defaults */
  --color-custom: #...;
}
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check CSS variables
const root = document.documentElement;
const computedStyles = window.getComputedStyle(root);
console.log('[DEBUG] CSS vars:', {
  brandColor: computedStyles.getPropertyValue('--color-brand-500'),
  fontDisplay: computedStyles.getPropertyValue('--font-display'),
  spacingUnit: computedStyles.getPropertyValue('--spacing-unit')
});
```

## 2. Variant Stacking Issues (CRITICAL)

### Problem: Variants not applying correctly

**Symptoms:**

- `hover:bg-blue-500 md:bg-blue-600` doesn't work as expected
- Dark mode variants inconsistent
- Responsive variants wrong order

#### Root Cause: v4 changed from right-to-left to left-to-right

```html
<!-- v3: Applied RIGHT-TO-LEFT -->
<div class="dark:md:bg-red-500">
<!-- Means: dark AND md apply bg-red-500 -->

<!-- v4: Applied LEFT-TO-RIGHT (BREAKING CHANGE) -->
<div class="dark:md:bg-red-500">
<!-- Means: When dark, AND when md within dark, apply bg-red-500 -->
```

**Migration Strategy:**

```html
<!-- WRONG in v4: Mixed order -->
<div class="hover:bg-blue-500 md:bg-blue-600">
<!-- Confusing: hover and md at same level -->

<!-- CORRECT: Responsive BEFORE pseudo-class -->
<div class="md:hover:bg-blue-600">
<!-- Clear: At md breakpoint, apply hover effect -->

<!-- Stacking multiple variants in v4 -->
<div class="dark:md:hover:bg-fuchsia-600">
<!-- Logic: Apply dark → IN dark apply md → IN dark+md apply hover -->
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check applied styles
const element = document.querySelector('.my-element');
const computedStyle = window.getComputedStyle(element);

// Check background at different states
console.log('[DEBUG] Current bg:', computedStyle.backgroundColor);

// Simulate dark mode
document.documentElement.classList.add('dark');
console.log('[DEBUG] Dark mode bg:', window.getComputedStyle(element).backgroundColor);

// Simulate md breakpoint
element.classList.add('md');
console.log('[DEBUG] Dark+md bg:', window.getComputedStyle(element).backgroundColor);
```

### Problem: Child variants broken

**Symptoms:**

- `first:*:pt-0` doesn't work
- `prose-headings:*:underline` is broken

#### Solution: Reverse the order

```html
<!-- v3: Right-to-left -->
<ul class="py-4 first:*:pt-0 last:*:pb-0">
<!-- Apply first/last to all children with pt-0/pb-0 -->

<!-- v4: Left-to-right (MUST REVERSE) -->
<ul class="*:first:pt-0 *:last:pb-0 py-4">
<!-- Apply to all children, then first/last -->
```

## 3. CSS Cascade Layers Issues

### Problem: Custom CSS overridden by utilities

**Symptoms:**

- Custom CSS has no effect
- Utilities always win
- `!important` doesn't work

#### Root Cause: Layer precedence

```text
utilities (highest precedence)
  ↓
components
  ↓
base
  ↓
theme (lowest precedence)
```

**Solutions:**

```css
/* Solution 1: Use ! modifier (v4 syntax: at END) */
.my-element {
  background-color: red !important;
}

/* In HTML */
<div class="bg-red-500!">
<!-- ! at END in v4 (not beginning) */

/* Solution 2: Use @utility API instead of @layer */
@utility tab-4 {
  tab-size: 4;
}
/* Automatically sorted by property count */

/* Solution 3: Move to utilities layer */
@layer utilities {
  .my-custom-class {
    /* Will override default utilities */
  }
}
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check layer precedence
const element = document.querySelector('.my-element');
const computedStyle = window.getComputedStyle(element);

console.log('[DEBUG] Computed background:', computedStyle.backgroundColor);
console.log('[DEBUG] All applied rules:', {
  length: computedStyle.length,
  cssText: computedStyle.cssText
});

// Check if utility is overriding
const testUtility = document.createElement('div');
testUtility.className = 'bg-red-500';
document.body.appendChild(testUtility);
const utilityStyle = window.getComputedStyle(testUtility);
console.log('[DEBUG] Utility bg:', utilityStyle.backgroundColor);
document.body.removeChild(testUtility);
```

## 4. Container Query Issues

### Problem: Container queries not working

**Symptoms:**

- `@md:` variants have no effect
- Container syntax errors
- Nested containers conflicting

**Solution:**

```html
<!-- WRONG: v3 syntax -->
<div class="container">
  <div class="@sm:grid-cols-3"></div>
</div>

<!-- CORRECT: v4 syntax with @container -->
<div class="@container">
  <div class="grid grid-cols-1 @md:grid-cols-3 @lg:grid-cols-4">
    <!-- @md = min-width container query -->
  </div>
</div>

<!-- Named containers for nested queries -->
<div class="@container/sidebar">
  <div class="flex flex-col @sm/sidebar:flex-row">
    <!-- Targets parent container named "sidebar" -->
  </div>
</div>

<!-- Arbitrary container sizes -->
<div class="@container">
  <div class="@min-[475px]:flex-row @max-[960px]:flex-col">
    <!-- Ad-hoc container widths -->
  </div>
</div>
```

**Custom Container Sizes:**

```css
@theme {
  --container-8xl: 96rem;
}

/* Now available: @8xl: variant */
<div class="@container">
  <div class="@8xl:grid-cols-6">
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check container size
const container = document.querySelector('.\\@container');
if (container) {
  const rect = container.getBoundingClientRect();
  console.log('[DEBUG] Container width:', rect.width);

  // Check which container query variants should apply
  console.log('[DEBUG] Should apply @md:', rect.width >= 768);
  console.log('[DEBUG] Should apply @lg:', rect.width >= 1024);
}
```

## 5. Preflight Breaking Changes

### Problem: Styles look different after upgrade

**Symptoms:**

- Buttons don't have pointer cursor
- Dialog not centered
- Placeholder color different
- Border color different

**Solutions:**

```css
/* Button cursor changed to default in v4 */
/* To restore v3 behavior: */
@layer base {
  button:not(:disabled),
  [role="button"]:not(:disabled) {
    cursor: pointer;
  }
}

/* Dialog margin removed in v4 */
/* To restore centering: */
@layer base {
  dialog {
    margin: auto;
  }
}

/* Placeholder color changed to 50% opacity in v4 */
/* To restore v3 gray-400: */
@layer base {
  input::placeholder,
  textarea::placeholder {
    color: var(--color-gray-400);
  }
}

/* Border color now currentColor in v4 */
/* MUST specify color explicitly: */
<div class="border border-gray-200">
<!-- NOT just: <div class="border"> -->
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check Preflight styles
const button = document.querySelector('button');
const dialog = document.querySelector('dialog');
const input = document.querySelector('input');

if (button) {
  console.log('[DEBUG] Button cursor:', window.getComputedStyle(button).cursor);
}

if (dialog) {
  console.log('[DEBUG] Dialog margin:', window.getComputedStyle(dialog).margin);
}

if (input) {
  console.log('[DEBUG] Placeholder color:', window.getComputedStyle(input).getPropertyValue('color'));
}
```

## 6. Important Modifier Issues

### Problem: !important doesn't work

**Symptoms:**

- `!flex` has no effect
- Important utilities are ignored

**Solution:**

```html
<!-- v3: ! at beginning -->
<div class="!flex !bg-red-500">

<!-- v4: ! at END (recommended) -->
<div class="flex! bg-red-500! hover:bg-red-600/50!">

<!-- Old syntax still supported but deprecated -->
<!-- But ! at END is the new standard -->
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check !important application
const element = document.querySelector('.my-element');
const styles = window.getComputedStyle(element);

console.log('[DEBUG] Is flex applied:', styles.display === 'flex');
console.log('[DEBUG] Is background applied:', styles.backgroundColor !== 'rgba(0, 0, 0, 0)');
```

## 7. Custom Utility Issues

### Problem: Custom utilities not working with variants

**Symptoms:**

- `hover:tab-4` doesn't work
- Custom utility gets overridden
- Variants don't apply

**Solution:**

```css
/* v3: @layer utilities or @layer components */
@layer utilities {
  .tab-4 {
    tab-size: 4;
  }
}
/* Problem: Doesn't work well with variants */

/* v4: @utility API */
@utility tab-4 {
  tab-size: 4;
}
/* Benefits: */
/* - Automatically sorted by property count */
/* - Works with ALL variants: hover:tab-4, md:tab-4, etc. */
/* - Proper cascade layer integration */

/* Component utilities (fewer properties) can be overridden */
@utility btn {
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  background-color: ButtonFace;
}
/* Can be overridden: bg-red-500 btn works as expected */
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Test custom utility with variants
const element = document.querySelector('.\\@hover:tab-4');
if (element) {
  console.log('[DEBUG] Tab size:', window.getComputedStyle(element).tabSize);

  // Trigger hover
  element.dispatchEvent(new MouseEvent('mouseover', { bubbles: true }));
  console.log('[DEBUG] Tab size on hover:', window.getComputedStyle(element).tabSize);
}
```

## 8. Modal & Backdrop Issues

### Problem: Modal backdrop doesn't cover entire screen

**Symptoms:**

- Backdrop displays but doesn't cover entire viewport
- Modal content visible but background transparent
- Click-through backdrop

#### Root Cause: Separate stacking contexts

```html
<!-- WRONG: Separate fixed elements create separate stacking contexts -->
<div class="fixed inset-0 bg-black/50 z-40">
  <!-- Backdrop -->
</div>
<div class="fixed inset-0 z-50 flex items-center justify-center">
  <div class="bg-white p-6 rounded-lg">
    <!-- Modal content -->
  </div>
</div>
```

**Solutions:**

```html
<!-- SOLUTION 1: Same stacking context (RECOMMENDED) -->
<div class="fixed inset-0 z-50">
  <!-- Backdrop - same context -->
  <div class="absolute inset-0 bg-black/50 backdrop-blur-sm"></div>

  <!-- Modal content - relative z to be above backdrop -->
  <div class="relative z-10 flex items-center justify-center min-h-screen">
    <div class="bg-white p-6 rounded-lg shadow-xl max-w-md mx-4">
      <h2 class="text-xl font-semibold mb-4">Modal Title</h2>
      <p>Modal content here...</p>
    </div>
  </div>
</div>

<!-- SOLUTION 2: React Portal (BEST for React) -->
// Modal.jsx
import { createPortal } from 'react-dom';

const Modal = ({ children }) => {
  return createPortal(
    <div className="fixed inset-0 z-50">
      <div className="absolute inset-0 bg-black/50 backdrop-blur-sm"></div>
      <div className="relative z-10 flex items-center justify-center min-h-screen">
        {children}
      </div>
    </div>,
    document.getElementById('modal-portal')
  );
};

export { Modal };

// In HTML
<div id="modal-portal"></div>

// Usage
<Modal>
  <div className="bg-white p-6 rounded-lg">
    Modal content
  </div>
</Modal>
```

### Problem: Modal backdrop opacity doesn't work

**Symptoms:**

- `bg-black/50` has no effect
- Backdrop too dark or too light
- Background opacity inconsistent

**Solution:**

```html
<!-- Background opacity syntax -->
<div class="bg-black/50"> <!-- 50% opacity -->
<div class="bg-black/30"> <!-- 30% opacity -->
<div class="bg-black/[0.5]"> <!-- Arbitrary value -->

<!-- Backdrop blur with opacity -->
<div class="absolute inset-0 bg-black/50 backdrop-blur-sm">
<div class="absolute inset-0 bg-black/40 backdrop-blur-md">
<div class="absolute inset-0 bg-black/60 backdrop-blur-lg">

<!-- Custom opacity with arbitrary values -->
<div class="bg-black/[0.35] backdrop-blur-[4px]">

<!-- Responsive backdrop -->
<div class="bg-black/50 backdrop-blur-none md:backdrop-blur-sm">
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check backdrop opacity
const backdrop = document.querySelector('.modal-backdrop');
if (backdrop) {
  const computedStyle = window.getComputedStyle(backdrop);
  console.log('[DEBUG] Backdrop background:', computedStyle.backgroundColor);
  console.log('[DEBUG] Backdrop blur:', computedStyle.backdropFilter);

  // Check opacity
  const rgba = computedStyle.backgroundColor.match(/[\d.]+/g);
  if (rgba) {
    const opacity = parseFloat(rgba[3]);
    console.log('[DEBUG] Opacity value:', opacity);
  }
}
```

### Problem: Z-index layering doesn't work

**Symptoms:**

- Modal content hidden by backdrop
- Other elements show through modal
- Z-index values have no effect

#### Root Cause: Stacking context isolation

```html
<!-- WRONG: Different stacking contexts -->
<div class="relative z-50">
  <div class="fixed inset-0 bg-black/50 z-40"></div>
</div>
<div class="relative z-50">
  <div class="fixed inset-0 z-50 modal-content"></div>
</div>
<!-- z-40 and z-50 in DIFFERENT stacking contexts -->

<!-- CORRECT: Use isolate to control stacking -->
<div class="isolate">
  <!-- Same stacking context -->
  <div class="fixed inset-0 bg-black/50 z-40"></div>
  <div class="fixed inset-0 z-50 modal-content"></div>
</div>
```

**Understanding Stacking Context:**

```css
/* Elements create new stacking contexts when: */
- position: fixed/relative + z-index
- transform/transform-style
- filter/backdrop-filter
- will-change
- opacity < 1
- isolation: isolate

/* Use isolate utility to control stacking context */
.isolate {
  isolation: isolate;
}
```

**Best Practice Modal Pattern:**

```html
<div class="fixed inset-0 z-50 isolate">
  <!-- Backdrop -->
  <div class="absolute inset-0 bg-black/50 backdrop-blur-sm"></div>

  <!-- Modal container -->
  <div class="relative z-10 flex items-center justify-center min-h-screen p-4">
    <!-- Modal content -->
    <div class="bg-white rounded-lg shadow-xl p-6 max-w-md w-full">
      <h2 class="text-xl font-semibold mb-4">Modal Title</h2>
      <p class="text-gray-600 mb-6">Modal content here...</p>
      <div class="flex justify-end gap-2">
        <button class="px-4 py-2 bg-gray-200 rounded">Cancel</button>
        <button class="px-4 py-2 bg-blue-500 text-white rounded">Confirm</button>
      </div>
    </div>
  </div>
</div>
```

### Problem: Dialog element backdrop

**Symptoms:**

- `<dialog>` backdrop style doesn't apply
- Default backdrop not customizable

**Solution:**

```html
<!-- Tailwind v4 has backdrop variant for dialog -->
<dialog class="backdrop:bg-gray-900/50 backdrop:backdrop-blur-sm">
  <form method="dialog">
    <p>Dialog content</p>
    <button value="cancel">Cancel</button>
    <button value="default">Confirm</button>
  </form>
</dialog>

<!-- Custom backdrop with JavaScript -->
const dialog = document.querySelector('dialog');
dialog.showModal();
// ::backdrop pseudo-element is styled automatically
```

**Debug Steps:**

```javascript
// TODO:DEBUG - Check dialog backdrop
const dialog = document.querySelector('dialog');
if (dialog) {
  console.log('[DEBUG] Dialog open:', dialog.open);

  // Show modal to create backdrop
  dialog.showModal();

  // Check backdrop styling
  const backdropStyle = window.getComputedStyle(dialog, '::backdrop');
  console.log('[DEBUG] Backdrop background:', backdropStyle.backgroundColor);
  console.log('[DEBUG] Backdrop backdrop-filter:', backdropStyle.backdropFilter);
}
```

## 9. Common Debug Patterns

### Check Tailwind Version

```javascript
// TODO:DEBUG - Check Tailwind version at runtime
try {
  const tailwindVersion = require('tailwindcss/package.json').version;
  console.log('[DEBUG] Tailwind version:', tailwindVersion);
  console.log('[DEBUG] Is v4:', tailwindVersion.startsWith('4.'));
} catch (e) {
  console.log('[DEBUG] Could not read Tailwind version');
}
```

### Check z-index Stacking

```javascript
// TODO:DEBUG - Verify z-index stacking
const backdrop = document.querySelector('.modal-backdrop');
const modal = document.querySelector('.modal-content');

if (backdrop && modal) {
  const backdropStyle = window.getComputedStyle(backdrop);
  const modalStyle = window.getComputedStyle(modal);

  console.log('[DEBUG] Backdrop z-index:', backdropStyle.zIndex);
  console.log('[DEBUG] Modal z-index:', modalStyle.zIndex);

  // Check stacking context
  const backdropParent = backdrop.parentElement;
  const modalParent = modal.parentElement;

  console.log('[DEBUG] Same parent:', backdropParent === modalParent);
  console.log('[DEBUG] Backdrop has stacking context:',
    backdropParent.style.transform !== 'none' ||
    backdropParent.style.position !== 'static'
  );
}
```

### Check Applied Classes

```javascript
// TODO:DEBUG - Check applied classes
const element = document.querySelector('.my-component');
if (element) {
  console.log('[DEBUG] Applied classes:', element.className);
  console.log('[DEBUG] Class list:', Array.from(element.classList));

  // Check specific Tailwind classes
  console.log('[DEBUG] Has bg-red-500:', element.classList.contains('bg-red-500'));
  console.log('[DEBUG] Has md:hover:bg-blue-600:', element.classList.contains('md\\:hover\\:bg-blue-600'));
}
```

### Check CSS Variables

```javascript
// TODO:DEBUG - Check theme CSS variables
const root = document.documentElement;
const computedStyles = window.getComputedStyle(root);

const themeVars = [
  '--color-brand-500',
  '--font-display',
  '--spacing-unit',
  '--breakpoint-md',
  '--container-lg'
];

themeVars.forEach(varName => {
  const value = computedStyles.getPropertyValue(varName);
  console.log(`[DEBUG] ${varName}:`, value || 'NOT SET');
});
```

## 10. Migration Checklist

### Before Upgrade

- [ ] Backup project (git commit)
- [ ] Read full [upgrade guide](https://tailwindcss.com/docs/upgrade-guide)
- [ ] Check current Tailwind version
- [ ] Review all `tailwind.config.js` usage
- [ ] Document all custom plugins

### During Upgrade

- [ ] Run `pnpx @tailwindcss/upgrade`
- [ ] Review all generated changes
- [ ] Update all `@tailwind` directives → `@import "tailwindcss"`
- [ ] Migrate `tailwind.config.js` → `@theme` in CSS
- [ ] Reverse variant stacking order
- [ ] Update important modifier syntax (`!flex` → `flex!`)
- [ ] Migrate `@layer utilities` → `@utility`
- [ ] Update container query syntax
- [ ] Add Preflight fixes if needed

### After Upgrade

- [ ] Test all pages in browser
- [ ] Check responsive breakpoints
- [ ] Test dark mode
- [ ] Verify hover/focus states
- [ ] Check modal/dialog elements
- [ ] Test forms and inputs
- [ ] Verify custom colors
- [ ] Check container queries
- [ ] Test all custom utilities
- [ ] Run visual regression tests

### Common Issues to Fix

1. **Variant stacking**: `hover:bg-blue-500 md:bg-blue-600` → `md:hover:bg-blue-600`
2. **Child variants**: `first:*:pt-0` → `*:first:pt-0`
3. **Important**: `!flex` → `flex!`
4. **Config file**: DELETE `tailwind.config.js`
5. **PostCSS**: Add `@tailwindcss/postcss` package
