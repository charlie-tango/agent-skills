---
name: react-effect-audit
description: >-
  enforce strict scrutiny around useeffect in react and next.js code. use when reviewing, refactoring, generating, or modifying react components, hooks, or client-side ui logic, especially whenever a useeffect exists or is about to be introduced. prefer patterns from react's "you might not need an effect": derive values during render, keep event-driven logic in event handlers, reset with keys, lift state, use refs for imperative values, and subscribe with purpose-built apis like usesyncexternalstore when appropriate. treat usememo as optional and justified, not default. require explicit justification before keeping or adding any effect.
---

# React Effect Audit

Treat `useEffect` as a last resort, not a default move.

When working on React or Next.js code, run this check before introducing or preserving any Effect:

1. Identify the purpose of the proposed Effect in one sentence.
2. Ask: **what external system is being synchronized?**
3. If there is no external system, do not use an Effect.
4. Replace the Effect with the simplest non-Effect pattern that fits.
5. If an Effect remains necessary, state the external system explicitly and keep the Effect minimal.

## Non-Effect alternatives to exhaust first

Try these in order before accepting an Effect:

### 1. Derive during render
Use plain variables for values that can be computed from props, state, or context.

- Avoid redundant state like `fullName`, `filteredItems`, `sortedRows`, `isEligible`, `selection`, or `hasError` when it can be computed during render.
- Do not mirror props into state just to keep them "in sync".
- If something can be calculated from existing props or state, prefer calculation during render over storing and syncing it.

### 2. Handle user actions in event handlers
If logic happens because the user clicked, typed, submitted, selected, dragged, or toggled something, keep that logic in the event handler.

- API calls caused by a button click belong in the click or submit handler.
- Notifications, tracking tied to a user action, and mutations triggered by a known interaction should stay with that interaction.
- Do not create state like `shouldSubmit` or `jsonToSubmit` just to trigger an Effect afterward.

### 3. Reset or separate state with keys
If state should reset when identity changes, prefer component structure and `key` over an Effect that manually clears state.

- Example: `key={userId}` to reset profile-specific local state.
- Prefer key-based reset for whole subtrees before reaching for manual cleanup state.

### 4. Lift state or reshape data flow
If an Effect exists only to notify a parent, copy data between siblings, or keep two pieces of state synchronized, revisit ownership of state.

- Prefer lifting state, passing values down, and invoking callbacks from the event that changed the state.
- Avoid child-to-parent synchronization through Effects when the parent can own the data directly.

### 5. Restructure before memoizing
If the goal is to avoid recomputing a pure calculation, first ask whether the code structure can avoid unnecessary re-renders.

- Prefer plain render-time calculation by default.
- Consider moving unrelated state into a child component before reaching for memoization.
- Consider `useMemo` only for pure calculations that are actually expensive or demonstrably hot.
- Do not treat `useMemo` as the automatic replacement for `useEffect`.
- React Compiler may remove the need for manual `useMemo`, so avoid adding it preemptively.

### 6. Use refs for imperative values that do not affect rendering
If the value is only needed for imperative access and should not trigger a re-render, prefer `useRef`.

### 7. Prefer purpose-built subscriptions
If the code is subscribing to an external store, prefer a dedicated API like `useSyncExternalStore` over ad hoc subscription logic in an Effect.

## Suspicious Effect patterns

Flag these as probably wrong unless there is unusually strong justification:

- `useEffect(() => setSomething(...), [deps])` where `something` is derived from props or other state
- Effects that copy props into state
- Effects that filter, map, sort, or aggregate data and then store the result in state
- Effects used to react to a submit, click, or other interaction already known in an event handler
- Effects that reset local state when an id or prop changes
- Effects whose only job is to keep React state in sync with other React state
- Effect chains where one state update triggers another Effect and then another state update
- Data passed upward to a parent from a child Effect when the parent could own the source of truth

## Legitimate Effect bar

Keep or add an Effect only when it is genuinely synchronizing React with something outside React.

Examples that may justify an Effect:

- subscribing and unsubscribing to an external source
- coordinating with browser APIs or DOM APIs outside normal declarative rendering
- wiring up third-party widgets
- synchronizing with timers, sockets, media playback, or network state that is not better handled by the framework
- fetching data that must stay synchronized with visible inputs like query params or pagination when framework data APIs are not available

If the Effect performs async work, verify cleanup or cancellation behavior when stale results could race with newer ones.

When you keep an Effect, explain it in this format:

`Keeping useEffect because it synchronizes React state/UI with [external system]. Non-Effect options were rejected because [reason].`

## Required response behavior

Whenever you remove or avoid an Effect, say so explicitly and name the replacement pattern.

Use phrasing like:

- `Removed unnecessary useEffect. This value is derived during render.`
- `Avoided useEffect. The logic belongs in the submit handler.`
- `Avoided useEffect. Reset state by keying the child component.`
- `Avoided useEffect. Lifted state instead of synchronizing two components.`
- `Avoided useEffect. Kept the calculation in render and did not memoize prematurely.`
- `Avoided useEffect. Used useMemo only because the calculation is pure and measurably expensive.`

Whenever you keep an Effect, include the justification sentence above.

## `useMemo` guardrail

Do not recommend `useMemo` unless all of these are true:

1. the work is a pure calculation done during render
2. the plain render-time version is correct but too costly
3. there is a concrete reason to believe the cost matters, such as scale, profiling, or an obviously expensive computation
4. simpler structural changes would not remove the pressure

If those conditions are not met, prefer plain render-time calculation.

## Rewrite examples

### Derived state
Avoid:

```tsx
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

Prefer:

```tsx
const fullName = `${firstName} ${lastName}`;
```

### Event-driven side effect
Avoid:

```tsx
useEffect(() => {
  if (submitted) {
    saveForm(formData);
  }
}, [submitted, formData]);
```

Prefer:

```tsx
async function handleSubmit(e: FormEvent) {
  e.preventDefault();
  await saveForm(formData);
}
```

### Reset by key
Avoid:

```tsx
useEffect(() => {
  setComment('');
}, [userId]);
```

Prefer:

```tsx
<ProfileForm key={userId} userId={userId} />
```

### Pure calculation, no memo by default
Avoid:

```tsx
const [visibleTodos, setVisibleTodos] = useState([]);
useEffect(() => {
  setVisibleTodos(getFilteredTodos(todos, filter));
}, [todos, filter]);
```

Prefer:

```tsx
const visibleTodos = getFilteredTodos(todos, filter);
```

Only consider `useMemo` if the calculation is pure and genuinely expensive.

### External store subscription
Avoid:

```tsx
const [isOnline, setIsOnline] = useState(true);
useEffect(() => {
  function updateState() {
    setIsOnline(navigator.onLine);
  }
  updateState();
  window.addEventListener('online', updateState);
  window.addEventListener('offline', updateState);
  return () => {
    window.removeEventListener('online', updateState);
    window.removeEventListener('offline', updateState);
  };
}, []);
```

Prefer:

```tsx
const isOnline = useSyncExternalStore(subscribe, () => navigator.onLine, () => true);
```

### Fetch with cleanup when Effect is necessary
If data must stay synchronized with visible state and framework data APIs are not available, an Effect can be acceptable, but verify stale response cleanup.

Avoid:

```tsx
useEffect(() => {
  fetchResults(query, page).then(setResults);
}, [query, page]);
```

Prefer:

```tsx
useEffect(() => {
  let ignore = false;
  fetchResults(query, page).then(json => {
    if (!ignore) setResults(json);
  });
  return () => {
    ignore = true;
  };
}, [query, page]);
```
