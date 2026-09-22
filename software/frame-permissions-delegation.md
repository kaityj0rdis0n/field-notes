---
title: How browsers decide what embedded content is allowed to do
category: software
date: 2026-09-22
tags: [browser, iframe, permissions-policy, security, cors, web-platform]
---

# How browsers decide what embedded content is allowed to do

## TL;DR

In the browser, powerful capabilities (camera, microphone, geolocation, payment) flow *down* the frame tree. A cross-origin or sandboxed iframe gets one of these features only if the page embedding it explicitly hands it down, and a page can hand down only what it was granted itself. So an embedded third-party widget or SDK can't turn on a capability that its host page withholds. When one of these fails, the fix is on the host page, not inside the embedded code.

## The explanation

When you embed something in a page (an iframe for a payment widget, a video player, a map), the browser treats that embedded frame as a separate, less-trusted context. By default it does not get access to the powerful browser features, even if it needs them. The host page has to grant them.

There are a few mechanisms, and they all point the same direction: the parent controls the child.

### Permissions-Policy

This is an HTTP response header (older name: Feature-Policy) the host page sends. It says which features are allowed on the page at all, and which of those get handed to embedded frames and which origins.

```
Permissions-Policy: payment=(self "https://widget.example.com")
```

That line means: the payment feature is allowed for the page's own origin, and delegated to frames from `widget.example.com`. A frame from any other origin gets nothing for `payment`. Most powerful features default to `self` only, so a cross-origin child starts with none of them until the header names it.

### The iframe `allow` attribute

A per-frame grant, written on the `<iframe>` tag itself:

```html
<iframe src="https://widget.example.com" allow="payment; camera"></iframe>
```

This is the parent choosing what this specific frame may use. Note the direction again: the attribute lives on the parent's tag, describing what the parent permits. The child can't add to it.

### The iframe `sandbox` attribute

Sandbox works the opposite way round: it strips almost everything, then you add back specific privileges with `allow-*` tokens (`allow-scripts`, `allow-forms`, and so on). The child still can't re-grant itself anything. Only the parent's tokens decide what comes back.

### CORS (the same idea, for reading responses)

When code fetches a cross-origin resource, the *server* returning the resource decides who is allowed to read it, via `Access-Control-Allow-Origin`. The caller can't grant itself read access. Same shape: the side that owns the resource controls access, not the side that wants it.

### A worked example: Payment Request / wallet buttons

A payment SDK usually drops its wallet button and logic into a cross-origin iframe it controls. Apple Pay and Google Pay on the web run through the browser's Payment Request API, which is one of the features governed by Permissions-Policy.

If the host page never delegates `payment` to that frame, the wallet's payment call is blocked by the browser. The visible result is a widget that renders fine and then does nothing when you try to pay, sometimes with only a permission-policy warning in the console. The SDK can't fix this in its own code, because it can't grant itself a permission the host page withheld. The one-line fix is on the host page: delegate `payment` to the SDK's origin.

### Is it *always* "embedded content can't grant itself permissions"?

No. The precise rule is narrower: **across an isolation boundary, a frame can't exceed what its ancestors delegate.** The looser phrasing breaks down in a few real cases:

- **Same-origin script, not in an iframe.** An SDK loaded as a plain `<script>` runs as part of the page itself. There's no sandbox boundary, so it just has the page's own permissions. "Embedded" here means a frame, not any third-party code.
- **Same-origin iframes** often inherit the parent's permissions by default (the default `self` allowlist includes them), so nothing has to be delegated explicitly.
- **New top-level windows.** Content opened with `window.open` is its own top-level context, not a child frame, so it isn't bound by the opener's policy.
- **User consent and extensions** can add capabilities at runtime. Even then, policy generally has to allow the feature for the frame before a permission prompt can appear, so consent works inside the delegated envelope.

## When does this matter?

- **Embedding a third-party widget or SDK that needs a powerful feature** (payments, camera for a QR scan, geolocation, microphone). If it works in the vendor's own demo but not in your page, suspect delegation.
- **Debugging "it renders but the action does nothing."** A frame that can't use a feature often produces a quiet failure plus a console warning, rather than a loud error.
- **Deciding where a fix goes.** When an embedded thing lacks a permission, the change belongs on the host page (its Permissions-Policy header or the iframe's `allow`), not in the vendor's SDK or on the vendor's side.

## Gotchas

- **The SDK's docs will tell you which features to delegate.** Payment SDKs commonly need `payment`, and some also want `camera`/`microphone`. Check their integration guide for the exact header value.
- **Header vs attribute.** For a cross-origin frame you often need both: the page's Permissions-Policy must allow and delegate the feature, and the iframe `allow` attribute must name it. Missing either one blocks the frame.
- **Same-origin masks the problem.** Something can work when the widget is same-origin in dev and then break once it's served from a real cross-origin CDN. The delegation was never needed until the boundary appeared.

## See also

- [Permissions-Policy — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy)
- [iframe `allow` attribute — MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#allow)
- [CORS — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Iframe widget analytics — the four-hop pipeline](iframe-widget-analytics-pipeline.md) — a companion note on how embedded iframes talk to the parent page
- [Debugging a stuck Apple Pay sheet on the web](../payments/apple-pay-web-debugging.md) — the payment case that this concept came out of
