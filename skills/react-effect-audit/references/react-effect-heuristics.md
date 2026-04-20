# React effect heuristics

Use this reference only when you need a deeper decision rule.

## Core principle
Effects are for synchronizing with external systems. If no external system is involved, prefer a non-Effect approach.

## Decision prompt
Before adding `useEffect`, answer all four:

1. What external system exists here?
2. Why can't this be derived during render?
3. Why can't this happen in an event handler or by restructuring state?
4. Why is plain render-time calculation insufficient without memoization?

If those answers are weak, do not use an Effect.

## Preferred replacements
- Derived value from props/state → calculate during render
- User-triggered workflow → event handler
- Reset on identity change → `key`
- State ownership confusion → lift state or pass callbacks
- Imperative mutable value that should not re-render → `useRef`
- External store subscription → `useSyncExternalStore`
- Expensive pure calculation → `useMemo` only if cost is real and structure cannot make it unnecessary

## `useMemo` rule
Treat `useMemo` as an optimization, not data-flow control.

Do not recommend it just because an Effect was removed. Prefer this order:

1. plain render-time calculation
2. restructure component boundaries to avoid unrelated re-renders
3. add `useMemo` only when the calculation is pure and the cost matters

Remember that React Compiler can often remove the need for manual memoization.

## Review cues
When editing existing code, inspect every Effect body for `setState`.

A `setState` inside an Effect is a strong hint that the code may be modeling data flow incorrectly.

Also inspect for these warning signs:

- chains of Effects updating state to trigger each other
- child Effects pushing data upward into parent state
- fetch Effects with no stale-response cleanup
- subscription Effects that could use a dedicated subscription API
