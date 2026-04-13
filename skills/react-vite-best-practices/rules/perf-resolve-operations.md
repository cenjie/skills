---
title: Reduce Resolve Operations
impact: HIGH
impactDescription: Can reduce dev server response time by 30-50% for import-heavy files
tags: performance, imports, typescript, resolve, dev-server
---

## Reduce Resolve Operations

**Impact: HIGH (30-50% faster import resolution)**

Import path resolution can be expensive when Vite has to check multiple file extensions. Each implicit import triggers multiple filesystem checks, and these add up quickly in large applications with many imports.

**Incorrect (expensive implicit imports):**

```typescript
// ❌ Vite checks 6 files: .mjs, .js, .mts, .ts, .jsx, .tsx
import { Button } from './components/Button'
import { Header } from './components/Header'
import { Footer } from './components/Footer'
import { Sidebar } from './components/Sidebar'
// 24 filesystem checks total for 4 imports!

// ❌ Overly broad resolve.extensions
export default defineConfig({
  resolve: {
    extensions: ['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json', '.vue', '.svelte']
    // Every implicit import checks 9 extensions
  }
})
```

**Correct (explicit imports with optimized config):**

```typescript
// ✅ Explicit extensions - only 1 filesystem check per import
import { Button } from './components/Button.tsx'
import { Header } from './components/Header.tsx'
import { Footer } from './components/Footer.tsx'
import { Sidebar } from './components/Sidebar.tsx'
// 4 filesystem checks total - 6x faster!

// ✅ Narrowed resolve.extensions
export default defineConfig({
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx']
    // Removed rarely-used extensions
    // Still works for node_modules
  }
})
```

**TypeScript Configuration for Explicit Extensions:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true
  }
}
```

This allows you to write `.ts` and `.tsx` extensions directly in your imports without TypeScript errors.

**Resolution Process Example:**

```text
import './Component'

Vite checks in order:
1. ./Component (no extension) ❌
2. ./Component.mjs ❌
3. ./Component.js ❌
4. ./Component.mts ❌
5. ./Component.ts ❌
6. ./Component.jsx ❌
7. ./Component.tsx ✅ FOUND

Total: 7 filesystem operations
```

vs.

```text
import './Component.tsx'

Vite checks:
1. ./Component.tsx ✅ FOUND

Total: 1 filesystem operation
```

**Additional Context:**

The performance impact scales with:
- Number of imports in your application
- Number of extensions in `resolve.extensions`
- Filesystem speed (especially on Windows or network drives)

For a typical React app with 500 imports and default extensions, you're looking at 3,500 filesystem checks vs. 500 with explicit extensions - a 7x reduction.

**Plugin Authors:**

If you're writing a Vite plugin, minimize calls to `this.resolve()` - each call triggers the full resolution process.

**Caveats:**

- Ensure explicit extensions work for your node_modules dependencies
- Some tools may not support `.ts`/`.tsx` extensions in imports
- Team must adopt the convention consistently

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#reduce-resolve-operations)
