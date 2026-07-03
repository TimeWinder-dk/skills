---
name: testing-strategy
description: |
  Practical testing strategy patterns for fast feedback and reliable releases.
  Use when: adding features, refactoring services, changing API contracts, or stabilizing flaky test suites.
  Covers: test-layer boundaries, fixture design, mocking rules, contract testing, regression protection, and CI quality gates.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Testing Strategy Skill

Test scope should match risk. Prefer small, deterministic tests first, then add broader coverage only where it buys confidence.

## DO NOT USE FOR

- Replacing production monitoring with test-only confidence
- Performance/load benchmarking as primary purpose
- Manual QA process documentation unrelated to automated strategy

## Fallback Workflow

- If full integration environment is unavailable, run contract and boundary tests with deterministic fakes.
- If flaky tests block delivery, quarantine with owner and expiry while adding regression coverage on stable layers.
- If CI runtime is constrained, prioritize changed-area suites before full matrix runs.

---

## Layer 1: Test Pyramid Boundaries

Balance fast unit tests with targeted integration and contract coverage.

---

## Layer 2: Fixture & Data Setup

Test data should be explicit, minimal, and easy to reason about.

---

## Layer 3: Mocking & Dependency Control

Mock at process boundaries, not inside pure domain logic.

---

## Layer 4: Contract & Regression Safety

Interfaces and API contracts must fail fast on drift.

---

## Layer 5: CI Quality Gates

CI should optimize signal quality, not just execution volume.

---

## Checklist

- ✅ Test layer matches change risk.
- ✅ Fixtures are deterministic and isolated.
- ✅ Mocks are boundary-focused.
- ✅ Contract drift is automatically detected in CI.
- ✅ Flaky tests have explicit owner and cleanup deadline.
- ❌ No reliance on execution order for correctness.
