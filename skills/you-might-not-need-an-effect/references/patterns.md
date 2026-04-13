# React Effect Removal Patterns

Use this file when the code needs concrete transformation patterns.

## Decision Table

| Smell | Better Pattern |
| --- | --- |
| `useEffect` writes derived state from props/state | Calculate during render |
| `useEffect` caches slow pure work | `useMemo` |
| `useEffect` reacts to a click or submit | Event handler |
| `useEffect` resets form state when entity ID changes | Keyed subtree |
| `useEffect` mirrors selected object from a list | Store `selectedId`, derive object |
| `useEffect` subscribes to mutable external data | `useSyncExternalStore` |
| `useEffect` fetches network data for current inputs | Keep effect, add cleanup or move to framework/data hook |

## Before/After Patterns

### Derived State

Avoid:

```js
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);
```

Prefer:

```js
const fullName = firstName + ' ' + lastName;
```

### Filtered Lists

Avoid:

```js
const [visibleTodos, setVisibleTodos] = useState([]);
useEffect(() => {
  setVisibleTodos(getFilteredTodos(todos, filter));
}, [todos, filter]);
```

Prefer:

```js
const visibleTodos = useMemo(
  () => getFilteredTodos(todos, filter),
  [todos, filter]
);
```

If the calculation is cheap, prefer plain render-time derivation.

### Event-Specific Effects

Avoid:

```js
useEffect(() => {
  if (jsonToSubmit !== null) {
    post('/api/register', jsonToSubmit);
  }
}, [jsonToSubmit]);
```

Prefer:

```js
function handleSubmit(e) {
  e.preventDefault();
  post('/api/register', { firstName, lastName });
}
```

### Reset On Entity Change

Avoid:

```js
useEffect(() => {
  setComment('');
}, [userId]);
```

Prefer:

```js
function ProfilePage({ userId }) {
  return <Profile key={userId} userId={userId} />;
}
```

### Partial State Adjustment

Use only when a better state model is not practical.

```js
const [prevItems, setPrevItems] = useState(items);
if (items !== prevItems) {
  setPrevItems(items);
  setSelection(null);
}
```

Prefer redesigning state shape first, for example:

```js
const [selectedId, setSelectedId] = useState(null);
const selection = items.find(item => item.id === selectedId) ?? null;
```

### External Store Subscription

Avoid:

```js
useEffect(() => {
  function updateState() {
    setIsOnline(navigator.onLine);
  }
  window.addEventListener('online', updateState);
  window.addEventListener('offline', updateState);
  return () => {
    window.removeEventListener('online', updateState);
    window.removeEventListener('offline', updateState);
  };
}, []);
```

Prefer:

```js
return useSyncExternalStore(
  subscribe,
  () => navigator.onLine,
  () => true
);
```

### Fetch With Cleanup

Keep the effect, but guard against stale responses:

```js
useEffect(() => {
  let ignore = false;
  fetchResults(query, page).then(json => {
    if (!ignore) {
      setResults(json);
    }
  });
  return () => {
    ignore = true;
  };
}, [query, page]);
```

## Review Questions

- Can this value be derived from current props/state on every render?
- Is this code running because the component is visible, or because the user performed a specific action?
- Is this state duplicated elsewhere?
- Would storing an ID or source value remove the need for synchronization?
- Would a `key` better model a change in identity?
- Is this effect actually synchronizing with a browser API, third-party store, or network request?
