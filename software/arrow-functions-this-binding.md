---
title: Arrow functions and `this` binding — why they matter in classes
category: software
date: 2026-05-21
tags: [javascript, typescript, classes, this, arrow-functions]
---

# Arrow functions and `this` binding — why they matter in classes

## TL;DR

In JavaScript, `this` inside a regular method depends on *how the method is called*, not where it's defined. Arrow functions fix `this` to wherever they were created. In a class, that means: regular methods need `.bind(this)` when passed as callbacks; arrow function class fields don't. Getting this wrong is a silent runtime bug — the function runs but `this` is `undefined` or the wrong object.

## The explanation

### The problem with `this` in regular methods

```ts
class Counter {
  count = 0;

  increment() {
    this.count++;
    console.log(this.count);
  }
}

const c = new Counter();
c.increment();              // logs 1 ✅ — called on c, so this === c

const fn = c.increment;
fn();                       // TypeError: Cannot read properties of undefined ❌
                            // called bare, this === undefined (strict mode)
```

When you pass a method as a callback — like `button.addEventListener('click', c.increment)` — it gets called bare, without the object it came from. `this` breaks.

### `.bind(this)` — the manual fix

`.bind(this)` returns a new version of the function with `this` permanently locked:

```ts
button.addEventListener('click', c.increment.bind(c));
//                                            ^^^^^^
//                             creates a new function where this === c always
```

It works, but it's boilerplate. You have to remember it for every method you pass as a callback.

### Arrow functions — the built-in fix

Arrow functions capture `this` from the surrounding context when they're *created*, not when they're called. As a class field (declared with `=`), they're created once per instance with `this` already locked in:

```ts
class Counter {
  count = 0;

  // Arrow function as a class field — this is always the Counter instance
  increment = (): void => {
    this.count++;
    console.log(this.count);
  };
}

const c = new Counter();
const fn = c.increment;
fn();   // logs 1 ✅ — this is always c, no matter how it's called
```

Now you can pass `c.increment` anywhere — as a callback, stored in a variable, passed to a timer — and `this` is always right. No `.bind(this)` needed.

### Comparison

| | Regular method | Arrow function class field |
|---|---|---|
| `this` when called on object (`c.method()`) | ✅ correct | ✅ correct |
| `this` when passed as callback | ❌ breaks | ✅ correct |
| Needs `.bind(this)` | Yes | No |
| Created once (shared on prototype) | Yes | No — one copy per instance |
| Can be overridden by subclass | Yes | Harder |

### The inconsistency to watch for

A class where most handlers are arrow functions but one is a regular method is a subtle trap. The regular method *looks* complete but will silently break `this` if it's ever passed as a callback:

```ts
class WidgetBridge {
  onPurchase(data: unknown): void {   // regular method — this depends on caller ⚠️
    this.notify(data);
  }

  onStepChange = (data: unknown): void => {   // arrow — this always works ✅
    this.notify(data);
  };
}
```

If both are registered as callbacks, `onPurchase` needs `.bind(this)`; `onStepChange` doesn't. Mixing the two styles in the same class without being deliberate about it is a common source of subtle bugs.

### When to use each

- **Arrow function class fields** — for methods you'll pass as callbacks (event handlers, `setTimeout`, `emitter.on`). Standard choice in React components and event-driven classes.
- **Regular methods** — for methods only ever called directly on the object (`c.method()`). Shared on the prototype, slightly more memory-efficient when you have many instances.
- **`.bind(this)`** — fine but verbose; arrow function fields are cleaner for the same purpose.

## When does this matter?

- You're adding an event handler to a class and it's not firing — check if `this` is broken
- You see `.bind(this)` in a constructor and wonder why it's there
- You're reviewing code that mixes arrow functions and regular methods in the same class — make sure the regular ones aren't being passed as callbacks
- You hit "cannot read properties of undefined" inside a callback — `this` is usually the culprit

## See also

- [Pub/sub (publish/subscribe)](pub-sub.md) — the pattern where `this` binding matters most
- [The silent registration gap](silent-registration-gap.md) — related bug where the method exists but is never registered
- [MDN — Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN — `this`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
