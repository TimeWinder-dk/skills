---
name: security-hardening
version: 1.0.0
releaseDate: 2026-07-03
description: |
  Authentication, authorization, input validation, CORS, rate limiting, abuse detection,
  review-mode isolation. Use when: implementing auth guards, adding new API endpoints,
  integrating external services, handling user input, or implementing feature gates.
  Covers: Auth chain middleware, multi-level authorization, store review mode sandbox,
  input validation framework, CORS fail-closed, rate limiting + bot detection, API key
  security, audit logging.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Security Hardening Skill

Multi-layered security approach: mandatory auth, fine-grained authorization, isolated review sandbox,
input validation, rate limiting with abuse detection, and immutable audit trail.

## DO NOT USE FOR

- Frontend-only UX validations as sole security controls
- Infrastructure hardening outside application/runtime boundaries
- Legal/compliance sign-off without technical review

## Fallback Workflow

- If full auth-chain migration is blocked, enforce route-level wrappers for all new endpoints first and backfill legacy endpoints in batches.
- If central validators are incomplete, apply strict per-endpoint validation and track migration to shared parsers.
- If full abuse-detection is not ready, deploy baseline rate limiting with conservative thresholds and monitoring.

---

## Layer 1: Authentication (Auth Chain Middleware)

All handlers must use `withAuth`, `withAdmin`, or `withTeamLead` wrapper. No exceptions.

---

## Layer 2: Authorization (Role & Team Checks)

Role and team-based access control with hierarchy inheritance.

---

## Layer 3: Store Review Mode (Isolated Sandbox)

Review accounts run against isolated Review DataContext. All outbound integrations must check `inReviewRun()` and no-op.

---

## Layer 4: Input Validation

Never call `await req.json()` directly; always use typed parsers.

---

## Layer 5: CORS & Rate Limiting

CORS: fail-closed whitelist. Rate limiting: per IP + abuse profile (minute/hour/day).

---

## Layer 6: API Keys & Secrets

Never log, never surface in responses, rotate regularly.

---

## Layer 7: Audit Trail (Immutable)

Append-only, never updated or deleted. Admin cannot mutate.

---

## Security Checklist

- ✅ All handlers wrapped with `withAuth`, `withAdmin`, or `withTeamLead`
- ✅ Authorization checks use `canAccessTeam()`, `isAdmin()`, etc. (server-side)
- ✅ Input validated via `requireString()`, `requireChoice()`, etc. (never raw `req.json()`)
- ✅ Review mode guarded via `inReviewRun()` on all external integrations (SMS, email, Graph, CrewNet)
- ✅ CORS: whitelist only known origins (fail-closed)
- ✅ Rate limiting: per IP + abuse profile (minute/hour/day)
- ✅ API keys: never logged, never returned to client, marked sensitive
- ✅ Audit trail: append-only, never mutated, captures action + delta + reason
- ✅ JWT validation: signature + expiration + issuer verified
- ✅ Team hierarchy inheritance: `canAccessTeamWithHierarchy()` applied
