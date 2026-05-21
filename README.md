# Field Notes

A personal library of things I learn about how stuff works — git, accounting, payments, software concepts — anything worth writing down once so I don't have to ask twice.

Each entry is a self-contained markdown file in a topic folder. Entries are written for **future me** picking it up cold: plain language, a TL;DR up top, examples where they help.

## Index

### Git & version control
- [Fast-forward merges](git/fast-forward.md) — what it means when git "fast-forwards" your branch

### Accounting & tax
- [Deferral](accounting-tax/deferral.md) — pushing revenue or expense recognition into a later period
- [Revenue recognition (accrual basis)](accounting-tax/revenue-recognition.md) — when revenue is "earned" vs. when cash hits the account
- [Merchant of Record vs. Principal/Agent](accounting-tax/mor-principal-agent.md) — two separate questions people conflate
- [State nexus](accounting-tax/state-nexus.md) — what gives a US state the right to tax you

### Payments
- [1099-K reporting](payments/1099-k.md) — the IRS information return for payment platforms, who files, threshold history

### Software

**React & component fundamentals**
- [What's a prop? (React)](software/react-props.md) — components are functions, props are their arguments
- [Prop drilling](software/prop-drilling.md) — when data has to be passed through many layers, and how stores/context sidestep it
- [Component mounting (React)](software/component-mounting.md) — when a component first hits the DOM and `useEffect([])` fires

**State management**
- [Stores (shared state)](software/stores-shared-state.md) — the shared-object pattern that lets distant components read/write the same value without prop drilling
- [MobX (state management)](software/mobx.md) — observable / action / observer trio, how MobX auto-wires subscriptions at render time
- [Pub/sub (publish/subscribe)](software/pub-sub.md) — the publisher-and-subscriber pattern that underlies event listeners, MobX, and most reactive systems
- [Shared-state population trap](software/shared-state-population-trap.md) — why a setter forgotten in one entry flow silently breaks only that flow's downstream behavior

**Programming fundamentals**
- [Function signatures](software/function-signatures.md) — how to use something without reading its body; the contract vs the implementation
- [Property shorthand & exact-match lookup (JS)](software/property-shorthand.md) — why `{ locale }` and `{ lang }` aren't interchangeable, even when they "mean" the same thing
- [Semantic versioning (semver)](software/semver.md) — the three numbers in `v1.2.3` and what they promise about what changed
- [TypeScript `as` casts can lie](software/typescript-as-casts.md) — `as` is an assertion to the compiler with no runtime check; misuse turns into a silent no-op

**Debugging**
- [Function interception for debugging](software/function-interception.md) — wrap a global function in a logging shim to see what's actually being called
- [Contract mismatch — fix the caller, not the receiver](software/contract-mismatch-fix-the-caller.md) — when the bug shows up downstream but the cause is upstream

**Architecture & migrations**
- [Migration parity is the observed contract, not the documented one](software/migration-parity-observed-contract.md) — replacements have to match what consumers actually depend on, not just the spec
- [Iframe widget analytics — the four-hop pipeline](software/iframe-widget-analytics-pipeline.md) — postMessage → SDK → dataLayer → tag manager; how embeddable widgets reach the host page's analytics

**Testing**
- [Unit tests](software/unit-tests.md) — what they are, arrange-act-assert structure, how they compare to integration and E2E
- [Testing a deployed branch via header injection](software/staging-build-header-testing.md) — hit production URLs but receive your branch's build via a header

## Adding entries

See [template.md](template.md) for the format. Short rules of thumb:

- One concept per file.
- Lead with the TL;DR — one or two sentences a tired version of me would understand.
- Use plain language, not jargon I'd have to look up.
- Include a small example or diagram when it helps.
- Link to other entries with relative paths.

## Companion repo

There's a private companion repo for stuff that's specific to my work (Universe / TM) and shouldn't be public.
