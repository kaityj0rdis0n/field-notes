---
title: GraphQL introspection
category: software
date: 2026-06-18
tags: [graphql, api, introspection, schema]
---

# GraphQL introspection

## TL;DR

A GraphQL API can describe itself. Introspection is asking the API to hand you its own schema (every type, field, and mutation, plus exactly what inputs each one takes) through a few special "meta" fields. So instead of guessing what a query or mutation looks like, or trusting stale docs, you ask the API directly and it answers in structured data.

## The explanation

I hit this building a small CLI that creates events on Universe's public GraphQL API. My first version *guessed* the mutation was `createEvent(input: EventInput!)`. That guess might be right, or the real name might be `eventCreate`, or the input might be a differently-named type with different required fields. Introspection replaces the guess with the truth.

### The three meta-fields

Every GraphQL server exposes these. They're how you interrogate it:

- `__schema` — the top-level map: what queries and mutations exist
- `__type(name: "X")` — zoom into one type and list its fields
- `__typename` — "what type is this object?" (handy, but less central here)

### A real example

To figure out how to list a user's events, I sent this (simplified):

```graphql
{ __type(name: "Viewer") { fields { name args { name } } } }
```

That's me asking: "List every field on the `Viewer` type." The API answered with the full list, including:

```
events(slugs, currency, states) -> EventConnection
```

That one line told me three true things, no guessing:

1. `events` is a real field on the viewer.
2. It accepts `slugs`, `currency`, and `states` as filters.
3. It returns an `EventConnection`.

I wrote the query against that and it worked first try, because I was reading the API's own description of itself instead of guessing.

### The trick for "what's required"

When you introspect a field or input, GraphQL wraps required things in a `NON_NULL` type. So on a mutation's input, the fields marked `NON_NULL` are mandatory and everything else is optional. That's how introspection hands you the *minimum* payload to send, precisely, with no trial and error.

### What it can't tell you

Introspection reveals the **shape** of the API (what exists, what's required). It does **not** tell you **behavior or permissions**, like whether your particular access token is allowed to run a mutation. Those are business rules, not part of the schema. So introspection answers "what do I send"; you still have to make a real call to learn "am I allowed, and does it behave as expected."

> Some APIs disable introspection in production for security. If `__type` / `__schema` come back empty, that's why. Fallback: send a small real query and read the error. GraphQL errors like "Cannot query field X on type Y, did you mean…?" leak the schema too.

## When does this matter?

- **Integrating with any GraphQL API** — confirm real field names and required inputs before writing a query, instead of guessing.
- **Replacing guessed or hardcoded queries** — check code's assumed shape against the live schema.
- **Finding required fields fast** — `NON_NULL` on the input type is the definitive must-provide list.
- **Exploring an unfamiliar API** — `__schema` for the lay of the land, `__type` to drill in, no docs required.

## See also

- [GraphQL.org: Introspection](https://graphql.org/learn/introspection/) — official explainer
- [GraphQL spec: Introspection](https://spec.graphql.org/draft/#sec-Introspection) — authoritative definition
- [function-signatures.md](function-signatures.md) — same theme: read the contract, don't guess the implementation
