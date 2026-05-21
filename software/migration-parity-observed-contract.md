---
title: Migration parity is the observed contract, not the documented one
category: software
date: 2026-05-21
tags: [migration, refactoring, contracts, debugging]
---

# Migration parity is the observed contract, not the documented one

## TL;DR

When you replace one system with another, "matching the API" is necessary but not sufficient. Downstream consumers built against everything they could observe — undocumented fields, ordering, timing, default values, paths, side-channel signals. The replacement is only at parity when consumers can't tell the difference. The documented interface is a small subset of what they actually depend on.

## The explanation

Migrations come in many flavors — strangler fig replacements, library swaps, framework upgrades, language rewrites — but they all carry the same trap. The team writes a spec: "the new system must match these N endpoints / events / message shapes." They build to that spec. They cut over. Then dashboards quietly stop populating, conversion events go missing, partner integrations break, and nobody knows why for weeks.

The reason is almost always that the *real* contract isn't the one in the spec. It's everything a consumer can observe and depend on:

- **Field names documented** in the spec (these usually match)
- **Field names *not* documented** but emitted anyway (these often don't)
- **Field values** — exact strings, casing, formatting
- **Ordering** of events, fields, records
- **Timing** — how soon after action X does event Y fire
- **Negative space** — what's *not* sent, what's null, what's omitted entirely
- **Paths and routes** — the URL or topic name through which the data flows
- **Headers, metadata, envelopes** — what wraps the payload
- **Default values** when a field is missing
- **Side effects** — auxiliary signals the old system happened to emit (logs, metrics, analytics events)

Any consumer that's been in production long enough has built triggers, alerts, dashboards, or business logic against any subset of these. They didn't ask permission. They saw something stable and used it.

When you migrate, you have to reproduce *all of that observable behavior*, not just the parts the old system documented.

## The shape of the bug

```
old system  →  emits 12 things    →  consumer uses 7 of them
                (5 documented, 7 not)

new system  →  emits 5 things     →  consumer is broken in ways
                (the 5 documented)    nobody can explain
```

The 7 things the consumer used? Three were documented. Four were stuff the old system happened to emit, and the consumer found them in a debugger one Friday afternoon and built a dashboard.

## A concrete example

A widget gets replaced. The old version emits cross-frame `postMessage` events the host page picks up:

```js
// old widget — every step
window.parent.postMessage({
  action: 'Step',
  data: { val: 4, session, referrer, user_agent, page: '/events/abc/checkout' }
}, '*');
```

The spec says "emit a Step event with a numeric val on every transition." The new widget does that:

```js
// new widget — every step
window.parent.postMessage({
  action: 'Step',
  data: { val: 4 }
}, '*');
```

Spec satisfied. The host page's analytics dashboard that grouped sessions by `referrer` is now permanently null. The marketer's GTM trigger keyed on `page contains "/checkout"` stops firing. Nobody told you. The dashboard owner files a bug six weeks later. You ship a fix. Three months later somebody else notices `user_agent` isn't there either.

This was preventable by diffing the actual wire payload, end to end, before cutover.

## How to scope a migration properly

Before writing any code, list every consumer of the old system. For each one, find:

1. **What do they say they use?** — docs, slack history, tickets
2. **What can you prove they use?** — search their code/config for field names, event names, paths. If they're external, read their integration code if you have access
3. **What can you infer they might use?** — their logs, dashboards, GTM containers, alert rules

The union is your real contract. The documented interface alone is a starting point.

## Practical heuristics

- **Run side-by-side as long as you can afford.** Have both systems emit, even if only one is the source of truth. Diff their output. Anything that diverges, decide whether it matters.
- **Diff payloads at the wire level, not the spec level.** A spec compliance test will pass while the wire payload is missing fields the spec didn't list.
- **Talk to consumers.** Ask "what would you notice change?" — they'll surface things the docs don't.
- **Treat "we'll add it later" as deferred breakage.** Every undocumented field you skip is an outage waiting for the consumer to flip from old to new.
- **Audit telemetry.** The old system probably emitted metrics, logs, analytics. They count as outputs even if the spec doesn't list them.

## When does this matter?

- Strangler fig migrations
- Replacing internal SDKs
- Library version upgrades with breaking changes
- Framework rewrites
- Vendor swaps (auth, analytics, payment providers)
- Anywhere external partners integrate against your output and you don't fully control their code

## When *not* to apply

- Greenfield work — no consumers yet
- Internal tools where you control all consumers and can refactor them in lockstep

## See also

- [Contract mismatch — fix the caller, not the receiver](contract-mismatch-fix-the-caller.md) — what to do when you discover a parity gap
- [Property shorthand & exact-match lookup (JS)](property-shorthand.md) — common low-level cause of "the spec matches but the wire doesn't"
- [Function signatures](function-signatures.md) — the documented contract is the signature; observed contract is what callers depend on in practice
