---
title: Function interception for debugging
category: software
date: 2026-05-15
tags: [javascript, debugging, browser-devtools]
---

# Function interception for debugging

## TL;DR

When you need to know what arguments a function is being called with — and you can't easily find or modify the caller — replace the function with a wrapper that logs and then calls the original. Especially useful for global functions or third-party scripts where the caller is in a minified bundle.

## The explanation

The core move is one pattern:

```js
const original = someGlobal.doThing;
someGlobal.doThing = function (...args) {
  console.log('doThing called with:', JSON.stringify(args));
  return original.apply(someGlobal, args);
};
```

Three things happen here:

1. **Save the original.** You're going to replace it; keep a reference so you can call it normally.
2. **Replace it with a wrapper that logs.** Use a rest parameter so the wrapper works regardless of how many arguments the caller passes.
3. **Call the original via `.apply()`** so `this` and the argument list are preserved exactly. Behavior of the system is identical from the outside; you just get a console message every time.

You install this in the **DevTools console**, *before* triggering the action you want to inspect. Then you click the thing, or whatever, and watch what gets logged.

Why this is so useful: the function may be called from a script you don't have source access to (or one that's minified into an opaque bundle). You don't need to find the caller. You just need to know what's reaching the function.

## When does this matter?

- **"Is the function I think is being called actually being called?"** If nothing logs when you trigger the action, the caller is going through a different path entirely.
- **"What arguments is it getting?"** If you suspected the caller is passing `{ locale }` and not `{ lang }`, this tells you in one click.
- **Investigating third-party / global APIs** — analytics SDKs, embedded widgets, anything that exposes a `window.foo.bar` function. You don't need the source code.
- **Sanity checks during refactors** — wrap the old function while your new path is parallel-running, to see traffic.

## Caveats

- This is a debugging tool, not production code. Removing it = refresh the page.
- If the function is destructured at call time (`const { open } = $u; open(args)`), your replacement on `$u.open` won't be picked up by code that already grabbed the reference earlier.
- Some build tools / minifiers inline calls so the function reference isn't dynamically looked up. Rare in browser globals, common in bundled internal modules.

## See also

- [Property shorthand & exact-match lookup](property-shorthand.md) — the kind of bug this technique helps you spot
- [Contract mismatch — fix the caller, not the receiver](contract-mismatch-fix-the-caller.md)
