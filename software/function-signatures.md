---
title: Function signatures
category: software
date: 2026-05-19
tags: [programming, typescript, fundamentals, interfaces]
---

# Function signatures

## TL;DR

A function's **signature** is the description of what it accepts and what it returns — just the shape, not the body. Signatures tell you how to *use* something without making you read the implementation. Same idea as declaring `f: ℝ → ℝ` in math.

## The explanation

In math:

```
f: ℝ → ℝ
```

That tells you "f takes a real number, returns a real number." It doesn't tell you whether f doubles, squares, or anything else. The mapping (signature) is separate from the rule (body).

In TypeScript:

```ts
(x: number, y: number) => number
```

Same thing — "this function accepts two numbers and returns a number." The body could be `x + y`, `x * y`, anything. The signature stays stable while the body can change.

For a React component:

```ts
CheckoutModal: (props: CheckoutModalProps) => React.ReactElement
```

"Takes a `CheckoutModalProps` object, returns a React element." That's the signature. The 400+ lines inside the component are the body.

## When does this matter?

- **Calling something you didn't write** — the signature is what you read. The body is implementation detail you can usually ignore.
- **Reviewing a PR** — signature changes are a bigger deal than body changes. A signature change is a contract change; a body change is "how it gets done."
- **Refactoring** — the goal is often to change the body while keeping the signature stable. Callers don't break.
- **Designing APIs** — the signature is what other developers see first. Make it readable, predictable, hard to misuse.
- **TypeScript errors that look scary** — usually they're the compiler telling you the signature you're calling doesn't match what you're passing. Read the signature, fix the call, ignore the rest.

## See also

- [What's a prop?](react-props.md) — the prop type is part of a component's signature
- [Contract mismatch — fix the caller, not the receiver](contract-mismatch-fix-the-caller.md) — what happens when the caller doesn't respect the signature
