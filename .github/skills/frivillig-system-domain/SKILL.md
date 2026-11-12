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

## Volunio admin-UI design boundaries

General admin-UI design rules live in `admin-ui-design`/`frontend-design`; only the domain-specific ones
belong here:

- Volunio admin surfaces write to the Volunio master (`frivillig.*` schema) — never Operations Hub
  `Volunteer`, `planning.Event`, Ops Hub `Team` or Ops Hub roles. Operations Hub may be used as the visual/
  interaction reference (shared `@ops-hub/ui` primitives) without becoming a write-master.
- Projected Operations Hub data shown inside Volunio (e.g. projection health/outbox) is a separate,
  read-only view — clearly distinct from the authoritative Volunio data the page edits.
- Team membership and team-leadership are administered team-centrically (Team → Members/Team leads) with the
  invariant team-lead ⊆ member enforced in the UI (adding a lead ensures membership; removing a member who is
  also a lead surfaces the cascade). The person-centric view is secondary; both write the same membership
  service. Team roles (`team_member`/`team_owner`) are membership-derived and are not grantable in the generic
  role surface.

## Transition-period identity and membership rules (CrewNet → Volunio → Operations Hub)

Learned in production (T-100-139, 2026-11-12). They apply until the CrewNet sync is stopped at cutover.

- **Merge identities in Volunio, never only in the Operations Hub projection.** An Ops Hub-only merge leaves
  the loser profile linked (`frivillig_user_id`) to a still-live Volunio identity, and the projection consumer
  re-activates it on its next run — the merge silently undoes itself. Ops Hub's merge must move the link to
  the keeper; two profiles linked to two different Volunio identities must be refused and routed to the
  Volunio merge first.
- **The duplicate scan never links on name alone.** AD/Entra-provisioned identities typically have a work
  email and no phone or birthday, while the same person's CrewNet identity has a private email — so the pair
  is never proposed. Offer an explicit manual pair merge (same preview + blockers). Keep the AD identity (it
  owns `entra_oid`, i.e. login) and absorb the CrewNet identity; the keeper inherits `crewnet_user_id`.
- **After a merge two rows carry the same CrewNet id** (there is no unique index on it). Every lookup by
  CrewNet id must prefer the non-deleted identity, or the sync writes to — or revives — the tombstone.
- **Membership chain:** CrewNet workplace membership (`is_supervisor` ⇒ lead) in the *active* CrewNet event →
  `frivillig.Membership` (`source='crewnet'`) via the scheduled sync's `memberships` step → projection →
  Ops Hub `planning.TeamMembership` (keyed `frivillig_membership_id`) + team-scoped `team_member`/`team_owner`
  (`source='frivillig-projection'`). In Ops Hub the `Volunteer` role exists only as per-team `team_member`
  rows, so a person with no team shows no role at all.
- **CrewNet membership is event-scoped.** `/v1/users?event_id=…` returns every organisation user regardless
  of the event; "takes part in the event" means a workplace membership or category application in that event.
- **Revocation is owned, never inferred.** The CrewNet import revokes only its own `source='crewnet'` grants
  and only for workplaces read completely; a suppressed/failed read must surface as incomplete, never as an
  empty member list. The Ops Hub projection only stamps and removes rows it created — admin- or Entra-made
  rows are never adopted or removed.
- **No Volunio → CrewNet writeback.** To get a user into the active CrewNet event, guide the user to CrewNet;
  the API's apply/add endpoints exist but writing back is forbidden by the transition architecture.

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
