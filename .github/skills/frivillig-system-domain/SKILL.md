---
name: frivillig-system-domain
description: Apply Volunio's authoritative product boundaries and business rules. Use when designing, implementing, reviewing, or documenting volunteer identity, lifecycle, roles, teams, memberships, events, workplaces, opportunities, shift plans, shifts, staffing, qualifications, attendance, communication, CrewNet migration, or Operations Hub projections. Also use when deciding which product owns data or authorization in a Volunio-related change.
---

# Volunio Domain

Protect Volunio as the standalone source of truth for TimeWinder's volunteer, staffing and shift domain.
Treat technical reuse as subordinate to product ownership.

The technical skill id remains `frivillig-system-domain` for compatibility. Use **Volunio** in new user-facing and product-level wording. Read [the product brand contract](references/product-brand.md) before changing names, product copy or white-label behavior.

## Required workflow

1. Classify the change as one of:
   - Volunio core,
   - an external product's add-on,
   - a read projection,
   - a migration/cutover adapter.
2. Name the authoritative entity, write owner, stable identifier, authorization scope, and user-facing
   product before choosing tables, repositories, routes, or UI components.
3. Read [the domain contract](references/domain-contract.md) for the affected capability.
4. Keep Volunio core functional when Operations Hub, CrewNet, and projection adapters are unavailable.
5. Prove the relevant invariants with server-side tests, including failure and concurrency cases.

Stop and correct the design if it points at an Operations Hub projection or CrewNet record as the
authoritative Volunio model.

## Non-negotiable boundaries

- Make Volunio the sole write master for volunteer identity/lifecycle, Volunio authorization, teams,
  memberships, leadership, events, workplaces, opportunities, shift plans/types/shifts, applications,
  assignments, qualifications, attendance, shift communication, and Volunio reporting.
- Let Operations Hub own only its add-on domains and local authorization. Consume Volunio data through a
  versioned contract and local projections; never require a synchronous add-on write for a core mutation.
- Treat CrewNet only as temporary migration input, parity evidence, one-time import source, or historical
  provenance. Do not design it as a permanent provider or fallback after cutover.
- Maintain one write owner per entity. Make projections idempotent, retryable, observable, and unable to
  roll back a successful Volunio mutation.
- Separate Volunio roles from Operations Hub roles. Never infer administrator or scoped access across
  products without an explicit, documented grant/federation.
- Reuse pure logic, contracts, and UI primitives where useful; do not reuse another product's entity merely
  because its fields look similar.

## Decision test

Before approving a change, answer all of these unambiguously:

- Where is the authoritative record created and updated?
- Which product can grant and revoke the relevant access?
- Can registration, login, profile, event, and shift flows work without Operations Hub?
- Does a projection failure leave the Volunio mutation committed and recoverable?
- Are CrewNet identifiers optional provenance rather than runtime keys?
- Does the UI live in Volunio unless the feature is explicitly an add-on?

If any answer is unclear, do not implement the write path yet.

## Validation checklist

- [ ] Every affected concept has exactly one named master and write path.
- [ ] Volunio authorization resolves from Volunio-owned identity, role, team, and membership stores.
- [ ] Operations Hub tables are projections or add-on data, never hidden Volunio masters.
- [ ] Core success does not depend on projection availability.
- [ ] Staffing rules are enforced server-side and concurrency-tested where capacity/overlap is involved.
- [ ] Lifecycle, impersonation, overrides, grants, revokes, and important mutations are auditable.
- [ ] Offline writes are idempotent and never locally confirm server-authoritative capacity or eligibility.
- [ ] User-facing UI and messages follow all supported locales in the consuming product.
- [ ] Migration is idempotent, non-destructive on empty/failed input, and ends with CrewNet removed from
      normal operation.
