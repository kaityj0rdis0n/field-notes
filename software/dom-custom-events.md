---
title: DOM CustomEvents — firing your own signals on a page
category: software
date: 2026-05-21
tags: [javascript, DOM, events, web, analytics]
---

# DOM CustomEvents — firing your own signals on a page

## TL;DR

The browser has built-in events (`click`, `scroll`, `keydown`). You can also create your own with `CustomEvent` and fire them on `document` — any other script on the same page can listen for them. This is how analytics bridges work: a signal comes in from one context, gets translated into a CustomEvent on `document`, and GTM or any other script picks it up without knowing where it came from.

## The explanation

### What the DOM is

DOM stands for Document Object Model. When a browser loads an HTML page, it turns the markup into a tree of objects that JavaScript can interact with. `document` is the root of that tree — the top-level object representing the entire page. When you open browser devtools and see the HTML elements nested inside each other, that's the DOM.

### Built-in events vs. custom events

The browser fires events constantly — when you click something, scroll, type, resize the window. These are built-in. But JavaScript also lets you create and fire your own:

```ts
// Create a custom event with a name and a payload
const event = new CustomEvent('bookr:reservation:confirmed', {
  detail: { reservation_id: 'res_123', party_size: 4 },
  bubbles: true,
});

// Fire it on document — any listener on document will receive it
document.dispatchEvent(event);
```

Any other script on the page can listen for it:

```js
document.addEventListener('bookr:reservation:confirmed', (event) => {
  console.log('reservation happened:', event.detail);
  // → reservation happened: { reservation_id: 'res_123', party_size: 4 }
});
```

### Why `document` specifically

You could dispatch a CustomEvent on any DOM element, but `document` is the conventional target for page-level signals — it's always available, always the same object, and any script on the page can reference it without needing to find a specific element.

### Namespacing with a prefix

Naming collisions are a real risk when multiple scripts on a page all fire CustomEvents. If your event is called `purchased` and so is another library's, they step on each other. A prefix makes clear who owns the event:

```ts
bookr:reservation:confirmed   // Bookr widget — reservation completed
bookr:step:changed            // Bookr widget — user moved to next step
stripe:payment:success        // Stripe — payment completed
```

### How a translation helper typically looks

A common pattern is a small wrapper that applies the namespace automatically:

```ts
function buildEvent(type: string, data: unknown): CustomEvent {
  const event = document.createEvent('CustomEvent');
  event.initCustomEvent(`bookr:${type}`, true, false, data);
  return event;
}

// Usage
document.dispatchEvent(buildEvent('reservation:confirmed', { reservation_id: 'res_123' }));
// fires: bookr:reservation:confirmed
```

### Why this comes up in widget/analytics contexts

An embedded widget lives in an iframe — a completely separate JavaScript context from the parent page. The parent page's analytics (GTM tags) need to know when things happen inside the widget, but they can't reach inside the iframe.

The chain: widget fires a postMessage → a small script on the parent page catches it → translates it into a `CustomEvent` on `document` → GTM sees the event and fires the conversion.

CustomEvents are the delivery format that GTM understands. The parent-page script is the translator between the two worlds.

## When does this matter?

- You're reading code that dispatches events on `document` and want to understand why
- You're debugging "tracking isn't firing" — check whether the CustomEvent is actually being dispatched (devtools → Event Listeners on `document`)
- You're building an analytics bridge between an iframe widget and a parent page
- You need two unrelated scripts on the same page to communicate without tight coupling

## See also

- [Iframe widget analytics — the four-hop pipeline](iframe-widget-analytics-pipeline.md)
- [Pub/sub (publish/subscribe)](pub-sub.md) — CustomEvents are pub/sub on the DOM
- [MDN — CustomEvent](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)
- [MDN — EventTarget.dispatchEvent](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)
