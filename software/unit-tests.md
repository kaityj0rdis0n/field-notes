---
title: Unit tests
category: software
date: 2026-05-19
tags: [testing, programming, fundamentals]
---

# Unit tests

## TL;DR

A **unit test** is automated code that verifies one specific behavior of a small piece of code (a "unit"). It runs without clicking through the app — the test runner just executes the code with controlled inputs and checks the outputs. Compared to other test types: fastest, most isolated, runs first in CI.

## The explanation

The "unit" is usually a function or component, small enough to isolate from the rest of the system.

### Test types, ranked by scope

| Type | What it tests | Speed | Trade-off |
|---|---|---|---|
| **Unit** | One function or component, isolated | ms | Fast and reliable, but doesn't catch integration bugs |
| **Integration** | Multiple units together (e.g. component + store + mocked API) | 100s of ms | Catches wiring bugs, slower |
| **End-to-end (E2E)** | A real browser clicking through real pages | seconds to minutes | Catches everything, slowest and flakiest |

The pyramid: lots of unit tests at the base, fewer integration in the middle, very few E2E at the top. Catches bugs cheap early, expensive tests last.

### The arrange-act-assert pattern

Most unit tests follow this structure:

```ts
test('updates store when prop arrives', () => {
  // Arrange — set up the inputs and the watcher
  const mockProps = { googleAdsSendTos: { purchase: ['AW-123'] } };
  const setSpy = jest.spyOn(store, 'setGoogleAdsSendTos');

  // Act — call the thing being tested
  render(<CheckoutModal {...mockProps} />);

  // Assert — verify the expected effect
  expect(setSpy).toHaveBeenCalledWith({ purchase: ['AW-123'] });
});
```

Three crisp sections. The test name describes the expected behavior in plain English.

### Mocking

Unit tests usually need to *isolate* the code under test, which means replacing real dependencies with controlled stand-ins ("mocks"):

- The real API call → a mock that returns a known response
- The real database → a mock that returns fixed data
- The real clock → a mock that returns a fixed time

The trade-off: more mocking means more isolated tests, but also tests that don't catch real-world bugs in the mocked dependencies. Balance.

## When does this matter?

- **Documenting expected behavior.** A good test name + assertion describes what the code is supposed to do better than a comment.
- **Catching regressions.** Someone removes a line that mattered → CI fails immediately, not in production three weeks later.
- **Refactoring safely.** Tests give you the confidence to change the implementation without breaking the contract — if the tests still pass, the behavior is preserved.
- **Onboarding.** New developers can read tests to learn how a function is meant to be used.
- **The "first test is expensive" rule.** Setting up the test infrastructure for a new component (mocks, fixtures, providers) is often more work than the fix itself. After that, adding tests is fast.

## See also

- [Function signatures](function-signatures.md) — unit tests typically check that a function's body honors its signature for various inputs
- [Jest docs](https://jestjs.io/docs/getting-started)
