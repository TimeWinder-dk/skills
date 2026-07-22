---
name: integration-resilience
version: 1.1.0
releaseDate: 2026-07-22
description: |
  Resilience patterns for external service integrations across backend services and workers.
  Use when: adding or changing HTTP/API integrations, background jobs, notifications, or sync flows.
  Covers: timeouts, retries with jitter, idempotency, circuit-breaker behavior, fallback handling, and failure observability.
owner: TimeWinder IT
lastReviewed: 2026-07-22
---

# Integration Resilience Skill

External dependencies fail in real systems. This skill defines safe defaults that prevent cascading failures and duplicate side effects.

## DO NOT USE FOR

- Pure in-process domain logic without external dependencies
- Security policy design unrelated to integration failure modes
- UI-only interaction patterns

## Fallback Workflow

- If circuit breaker implementation is unavailable, enforce strict timeout budgets and reduced retry counts as a temporary safeguard.
- If idempotency storage is unavailable, disable automatic retries for side-effecting operations until dedup support exists.
- If telemetry pipeline is delayed, emit structured logs with operation and attempt context as minimum observability baseline.

---

## Layer 1: Call Contracts

Every integration call has explicit timeout, classification, and error mapping.

---

## Layer 2: Retry Strategy

Retry only transient failures with bounded exponential backoff and jitter.

**Retry on:** 408, 429, 5xx, network timeouts  
**Never retry:** client errors (4xx except 408/429) without idempotency key

---

## Layer 3: Idempotency & Deduplication

Side effects must be safe under retries, replays, and at-least-once execution.

Any coordination that must survive another invocation—dedup keys, cursors, throttles, leases, pause state,
or retry progress—must use durable storage. Module/global memory is only an optimization or local fallback:
serverless scale-to-zero, process restarts, and parallel instances make it non-authoritative. Choose storage
that does not wake or depend on the resource the coordination is intended to protect.

---

## Layer 4: Circuit Breaker & Fallback

Protect the system from repeated upstream failure bursts.

---

## Layer 5: Telemetry & Operations

Integration health must be visible and actionable.

## Layer 6: Side-effect suppression

Review/sandbox mode, operator kill switches, and maintenance gates must suppress outbound effects at the
integration choke point, not only in one caller:

- Resolve request/job-level policy once at each HTTP, timer, and queue entry point and propagate it through
  the execution context.
- Re-check suppression synchronously in the adapter immediately before the external effect.
- Background workers require explicit policy resolution because they do not inherit HTTP request context.
- Define which dependencies are effects versus internal data stores; do not let an ambiguous kill switch
  partially disable a workflow.
- A suppressed operation must have a deterministic no-op/deferred result and appropriate audit/telemetry.

---

## Checklist

- ✅ Timeout defined for every outbound call.
- ✅ Retry policy scoped to transient failure classes.
- ✅ Idempotency strategy present for side effects.
- ✅ Circuit-breaker and fallback behavior documented.
- ✅ Metrics/logs support incident triage.
- ✅ Cross-invocation coordination uses durable state rather than process memory.
- ✅ Outbound effects enforce review/kill-switch policy at the adapter choke point.
- ✅ Timers and queue workers resolve suppression policy explicitly.
- ❌ No unbounded retries or infinite queues.
