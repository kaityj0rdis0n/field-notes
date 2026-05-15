---
title: Testing a deployed branch via header injection
category: software
date: 2026-05-15
tags: [testing, ci-cd, deployment, browser-devtools]
---

# Testing a deployed branch via header injection

## TL;DR

Some teams configure their hosting layer to route requests to a specific build artifact based on a header (e.g. a commit SHA). You can hit your production URL but receive your branch's build by injecting that header from your browser via a tool like ModHeader. This avoids local dev setup entirely and tests in a real environment.

## The explanation

The setup, on the deploy side, looks like:

```
[CDN / hosting layer]
       │
       │  inspect request headers
       ↓
   ┌─────────────────────┐
   │ If header X is set: │ → serve build for that branch
   │ Otherwise:          │ → serve master/main build
   └─────────────────────┘
```

Your team has to actually wire this up (the CDN/load balancer must know how to look up build artifacts by SHA or branch name). When it exists, the workflow is:

1. **Open a PR.** CI builds a deployable artifact for your branch and stores it somewhere reachable.
2. **Find the identifier** — usually the commit SHA or branch name. Often exposed in PR comments or CI logs.
3. **In your browser, install a header-injection extension** (ModHeader is the popular one). Add a request header like `x-branch-sha: <your-sha>`, scoped to the production domain.
4. **Visit the production URL.** Your branch's build is served. Everyone else sees the normal build.

You're testing in the actual production environment — real services, real auth, real data — without merging or deploying.

## When does this matter?

- **The codebase is hard to run locally** (old toolchain, exotic dependencies, services that need VPN). Setup cost dwarfs the value.
- **The bug needs production data or services** to reproduce. Local mocks don't catch it.
- **You want pre-merge confidence beyond CI's automated tests** — the "click around in a real browser" check you'd normally need a deploy for.
- **You're verifying a fix where the deployed artifact differs from local source** (minified bundles, build-time transforms, polyfills). Local testing wouldn't show what production would.

## Gotchas

- **The header must reach the host.** If your CDN/proxy strips custom headers before forwarding, this won't work.
- **Scope the header.** Without a URL filter, the extension sends it on every request from your browser — usually harmless but messy. Limit to your domain.
- **Incognito needs extension permission.** Most extensions don't run in incognito by default — opt in via the extension settings. The toggle doesn't apply to incognito windows that were already open — close and reopen.
- **Cookies and account preferences may override what you're testing.** If you're testing locale and you're logged in with a language preference, the server may ignore browser hints. Test logged-out (incognito) or change the preference.
- **It's not the same as a staging environment.** You're getting a one-off build served in production context. Side effects (real emails, real charges, real bookings) still apply unless your code paths know to behave differently.

## See also

- [ModHeader extension](https://chromewebstore.google.com/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj)
