# Frontend Debug Workflows

Copy checklist and track progress through each workflow.

## Table of Contents

- [Console errors](#console-errors) - JavaScript errors, warnings, React warnings
- [Network issues](#network-issues) - Failed requests, wrong data, CORS
- [DOM issues](#dom-issues) - Elements not found, styling wrong, layout problems
- [Framework issues](#framework-issues) - React/Next.js/Astro weird behavior
- [Console.log debugging](#consolelog-debugging-patterns) - Runtime value inspection with styled logs
- [Verification loop](#verification-loop) - Mandatory after every fix

---

## Console errors

When you have console errors, this is the clearest clue.

```text
Console Debug Progress:
- [ ] Step 1: Reproduce and capture error
- [ ] Step 2: Get error details (stack trace)
- [ ] Step 3: Locate error source
- [ ] Step 4: Form hypothesis
- [ ] Step 5: Test hypothesis
- [ ] Step 6: Fix and verify
```

### Step 1: Reproduce and capture error

```bash
# Navigate to page
mcp__chrome-devtools__navigate_page({ "type": "url", "url": "<url>" })

# Trigger action
mcp__chrome-devtools__take_snapshot({})
mcp__chrome-devtools__click({ "uid": "<element>" })

# Check console
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })
```

### Step 2: Get error details

```bash
# Get detailed error message
mcp__chrome-devtools__get_console_message({ "msgid": <id> })

# Find stack trace → file:line
```

### Step 3: Locate error source

```bash
# Read file with error
Read({ "file_path": "src/components/Component.jsx" })
# Find line number from stack trace
```

### Step 4: Form hypothesis

Example hypotheses:

- "Element doesn't exist when selector runs"
- "Undefined property access"
- "Async function not awaited"

### Step 5: Test hypothesis

```bash
# Test element existence
mcp__chrome-devtools__evaluate_script({
  "function": `() => {
    const el = document.querySelector('.my-element');
    return { exists: !!el, visible: el?.offsetParent !== null };
  }`
})
```

### Step 6: Fix and verify

```bash
# Apply fix
Edit({ "file_path": "...", "old_string": "...", "new_string": "..." })

# Reload
mcp__chrome-devtools__navigate_page({ "type": "reload" })

# Verify error gone
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })
# Expected: [] (empty)
```

---

## Network issues

Request failed or response incorrect.

```text
Network Debug Progress:
- [ ] Step 1: Reproduce and list requests
- [ ] Step 2: Find failed request
- [ ] Step 3: Inspect request/response
- [ ] Step 4: Identify root cause
- [ ] Step 5: Fix and verify
```

### Step 1: Reproduce and list requests

```bash
mcp__chrome-devtools__navigate_page({ "type": "url", "url": "<url>" })

# Trigger action
mcp__chrome-devtools__click({ "uid": "<submit-button>" })

# List XHR/Fetch requests
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })
```

### Step 2: Find failed request

```bash
# Find request with 4xx, 5xx status
mcp__chrome-devtools__get_network_request({ "reqid": <id> })
```

### Step 3: Inspect request/response

```bash
# Check:
# - Request payload (body, headers)
# - Response status, body
# - Response headers (CORS issues)
```

### Step 4: Identify root cause

Common causes:

- **4xx**: Client error (wrong payload, missing field, validation failed)
- **5xx**: Server error (contact backend)
- **CORS**: Missing headers, wrong origin
- **Timeout**: Server slow, request too large

### Step 5: Fix and verify

```bash
# Apply fix (client side)
Edit({ "file_path": "...", ... })

# Reload and test
mcp__chrome-devtools__navigate_page({ "type": "reload" })
mcp__chrome-devtools__click({ "uid": "<submit-button>" })

# Verify success
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })
# Expected: 200 OK, 201 Created
```

---

## DOM issues

Element not found, styling wrong, layout broken.

```text
DOM Debug Progress:
- [ ] Step 1: Reproduce and take snapshot
- [ ] Step 2: Inspect element properties
- [ ] Step 3: Check computed styles
- [ ] Step 4: Check parent/child relationships
- [ ] Step 5: Fix and verify
```

### Step 1: Reproduce and take snapshot

```bash
mcp__chrome-devtools__navigate_page({ "type": "url", "url": "<url>" })
mcp__chrome-devtools__take_snapshot({})
```

### Step 2: Inspect element properties

```bash
# Check element exists and visible
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    const styles = window.getComputedStyle(el);
    return {
      exists: true,
      visible: el.offsetParent !== null,
      display: styles.display,
      visibility: styles.visibility,
      opacity: styles.opacity,
      zIndex: styles.zIndex
    };
  }`,
  "args": [{ "uid": "<element-uid>" }]
})
```

### Step 3: Check computed styles

```bash
# See actual applied styles
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    const styles = window.getComputedStyle(el);
    return {
      backgroundColor: styles.backgroundColor,
      color: styles.color,
      width: styles.width,
      height: styles.height,
      margin: styles.margin,
      padding: styles.padding
    };
  }`,
  "args": [{ "uid": "<element-uid>" }]
})
```

### Step 4: Check parent/child relationships

```bash
# Find element being covered by other element
mcp__chrome-devtools__evaluate_script({
  "function": `(el) => {
    let current = el;
    let depth = 0;
    while(current && depth < 10) {
      const styles = window.getComputedStyle(current);
      if(styles.zIndex !== 'auto' && parseInt(styles.zIndex) > 0) {
        return { foundAt: depth, element: current.tagName, zIndex: styles.zIndex };
      }
      current = current.parentElement;
      depth++;
    }
    return { foundAt: -1 };
  }`,
  "args": [{ "uid": "<element-uid>" }]
})
```

### Step 5: Fix and verify (DOM)

```bash
# Apply fix (CSS, HTML structure, z-index)
Edit({ "file_path": "...", ... })

# Reload
mcp__chrome-devtools__navigate_page({ "type": "reload" })

# Verify element visible and styled correctly
mcp__chrome-devtools__take_snapshot({})
mcp__chrome-devtools__evaluate_script({ ... })
```

---

## Framework issues

React/Next.js/Astro weird behavior, infinite loops, stale closures.

```text
Framework Debug Progress:
- [ ] Step 1: Check console for framework warnings
- [ ] Step 2: Query framework docs (Context7)
- [ ] Step 3: Use performance trace
- [ ] Step 4: Locate problematic component
- [ ] Step 5: Fix and verify
```

### Step 1: Check console

```bash
mcp__chrome-devtools__list_console_messages({ "types": ["warning", "error"] })

# Common warnings:
# - "Maximum update depth exceeded" (infinite loop)
# - "Each child should have a unique key prop"
# - "useEffect has missing dependencies"
```

### Step 2: Query framework docs

```bash
# ALWAYS query docs before guessing
mcp__context7__query-docs({
  "libraryId": "reactjs/react.dev",
  "query": "infinite render loop useEffect dependency debugging"
})
```

### Step 3: Use performance trace

```bash
# Start trace
mcp__chrome-devtools__performance_start_trace({ "reload": false, "autoStop": true })

# Interact with app
# ... wait 2-3 seconds ...

# Stop trace
mcp__chrome-devtools__performance_stop_trace({})

# Check results
# - Component render count
# - Long tasks (>50ms)
# - Layout shifts
```

### Step 4: Locate problematic component

```bash
# Grep component file
Grep({ "pattern": "UserProfile|function UserProfile", "type": "jsx" })
Read({ "file_path": "src/components/UserProfile.jsx" })

# Check for:
# - Object/array in dependency array
# - Missing dependencies
# - Stale closures
# - Unbounded re-renders
```

### Step 5: Fix and verify (Framework)

```bash
# Apply fix (component code)
Edit({ "file_path": "...", ... })

# Reload
mcp__chrome-devtools__navigate_page({ "type": "reload" })

# Verify:
mcp__chrome-devtools__list_console_messages({ "types": ["warning"] })
# Expected: [] (no framework warnings)

mcp__chrome-devtools__performance_start_trace({ "reload": false, "autoStop": true })
mcp__chrome-devtools__performance_stop_trace({})
# Expected: Component render count reasonable (<10 times)
```

---

## Console.log debugging patterns

When needing to inspect runtime values, use console.log with CSS styling for readability.

```text
Console Debug Progress:
- [ ] Step 1: Identify points to log
- [ ] Step 2: Add console.log with styling
- [ ] Step 3: Reproduce and inspect output
- [ ] Step 4: Remove logs after debugging
```

### Basic console types

```javascript
// Regular log
console.log('Regular message');

// Info (blue icon)
console.info('Informational message');

// Warning (yellow icon)
console.warn('Warning message');

// Error (red icon, has stack trace)
console.error('Error message');

// Table format for arrays/objects
console.table([{ name: 'Item 1', value: 100 }, { name: 'Item 2', value: 200 }]);
```

### Console with CSS styling (browser only)

```javascript
// Styled log with %c directive
const style = "color: red; font-size: 20px; font-weight: bold;";
console.log("%c[DEBUG] Hello World", style);

// Multiple styles in same line
console.log(
  "%c[ERROR] %cSomething went wrong",
  "color: red; font-weight: bold;",
  "color: orange; font-weight: normal;"
);

// Background color
console.log("%c[SUCCESS] Operation completed", "background: green; color: white; padding: 4px;");
```

### ANSI colors (Terminal + Node.js)

```javascript
// ANSI escape codes for colors
const textRed = "\x1b[31m";
const textGreen = "\x1b[32m";
const textYellow = "\x1b[33m";
const textBlue = "\x1b[34m";
const bgRed = "\x1b[41m";
const reset = "\x1b[0m";

console.log(textRed + "ERROR" + reset + " Something went wrong");
console.log(textGreen + "SUCCESS" + reset + " Operation completed");
console.log(textYellow + "WARNING" + reset + " Check this value");
console.log(textGreen + bgRed + "ALERT" + reset + " Critical issue");
```

### Conditional logging

```javascript
// Log conditionally
const shouldDebug = true;

if (shouldDebug) {
  console.log('[DEBUG] Current state:', state);
}

// Shorter with && operator
shouldDebug && console.log('[DEBUG] State:', state);

// Assert-style logging
console.assert(condition, '[ASSERT] Condition failed, value:', value);
```

### Grouping logs

```javascript
// Group related logs
console.group('User Login Flow');
console.log('Step 1: Validate credentials');
console.log('Step 2: Check database');
console.log('Step 3: Generate token');
console.groupEnd();

// Collapsed group
console.groupCollapsed('Detailed Debug Info');
console.log('Variable A:', a);
console.log('Variable B:', b);
console.groupEnd();
```

### Timing operations

```javascript
// Measure execution time
console.time('API Request');
// ... API call ...
console.timeEnd('API Request');
// Output: API Request: 245ms

// Multiple timers
console.time('Operation A');
console.time('Operation B');
// ...
console.timeEnd('Operation A');
console.timeEnd('Operation B');
```

### Counting occurrences

```javascript
// Count how many times function is called
function handleClick() {
  console.count('Button clicked');
  // ...
}

// Reset counter
console.countReset('Button clicked');
```

### Tracing execution

```javascript
// Stack trace
console.trace('Execution path');

// Debug function calls
function debugFunctionCall(funcName, args) {
  console.log(`%c[CALL] ${funcName}`, "color: blue;", args);
}
```

### Best practices

```javascript
// 1. Prefix logs for easy filtering
console.log('[AUTH] User logged in');
console.log('[API] Request sent');
console.log('[STATE] Component updated');

// 2. Log meaningful context
console.log('[DEBUG] User ID:', user.id, 'Role:', user.role);

// 3. Log objects destructured for clarity
console.log('[DEBUG] User data:', { id, name, email });

// 4. Use table for arrays
console.table(users.map(u => ({ id: u.id, name: u.name })));

// 5. Remove/skip logs in production
const DEBUG = process.env.NODE_ENV === 'development';
DEBUG && console.log('[DEBUG] Development only log');
```

### Chrome DevTools integration

```bash
# Inject console.log into runtime
mcp__chrome-devtools__evaluate_script({
  "function": `() => {
    // Add debug logging to existing code
    const originalFetch = window.fetch;
    window.fetch = function(...args) {
      console.log('%c[FETCH]', 'color: blue;', args[0]);
      return originalFetch.apply(this, args);
    };
  }`
})

# Check console output
mcp__chrome-devtools__list_console_messages({})
```

### Important: Remove logs after debugging

```bash
# Search for console.log before committing
Grep({ "pattern": "console\\.(log|debug|info)", "type": "js" })

# Remove TODO comments related to debug
Grep({ "pattern": "TODO:DEBUG|FIXME:debug", "type": "js" })
```

---

## Verification loop

**MANDATORY** after every fix. Skip steps → risk regression.

```text
Verification Checklist:
- [ ] Step 1: Reload page
- [ ] Step 2: Reproduce original issue
- [ ] Step 3: Verify issue resolved
- [ ] Step 4: Check for regressions
- [ ] Step 5: Verify console clean
- [ ] Step 6: Test edge cases
```

### Step 1: Reload page

```bash
mcp__chrome-devtools__navigate_page({ "type": "reload" })
```

### Step 2: Reproduce original issue

```bash
# Perform original action
mcp__chrome-devtools__click({ "uid": "<element>" })
```

### Step 3: Verify issue resolved

```bash
# Verify expected result
# - Error gone from console
# - Request returns 200 OK
# - Element visible with correct styles
```

### Step 4: Check for regressions

```bash
# Check console for new errors
mcp__chrome-devtools__list_console_messages({ "types": ["error", "warning"] })
# Expected: [] (empty)

# Check network for failed requests
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })
# Expected: All requests 200-299 status
```

### Step 5: Verify console clean

```bash
# Final console check
mcp__chrome-devtools__list_console_messages({})
# Expected: No errors, no warnings
```

### Step 6: Test edge cases

```bash
# Test:
# - Different screen sizes (resize)
# - Different user inputs
# - Dark mode (if applicable)
# - Different browsers (if possible)

mcp__chrome-devtools__resize_page({ "width": 375, "height": 667 })
mcp__chrome-devtools__resize_page({ "width": 1280, "height": 800 })
```

---

## General Anti-patterns

**DON'T:**

- Guess root cause without inspecting
- Skip verification loop
- Fix multiple issues at once (harder to verify)
- Assume code works after edit (MUST verify)

**DO:**

- Follow evidence (console → network → DOM)
- Form hypothesis → Test → Confirm
- Verify every fix
- Document findings
