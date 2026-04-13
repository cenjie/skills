---
name: you-might-not-need-an-effect
description: Refactor React components to remove unnecessary useEffect usage. Use when reviewing or rewriting React code that derives state from props, transforms data for rendering, mirrors values between components, resets state on prop change, performs event-specific work inside effects, subscribes to external stores, or needs guidance on when useEffect is actually appropriate.
---

# You Might Not Need an Effect

Identify whether each `useEffect` synchronizes with an external system. Keep the effect only when it coordinates React with something outside React such as the network, browser APIs, timers, subscriptions, imperative widgets, or analytics tied to display.

For every candidate effect, classify it before editing:

1. Derives render data from props or state.
2. Responds to a user event.
3. Resets or adjusts local state because props changed.
4. Synchronizes with an external system.

Apply the smallest replacement that preserves behavior:

- Derive values during render instead of storing redundant state.
- Use `useMemo` only for expensive pure calculations.
- Move event-specific logic into the event handler that caused it.
- Reset an entire subtree with a changing `key`.
- Prefer storing IDs or source-of-truth values instead of mirrored objects.
- Use `useSyncExternalStore` for external subscriptions.
- Keep fetch effects only when the UI must stay synchronized with current inputs, and add cleanup to ignore stale responses.

## Review Workflow

1. Find each `useEffect`, the state it reads, and the state it writes.
2. Ask what external system it synchronizes with.
3. If the answer is "none", remove the effect and simplify the state model.
4. If the effect exists only to trigger work after a click, submit, toggle, or navigation initiated by the user, move that work into the event handler.
5. If the effect mirrors props into state, prefer render-time derivation, a `key` reset, or a better state shape.
6. If the effect subscribes to browser or library state, replace manual subscription code with `useSyncExternalStore` when practical.
7. If the effect fetches data, keep it only if render/event alternatives do not fit, and ensure cleanup prevents race conditions.

## Replacement Rules

### Derive During Render

Remove state like `fullName`, filtered arrays, JSX fragments, counters, or selected objects when they can be computed from existing props/state on every render.

Prefer:

```js
const fullName = firstName + ' ' + lastName;
const activeTodos = todos.filter(todo => !todo.completed);
const selection = items.find(item => item.id === selectedId) ?? null;
```

### Memoize Only Expensive Pure Work

If derivation is expensive and inputs are stable, wrap the calculation in `useMemo`. Do not use an effect plus state as a cache.

### Keep Event Logic in Events

Move POST requests, notifications, parent callbacks, and multi-state updates into the event handler or a helper called by that handler.

### Reset State with Keys

When switching between conceptual entities like `userId`, `contact.id`, or `threadId`, split the component and pass the identity as `key` to the inner stateful component.

### Adjust State Carefully

If only part of local state must change after props change and you cannot avoid local state entirely, prefer changing state during render with a guarded previous-value check over using an effect. Use this sparingly and only for the same component's state.

### External Subscriptions

Replace patterns like manual `addEventListener` plus `useState` with `useSyncExternalStore` when subscribing to mutable external data.

### Data Fetching

Fetching can stay in an effect when results must track visible inputs such as query/page. Add cleanup to ignore stale responses. Prefer framework loaders or shared data hooks when available.

## Guardrails

- Do not remove effects that manage timers, DOM APIs, sockets, media queries, observers, third-party widgets, or network synchronization without replacing the synchronization mechanism.
- Do not introduce `useMemo` by default unless computation cost or re-render boundaries justify it.
- Do not preserve redundant state just to avoid recalculating simple expressions.
- Do not move display-driven analytics into event handlers when the requirement is "send on view".

## Files To Read When Needed

- Read [references/patterns.md](C:\Users\cenji\.agents\skills\you-might-not-need-an-effect\references\patterns.md) for concrete before/after transformations and a decision table.

## Output Expectations

When using this skill:

1. Name the effect category and why it is unnecessary or justified.
2. Describe the replacement pattern in one sentence.
3. Implement the refactor with the smallest state surface possible.
4. Call out any behavior changes or residual risks, especially around fetch timing and subscriptions.
