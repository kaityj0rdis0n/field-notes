---
title: The ternary operator (one-line if/else)
category: software
date: 2026-08-23
tags: [javascript, operators, conditionals, readability]
---

# The ternary operator (one-line if/else)

## TL;DR

A one-line if/else that produces a value. Picking a value: ternary. Doing things: if/else.

## The explanation

The name "ternary" just means three-part (condition, true-branch, false-branch), the same way "binary" means two-part. It's the only three-part operator in JavaScript, so people say "the ternary."

```js
const shippingCost = orderTotal >= 50 ? 0 : 6.99;
```

It reads as: **condition `?` value-if-true `:` value-if-false**. "Is the order total at least 50? If yes, shipping is 0. If no, shipping is 6.99."

It's exactly equivalent to:

```js
let shippingCost;
if (orderTotal >= 50) {
  shippingCost = 0;
} else {
  shippingCost = 6.99;
}
```

## Habits and when to use it

Ternaries are great for picking between two values, like the shipping line above. They get unreadable when people nest them or hide side effects in the branches; at that point the honest if/else is better. Picking a value: ternary. Doing things: if/else.

## See also

- [Arrow functions and `this` binding](./arrow-functions-this-binding.md)
- [MDN: Conditional (ternary) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator)
