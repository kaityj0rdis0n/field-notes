---
title: Stores (shared state)
category: software
date: 2026-05-20
tags: [react, state, architecture, fundamentals]
---

# Stores (shared state)

## TL;DR

A **store** is a single shared JavaScript object that holds state any component can read from or write to, without needing the value passed through props. Stores exist to sidestep prop drilling for values that many components — at very different depths in the component tree — need access to. The store is imported everywhere it's needed; there's only ever one instance of it, so everyone is looking at the same data.

## The explanation

### A store is just an object

Strip away the framework noise and a store is literally just a JavaScript object with some properties on it:

```ts
class CheckoutStore {
  googleAdsSendTos?: GoogleAdsSendTos;
  orderId?: string;
  buyerEmail?: string;
  facebookPixels: FacebookPixel[] = [];

  setGoogleAdsSendTos = (value) => {
    this.googleAdsSendTos = value;
  };

  setOrderId = (id) => {
    this.orderId = id;
  };
}

export const checkoutStore = new CheckoutStore();
```

That's it. A class, instantiated once, exported. Every file that imports `checkoutStore` is pointing at the same object in memory — so what you write in one place, you can read in another.

### Why this exists: avoiding prop drilling

Imagine `CheckoutModal` knows `googleAdsSendTos`, and `Confirmation` (five layers deep in the tree) also needs it. Without a store, you'd have to pass it as a prop through every component in between:

```
CheckoutModal (knows the value)
  └── CheckoutLayout (passes it through, doesn't use it)
       └── Steps (passes it through, doesn't use it)
            └── PaymentInfo (passes it through, doesn't use it)
                 └── Confirmation (finally uses it)
```

That's [prop drilling](prop-drilling.md). The middle components don't care about `googleAdsSendTos` but are forced to know about it. Adding a new "intermediate" component means another file to touch. Removing the value means cleaning up four files.

With a store, the value teleports:

```
CheckoutModal — writes:   checkoutStore.setGoogleAdsSendTos(value)
                           ↓
                  (anywhere in the codebase)
                           ↓
Confirmation  — reads:    checkoutStore.googleAdsSendTos
```

No props in between. Confirmation reads directly from the store. Adding intermediate components is free — they don't need to know the value exists.

### Reading and writing

By convention, you don't mutate the store directly:

```ts
// ❌ don't do this even though you could
checkoutStore.googleAdsSendTos = value;

// ✅ use the setter method
checkoutStore.setGoogleAdsSendTos(value);
```

The setter exists for two reasons:
- **Intent**: makes it grep-able. You can find every place that changes the value by searching for `setGoogleAdsSendTos`. Direct assignments are easy to miss.
- **Reactivity hooks**: in libraries like MobX, setter methods are where the library hooks in to notify watchers that the value changed. (More on that below.)

### The reactivity question

A plain class-based store like the one above has a quiet problem: when you change a value, **nothing else knows**. The store is updated, but any component that already rendered with the old value won't re-render to show the new one. The UI is stale until something else (a state change, a prop update) triggers a render.

This is why most "stores" in real applications are paired with a reactivity library:

- **MobX** — wraps the store in observable tracking. Reads at render time auto-subscribe; writes auto-notify. (See [MobX](mobx.md).)
- **Redux** — a single big store with a strict update protocol; components subscribe via `useSelector`.
- **Zustand** — a tiny hooks-based store where components opt into the slices of state they care about.
- **Plain React Context** — built in to React, broader-stroke. Every consumer of a context re-renders when any value in it changes.

The store *is* the shared object. The reactivity library *makes it interactive* — turns "shared state" into "shared state that the UI keeps in sync with."

### Store vs props — which to use?

| Use props when… | Use a store when… |
|---|---|
| Only one or two layers between sender and receiver | The receiver is many layers down |
| Value is specific to a single subtree | Many unrelated subtrees need the same value |
| Value changes often and locally | Value is set once and read in lots of places |
| You want the data flow to be obvious in the JSX | The data flow would clutter every intermediate component |

Props make data flow explicit; stores make it implicit. Both have trade-offs. Stores can be hard to trace ("where did this value come from?") because the answer isn't in the parent's JSX — it's in whichever file last called the setter. Props are easier to trace but harder to scale.

### Mental model

A store is **a whiteboard in a shared room**. Anyone in the building can walk in, write on it, or read what's currently written. No need to hand-deliver a note (prop) from one person to another.

Props are like passing a note from desk to desk: explicit, traceable, but tedious if the recipient is far away.

## When does this matter?

- **Reading code with a store.** First question: "Who writes to this slot, and when?" A grep for `setX` (or whatever the setter is) gives you the entry paths. A grep for `store.x` gives you the consumers.
- **Many unrelated places need the same value.** Tracking IDs, the current user, theme preferences, feature flags — anything that's "ambient" across the app. Drilling all of these through props would be insane.
- **Persistent state across navigation.** Stores often outlive component lifecycles. A modal that closes and reopens still finds its values in the store. Useful for "remember the user's place in the checkout if they refresh."
- **Multiple entry points populating the same shared state.** This is the design that powers the modal vs. calendar checkout flows in Universe. Both flows are *supposed to* populate the store on mount; if one forgets, only that flow silently breaks. (See [Shared-state population trap](shared-state-population-trap.md).)
- **Testing components in isolation.** A component that reads from a store will read `undefined` in a unit test unless you populate the store first. Tests for store-reading components often have a setup step like `checkoutStore.setOrderId('test-id')` before rendering.

## See also

- [Prop drilling](prop-drilling.md) — the problem stores solve
- [MobX](mobx.md) — the library this codebase uses to make stores reactive
- [Pub/sub](pub-sub.md) — the underlying pattern that reactive stores implement
- [Shared-state population trap](shared-state-population-trap.md) — what goes wrong when one entry path forgets to populate the store
- [Component mounting](component-mounting.md) — the lifecycle moment when init code (often: populate the store) runs
