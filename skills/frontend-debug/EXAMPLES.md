# Frontend Debug Examples

5 real-world examples. For detailed step-by-step, see [WORKFLOWS.md](WORKFLOWS.md).

## Example 1: Button Click Not Working

**Issue:** Clicking Submit button does nothing.

**Root Cause:** Selector mismatch - HTML uses `id="submitBtn"` but JS looks for `#submit-btn`.

**Flow:**

```bash
# Console error when clicking
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })
# → "TypeError: Cannot read properties of null (reading 'click')"

# Test selector
mcp__chrome-devtools__evaluate_script({
  "function": `() => {
    const btn = document.querySelector('#submitBtn');
    return btn ? 'Found' : 'Not found';
  }`
})
# → "Found" (camelCase correct)

# Fix selector
Edit({ "file_path": "src/form.js", ... })
# "#submit-btn" → "#submitBtn"

# Verify
mcp__chrome-devtools__navigate_page({ "type": "reload" })
mcp__chrome-devtools__click({ "uid": "button-123" })
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })
# → [] (no errors)
```

---

## Example 2: Form Submit Fails Silently

**Issue:** Clicking Submit but data doesn't save, no console errors.

**Root Cause:** Validation function has buggy regex that removes domain extension.

**Flow:**

```bash
# Console clean → check network
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })
# → [reqid=45] POST /api/signup 400 Bad Request

mcp__chrome-devtools__get_network_request({ "reqid": 45 })
# Response: { "error": "Validation failed", "details": { "email": "Invalid email format" } }
# Request: { "email": "john@example" } ← Missing .com!

# Validation code
Read({ "file_path": "src/utils/validation.js" })
# const validateEmail = (email) => email.replace(/\.\w+$/, ''); ← WRONG!

# Fix regex
Edit({ "file_path": "src/utils/validation.js", ... })
# email.replace(/\.\w+$/, '') → /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)

# Verify
mcp__chrome-devtools__navigate_page({ "type": "reload" })
mcp__chrome-devtools__get_network_request({ "reqid": 48 })
# → 200 OK
```

---

## Example 3: Modal Shows But Blank

**Issue:** Modal opens but content is empty, no errors.

**Root Cause:** API endpoint has typo → 404 → no data → empty modal.

**Flow:**

```bash
# Check network
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["fetch"] })
# → No modal data request

# Check DOM
mcp__chrome-devtools__take_snapshot({})
# → uid=modal-body EMPTY

# Find API call
Grep({ "pattern": "fetchModalData|loadModalContent", "type": "js" })
Read({ "file_path": "src/api.js" })
# fetch('/api/modal-deta') ← Typo!

# Fix endpoint
Edit({ "file_path": "src/api.js", ... })
# '/api/modal-deta' → '/api/modal-data'

# Verify
mcp__chrome-devtools__navigate_page({ "type": "reload" })
mcp__chrome-devtools__wait_for({ "text": "Modal loaded successfully" })
# → Success
```

---

## Example 4: React Infinite Loop

**Issue:** Page becomes slow, CPU 100%.

**Root Cause:** Object literal in dependency array creates new reference every render → infinite useEffect.

**Flow:**

```bash
# Console warning
mcp__chrome-devtools__list_console_messages({ "types": ["warning"] })
# → "Warning: Maximum update depth exceeded"

# Performance trace
mcp__chrome-devtools__performance_start_trace({ "reload": false, "autoStop": true })
mcp__chrome-devtools__performance_stop_trace({})
# → Component rendering 100+ times/second

# Query React docs
mcp__context7__query-docs({
  "libraryId": "/websites/react_dev",
  "query": "infinite render loop useEffect dependency debugging"
})

# Find component
Grep({ "pattern": "UserProfile|function UserProfile", "type": "jsx" })
Read({ "file_path": "src/components/UserProfile.jsx" })
# useEffect(() => { ... }, [userId, { fetch: fetchUserData }]) ← Object!

# Fix dependency array
Edit({ "file_path": "src/components/UserProfile.jsx", ... })
# [userId, { fetch: fetchUserData }] → [userId]

# Verify
mcp__chrome-devtools__navigate_page({ "type": "reload" })
mcp__chrome-devtools__performance_start_trace({ "reload": false, "autoStop": true })
# → UserProfile rendered ONCE
```

---

## Example 5: Tailwind v4 Variant Stacking

**Issue:** Hover button doesn't change color on mobile.

**Root Cause:** Tailwind v4 changed variant stacking order (right-to-left → left-to-right).

**Flow:**

```bash
# Check computed style
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    const styles = window.getComputedStyle(el);
    return {
      backgroundColor: styles.backgroundColor,
      allClasses: el.className
    };
  }`,
  "args": [{ "uid": "hover-button" }]
})
# → backgroundColor: "rgb(229, 231, 235)" (gray, not blue)
# → allClasses: "px-4 py-2 hover:bg-blue-500 md:bg-blue-600"

# Check Tailwind version
Read({ "file_path": "package.json" })
# → "tailwindcss": "^4.0.0"

# Query Tailwind docs
mcp__context7__query-docs({
  "libraryId": "/tailwindlabs/tailwindcss",
  "query": "v4 variant stacking order responsive pseudo-class hover"
})
# → In v4, variants stack LEFT-TO-RIGHT

# Fix variant order
Edit({ "file_path": "src/components/Button.jsx", ... })
# hover:bg-blue-500 md:bg-blue-600 → md:hover:bg-blue-600

# Verify
mcp__chrome-devtools__resize_page({ "width": 375, "height": 667 })
mcp__chrome-devtools__hover({ "uid": "hover-button" })
# → Blue on hover ✓
```

---

## Debug Patterns Quick Reference

### Pattern: "Click/Action Not Working"

```bash
# 1. Console errors
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })

# 2. Element exists and clickable
mcp__chrome-devtools__evaluate_script({
  "function": `() => {
    const btn = document.querySelector('button');
    const styles = window.getComputedStyle(btn);
    return {
      exists: !!btn,
      visible: btn?.offsetParent !== null,
      disabled: btn?.disabled,
      pointerEvents: styles.pointerEvents
    };
  }`
})

# 3. Event listeners
mcp__chrome-devtools__evaluate_script({
  "function": `() => {
    const btn = document.querySelector('button');
    const listeners = getEventListeners(btn);
    return listeners?.click?.length || 0;
  }`
})
```

### Pattern: "Data Not Loading"

```bash
# 1. Console
mcp__chrome-devtools__list_console_messages({})

# 2. Network
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })
mcp__chrome-devtools__get_network_request({ "reqid": <id> })

# 3. Code logic
Grep({ "pattern": "fetch.*data|loadData", "type": "js" })
```

### Pattern: "Style Not Applying"

```bash
# 1. Computed styles
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    const styles = window.getComputedStyle(el);
    return {
      display: styles.display,
      color: styles.color,
      backgroundColor: styles.backgroundColor
    };
  }`,
  "args": [{ "uid": "<element-uid>" }]
})

# 2. Class list
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => el.className`,
  "args": [{ "uid": "<element-uid>" }]
})

# 3. Z-index stacking
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    let current = el;
    let depth = 0;
    while(current && depth < 10) {
      const styles = window.getComputedStyle(current);
      if(parseInt(styles.zIndex) > 0) {
        return { foundAt: depth, zIndex: styles.zIndex };
      }
      current = current.parentElement;
      depth++;
    }
    return { foundAt: -1 };
  }`,
  "args": [{ "uid": "<element-uid>" }]
})
```

### Pattern: "React Weird Behavior"

```bash
# 1. Console warnings
mcp__chrome-devtools__list_console_messages({ "types": ["warning", "error"] })

# 2. Query docs
mcp__context7__query-docs({
  "libraryId": "/websites/react_dev",
  "query": "<specific issue>"
})

# 3. Performance trace
mcp__chrome-devtools__performance_start_trace({ "reload": false, "autoStop": true })
# ... interact ...
mcp__chrome-devtools__performance_stop_trace({})
```

---

## Key Takeaways

1. **Console FIRST** - Errors are most obvious
2. **Network second** - 4xx/5xx reveals issues
3. **DOM third** - Inspect element properties
4. **Hypothesis-driven** - "I think X, let's test"
5. **Use Context7** - Don't guess, query docs
6. **Verify fixes** - Reload, test, confirm
