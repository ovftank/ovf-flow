---
name: frontend-debug
description: Debug frontend issues using Chrome DevTools MCP. Find root causes of UI bugs, console errors, network failures, DOM problems. Use when user mentions "debug", "fix bug", "troubleshoot", "not working", "error", "console", or has UI/DOM/network issues.
allowed-tools: Read, Grep, Glob, Bash(choco:*, uv:*, rg:*, wget:*, curl:*, html2text:*), Edit, Write, mcp__chrome-devtools__*, mcp__context7__*, mcp__web_reader__webReader, WebSearch, AskUserQuestion
---

# Frontend Debug

Systematically debug frontend issues to find **root cause** before fixing.

## CRITICAL: Communication with User

**MUST use `AskUserQuestion` tool for ALL questions.**

- **NEVER ask questions in plain text** - User cannot respond to text questions
- **ALWAYS use AskUserQuestion** - This is the ONLY way to get user input
- **Present 2-4 options** - Each with clear label and description
- **Allow "Other" option** - For custom input when needed

**Example:**

```javascript
// WRONG - User cannot see or respond to this:
"What is the URL of the page with the bug?"

// CORRECT - Uses AskUserQuestion:
AskUserQuestion({
  "questions": [{
    "question": "What is the URL of the page with the bug?",
    "header": "Bug URL",
    "options": [
      { "label": "http://localhost:3000", "description": "Local development server" },
      { "label": "https://staging.example.com", "description": "Staging environment" },
      { "label": "https://production.example.com", "description": "Production site" }
    ],
    "multiSelect": false
  }]
})
```

**When to use AskUserQuestion:**

- Need URL, page name, or location
- Need reproduction steps
- Need to clarify expected vs actual behavior
- Need browser, device, or environment info
- Need to choose between debugging approaches
- Need confirmation before applying fixes

## MANDATORY PRINCIPLES

- **NO guessing** - Every hypothesis must be traceable
- **NO premature fixing** - Find root cause first
- **Every conclusion must have evidence** - Console, network, DOM, performance trace

## 7-Step Debugging Process

### 1. Clear Reproduction

```text
- When does the bug appear?
- Minimum conditions to reproduce?
- Deterministic or random?
```

**If NOT reproduced → STOP.**

### 2. Decision Tree Mapping

Trace full flow:

- user action
- event handler
- async / sync
- state change
- render
- DOM update
- CSS / layout

**Pinpoint which node is wrong.**

### 3. Five Whys (at least 3 levels)

Ask repeatedly:

- Why is UI wrong?
- Why is state like that?
- Why does that assumption exist?

### 4. Assumption Reversal

List assumptions you implicitly believe:

- Data always exists
- Component mounts once
- Effect runs once
- CSS loads in correct order

**Flip each assumption and test.**

### 5. Constraint Mapping

Identify constraints:

- Browser
- Device
- Network
- Timing (race, debounce, hydration)
- Feature flag
- Permission

**Eliminate each constraint.**

### 6. Failure Analysis

Compare:

- Working case vs failing case
- What differs in data, timing, env, version?

### 7. Reversal / Inversion (when stuck)

- If you intentionally make it fail, how would you do it?
- If you remove framework, leaving only DOM, what happens?

## Expected Output

- **Clear root cause** (1-2 sentences)
- **Fix targets root cause directly**, no workaround
- **State side-effect risks**

## Triage decision tree

**Seeing console errors?** → Follow [Console workflow](WORKFLOWS.md#console-errors)
**Network request failed?** → Follow [Network workflow](WORKFLOWS.md#network-issues)
**UI not rendering correctly?** → Follow [DOM workflow](WORKFLOWS.md#dom-issues)
**React/Next.js/Astro weird behavior?** → Follow [Framework workflow](WORKFLOWS.md#framework-issues)

## Quick reference

```bash
# Console errors
mcp__chrome-devtools__list_console_messages({ "types": ["error"] })

# Network requests
mcp__chrome-devtools__list_network_requests({ "resourceTypes": ["xhr", "fetch"] })

# Take snapshot
mcp__chrome-devtools__take_snapshot({})

# Navigate
mcp__chrome-devtools__navigate_page({ "type": "url", "url": "<url>" })
```

## Runtime debugging

**Need to inspect runtime values?** → Use [console.log debugging patterns](WORKFLOWS.md#consolelog-debugging-patterns)

CSS styling, ANSI colors, grouping, timing, counting for production-grade logs.

## Documentation lookup

**ALWAYS use Context7 for framework issues:**

```javascript
// Direct library ID (preferred)
mcp__context7__query-docs({
  "libraryId": "/websites/react_dev",
  "query": "useEffect stale closure debugging"
})

// Resolve when library ID unknown
mcp__context7__resolve-library-id({ "libraryName": "astro" })
```

Common libraries: `/websites/react_dev`, `/vercel/next.js`, `/withastro/astro`, `/tailwindlabs/tailwindcss.com`

## External docs

```bash
wget -O - https://docs.astro.build/llms-full.txt | html2text -
wget -O - https://rsbuild.rs/llms-full.txt | html2text -
```

## Reference

- [WORKFLOWS.md](WORKFLOWS.md) - Step-by-step workflows for each issue type
- [EXAMPLES.md](EXAMPLES.md) - 5 realistic debugging scenarios
- [TAILWIND-DEBUG.md](reference/TAILWIND-DEBUG.md) - Tailwind v4 specialized guide
