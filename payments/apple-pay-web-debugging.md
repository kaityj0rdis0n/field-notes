---
title: Debugging a stuck Apple Pay sheet on the web
category: payments
date: 2026-09-22
tags: [apple-pay, payments, web-checkout, browser-devtools, debugging, macos]
---

# Debugging a stuck Apple Pay sheet on the web

## TL;DR

If the Apple Pay sheet opens on a web checkout, shows the card and amount, but does nothing when you try to confirm and prints no error: check whether your screen is being shared or recorded. Apple Pay blocks authorization during screen capture, and it fails silently. That one non-obvious cause wasted a long debugging session for me. When it still misbehaves after that, the console flow tells you which side the problem is on.

## The explanation

### There is no "Pay" button on a Mac

The Apple Pay sheet on the web has no on-screen confirm button. You authorize with Touch ID, or by double-clicking the side button on a paired iPhone or Apple Watch signed into the same Apple ID. On a Mac with no Touch ID and no paired device nearby, there is nothing to confirm with, and the sheet just sits there. First-time testers look for a button that was never going to be there.

So before anything else, confirm the machine can actually authorize: a Mac with Touch ID (with a fingerprint enrolled and Apple Pay toggled on in System Settings > Touch ID & Password), or a paired iPhone/Watch within Bluetooth range.

### The silent-failure trap

The confusing symptom: the sheet opens, shows a valid card and the correct amount, and pressing Touch ID does nothing. No spinner, no error dialog, nothing in the console tied to the press. Every instinct says config or permissions. The cause can be much dumber.

**The first thing to rule out is screen capture.** If you are sharing your screen (a call, a recording, anything capturing the display), Apple Pay refuses to authorize, for privacy reasons, and it does so quietly. Stop the share, reopen the sheet, and try again before you touch any code or settings. This is the single highest-value check because it is invisible and unrelated to the payment stack.

### Triage: which side is the problem on?

Once you can actually authorize, one question sorts most Apple Pay web bugs into the right half:

- **No reaction at all to the confirm** (Touch ID does nothing, sheet frozen): the block is *before* authorization. Look at the environment (screen share), the device (can it authorize), or the merchant validation step.
- **It accepts the confirm, shows processing, then errors or hangs**: the payment token reached your backend and the *submit* failed. Look at your server, the payment gateway, or anything else the order submission depends on (fraud checks, captcha, validation).

That split saves you from debugging the backend when the block is on the device, or vice versa.

### Reading the console flow

Apple Pay on the web runs a predictable sequence. Watching it in the browser console tells you exactly how far you got:

```
canMakePayments                 → is Apple Pay available on this device?
build the payment request       → sets amount, networks, merchant id
onmerchantvalidation fires      → the site asks Apple to start a session
"session created" +             → merchant validation SUCCEEDED
  merchantSessionIdentifier        (domain + merchant cert are fine)
onpaymentmethodchange           → sheet is interactive
[user confirms]
onpaymentauthorized             → the token you actually need
```

The most useful landmark is the **session-created line with a `merchantSessionIdentifier`**. Once you see it, merchant validation passed, which means the domain registration and merchant certificate are fine. I burned time chasing a "domain not registered" theory before I noticed the session had already been created, which had ruled that out. Read for that line first, and skip the domain rabbit hole if it is there.

If the log stops at `onpaymentmethodchange` and never reaches `onpaymentauthorized`, you are stuck at the confirm step, which is exactly where the screen-capture block lands.

## When does this matter?

- **Testing Apple Pay on a web checkout in a dev or sandbox environment**, especially over a screen share with a colleague. The share itself is what breaks it.
- **A bug report that says "Apple Pay does nothing"** with no error. Start with device capability and screen capture before the payment code.
- **Deciding whether a payment problem is front-end or back-end.** The no-reaction vs. accepts-then-fails split answers it fast.

## Gotchas

- **Screen recording counts, not just live sharing.** Any capture of the display can trigger the block.
- **A test card showing in the sheet is not proof it can authorize.** For sandbox testing you need a properly provisioned sandbox card via an Apple sandbox tester account, signed into iCloud on the device.
- **Custom diagnostic logs can lie.** A homegrown "has active card?" log said "no cards in wallet" while a card was clearly in the sheet. Trust the native Apple Pay flow (the session-created and authorized events) over a custom wrapper's opinion.
- **Other red errors in the console may be unrelated.** A reCAPTCHA "invalid domain" error and a wallet-provider permission-policy warning were both in the log and neither was the Apple Pay blocker. Don't anchor on the loudest error; confirm it is on the path you care about.

## See also

- [Apple Pay on the Web — Apple Developer](https://developer.apple.com/documentation/apple_pay_on_the_web)
- [Payment Request API — MDN](https://developer.mozilla.org/en-US/docs/Web/API/Payment_Request_API)
- [Testing a deployed branch via header injection](../software/staging-build-header-testing.md) — companion note on testing checkout flows in a real environment
