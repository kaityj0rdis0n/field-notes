---
title: The string "0" is truthy (parse before you compare)
category: software
date: 2026-08-24
tags: [javascript, ruby, validation, forms, type-coercion]
---

# The string "0" is truthy (parse before you compare)

## TL;DR

Form inputs deliver strings, and the only falsy string is the empty one. An emptiness check like `!value` happily passes `"0"`. Parse the value into a number first, then compare.

## The explanation

JavaScript has exactly eight falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. Everything else is truthy, including the strings `"0"` and `"false"`.

That matters because HTML form inputs always deliver strings. A user typing 0 into a number field gives you `"0"`, not `0`. So this common validation pattern has a hole in it:

```js
if (!donationAmount) {
  return 'Please enter an amount';
}
```

Walk `"0"` through it: `!"0"` asks "is this string falsy?" It has a character in it, so it's truthy, and the check passes. A zero-dollar donation sails through a validator whose entire job was to catch missing amounts. If the same variable had held the *number* `0`, the check would have caught it, which makes the bug worse: it appears and disappears depending on where the value came from.

The fix is to parse before comparing:

```js
const amount = parseFloat(donationAmount);
if (donationAmount === '' || isNaN(amount) || amount <= 0) {
  return 'Please enter an amount greater than zero';
}
```

Once `amount` is a real number, the comparisons mean what they say. Note the order: the blank check comes first, because `parseFloat('')` is `NaN` and `NaN <= 0` is false, so an empty input would slip past the zero check alone.

## Same bug, two languages

Ruby has the mirror-image version. Comparing a numeric string with a number doesn't silently coerce like JavaScript; it raises:

```ruby
'0.13' <= 0
# ArgumentError: comparison of String with 0 failed
```

I hit both versions of this in the same week, in the same fix. The JavaScript form of the bug (comparing a raw form string) had lived in a codebase for eight years, because JS coerces silently and the code merely gave a wrong answer. My Ruby fix for it made the same class of mistake and lived for three minutes, because Ruby refused the comparison and crashed in CI.

Same defect both times: a loose comparison across a type boundary. The language only decides whether you find out loudly or quietly. The fix is also the same in both: parse first (`parseFloat` in JS, `.to_f` in Ruby), then compare numbers with numbers.

## Habits and when it bites

Any time a value crosses a boundary (a form, a CSV, an env var, a URL param, an API payload you didn't type-check), assume it's a string until you've parsed it. Validate the parsed value, not the raw one, because the parsed value is what your code will actually store or act on. And if you're writing an emptiness check with `!value`, ask whether zero is a value your users can legitimately type; if it is, truthiness is the wrong tool.

## See also

- [The ternary operator (one-line if/else)](./ternary-operator.md)
- [MDN: Falsy values](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)
