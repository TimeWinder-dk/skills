---
name: observability-ops
version: 1.0.0
releaseDate: 2026-07-03
description: |
  Operational observability patterns for reliable diagnosis, alerting, and incident response.
  Use when: building new backend flows, adding integrations, introducing background jobs, or preparing production rollout.
  Covers: structured logging, metrics, tracing, SLO/alert design, runbooks, and release/rollback observability gates.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Observability & Ops Skill

Observability is a delivery requirement, not an afterthought. Every critical flow must be diagnosable within minutes.

## DO NOT USE FOR

- Replacing security controls with logging-only mitigations
- Local debugging-only tasks that do not require production-grade observability
- Product analytics taxonomy design unrelated to operational reliability

## Fallback Workflow

- If distributed tracing is not yet available, enforce correlation IDs in logs and request headers.
- If metric dashboards are incomplete, define temporary alert queries directly on structured logs.
- If SLO tooling is pending, use explicit error budget thresholds documented in runbooks.

---

## Layer 1: Structured Logging

Logs are machine-queryable events with stable fields (correlation ID, actor, operation, entity, outcome, duration).

---

## Layer 2: Metrics Design

Publish success rate, error rate, p50/p95 latency, saturation/backlog signals.

---

## Layer 3: Distributed Tracing

Trace context follows requests across handlers, services, and integrations.

---

## Layer 4: SLOs & Alerts

Alerts should be actionable and mapped to reliability goals.

---

## Layer 5: Runbooks & Release Gates

Incident handling and release validation are codified.

---

## Checklist

- ✅ Critical flows emit structured logs with correlation.
- ✅ Core SLI metrics and latency percentiles exist.
- ✅ Trace propagation is validated end-to-end.
- ✅ SLOs and actionable alerts are documented.
- ✅ Runbook and rollback criteria are linked to deployment.
- ❌ No secrets or sensitive payload dumps in logs.
