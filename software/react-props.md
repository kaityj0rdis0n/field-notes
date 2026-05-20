---
title: What's a prop? (React)
category: software
date: 2026-05-19
tags: [react, javascript, components, fundamentals]
---

# What's a prop? (React)

## TL;DR

A **prop** ("property") is how a parent React component hands data down to a child. It's a function argument, but for components. The component declares what shape of props it accepts (its TypeScript interface), and whoever renders it has to pass that shape in.

## The explanation

A React function component literally IS a function — and props are its arguments. Math-style:

```
Math:     F(x, y) = ...
React:    CheckoutModal({ host, googleAdsSendTos }) = ...
```

```tsx
// Declaration — defining what the function expects
interface CheckoutModalProps {
  host: User;
  googleAdsSendTos: GoogleAdsSendTos;
}

function CheckoutModal({ host, googleAdsSendTos }: CheckoutModalProps) {
  return <div>...</div>
}
```

```tsx
// Usage — passing the args in
<CheckoutModal
  host={someHost}
  googleAdsSendTos={someConfig}
/>
```

The JSX `<Component prop={value} />` syntax is just sugar for `Component({ prop: value })`. React conventionally groups all the arguments into a single object so the call site reads like HTML attributes.

Inside the component, props are usually destructured at the top so you can use them as regular variables:

```tsx
function CheckoutModal({ host, googleAdsSendTos }) {
  // now `host` and `googleAdsSendTos` are local variables
}
```

The TypeScript interface (e.g. `CheckoutModalProps`) is the component's **signature** — what it accepts. Like declaring `F: (x: number, y: number) => number` in math.

## When does this matter?

- **Reading any React component.** First place to look: what props does it accept? That's the contract.
- **Passing data to a child.** You name the prop in JSX, the child reads it from its destructured arguments.
- **TypeScript will yell at you** if the parent passes the wrong shape — that's the prop interface doing its job.
- **Optional vs required props** — `googleAdsSendTos?: GoogleAdsSendTos` (with `?`) means the parent can omit it; without `?`, TypeScript requires it.

## See also

- [Function signatures](function-signatures.md) — the broader concept: what a function's input/output type tells you
- [Prop drilling](prop-drilling.md) — what happens when you pass a prop through many layers to reach a deep component
- [React docs — Passing props to a component](https://react.dev/learn/passing-props-to-a-component)
