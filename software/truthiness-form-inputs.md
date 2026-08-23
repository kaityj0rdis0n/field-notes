---
title: "Truthiness and form inputs: the string \"0\" is truthy"
category: software
date: 2026-08-23
tags: [javascript, ruby, validation, forms, type-coercion]
---

# Truthiness and form inputs: the string "0" is truthy

## TL;DR

Form inputs always deliver strings, and the only falsy string is the empty one. A truthiness check on a raw input silently accepts "0". Parse before you compare.

## The explanation

JavaScript has exactly seven falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. Everything else is truthy. That includes every non-empty string: `"0"` is truthy, and so is `"false"`.

Form inputs make this dangerous, because `event.target.value` is always a string, no matter what the user typed and no matter what `type` the input has. So this common emptiness check has a hole in it:

```js
if (!discountValue) {
  showError('Value is required');
}
```

It reads as: "if the value is falsy, complain." Walk two values through it. The user leaves the field empty: `discountValue` is `""`, which is falsy, so the error shows. Correct. The user types 0: `discountValue` is the string `"0"`, which is truthy, so no error, and a zero-value discount sails through to the database. The check was written to mean "is this field empty?" but what it actually asks is "is this string empty?", and those are only the same question until someone types a zero.

The fix is to parse once, at the boundary, and compare numbers with numbers:

```js
const amount = parseFloat(discountValue);

if (discountValue === '' || isNaN(amount)) {
  showError('Value is required');
}
else if (amount <= 0) {
  showError('Value must be greater than zero');
}
```

Note the order: the emptiness check comes first, because `parseFloat('')` is `NaN`, and `NaN <= 0` is false. Each check can then assume the previous one already handled its case: by the time you compare against zero, you know you have a real number.

## Same bug, two languages

The exact same defect class behaves completely differently depending on the language's attitude to mixed-type comparison.

JavaScript quietly gives you an answer. `"0" > 100` coerces the string and returns false; `!"0"` returns false; nothing ever errors. In a real codebase, a validator built on these checks passed the string "0" for eight years without anyone noticing.

Ruby refuses. The equivalent guard, `'0.13' <= 0`, raises `ArgumentError: comparison of String with 0 failed` the first time it runs. The same mistake, written the same week, lasted three minutes in CI.

The lesson is not that one language is safer. It's that comparing across a type boundary is the defect, and the language only decides when you find out: at the review table, in CI, or eight years later in production.

## Habits and when this bites

Parse at the boundary, once, and let everything inward trust that it has a number. Never use truthiness to mean "empty" on a value that can legitimately be zero: amounts, quantities, percentages, indexes. And when you write a test for numeric validation, feed it the *string* form of the value, because that's what the form actually delivers; a test that passes the number would have passed against the broken code too.

## See also

- [Presence is not validity](./presence-is-not-validity.md)
- [Validation has to live at every door](./validation-at-every-door.md)
- [The ternary operator](./ternary-operator.md)
