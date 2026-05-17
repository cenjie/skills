---
title: Use Lesser or Native Tooling
impact: HIGH
impactDescription: Can reduce build time by 50-80% and dev server startup by 40-60%
tags: performance, build, tooling, css, native
---

## Use Lesser or Native Tooling

**Impact: HIGH (50-80% faster builds, 40-60% faster dev startup)**

Reducing the amount of transformation work and using native tooling significantly improves both dev server and build performance. The less code that needs processing, the faster your builds.

**Incorrect (unnecessary transformations):**

```typescript
// ❌ Using Sass for features CSS can handle natively
// styles.scss
$primary-color: #007bff;

.button {
  color: $primary-color;
  
  &:hover {
    color: darken($primary-color, 10%);
  }
  
  .icon {
    margin-right: 8px;
  }
}

// ❌ Transforming SVGs into React components
import Logo from './logo.svg?component'
import Icon from './icon.svg?component'
import Banner from './banner.svg?component'
// Each SVG is transformed into a React component
// Increases bundle size and transform time

// vite.config.ts
import { defineConfig } from 'vite'
import svgr from 'vite-plugin-svgr'

export default defineConfig({
  plugins: [svgr()] // Transforms all SVGs
})
```

**Correct (native CSS and optimized SVG handling):**

```typescript
// ✅ Use native CSS features (no Sass needed)
// styles.css
.button {
  color: #007bff;
  
  /* Native CSS nesting (supported in modern browsers) */
  &:hover {
    color: color-mix(in srgb, #007bff 90%, black);
  }
  
  & .icon {
    margin-right: 8px;
  }
}

// ✅ Import SVGs as URLs or strings (no transformation)
import logoUrl from './logo.svg?url'
import iconString from './icon.svg?raw'

function Logo() {
  return <img src={logoUrl} alt="Logo" />
}

function Icon() {
  return <div dangerouslySetInnerHTML={{ __html: iconString }} />
}

// ✅ Only transform SVGs that truly need component behavior
// vite.config.ts
export default defineConfig({
  plugins: [
    // No svgr plugin - use native SVG handling
  ]
})
```

**CSS: Native vs. Preprocessor:**

```css
/* ✅ Modern CSS can handle most Sass/Less features */

/* Variables */
:root {
  --primary-color: #007bff;
  --spacing: 1rem;
}

/* Nesting (native in modern browsers) */
.card {
  padding: var(--spacing);
  
  & .title {
    font-size: 1.5rem;
  }
  
  &:hover {
    transform: scale(1.02);
  }
}

/* Color manipulation */
.button {
  background: var(--primary-color);
  border: 1px solid color-mix(in srgb, var(--primary-color) 80%, black);
}

/* Math */
.container {
  width: calc(100% - 2rem);
  padding: calc(var(--spacing) * 2);
}
```

**When to Use Preprocessors:**

```scss
// ✅ Use Sass/Less only when you need features CSS doesn't have:

// Complex mixins
@mixin responsive-grid($columns) {
  display: grid;
  grid-template-columns: repeat($columns, 1fr);
  
  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
}

// Advanced color functions not in CSS
$color: adjust-hue($primary, 30deg);

// Complex loops
@for $i from 1 through 12 {
  .col-#{$i} {
    width: percentage($i / 12);
  }
}
```

**SVG Handling Strategy:**

```typescript
// ✅ Decision tree for SVG handling

// Static images → Use ?url
import logo from './logo.svg?url'
<img src={logo} alt="Logo" />

// Inline SVG for styling → Use ?raw
import icon from './icon.svg?raw'
<div dangerouslySetInnerHTML={{ __html: icon }} />

// Need React props/state → Use component (sparingly)
import { ReactComponent as AnimatedIcon } from './animated.svg'
<AnimatedIcon className={isActive ? 'active' : ''} />
```

**Native Tooling: Lightning CSS:**

```typescript
// ✅ Experimental: Use Lightning CSS for faster CSS processing
export default defineConfig({
  css: {
    transformer: 'lightningcss',
    lightningcss: {
      // Lightning CSS is 100x faster than PostCSS
      // Written in Rust, native performance
    }
  }
})
```

**Performance Comparison:**

```text
Sass compilation:
- 1000 files: ~8 seconds
- Requires node-sass or dart-sass
- Additional transform step

Native CSS:
- 1000 files: ~0.5 seconds
- No transformation needed
- Direct browser support

Lightning CSS:
- 1000 files: ~0.1 seconds
- Native Rust performance
- Modern CSS features
```

**Migration Strategy:**

```typescript
// Step 1: Identify what preprocessor features you actually use
// Most projects only use: variables, nesting, and basic math

// Step 2: Replace with native CSS equivalents
// Variables → CSS custom properties
// Nesting → Native CSS nesting
// Math → calc()

// Step 3: Keep preprocessor only for advanced features
// Complex mixins, loops, advanced color functions

// Step 4: Consider Lightning CSS for remaining transforms
```

**Additional Context:**

The performance benefits compound:
- Fewer dependencies to install
- Faster dev server startup (no preprocessor initialization)
- Faster HMR (no Sass recompilation)
- Smaller build pipeline
- Better browser caching (native CSS)

Modern CSS has caught up to preprocessors for most use cases:
- Custom properties (variables)
- Native nesting
- color-mix() for color manipulation
- calc() for math
- @layer for cascade control
- @container for container queries

Only use preprocessors when you need features CSS truly doesn't have.

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#use-lesser-or-native-tooling) | [Lightning CSS Discussion](https://github.com/vitejs/vite/discussions/13835)
