---
title: The silent registration gap — writing a handler vs. wiring it up
category: software
date: 2026-05-21
tags: [javascript, typescript, events, bugs, pub-sub]
---

# The silent registration gap — writing a handler vs. wiring it up

## TL;DR

Writing a method that handles an event is not the same as registering it to listen for that event. If you write the handler but forget to call `addEventListener` / `emitter.on` / equivalent, the handler exists but never runs — and nothing breaks visibly. No error. No warning. Events fire, the handler sits there, and the thing you expected to happen just silently doesn't.

## The explanation

### The two steps you have to complete

Any event-driven system requires two things:

1. **Write the handler** — the function that runs when the event fires
2. **Register the handler** — tell the system "when this event fires, call this function"

Both are required. Writing the handler without registering it is the same as writing a function that's never called.

```ts
class WidgetBridge {
  // Step 1: handler written ✅
  onPurchaseComplete(data: unknown): void {
    document.dispatchEvent(new CustomEvent('widget:purchase', { detail: data }));
  }

  constructor() {
    // Step 2: registration — but it's missing ❌
    // messenger.on('PURCHASE:COMPLETE', this.onPurchaseComplete.bind(this));
  }
}
```

The method exists. It does the right thing. But without the `messenger.on` call, nothing ever invokes it. The widget sends `PURCHASE:COMPLETE` messages — and they go unanswered.

### Why it's so hard to catch

Normal bugs produce errors: a thrown exception, a 500, a red screen. This one produces nothing. The system works fine — it just silently omits the thing it was supposed to do:

- The purchase completes ✅
- The widget emits the event ✅
- The parent page receives it ✅
- The handler is never called ❌ (no error)
- GTM never sees `widget:purchase` ❌ (no error)
- Partner conversion tracking silently stops ❌ (no error)

You only discover it when a partner says "our conversions are missing" weeks later.

### The specific pattern to watch for

Any class that sets up listeners in a constructor is vulnerable when the list grows:

```ts
constructor() {
  // These are registered:
  messenger.on('GOOGLE_ADS:EVENT', this.onGoogleAds.bind(this));
  messenger.on('FACEBOOK:EVENT', this.onFacebook.bind(this));
  messenger.on('TIKTOK:EVENT', this.onTikTok.bind(this));

  // Easy to miss when adding a new one:
  // messenger.on('PURCHASE:COMPLETE', this.onPurchaseComplete.bind(this));
  // messenger.on('STEP:CHANGED', this.onStepChanged.bind(this));
}
```

The longer the list, the easier it is to write the method and forget the constructor line. Code review can miss it because the method looks complete.

### How to catch it

**In tests** — test the constructor registrations explicitly. Don't just test that handlers do the right thing; test that they're registered at all:

```ts
it('registers a listener for PURCHASE:COMPLETE', () => {
  new WidgetBridge();
  const registeredEvents = mockMessenger.on.mock.calls.map(([name]) => name);
  expect(registeredEvents).toContain('PURCHASE:COMPLETE');
});
```

**In code review** — when you see a new handler method added to a class, look for the corresponding registration. If you don't see it, the method is an orphan.

**In debugging** — if an event-driven feature silently doesn't work, your first question is: "is the handler registered?" Not "is the handler correct?" The handler might be perfect and still never run.

## When does this matter?

- Reading or reviewing any class that registers event listeners in a constructor
- Debugging "this should happen when X occurs but doesn't" — check registration first
- Adding a new event handler to an existing class — make sure you add both the method and the registration line
- Writing tests for event-driven classes — test registrations explicitly, not just handler logic

## See also

- [Pub/sub (publish/subscribe)](pub-sub.md) — the pattern this bug lives inside
- [DOM CustomEvents](dom-custom-events.md) — the delivery mechanism on the parent page
- [Iframe widget analytics — the four-hop pipeline](iframe-widget-analytics-pipeline.md)
