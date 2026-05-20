---
title: Pub/sub (publish/subscribe)
category: software
date: 2026-05-20
tags: [patterns, events, fundamentals, javascript]
---

# Pub/sub (publish/subscribe)

## TL;DR

**Pub/sub (publish/subscribe) is a pattern where one piece of code (the publisher) announces events, and other pieces of code (subscribers) register interest in being notified.** Subscribers explicitly opt in via a `subscribe` call, and opt out via `unsubscribe` — otherwise they keep getting notified. It's how `addEventListener` works, how MobX works under the hood, how WebSockets work, and how most "reactive" systems work. The pattern shows up everywhere once you start looking for it.

## The explanation

### The four terms

- **Publisher** — something that puts out notifications when interesting things happen. (A button, an event bus, a MobX observable, a Slack channel.)
- **Subscriber** — something that wants to know about those notifications. (A click handler, a re-render callback, a user.)
- **Subscribe** — the act of registering as a subscriber. Hands the publisher a callback (or a phone number, or an email address) and says "tell me when something happens."
- **Unsubscribe** — the act of removing yourself from the list so you stop getting notified.

The mental model: a newsletter. You hit "subscribe" → emails start arriving. You hit "unsubscribe" → they stop. The newsletter doesn't know about you until you ask, and stops caring about you when you opt out.

### A concrete example you've already used

`addEventListener` is pub/sub:

```ts
function handleClick() {
  console.log('button clicked!');
}

// SUBSCRIBE — tell the button: "when you publish a 'click', call this function"
button.addEventListener('click', handleClick);

// ...time passes, user clicks the button, handleClick fires...

// UNSUBSCRIBE — tell the button: "stop calling me when you publish 'click'"
button.removeEventListener('click', handleClick);
```

- The button is the **publisher**. It emits `click` events whenever a click happens.
- `handleClick` is the **subscriber**. It runs when the publisher emits.
- `addEventListener` is the **subscribe** call.
- `removeEventListener` is the **unsubscribe** call.

If you never call `addEventListener`, `handleClick` will never run — clicks happen but nobody asked to be told about them. The publisher publishes into the void.

### Why unsubscribe matters

If you subscribe and then walk away without unsubscribing, the publisher still holds a reference to your function. Two problems:

**1. Memory leak.** The function (and everything it references — closures, DOM nodes, app state) can't be garbage collected. The publisher is still "expecting" to call it someday, so the runtime can't free it. If you keep subscribing without unsubscribing, your app's memory usage grows over time. In a single-page app that never reloads, this is a slow poison.

**2. Stale callbacks.** The function might still get called after the component that owns it has been unmounted. The function tries to do something with state that no longer exists, or update a UI element that's been removed, and you get:

```
Warning: Can't perform a React state update on an unmounted component.
This is a no-op, but it indicates a memory leak in your application.
```

That warning is React noticing that you subscribed without unsubscribing.

### useEffect cleanup — explicit pub/sub in React

React makes the pattern explicit when you set up subscriptions inside a component:

```tsx
useEffect(() => {
  window.addEventListener('resize', handleResize);   // subscribe on mount

  return () => {
    window.removeEventListener('resize', handleResize);  // unsubscribe on unmount
  };
}, []);
```

The function you return from `useEffect` is the **cleanup**. React runs it when the component unmounts (or before the effect re-runs, if deps changed). It's the place to put your unsubscribe call.

The shape is always:

```ts
useEffect(() => {
  // subscribe
  return () => {
    // unsubscribe
  };
}, [deps]);
```

If your effect *doesn't* set up a subscription (or a timer, or anything external), you don't need a cleanup. But anything that lives outside React's tree and could outlive the component needs one.

### Implicit vs. explicit subscription

In raw pub/sub, **subscription is explicit**. You write `addEventListener(...)` or `emitter.on('event', cb)` or `socket.subscribe(...)` by hand. Same for unsubscribing.

Some libraries skip that step and make subscription **implicit** — they figure out who wants to subscribe by watching what your code reads.

The classic example is MobX:

```tsx
export const Confirmation = observer(() => {
  // Just reading the value. No subscribe call.
  if (checkoutStore.googleAdsSendTos?.purchase) { ... }
});
```

That `observer(...)` wrapper sets up a hidden subscription system. During the component's render, MobX silently records: *"this component read `googleAdsSendTos`."* That counts as the subscription — no explicit call needed.

When the component unmounts, MobX automatically tears down the subscriptions it set up. No explicit unsubscribe needed.

So in MobX:
- **Subscribe** = "read this observable inside an `observer` component" → MobX wires it up for you
- **Unsubscribe** = "component unmounts" → MobX cleans it up for you

The pattern is still pub/sub. The subscribe/unsubscribe calls are still happening — they're just done by the library, not by you.

### Two analogies

- **Manual pub/sub** (`addEventListener`, Redux, EventEmitter): filling out a paper form to subscribe to a magazine. You sign up explicitly. When you move, you have to mail in a cancellation form, or magazines pile up at your old address forever.
- **Implicit pub/sub** (MobX, Vue's reactivity, Svelte): walking into a library, picking up a book. The library notices what you picked up and automatically tells you when there's a new edition. When you leave, the tracking stops on its own.

Both deliver the magazine. Only one requires you to fill out forms.

### Where else pub/sub shows up

Once you know the pattern, you'll see it everywhere:

- **DOM events** — every `addEventListener` is pub/sub
- **Node.js EventEmitter** — explicit `.on('event', cb)` and `.off('event', cb)`
- **WebSockets** — `socket.onmessage = ...` is a subscription to incoming messages
- **MobX, Vue, Svelte, SolidJS** — implicit subscriptions to reactive state
- **Redux** — `store.subscribe(listener)` (you usually don't see this because `react-redux` wraps it for you)
- **Server-sent events / EventSource** — `eventSource.addEventListener('update', cb)`
- **Message queues** (Kafka, RabbitMQ, SNS/SQS) — publishers send messages, subscribers register topics
- **Slack/Discord notifications** — channels publish, users subscribe to channels
- **Newsletters, podcasts, YouTube channels** — the original analogy still holds

## When does this matter?

- **Setting up any event listener.** Always think about the unsubscribe. If the listener is inside a component, that means a `useEffect` cleanup. If it's at module load, ask yourself whether it should ever unsubscribe (often: no, but be deliberate about it).
- **Debugging memory leaks.** A common cause is subscribing inside a component without cleaning up. Each mount/unmount cycle leaks one more listener. Check `useEffect`s for missing return functions.
- **Understanding "reactive" libraries.** Anything advertised as "reactive" — MobX, Vue refs, Svelte stores, RxJS — is doing pub/sub under the hood. Knowing the pattern means you can reason about why your component does or doesn't update.
- **Debugging "X isn't reacting to Y."** Three questions: Is X subscribed? Is Y actually publishing (e.g., is the setter actually being called)? Is X still alive when Y publishes (or did it unmount)?
- **Designing a system that needs to broadcast.** When one piece of code needs to notify N other pieces, and you don't want them tightly coupled, pub/sub is usually the right pattern. The publisher doesn't need to know who its subscribers are.

## See also

- [Component mounting](component-mounting.md) — subscriptions are commonly set up on mount and torn down on unmount
- [MobX](mobx.md) — implicit pub/sub for state changes
- [Stores (shared state)](stores-shared-state.md) — what reactive stores are built on top of
- [Unit tests](unit-tests.md) — testing subscription cleanup is a common case for "test the cleanup function"
- [MDN — EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
