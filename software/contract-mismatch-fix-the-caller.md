---
title: Contract mismatch — fix the caller, not the receiver
category: software
date: 2026-05-15
tags: [debugging, architecture, software-patterns]
---

# Contract mismatch — fix the caller, not the receiver

## TL;DR

When a system has a sender and a receiver — code that produces data, then code that consumes it — bugs often show up at the receiver but live at the sender. The receiver is doing exactly what it was designed to do; the sender is using the wrong label, wrong shape, wrong path. Fixing the receiver to "be more forgiving" usually masks the bug elsewhere. Fix the caller.

## The explanation

A common bug shape:

```
Caller code       →   Boundary (function call,   →   Receiver code
                      URL, postMessage, etc.)        expects: { lang }
sends: { locale }                                    sees: { lang: undefined }
```

The caller looks reasonable. The receiver looks reasonable. The data crossing the boundary doesn't match what the receiver was built to read. The symptom shows up at the receiver — a default value, a missing feature, an empty field. The cause is the caller, often far away.

The instinct is to patch the receiver: "if `lang` isn't set, fall back to `locale`." This usually:

1. Doesn't address why the contract was broken in the first place.
2. Creates a precedent — now other callers can break the contract too, because there's a fallback. The contract decays into "anything goes."
3. Makes the receiver harder to reason about. It's now doing schema sniffing.

Fixing the caller:

1. Restores the contract — there's one shape, both sides agree on it.
2. Lets the receiver stay simple.
3. Often surfaces *other* callers that have the same bug (because once you start grepping for the wrong shape, you find the rest).

## The "three actors" frame

For non-trivial cases, draw three boxes:

```
[Producer]  →  [Bridge / contract]  →  [Consumer]
```

- The **producer** is your code.
- The **bridge** is the API/URL/message format that defines the shape both sides agree on.
- The **consumer** is the receiver — sometimes in a totally different repo or system.

When the bug manifests at the consumer, the question to ask is: "is the consumer reading the bridge correctly, or is the producer writing it wrong?" Almost always the producer.

## When does this matter?

- Cross-iframe / cross-window messaging where one side passes `postMessage` payload with the wrong field.
- Frontend-to-API calls where the body shape diverges from what the server expects.
- React component prop names — child expects `onChange`, parent passes `onUpdate`.
- Embedded third-party widgets (ad SDKs, analytics, payment iframes) where you control the init code but not the widget itself.
- Anywhere a bug shows up in a system you didn't write — check your handoff to it before you blame it.

## When *not* to apply this

- The contract is genuinely ambiguous and the receiver should be tolerant (e.g. backwards-compatibility shims during a migration).
- You don't control the consumer and the producer is a third party. Then yes, you may need to layer a translation.

But default to fixing the caller.

## See also

- [Property shorthand & exact-match lookup](property-shorthand.md) — common low-level source of contract mismatches
- [Function interception for debugging](function-interception.md) — how to actually confirm which side is wrong
