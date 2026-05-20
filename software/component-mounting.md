---
title: Component mounting (React)
category: software
date: 2026-05-20
tags: [react, lifecycle, testing, useeffect, fundamentals]
---

# Component mounting (React)

## TL;DR

A component **mounts** the first time it gets rendered into the DOM. That moment is when `useEffect(() => {...}, [])` (an effect with an empty or initially-defined dependency array) fires for the first time. In tests, "mounting the component" means calling `render(<Component />)` so the lifecycle actually runs — without that, none of the `useEffect`s execute and you can't observe their side effects.

## The explanation

### The lifecycle

A React component goes through three lifecycle moments:

| Moment | What happens | What runs |
|---|---|---|
| **Mount** | Component appears in the DOM for the first time | Constructor, initial render, `useEffect(() => {...}, [])` |
| **Update** | Props or state change, component re-renders | New render, `useEffect`s whose deps changed |
| **Unmount** | Component is removed from the DOM | `useEffect` cleanup functions |

"Mounting" is the first one. It happens once per instance of the component. If the user closes the modal and reopens it, that's a new mount.

### Why mounting matters for `useEffect`

```tsx
useEffect(() => {
  // This block runs once, on mount.
  checkoutStore.setGoogleAdsSendTos(googleAdsSendTos);
}, []);
```

The empty `[]` dep array means: only run this on mount, never on updates. It's the React equivalent of "initialize once."

If `googleAdsSendTos` is in the dep array instead (`[googleAdsSendTos]`), the effect runs on mount AND every time `googleAdsSendTos` changes. Same code, different trigger pattern.

Either way, **mount is when it first runs.** No mount, no effect, no side effect, nothing happens.

### Mounting in tests

A unit test for a component has to actually trigger the mount, or the `useEffect` never fires and there's nothing to assert against:

```tsx
import { render } from '@testing-library/react';

// Before this line: the component is just a function definition. Nothing has run.
render(<CheckoutModal googleAdsSendTos={{ purchase: ['AW-123'] }} />);
// After this line: the component has mounted, the init useEffect has run,
// and any side effects (store updates, API calls, console logs) have happened.

expect(checkoutStore.setGoogleAdsSendTos).toHaveBeenCalledWith({
  purchase: ['AW-123'],
});
```

`render()` from `@testing-library/react` is doing the mount: it creates a virtual DOM (jsdom), inserts the component, and runs the full mount lifecycle synchronously. After it returns, the effect has already fired.

### Why mounting is sometimes expensive in tests

The component you want to mount usually doesn't stand alone — it expects:

- A router context (`<BrowserRouter>`)
- An i18n provider (`<IntlProvider>`)
- A MobX store provider
- Mocked gateways for any API calls the mount triggers

Setting all that up so the component can mount without crashing is "test infrastructure," and it's the expensive part. Once it exists, every additional test is cheap. (See [Unit tests](unit-tests.md) — the "first test is expensive" rule.)

## When does this matter?

- **Reading any `useEffect`.** First question: when does it run? Mount-only (`[]`), on every render (no deps), or when specific deps change (`[a, b]`)?
- **Bugs in init code.** If a side effect "isn't happening," check whether the mount actually ran. A component that's never rendered never mounts, so its init never executes.
- **Writing tests for init logic.** You can't unit-test "what happens on mount" without rendering the component — there's no shortcut.
- **Strict mode in dev.** React 18+ deliberately mounts → unmounts → mounts again in development to catch effect cleanup bugs. Don't be surprised if your init `useEffect` appears to run twice locally; it won't in production.

## See also

- [What's a prop?](react-props.md) — props are the inputs to the function that gets mounted
- [Unit tests](unit-tests.md) — `render()` is how you trigger a mount in a test
- [Prop drilling](prop-drilling.md) — sometimes the mount of one component triggers a cascade of child mounts
- [React docs — Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
