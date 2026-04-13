---
name: react-native-instant
description: Eliminate loading states in Postgres-backed React Native or Expo apps by combining React Query cache persistence with MMKV, targeted prefetching, and optimistic updates. Use when implementing or debugging instant app launch, instant screen navigation, instant saves, offline-friendly sync, React Query persistence, MMKV hydration, optimistic mutations, or local-first UX on top of a server-truth API.
---

Apply this skill with one framing rule: native feel requires native UI and instant data. Prefer stale local data now over fresh remote data later.

## Core workflow

Follow this sequence unless the user explicitly narrows scope:

1. Persist query cache to disk.
2. Hydrate and render cached data immediately.
3. Refetch in the background and reconcile.
4. Prefetch likely next queries before navigation.
5. Apply optimistic mutations with rollback and invalidation.
6. Keep the server as source of truth and the UI local-first.

State these UX constraints when relevant:

- Do not show spinners on app reopen, routine navigation, or normal saves.
- Show blocking loading UI only on a true first launch with no cache.
- Queue work offline instead of blocking the interaction.

## Data lifecycle

```text
App launch or screen reopen
  -> hydrate React Query cache from MMKV
  -> render immediately from cache
  -> refetch stale data in background
  -> reconcile cache with server response
  -> keep subsequent interactions instant
```

## Implementation recipe

### 1. Persist cache for instant launch

Target outcome: show meaningful content immediately after reopening the app.

Do this:

- Use `PersistQueryClientProvider` with an MMKV-backed persister.
- Set `staleTime` in minutes, not seconds, for user-facing data that can tolerate brief staleness.
- Set `gcTime` long enough to survive normal reopen flows, typically around 24 hours.
- Disable focus refetch patterns that behave poorly on mobile resume.

Enforce this:

- Version the persisted cache key and bump it whenever query keys, shape, or hydration assumptions change.

```text
key: app-cache-v{N}
```

During hydration:

- Render cached data immediately.
- Refetch stale queries in the background.
- Reconcile quietly instead of clearing the screen first.

### 2. Prefetch for instant navigation

Target outcome: the next screen reads warm cache instead of waiting on the network.

Use the lightest prefetch that fits the flow:

- Prefetch likely next screens on current screen mount.
- Prefetch on `onPressIn` for high-intent navigation.
- Prefetch related detail queries in parallel.
- Prefetch a small critical set during app startup.

Constrain it:

- Never prefetch unbounded lists.
- Prefetch first page, visible items, recent items, or top-N only.
- Keep query keys stable, for example `['entity', id]` and `['list', { filter }]`.

### 3. Use optimistic updates for instant saves

Target outcome: the UI commits immediately and sync happens in the background.

Use this mutation pattern:

- `cancelQueries` for affected keys.
- Snapshot prior cache state.
- `setQueryData` for optimistic writes.
- Roll back on error.
- `invalidateQueries` on settle to reconcile with server truth.

If the mutation changes more than one cache surface, update and roll back all of them:

- list queries
- detail queries
- aggregate or stats queries

### 4. Preserve continuity offline

Target outcome: the app stays responsive without connectivity.

Do this:

- Use `networkMode: 'offlineFirst'` for relevant queries and mutations.
- Persist pending mutations with `shouldDehydrateMutation: () => true`.
- Resume paused mutations on app startup.
- Drive online status from device connectivity rather than assumptions.

Rule: offline should defer synchronization, not block the interaction.

## Review checklist

Check these before shipping:

- Reopen path renders cached content immediately.
- Navigation path avoids loading UI on common screen transitions.
- Mutation path updates the UI immediately and reconciles later.
- Offline path keeps interactions usable and resumes sync later.
- Persisted cache version changes when cache shape changes.
- Background refetch does not visibly thrash the UI.

## Common failure modes

- Missing cache versioning causes stale or invalid hydrated data after schema changes.
- Over-prefetching wastes memory, battery, and network.
- Partial optimistic updates leave list, detail, and aggregate views out of sync.
- Default spinner-first UI erases the benefit of persistence.
- Aggressive refetch settings make the app feel jittery and network-bound.

## Definition of done

- App reopen shows content immediately.
- Normal navigation shows no avoidable loading states.
- Saves feel instant and recover correctly on failure.
- Offline actions queue and recover without user intervention.
