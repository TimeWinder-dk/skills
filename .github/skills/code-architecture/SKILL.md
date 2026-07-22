---
name: code-architecture
version: 1.1.0
releaseDate: 2026-07-22
description: |
  Backend layered architecture: handler → domain service → repository → persistence provider.
  Use when: creating new backend features, adding repositories, implementing business logic,
  building Azure Functions handlers, or onboarding to the codebase structure. Covers: Handler
  pattern with withAuth wrapper, Domain services layer, IRepository interface with multi-provider
  support (SQL/SharePoint/memory), DataContext aggregation, Audit trail immutability,
  File organization conventions, TypeScript + ES5 constraints.
owner: TimeWinder IT
lastReviewed: 2026-07-22
---

# Code Architecture Skill

Proven 4-layer architecture ensuring clean separation of concerns, testability, and provider agnostic design.

## DO NOT USE FOR

- Pure frontend rendering/component styling decisions
- One-off scripts where full layering adds unnecessary complexity
- Infrastructure provisioning concerns (Bicep/Terraform specifics)

## Fallback Workflow

- If full repository abstraction is not feasible in a hotfix, isolate direct provider access behind a temporary adapter and schedule consolidation.
- If DataContext aggregation cannot be completed in one change, add a thin composition function and migrate handlers incrementally.
- If domain service extraction is blocked, keep business rules out of handlers by moving them to local helper modules as an interim step.

---

## Layer 1: Handlers (HTTP Entry Points)

**File Location:** `backend/src/functions/*.ts`  
**Pattern:** Mandatory `withAuth` wrapper; never handle auth inside handler logic.

Handler wraps request, validates input, calls domain service, writes audit.

---

## Layer 2: Domain Services

**File Location:** `backend/src/domain/*.ts`  
**Responsibility:** Business logic, validation, lifecycle state machines, deduplication.

Service contains business rules and state transitions.

---

## Layer 3: Repositories (IRepository Pattern)

**File Location:** `backend/src/repositories/*.ts`

Repository interface contracts multi-provider implementation (SQL, SharePoint, memory).

Concrete repositories with custom SQL, joins, projections, or relation loading are explicit adapters—not
automatic extensions of a generic field map:

- Map every new persisted field in read, create, and update paths as applicable.
- Validate or reject unsupported filter keys. Never silently drop an unknown `where` field, because a
  supposed point lookup can otherwise degrade into “first arbitrary row”.
- Normalize provider-specific runtime types at the repository boundary. In particular, SQL drivers may
  return `BIGINT` as a string while `INT` is a number; canonicalize both sides before equality or Map/Set
  lookup, and preserve values beyond JavaScript's safe integer range when required.
- Quote/bracket reserved table and column identifiers consistently in all DDL and DML.
- Prove custom mappings and filters against the concrete repository/provider. A memory implementation can
  satisfy the interface while hiding SQL mapping, parser, or driver-shape failures.

---

## Layer 4: DataContext Aggregation

**File Location:** `backend/src/repositories/DataContext.ts`

Single entry point for all repository access; ensures consistency and lazy loading.

## Runtime entry-point wiring

Framework discovery is part of the architecture contract. A handler or trigger file is not deployed merely
because tests import it directly:

- Wire every new route, timer, queue, or trigger module into the runtime's real composition root.
- Add a wiring/completeness test that walks the production import graph or inspects registered triggers.
- Verify the built artifact through the same entry point the host loads.

---

## File Organization

```
backend/src/
├── functions/           ← HTTP handlers (60+), each wraps withAuth
├── middleware/          ← authChain, cors, rateLimit
├── auth/                ← authorize, validateToken, planningAuth
├── domain/              ← business logic services (incidentService, etc.)
├── services/            ← external integrations (CrewNet, Zettler, SMS)
├── repositories/        ← IRepository, DataContext, providers
├── config/              ← Zod-validated LoadedConfig
└── api-docs/            ← OpenAPI contract
```

---

## Summary Checklist

- ✅ Handler wrapped with `withAuth` or `withAdmin`
- ✅ Business logic in domain service (never in handler)
- ✅ Repository calls via `DataContext` (never direct SQL/SharePoint)
- ✅ All repository operations implement `IRepository<T>`
- ✅ Audit trail written for state changes
- ✅ Input validation via `requestParse` utilities
- ✅ File organization follows layered structure
- ✅ Tests use `setDataContextForTests()` for mocking
- ✅ Custom repositories explicitly map reads, writes, and supported filters
- ✅ Provider-specific scalar types are normalized at the repository boundary
- ✅ New runtime handlers/triggers are reachable from the production composition root
- ✅ No ES5 forbidden patterns
