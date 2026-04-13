---
title: Warm Up Frequently Used Files
impact: MEDIUM
impactDescription: Can reduce initial page load by 20-40% by pre-transforming critical files
tags: performance, dev-server, warmup, optimization
---

## Warm Up Frequently Used Files

**Impact: MEDIUM (20-40% faster initial page load in dev)**

Vite's dev server transforms files on-demand, which can create request waterfalls when files take time to transform. The `server.warmup` option pre-transforms frequently used files so they're ready when requested.

**The Problem (request waterfall):**

```text
Import graph:
main.tsx -> BigComponent.tsx -> big-utils.ts -> large-data.json

Timeline:
0ms:   Browser requests main.tsx
50ms:  main.tsx transformed, discovers BigComponent.tsx import
50ms:  Browser requests BigComponent.tsx
150ms: BigComponent.tsx transformed, discovers big-utils.ts import
150ms: Browser requests big-utils.ts
250ms: big-utils.ts transformed, discovers large-data.json import
250ms: Browser requests large-data.json
300ms: Page fully loaded

Total: 300ms (sequential waterfall)
```

**Incorrect (no warmup):**

```typescript
// ❌ No warmup - files transformed on-demand
export default defineConfig({
  // BigComponent.tsx takes 100ms to transform
  // big-utils.ts takes 100ms to transform
  // Sequential loading = slow initial page load
})
```

**Correct (strategic warmup):**

```typescript
// ✅ Warm up frequently used, slow-to-transform files
export default defineConfig({
  server: {
    warmup: {
      clientFiles: [
        './src/components/BigComponent.tsx',
        './src/utils/big-utils.ts',
        './src/data/large-data.json'
      ]
    }
  }
})
```

**With warmup:**

```text
Timeline:
0ms:   Dev server starts, begins warming up files
0ms:   BigComponent.tsx transforming in background
0ms:   big-utils.ts transforming in background
0ms:   large-data.json transforming in background
100ms: All files warmed up and cached
---
User opens browser:
0ms:   Browser requests main.tsx
50ms:  main.tsx transformed, discovers BigComponent.tsx import
50ms:  Browser requests BigComponent.tsx (already cached!)
51ms:  Browser requests big-utils.ts (already cached!)
52ms:  Browser requests large-data.json (already cached!)
52ms:  Page fully loaded

Total: 52ms (vs 300ms without warmup)
```

**Identifying Files to Warm Up:**

```bash
# Run dev server with debug flag
vite --debug transform

# Look for slow transforms in the output
vite:transform 28.72ms /@vite/client +1ms
vite:transform 62.95ms /src/components/BigComponent.tsx +1ms
vite:transform 102.54ms /src/utils/big-utils.ts +1ms
vite:transform 45.23ms /src/data/large-data.json +1ms

# Warm up files that take >50ms
```

**What to Warm Up:**

```typescript
export default defineConfig({
  server: {
    warmup: {
      clientFiles: [
        // ✅ Large components used on every page
        './src/components/Layout.tsx',
        './src/components/Navigation.tsx',
        
        // ✅ Heavy utility files
        './src/utils/api-client.ts',
        './src/utils/validation.ts',
        
        // ✅ Large data files
        './src/data/config.json',
        
        // ✅ Frequently imported libraries (if slow to transform)
        './src/lib/chart-setup.ts'
      ]
    }
  }
})
```

**What NOT to Warm Up:**

```typescript
export default defineConfig({
  server: {
    warmup: {
      clientFiles: [
        // ❌ Don't warm up rarely used files
        './src/pages/AdminPanel.tsx',
        
        // ❌ Don't warm up fast-transforming files
        './src/utils/tiny-helper.ts',
        
        // ❌ Don't warm up everything (defeats the purpose)
        './src/**/*.tsx'
      ]
    }
  }
})
```

**Combine with server.open:**

```typescript
export default defineConfig({
  server: {
    // ✅ Auto-open browser + warmup entry point
    open: true, // or open: '/dashboard'
    warmup: {
      clientFiles: [
        './src/main.tsx',
        './src/App.tsx'
      ]
    }
  }
})
```

When `server.open` is enabled, Vite automatically warms up the entry point of your app, providing an additional performance boost.

**Additional Context:**

Warmup is most effective for:
- Large component files with heavy transforms
- Files with complex TypeScript types
- JSON/data files that need parsing
- Files imported by many other files (high fan-in)

Don't overuse warmup:
- Warming up too many files slows down dev server startup
- Only warm up files that are actually slow to transform
- Focus on files used on initial page load

The goal is to eliminate waterfalls for critical files, not to pre-transform your entire codebase.

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#warm-up-frequently-used-files) | [server.warmup docs](https://vite.dev/config/server-options#server-warmup)
