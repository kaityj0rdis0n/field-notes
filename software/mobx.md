---
title: MobX (state management)
category: software
date: 2026-05-20
tags: [react, state, mobx, pub-sub, fundamentals]
---

# MobX (state management)

## TL;DR

**MobX is a state management library that makes plain JavaScript objects "observable"** — when their values change, any component that read from them automatically re-renders. It works through three pieces: **observable state** (the data), **actions** (functions that change it), and **observers** (components that react to changes). The magic is that MobX figures out who's subscribed to what by quietly watching what your code reads — no explicit subscribe calls.

## The explanation

### The problem MobX solves

A plain JavaScript [store](stores-shared-state.md) — a shared object with some properties — has one big gap: when you change a value, **nothing else knows**. The store is updated, but the UI doesn't refresh. You need a way to broadcast "this value changed" to everything that cares.

You could write that broadcast system by hand using [pub/sub](pub-sub.md) — every consumer calls `subscribe()` to register, every setter calls `notify()` to broadcast. It works, but it's a lot of plumbing for every value in every store. MobX does it for you.

### The three pieces

#### 1. Observable state — the data that broadcasts changes

```ts
import { makeAutoObservable } from 'mobx';

class CheckoutStore {
  googleAdsSendTos?: GoogleAdsSendTos;
  orderId?: string;
  buyerEmail?: string;

  constructor() {
    makeAutoObservable(this);   // ← this is the magic line
  }
}
```

`makeAutoObservable(this)` tells MobX: "watch every property on this object. Any time someone reads one, remember who's reading. Any time someone writes one, notify everyone who was reading."

After that line runs, `checkoutStore.googleAdsSendTos` isn't a plain property anymore. It's an **observable** — MobX has wrapped it in tracking machinery.

There's also a more explicit form, `makeObservable(this, { ... })`, where you declare which properties are observable and which methods are actions:

```ts
makeObservable(this, {
  googleAdsSendTos: observable,
  orderId: observable,
  setGoogleAdsSendTos: action,
  setOrderId: action,
});
```

`makeAutoObservable` is shorthand — MobX inspects the class and figures out which is which.

#### 2. Actions — the legitimate mutators

```ts
import { action } from 'mobx';

class CheckoutStore {
  // ...

  setGoogleAdsSendTos = (value) => {
    this.googleAdsSendTos = value;
  };
}
```

The setter methods are **actions**. MobX wants every state change to happen inside an action because:

- **Batching**: multiple changes inside one action trigger one re-render, not one per line. Performance.
- **Intent**: actions are the API. Random assignments anywhere in the codebase are an anti-pattern — they make state changes hard to grep for.
- **Dev warnings**: MobX shouts in development if you mutate observable state outside an action.

If you used `makeAutoObservable`, MobX figures out actions automatically. If you used `makeObservable`, you declare them explicitly (as in the snippet above).

#### 3. Observers — the components that react

```tsx
import { observer } from 'mobx-react';

export const Confirmation = observer(() => {
  // Just read from the store. No subscribe call anywhere.
  if (checkoutStore.googleAdsSendTos?.purchase) {
    googleAds.send(...);
  }

  return <div>Order confirmed</div>;
});
```

Wrapping a component in `observer(...)` makes it react to MobX observables. **During the component's render, MobX silently records every observable it reads.** When any of those observables changes later, MobX automatically re-renders this component.

You write the component as if you're just reading variables. MobX does the wiring.

### How the magic actually works

The "automatic" part of MobX is two pieces of read-time tracking:

```
RENDER PHASE
  Confirmation runs its render function
    Reads checkoutStore.googleAdsSendTos
      MobX intercepts the read → records:
        "Confirmation is observing googleAdsSendTos"
    Reads checkoutStore.orderId
      MobX intercepts the read → records:
        "Confirmation is also observing orderId"

LATER, SOMEWHERE ELSE
  CheckoutModal calls checkoutStore.setGoogleAdsSendTos(value)
    MobX intercepts the write → looks up:
      "Who's observing googleAdsSendTos?"
        Answer: Confirmation
    MobX triggers Confirmation's re-render
```

Read-time subscription and write-time notification. No `subscribe()` / `unsubscribe()` in any of your component code — MobX inserts that machinery via the `observer()` wrapper and the `makeAutoObservable()` constructor call.

### Putting it together — your UNI-7451 bug

```
1. CheckoutModal mounts → calls setGoogleAdsSendTos(value)        ← action
                                  ↓
2. checkoutStore.googleAdsSendTos slot changes                     ← observable
                                  ↓
3. MobX checks: who's observing this slot?                         ← MobX runtime
                                  ↓
4. "Confirmation reads it" → trigger re-render of Confirmation     ← observer
                                  ↓
5. Confirmation re-runs; this time `checkoutStore.googleAdsSendTos?.purchase`
   is defined, so googleAds.send(...) fires
```

Before the fix, step 1 didn't happen in the modal flow (the setter call was missing). So step 2 didn't happen. So step 3 had nothing to find. So step 4 never fired. So step 5 never ran — and the purchase event silently didn't send.

### MobX vs. alternatives

| Library | Mental model | Boilerplate | When to reach for it |
|---|---|---|---|
| **MobX** | "Observable objects auto-notify everyone who read them" | Low | Class-based stores, lots of interdependent state, you want the tracking to feel invisible |
| **Redux** | "Single big state object, dispatch actions, reducers return new state" | High | Strict audit trail of every state change matters; large team with strong conventions |
| **Zustand** | "Tiny hooks-based store, you opt-in to subscriptions" | Very low | Simple apps, function-style stores, hooks-native |
| **React Context** | Built in to React; provider wraps the tree, every consumer re-renders on any change | Low for setup, can be high for performance | Theme, current user, feature flags — values that change rarely |
| **TanStack Query** / **SWR** | State management for *server* data specifically — cache + revalidation | Low | Anything fetched from an API; not really "global app state" |

MobX trades some explicitness for ergonomics. You don't see subscriptions in your code; MobX does them for you. The downside: a value can change for "no apparent reason" and tracing why takes you into MobX-land. The upside: you write code like you'd write it without any state management, and reactivity Just Works.

### Why the checkout repo uses MobX

You can see all three pieces in [src/stores/checkoutStore.ts](src/stores/checkoutStore.ts):

- `import { action, makeAutoObservable } from 'mobx';`
- Observable properties (`googleAdsSendTos`, `orderId`, etc.)
- Setter methods marked `action`
- `makePersistable(this, {...})` from `mobx-persist-store` — a sister library that auto-saves observable state to `localStorage`, so a buyer who refreshes mid-checkout gets their state back

The persistence is a big tell: this is state that has to outlive a page load. You couldn't do that easily with props or Context. The store + localStorage combo gives the checkout flow durability through refreshes, navigation, and even brief tab-close-and-reopen.

### Mental model

MobX is **read-time subscription, write-time notification, both invisible**:

- **Observable** = a slot in the store that broadcasts when it changes
- **Action** = a function that's allowed to change a slot
- **Observer** = a component that auto-subscribes to whatever slots it reads

If a component isn't wrapped in `observer(...)`, it will *read* the right value at render time but won't re-render when the value changes later. That's a common silent bug.

## When does this matter?

- **Reading any MobX-based codebase.** Find the store class (search for `makeAutoObservable` or `makeObservable`). That tells you what's observable. Then `observer(` tells you which components react.
- **Adding new state to a store.** Add the property + the setter. If the class uses `makeObservable` (not Auto), don't forget to add the property to the observable map and the setter to the action map. Easy to miss; MobX won't notify if you forget.
- **A component "isn't updating."** First check: is it wrapped in `observer(...)`? An unwrapped component reads the right value the first time and then never updates again, even as the store changes.
- **A value "isn't firing."** First check: is the setter actually being called? MobX can only notify observers of changes that go through the setter. (This was the UNI-7451 bug — the setter was never called in the modal flow.)
- **Debugging unexpected re-renders.** Components observe whatever they read during render. If a component reads three observables, it'll re-render when *any* of them change. To re-render less, read fewer observables or split the component.
- **Persistence.** `mobx-persist-store` (or similar) hooks into observable changes and writes them to storage. If you add a new field that needs to survive refresh, add it to the persist config too — easy to miss.

## See also

- [Stores (shared state)](stores-shared-state.md) — what a "store" is without the reactivity layer
- [Pub/sub](pub-sub.md) — the pattern MobX implements invisibly
- [Prop drilling](prop-drilling.md) — the problem stores (with MobX or otherwise) solve
- [Component mounting](component-mounting.md) — `useEffect([])` is typically where store init writes happen
- [MobX docs — Concepts & Principles](https://mobx.js.org/the-gist-of-mobx.html)
