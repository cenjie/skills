---
title: Review Browser Setup for Development
impact: MEDIUM
impactDescription: Can reduce dev server startup and reload times by 30-50%
tags: performance, dev-server, browser, debugging
---

## Review Browser Setup for Development

**Impact: MEDIUM (30-50% faster dev server startup and reloads)**

Browser extensions and dev tools settings can significantly interfere with Vite's dev server performance, especially for large applications. The Vite dev server uses hard caching for pre-bundled dependencies and fast 304 responses, but browser settings can negate these optimizations.

**Incorrect (performance-killing browser setup):**

```text
❌ Using regular browser profile with many extensions
❌ "Disable Cache" enabled in Browser Dev Tools
❌ Multiple security/privacy extensions intercepting requests
❌ Ad blockers interfering with local dev requests
```

**Correct (optimized browser setup):**

```text
✅ Create a dev-only browser profile without extensions
✅ Use incognito/private mode for development
✅ Keep "Disable Cache" OFF in Browser Dev Tools (unless debugging cache issues)
✅ Allow Vite dev server requests through any security extensions
```

**Additional Context:**

The Vite dev server relies on aggressive caching strategies:
- Pre-bundled dependencies are cached with strong ETags
- Source code uses 304 Not Modified responses
- Disabling cache forces full re-fetches on every request

When debugging cache-specific issues, temporarily enable "Disable Cache", but remember to turn it off for normal development work.

For teams, document the recommended browser setup in your project README to ensure consistent performance across developers.

Reference: [Vite Performance Guide](https://vite.dev/guide/performance#review-your-browser-setup)
