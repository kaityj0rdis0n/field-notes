---
title: Shared-state population trap
category: software
date: 2026-05-20
tags: [state, architecture, bugs, debugging, fundamentals]
---

# Shared-state population trap

## TL;DR

When two or more flows share a piece of state through a [store](stores-shared-state.md), **every flow has its own obligation to populate it**. Miss the population step in one flow and only that flow silently breaks — the value reads as `undefined`, any null guard in the consumer skips silently, and the bug is invisible to error monitoring. The architecture is fine; what's broken is that one entry point forgot to do its share of the setup.

## The explanation

### The setup

You have shared state. Maybe a MobX store, maybe a Redux slice, maybe a plain singleton object. Multiple flows in your app need to write to it, and some downstream component needs to read from it.

```
                  ┌────────────────────────┐
   Flow A  ──→    │                        │   ──→  Confirmation
                  │       sharedStore      │        (reads value)
   Flow B  ──→    │   .setValue(v)         │
                  │                        │
   Flow C  ──→    └────────────────────────┘
```

Each entry flow is supposed to call `sharedStore.setValue(...)` somewhere in its initialization — typically inside a mount-time `useEffect`. The consumer reads the value from the store and acts on it.

This is a good architecture for the right problem: it sidesteps [prop drilling](prop-drilling.md), it scales to N flows without adding props to intermediate components, and it lets the consumer be at any depth in the tree.

### What goes wrong

The architecture has one weak point: the population step is the responsibility of each entry path, separately. There's no compile-time check that flows A, B, and C *all* populate the store. They each have their own `useEffect` (or `onMount`, or whatever), each containing its own list of setters, maintained by different people at different times.

So when a new property is added to the store, the developer adding it has to remember to add the setter call in every entry path. If they only add it in flow A, then flows B and C will silently read `undefined` for that property.

```
Flow A's init:   sharedStore.setValue(v);         ← happy path

Flow B's init:   // ❌ setter was forgotten here

Flow C's init:   sharedStore.setValue(v);         ← happy path
```

In flow B:

```ts
// Consumer code, runs after the user reaches Confirmation
if (sharedStore.value?.somePath) {     // ← undefined, guard returns false
  doTheThing();                         // ← never runs, no error logged
}
```

The consumer does the right thing — it null-guards the value. So when the value is `undefined`, nothing crashes. The function that was supposed to fire just… doesn't. No exception, no error in Sentry, no failed network request. The feature silently doesn't work, and only the user (or the host, or the buyer) notices it's broken.

### The real-world example: UNI-7451

Universe's checkout has two entry flows that share `checkoutStore`:

- **Calendar flow** — buyer hits a calendar widget on a listing page → picks a date → ticket selection happens.
- **Modal flow** — buyer hits a "Buy Tickets" button → a modal opens with ticket selection inside.

Both flows have an init `useEffect` that writes a bunch of values into `checkoutStore` so `Confirmation.tsx` can read them later. One of those values is `googleAdsSendTos` — the host's Google Ads conversion config.

The calendar flow's init had:

```ts
checkoutStore.setGoogleAdsSendTos(googleAdsSendTos);
```

The modal flow's init didn't.

So in the modal flow, `Confirmation.tsx` ran this check:

```ts
if (checkoutStore.googleAdsSendTos?.purchase) {
  checkoutStore.googleAdsSendTos.purchase.forEach((conversionId) => {
    googleAds.send(conversionId, purchaseData);
  });
}
```

…and the guard returned false (because `googleAdsSendTos` was `undefined`), and `googleAds.send` never fired. Conversion events for every modal-flow purchase silently went missing — only the host noticed, weeks later, when they checked Google Tag Assistant.

### Why this is hard to catch

- **No exception.** The consumer's null guard means there's no thrown error to land in Sentry.
- **No failed network request.** It's an event-not-sent bug. The thing that didn't happen leaves no trace.
- **The architecture looks correct.** The store has the field. The setter exists. The consumer reads it correctly. Each piece in isolation is fine.
- **Other reads of the same value still work.** In UNI-7451, page-view and widget-init Google Ads events still fired correctly — they read `googleAdsSendTos` from props directly, not from the store. So most monitoring would say "Google Ads is working." Only the specific purchase path was broken.
- **The wiring is in code, not types.** TypeScript can't enforce that flow B's init useEffect calls every setter, because the setter calls are statements, not part of a type contract.

### Why the architecture still wins

Despite the trap, this design is usually still the right call when:

- The consumer is many layers below the entry path (prop drilling would touch 4+ files)
- Multiple unrelated flows need to populate the same downstream behavior
- The value has to survive page reloads (the store can be persisted to localStorage)

The alternative — drilling the prop through every layer — would have its own bugs and would be invasive to add new fields. The "asymmetric population obligation" is the cost you pay for the indirection.

### How to mitigate it

The trap isn't fully avoidable in dynamic languages, but you can reduce the surface area:

**1. Centralize the population list.** Instead of each flow having its own list of `set...` calls, write a `populateCheckoutStore(props)` helper that takes everything as an argument and does all the writes. Every flow calls the helper. Now there's exactly one place to add a new field, and TypeScript can require the new field be in the helper's argument shape.

```ts
function populateCheckoutStore(props: PopulateProps) {
  checkoutStore.setOrderLevelFee(props.orderLevelFee);
  checkoutStore.setHost(props.host);
  checkoutStore.setFacebookPixels(props.facebookPixels);
  checkoutStore.setGoogleAdsSendTos(props.googleAdsSendTos);  // ← add here once
  checkoutStore.setTiktokApiTokens(props.tiktokPixels.map(...));
}
```

Now flow A and flow B both call `populateCheckoutStore({...})`, and the only place to keep in sync is the helper.

**2. Lint or grep for symmetry.** Have a test (or a custom ESLint rule, or even just a CI grep) that asserts every setter declared in the store class is called in every entry path. Brittle but catches forgotten wires.

**3. Required init function.** Make the store throw if you read a value before it was set. Doesn't help if the consumer null-guards, but does help if the consumer assumes the value exists.

**4. Smoke tests for every entry path.** Write a test that mounts each entry path's root component and asserts that every store field that *should* be populated is populated. End-to-end tests can also catch this if they exercise both flows.

**5. Documentation.** If you can't enforce it in code, document it as a constraint in the repo's CLAUDE.md or README. The Universe checkout repo does exactly this — there's a "Known Issues & Gotchas" section that explicitly flags the modal-vs-calendar setter mismatch as a recurring bug pattern. Future you greps for the error symptom and finds the pattern documented.

### The general lesson

Any time shared state has more than one entry path, you have an **asymmetric population obligation**: each entry path has to do its share, independently, and there's no central enforcement that they all do it. The bug shape is always the same — value reads as `undefined`, the guard skips silently, the behavior just doesn't happen.

When you see this shape in unfamiliar code, the suspect lines are the init useEffects (or onMount handlers) of every entry path. Grep for the setter name; whichever entry path doesn't call it is the broken one.

## When does this matter?

- **Debugging "feature X works on flow A but not on flow B."** First check: does flow B's init populate the store with everything flow A's init does? Diff the setter lists.
- **Adding a new field to a store.** Don't just add the property and the setter — also add the setter call to every entry path. Easy to forget; very common bug.
- **Reviewing a PR that adds a store field.** Check that the setter is called from every flow that needs the value. The reviewer is often in a better position to catch this than the author, who's focused on the one flow they're working on.
- **Reading a "silently not firing" bug report.** The shape of the bug — no error, just an event not happening, only on one flow — is a strong signal that this trap is the cause.
- **Designing shared state from scratch.** Decide up front: will you have one helper that populates the store (lower trap risk), or per-flow inline setters (more flexible, higher trap risk)? Trade-off, but worth being deliberate.

## See also

- [Stores (shared state)](stores-shared-state.md) — the architecture that creates the trap
- [Contract mismatch — fix the caller, not the receiver](contract-mismatch-fix-the-caller.md) — the related insight: when downstream code can't see a value, the bug is upstream
- [Prop drilling](prop-drilling.md) — the problem stores solve, which is why we accept the trap
- [Component mounting](component-mounting.md) — the lifecycle moment when these setter calls usually happen
- [MobX](mobx.md) — one specific store technology where this trap commonly shows up
