---
title: Avoid Barrel Files
impact: CRITICAL
impactDescription: Can reduce initial page load by 40-70% by eliminating unnecessary module loading
tags: performance, imports, tree-shaking, barrel-files, code-splitting
---

## Avoid Barrel Files

**Impact: CRITICAL (40-70% reduction in initial bundle size)**

Barrel files (index files that re-export APIs from other files) force Vite to load and transform all re-exported files, even when you only need one export. This defeats tree-shaking and loads unnecessary code on initial page load.

**Incorrect (barrel file pattern):**

```typescript
// ❌ src/utils/index.ts (barrel file)
export * from './color.js'
export * from './dom.js'
export * from './slash.js'
export * from './format.js'
export * from './validation.js'
// All 5 files must be loaded and transformed

// ❌ Component imports from barrel
import { slash } from './utils'
// Vite must load ALL utils files to find 'slash'
// Even though you only need slash.js
```

**Why This Is Expensive:**

```text
import { slash } from './utils'

Vite must:
1. Load utils/index.ts
2. Load utils/color.js (might have side effects)
3. Load utils/dom.js (might have side effects)
4. Load utils/slash.js (the one you actually need)
5. Load utils/format.js (might have side effects)
6. Load utils/validation.js (might have side effects)
7. Transform all 6 files
8. Check each for side effects

Result: 6x more work than necessary
```

**Correct (direct imports):**

```typescript
// ✅ Direct import - only loads what you need
import { slash } from './utils/slash.js'
// Vite loads and transforms ONLY slash.js

// ✅ Multiple direct imports
import { slash } from './utils/slash.js'
import { formatDate } from './utils/format.js'
// Only 2 files loaded instead of all 6
```

**Real-World Impact:**

```typescript
// ❌ Barrel import from large utility library
import { debounce } from './utils'
// Loads: 50+ utility files, 200KB of code
// Initial page load: 2.5s

// ✅ Direct import
import { debounce } from './utils/debounce.js'
// Loads: 1 file, 4KB of code
// Initial page load: 0.8s
```

**When Barrel Files Are Acceptable:**

```typescript
// ✅ Public API for a library (external consumers)
// src/index.ts
export { Button } from './components/Button'
export { Input } from './components/Input'
// This is fine for library entry points

// ❌ Internal barrel files within your app
// src/components/index.ts
export * from './Button'
export * from './Input'
// Avoid this pattern internally
```

**Migration Strategy:**

```bash
# Find barrel file imports
grep -r "from '\./[^/]*'$" src/

# Replace with direct imports
# Before: import { Button } from './components'
# After:  import { Button } from './components/Button.tsx'
```

**Additional Context:**

Barrel files were popular in the webpack era for cleaner imports, but they're a performance anti-pattern in Vite because:

1. **Side effects**: Vite must execute all re-exported modules to check for side effects
2. **Transform waterfall**: Each file must be transformed before Vite knows what it exports
3. **Tree-shaking limitations**: `export *` makes it harder for bundlers to eliminate dead code
4. **Dev server slowdown**: More files to watch, transform, and serve

The performance cost scales with:
- Number of files in the barrel
- Size of each re-exported file
- Depth of barrel file nesting

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#avoid-barrel-files) | [GitHub Issue #8237](https://github.com/vitejs/vite/issues/8237)
