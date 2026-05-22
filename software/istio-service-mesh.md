---
title: Istio / service mesh
category: software
date: 2026-05-21
tags: [infrastructure, networking, microservices, kubernetes]
---

# Istio / service mesh

## TL;DR

A service mesh is the "smart network" layer that sits between all your microservices and handles retries, auth, encryption, and observability — so the services themselves don't have to. Istio is the most common implementation; it works by injecting a sidecar proxy (Envoy) into every pod.

## The explanation

When you have 20+ microservices, each one needs to talk to the others reliably. Without a mesh, every service has to implement its own retry logic, handle mTLS, emit traces, set timeouts, and decide how to route traffic. That's a lot of duplicated work — and it tends to be inconsistent.

A service mesh moves all of that into the network layer instead.

### How Istio works

Istio injects a small proxy (called **Envoy**) as a sidecar into every pod. All traffic in and out of a pod flows through that proxy. The services themselves think they're talking directly to each other; Envoy intercepts and handles everything in between.

```
[Service A]
    ↓  (thinks it's talking to B)
[Envoy sidecar]   ← Istio controls this
    ↓
[Envoy sidecar]   ← and this
    ↑  (thinks it's talking to A)
[Service B]
```

Because Envoy sits in the middle, Istio can give you:

- **Traffic routing** — route requests based on URL prefix, headers, or weights (e.g. "send 10% of traffic to the canary version"). Configured via `VirtualService` resources.
- **Load balancing** — distribute traffic across pod replicas.
- **mTLS everywhere** — automatic mutual TLS between services, with zero app code changes. Every service-to-service call is encrypted and authenticated.
- **Retries + timeouts** — define retry policies and timeouts in the mesh config, not in each service's code.
- **Observability** — distributed traces, per-request metrics, and logs for every service-to-service call. Again, no app code changes needed.

### Key config primitives

- `VirtualService` — defines routing rules (where traffic goes, based on what conditions)
- `DestinationRule` — defines policies for a destination (e.g. load balancing strategy, circuit breaking)
- `Gateway` — controls ingress/egress traffic at the edge of the mesh

### Why this matters at scale

Without a mesh, if you want retries on every service call, you need every team to implement retries. If one team forgets or does it wrong, you have a gap. With a mesh, you configure it once in the infrastructure and it applies uniformly across everything — no trust required in each service's implementation.

## When does this matter?

- **Reading Kubernetes configs** — if you see `VirtualService` or `DestinationRule` resources, that's Istio routing config.
- **Debugging service-to-service failures** — problems might be in the app code or in the mesh config; knowing which layer to look at saves time.
- **Understanding "zero-trust" networking** — mTLS in a mesh is how teams achieve encryption between services without wiring up certificates in every app.
- **Canary deploys** — traffic splitting (`send 5% to v2`) is typically done via Istio `VirtualService` weights, not app logic.
- **Distributed tracing** — when you see traces showing every hop between services, that's the mesh collecting them automatically.

## See also

- [Istio docs — Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [Envoy proxy](https://www.envoyproxy.io/) — the sidecar Istio injects
