# Frontend Debug Skill

Skill debug frontend có hệ thống cho Claude Code, sử dụng Chrome DevTools MCP và Context7.

## Cấu trúc

```
frontend-debug/
├── SKILL.md                   # Main skill file (REQUIRED)
├── README.md                  # File này
├── EXAMPLES.md                # Real-world debugging examples
└── reference/                 # Specialized guides
    └── TAILWIND-DEBUG.md      # Tailwind CSS v4 issues
```

## Quick Start

### Yêu cầu

- **Chrome DevTools MCP server** phải được cài đặt
- **Context7** cho framework/library documentation (LUÔN dùng Context7)
- **wget** và **html2text** cho external docs (optional)

### Usage

Skill tự động kích hoạt khi bạn dùng các keywords:

- "debug", "debugging", "troubleshoot"
- "fix bug", "broken", "not working"
- "console error", "network error"
- "lỗi", "sửa lỗi"

### Workflow

1. **Information Gathering** - Thu thập bug details
2. **Inspect Page** - DOM snapshot, console, network
3. **Root Cause Analysis** - Tìm nguyên nhân gốc rễ
4. **Add Debug Logs** - TODO:DEBUG prefix
5. **Verify Fix** - Confirm và clean up

## Files

### SKILL.md

Main skill file với:

- Trigger keywords
- Quick reference commands
- 5-phase debug workflow
- Chrome DevTools MCP reference
- Context7 integration
- External docs access (wget+html2text, webReader MCP)
- Best practices

### EXAMPLES.md

7 real-world debugging scenarios:

1. Console Error Debugging
2. Network API Failure
3. Element Not Visible
4. React useEffect Bug
5. Tailwind v4 Variant Stacking
6. Modal Backdrop Issue
7. Race Condition

### Framework/Library Debugging

**LUÔN dùng Context7** - Documentation mới nhất, version-specific.

```javascript
// React
mcp__context7__query-docs({
  "libraryId": "/facebook/react",
  "query": "useEffect debugging stale closure"
})

// Next.js
mcp__context7__query-docs({
  "libraryId": "/vercel/next.js",
  "query": "hydration error server client component"
})

// Vue
mcp__context7__query-docs({
  "libraryId": "/vuejs/core",
  "query": "reactivity debugging"
})

// Tailwind CSS
mcp__context7__query-docs({
  "libraryId": "/tailwindlabs/tailwindcss",
  "query": "v4 variant stacking configuration"
})
```

### reference/TAILWIND-DEBUG.md

Tailwind CSS v4 breaking changes:

- Configuration (@theme vs tailwind.config.js)
- Variant stacking (left-to-right)
- CSS cascade layers
- Container queries
- Preflight changes
- Important modifier syntax
- Modal & backdrop issues

## Documentation Access

### Context7 (recommended)

Context7 mang documentation mới nhất, version-specific trực tiếp vào prompt.

**Best practices:**

```bash
# Dùng library ID trực tiếp khi đã biết
mcp__context7__query-docs({
  "libraryId": "/facebook/react",
  "query": "useEffect debugging"
})

# Specify version trong query
mcp__context7__query-docs({
  "libraryId": "/vercel/next.js",
  "query": "Next.js 14 middleware JWT"
})

# Resolve khi không biết library ID
mcp__context7__resolve-library-id({
  "libraryName": "astro"
})
```

### External docs (wget + html2text)

```bash
wget -O - https://docs.astro.build/llms-full.txt | html2text -
wget -O - https://rsbuild.rs/llms-full.txt | html2text -
```

### webReader MCP

```bash
mcp__web_reader__webReader({
  "url": "https://docs.astro.build/llms-full.txt",
  "return_format": "markdown"
})
```

## Best Practices

1. **Luôn tìm root cause trước khi fix**
2. **LUÔN dùng Context7** cho framework/library issues (không dùng docs lỗi thời)
3. **Dùng TODO:DEBUG prefix** cho tất cả debug logs
4. **Clean up debug logs** sau khi fix
5. **Document learnings** cho future reference

## Progressive Disclosure

Skill này sử dụng progressive disclosure pattern:

- **SKILL.md**: Essential info + Context7 workflows
- **EXAMPLES.md**: Real-world scenarios
- **reference/TAILWIND-DEBUG.md**: Tailwind CSS v4 specialized guide
- **Context7**: Framework/library documentation (luôn dùng)
