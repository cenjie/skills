---
name: react-vite-best-practices
description: Improve React applications built with Vite. Use this whenever the user mentions Vite, `vite.config.*`, bundle size, slow builds, slow dev server startup, broken or slow HMR/Fast Refresh, route or component lazy loading, `React.lazy`, `Suspense`, manual chunks, dependency prebundling, asset loading, tree shaking, or asks for performance/review help on a React + Vite codebase even if they do not explicitly ask for "best practices."
license: MIT
metadata:
  author: agent-skills
  version: "1.1.0"
---

# React + Vite Best Practices

Use this skill to make React + Vite changes that are measurably better, not just conventionally tidy. Focus on the user's actual bottleneck first: production bundle size, runtime performance, startup/build speed, or dev-loop friction.

## Apply This Skill When

Use it when the request involves any of these:

- React apps built with Vite
- `vite.config.ts`, `vite.config.js`, or plugin setup
- Bundle splitting, oversized chunks, or bad caching
- Slow page loads caused by JS, images, fonts, or SVG handling
- Broken, noisy, or slow HMR / Fast Refresh
- `React.lazy`, `Suspense`, dynamic imports, or route-level splitting
- Dependency bloat, tree-shaking issues, or bundle analysis
- Client env vars via `import.meta.env`
- Review or refactor work where React + Vite performance is part of the acceptance criteria

Do not use this skill for generic React work that does not involve Vite, build tooling, or performance-sensitive frontend delivery.

## Operating Rules

1. Confirm this is a React + Vite project before leaning on Vite-specific guidance.
2. Start from the user's symptom, not from a generic checklist.
3. Prefer the smallest change that improves the measured bottleneck.
4. Avoid cargo-cult optimizations.
5. Keep recommendations compatible with the existing stack unless the user asks for broader refactoring.

## Workflow

### 1. Identify the bottleneck

Classify the task before editing:

- `perf`: core performance issues (barrel files, plugin performance, import resolution, file warmup, native tooling)
- `build`: production bundle size, chunking, caching, minification, sourcemaps
- `split`: route/component lazy loading, dynamic imports, chunk boundaries
- `dev`: startup time, dependency prebundling, dev server responsiveness
- `hmr`: Fast Refresh resets, full reloads, HMR misconfiguration
- `asset`: images, fonts, SVG strategy, static assets
- `env`: `import.meta.env`, `VITE_` usage, mode-specific config
- `bundle`: tree shaking, dependency weight, visualizer-driven cleanup

If multiple categories apply, handle them in this order:

1. Broken behavior
2. Large user-facing performance wins
3. Developer-experience improvements

### 2. Inspect before changing

Look for the relevant files first:

- `vite.config.*`
- `package.json`
- `src/main.*`
- router setup and route definitions
- heavyweight feature entry points
- asset-loading components
- env typings such as `vite-env.d.ts`

When reviewing, call out the concrete problem you found, such as:

- one large vendor chunk
- eager imports for rarely used routes
- barrel imports defeating tree shaking
- `import.meta.env` misuse
- SVGs handled inconsistently
- component files structured in ways that break Fast Refresh

### 3. Choose targeted fixes

Prefer changes that map directly to the symptom:

- Big initial bundle: route lazy loading, dynamic imports, chunk strategy, vendor separation
- Slow repeat visits: stable chunking and hashed assets for better caching
- Slow dev startup: `optimizeDeps` tuning, plugin cleanup, fewer unnecessary transforms
- HMR resets: fix Fast Refresh boundaries and mixed exports
- Asset-heavy pages: image sizing, lazy loading, SVG/component strategy
- Dead code in bundles: direct imports, ESM-friendly libraries, tree-shaking fixes

### 4. Explain tradeoffs

When proposing or applying a change, explain the tradeoff in one sentence:

- smaller initial bundle vs additional async requests
- faster builds vs weaker minification
- better caching vs more chunk-management complexity

Do not present an optimization as free if it increases complexity or shifts cost elsewhere.

## Default Recommendations

Use these defaults unless the repo context suggests otherwise:

- Prefer route-level splitting before fine-grained component splitting
- Keep chunking strategy understandable; avoid generating many tiny chunks
- Prefer direct imports over broad barrel imports in performance-sensitive code paths
- Use `vite-plugin-svgr` only when SVGs need component behavior
- Type `import.meta.env` when env vars are part of app logic
- Keep dev-only tooling out of production bundles
- Preserve Fast Refresh by keeping component files focused on component exports

## Rule File Map

Open only the files relevant to the current task.

### Build and bundle output

- `rules/build-manual-chunks.md`
- `rules/build-chunk-strategy.md`
- `rules/build-code-splitting.md`
- `rules/build-minification.md`
- `rules/build-minify-terser.md`
- `rules/build-target-modern.md`
- `rules/build-sourcemaps.md`
- `rules/build-asset-hashing.md`
- `rules/build-compression.md`
- `rules/build-tree-shaking.md`
- `rules/build-vendor-splitting.md`
- `rules/bundle-tree-shaking.md`

### Code splitting

- `rules/split-route-lazy.md`
- `rules/split-lazy-routes.md`
- `rules/split-component-lazy.md`
- `rules/split-dynamic-imports.md`
- `rules/split-suspense-boundaries.md`
- `rules/split-prefetch-hints.md`
- `rules/split-library-chunks.md`

### Development and HMR

- `rules/dev-dependency-prebundling.md`
- `rules/dev-fast-refresh.md`
- `rules/dev-hmr-config.md`
- `rules/hmr-fast-refresh.md`

### Assets and env

- `rules/asset-image-optimization.md`
- `rules/asset-svg-components.md`
- `rules/env-vite-prefix.md`

### Performance optimization

- `rules/perf-avoid-barrel-files.md`
- `rules/perf-plugin-audit.md`
- `rules/perf-resolve-operations.md`
- `rules/perf-warmup-files.md`
- `rules/perf-native-tooling.md`
- `rules/perf-browser-setup.md`

## Response Pattern

When using this skill, structure the output around the current task:

1. State the main bottleneck or risk.
2. Name the concrete files/settings causing it.
3. Apply or recommend the minimum set of changes.
4. Mention expected impact in practical terms.
5. Note any validation still needed, such as build output or bundle analysis.

## Anti-Patterns

Avoid these common mistakes:

- Recommending lazy loading for tiny, always-used components
- Splitting every dependency into its own chunk
- Using webpack-specific comments or assumptions in Vite guidance
- Tuning `manualChunks` without checking actual dependency usage
- Suggesting client-side env exposure for secrets
- Claiming tree shaking works while imports still force whole-module inclusion
- Breaking Fast Refresh by mixing components and non-component exports carelessly

## Example Triggers

- "Our React app uses Vite and the dashboard bundle is huge. Can you fix the chunking?"
- "Fast Refresh keeps doing full reloads after I edit components."
- "Can you review this `vite.config.ts` and make the build output saner?"
- "I need route lazy loading and better Suspense boundaries in this Vite app."
- "Why is `import.meta.env.MY_API_URL` undefined in production?"

## Reference

If you need detailed examples, open the matching rule files in `rules/` rather than expanding this file with long embedded code samples.
