---
title: Audit Vite Plugin Performance
impact: HIGH
impactDescription: Can reduce dev server startup by 50-80% and transform times by 40-60%
tags: performance, plugins, dev-server, build, profiling
---

## Audit Vite Plugin Performance

**Impact: HIGH (50-80% faster startup, 40-60% faster transforms)**

Community Vite plugins can significantly impact dev server startup time and file transformation speed. While Vite's internal plugins are optimized, third-party plugins may perform expensive operations that slow down the development experience.

**Incorrect (performance-killing plugin patterns):**

```typescript
// ❌ Plugin with heavy dependencies loaded eagerly
import { someHeavyLibrary } from 'heavy-package' // 50MB dependency

export default defineConfig({
  plugins: [
    {
      name: 'my-plugin',
      // ❌ Expensive operation in buildStart (blocks dev server startup)
      buildStart() {
        const result = someHeavyLibrary.processEverything()
        // This runs on EVERY dev server start
      },
      // ❌ Transform hook that processes all files
      transform(code, id) {
        // No early return - processes every single file
        return someHeavyLibrary.transform(code)
      }
    }
  ]
})
```

**Correct (optimized plugin patterns):**

```typescript
export default defineConfig({
  plugins: [
    {
      name: 'my-plugin',
      // ✅ Minimal work in buildStart
      buildStart() {
        // Only essential initialization
      },
      // ✅ Early returns and conditional processing
      transform(code, id) {
        // Check if this file needs processing
        if (!id.endsWith('.special')) return null
        if (!code.includes('SPECIAL_KEYWORD')) return null
        
        // ✅ Dynamic import for heavy dependencies
        const { transform } = await import('heavy-package')
        return transform(code)
      }
    }
  ]
})
```

**Plugin Performance Checklist:**

1. **Large dependencies**: Dynamically import heavy dependencies only when needed
2. **buildStart/config/configResolved hooks**: Keep these fast - they block dev server startup
3. **resolveId/load/transform hooks**: Add early returns based on file extension or content checks
4. **Avoid full AST parsing**: Use regex for dev, save complete parsing for build

**Profiling Plugin Performance:**

```bash
# Debug plugin transform times
vite --debug plugin-transform

# Generate CPU profile
vite --profile
# Then press 'p + enter' in terminal to save .cpuprofile
# Analyze with https://www.speedscope.app
```

**Inspect Transform Times:**

```bash
npm install -D vite-plugin-inspect

# Add to vite.config.ts
import Inspect from 'vite-plugin-inspect'

export default defineConfig({
  plugins: [Inspect()]
})

# Visit /__inspect/ in browser to see transform times
```

**Additional Context:**

Request waterfalls become more significant with slow transforms. If a file takes 100ms to transform, and it imports 5 other files that each take 100ms, you're looking at 600ms+ just for transforms.

Common plugin performance issues:
- Loading entire heavy libraries at plugin initialization
- Running expensive operations in hooks that fire on every dev server start
- Processing files that don't need transformation
- Synchronous operations that could be async
- Full AST parsing when regex would suffice

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#audit-configured-vite-plugins)
