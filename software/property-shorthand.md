---
title: Property shorthand & exact-match lookup (JS)
category: software
date: 2026-05-15
tags: [javascript, objects, debugging]
---

# Property shorthand & exact-match lookup (JS)

## TL;DR

In JavaScript, `{ apiKey }` is shorthand for `{ apiKey: apiKey }` — same value, but the property *name* is locked to the variable name. When the receiver expects a different name, drop the shorthand and use explicit `key: value` syntax. And remember: JS property lookup is exact-string-match — `obj.lang` doesn't find a property called `locale`, even if they "mean" the same thing.

## The explanation

Two related ideas that bite together.

**Shorthand**: when the key and the variable have the same name, you can write the variable once.

```js
const apiKey = 'abc123';

const config = { apiKey };
// same as:
const config = { apiKey: apiKey };
```

This is great for readability — *until* the receiver wants a different name. If the API expects the field to be called `api_key` (snake_case), you can't just keep the shorthand:

```js
sendRequest({ apiKey });           // sends { "apiKey": "abc123" } — wrong
sendRequest({ api_key: apiKey });  // sends { "api_key": "abc123" } — right
```

**Exact-match lookup**: when code does `obj.something`, it asks for a property literally named `something`. No fuzzy match.

```js
const pkg = { lang: 'fr' };
pkg.locale;   // undefined — no property called "locale"
pkg.lang;     // 'fr'
```

This is the post-office mailbox model. Your package can be addressed to "John" or "Jonathan" — they look related, but the mailbox slot is labeled exactly one of those. The mail carrier doesn't unify them.

## When does this matter?

- **API contracts.** Receiver expects `{ user_id }`, your variable is `userId` — shorthand won't help, you need `{ user_id: userId }`.
- **Cross-system bugs that look like null/undefined.** "Why is `obj.locale` undefined? I set it!" Often: you set `obj.lang`. The names look interchangeable to humans, but the runtime sees them as totally different.
- **Renaming a field while keeping the value.** When the variable name is right but the property name needs to change, drop the shorthand and use explicit `key: value`.
- **Debugging a serialization/IPC layer.** Whenever data crosses a boundary (web ↔ iframe, parent ↔ child component, JSON over the wire), the receiving side reads property names by exact string. Mismatched names = silent data loss.

## See also

- [Function interception for debugging](function-interception.md) — useful when you suspect a call site is passing the wrong field name
- [Contract mismatch — fix the caller, not the receiver](contract-mismatch-fix-the-caller.md) — the higher-level pattern this often shows up inside of
- [MDN — Object initializer (shorthand property names)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#property_definitions)
