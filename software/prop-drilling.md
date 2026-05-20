---
title: Prop drilling
category: software
date: 2026-05-19
tags: [react, javascript, state-management, architecture]
---

# Prop drilling

## TL;DR

**Prop drilling** is when a piece of data has to be passed through multiple layers of components — as a prop at each level — just to reach a deep component that actually needs it. The intermediate components don't use it themselves; they just relay it. Annoying because it clutters every level's signature and makes data flow hard to trace.

## The explanation

Imagine you want to get a value from a top-level component down to a deeply-nested grandchild:

```
TopLevel              ← data arrives here
  ↓ passes as prop
ContainerA            ← doesn't need it, but has to accept & forward
  ↓ passes as prop
ContainerB            ← doesn't need it, but has to accept & forward
  ↓ passes as prop
ContainerC            ← doesn't need it, but has to accept & forward
  ↓ passes as prop
DeepChild             ← finally uses it
```

Each intermediate component has to:

1. Add the prop to its TypeScript interface (even though it doesn't *care*)
2. Pass it down to its child
3. Get re-edited every time the data shape changes

That's the "drilling" — the prop has to bore its way through every layer, like a drill going through floors.

### Why it's annoying

- Intermediate components' signatures get cluttered with props they don't actually use
- Renames cascade — touch one level, touch them all
- Reading the code, you can't tell where the data originates without tracing every hop
- New developers see the prop in middle layers and wonder why it's there

### Common solutions

- **React Context** — built-in. Put data "in the air" at a high level, descendants pull it down directly.
- **State management library** (Redux, MobX, Zustand) — same idea but with more features.
- **Composition** — pass JSX children instead of plumbing data, letting the consumer access the data scope directly.

The point of all of these: avoid threading data through layers that don't care about it.

## When does this matter?

- Any time you find yourself adding a prop to a component "just to pass it down" — the component is acting as a courier, not a consumer.
- When component trees deepen (a redesign adds wrapper components) — what was 2 hops becomes 5, drilling gets uglier.
- When a single piece of data is needed in many parts of the tree (locale, current user, theme) — drilling each one separately is exhausting; this is where Context or a store shines.
- When you see a prop in a TypeScript interface and can't tell whether the component uses it — it might be drilled.

### When *not* to drill-avoid

- One or two hops? Drilling is fine. It's the simplest and most explicit pattern.
- Don't reach for a global store just because you don't want to add one prop to one component. That's premature complexity.

## See also

- [What's a prop?](react-props.md) — props are the thing that gets drilled
- [React docs — Passing data deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
