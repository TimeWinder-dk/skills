---
name: operations-hub-domain
description: Apply TimeWinder Operations Hub product ownership and business invariants. Use when designing, implementing, reviewing, or documenting public reporting, incidents, operational coordination, order locations, products, orders, refill, planning events/tasks/year-wheel, Ops Hub roles, administration, or projections from the Frivillig product. Always pair this skill with frivillig-system-domain when a change touches people, teams, memberships, roles, events, shifts, qualifications, attendance, or CrewNet migration.
---

# Operations Hub Domain

Protect Operations Hub as the operational coordination and planning add-on. Keep its genuine domain data
authoritative while treating Frivillig-owned data as projections with explicit field ownership.

## Required workflow

1. Classify the capability as:
   - Ops Hub operations core,
   - Ops Hub planning/add-on core,
   - platform/administration support,
   - a projection from another product.
2. If the change touches a person, team, membership, role, event, shift, qualification, attendance, or
   CrewNet-derived value, first read:
   - [Frivillig System Domain](../frivillig-system-domain/SKILL.md), and
   - [Frivillig domain contract](../frivillig-system-domain/references/domain-contract.md).
3. Name the master and write owner for every affected field—not merely the containing table or DTO.
4. Read [the Operations Hub domain contract](references/domain-contract.md) for the affected module.
5. Enforce transitions, scope, deduplication, and mutation rules server-side and prove them in tests.

If the Frivillig skill is unavailable, apply the boundary summary below and flag the missing dependency
before approving any cross-product write.

## Boundary summary

- Let Operations Hub own public reporting, incidents, operational workflows, order/logistics flows,
  planning tasks/year-wheel, Ops Hub-local roles, configuration, and operational administration.
- Let Frivillig own volunteer identity/lifecycle, Frivillig roles/admins, volunteer teams/memberships,
  Frivillig events, workplaces/opportunities, shifts/staffing, qualifications, attendance, and shift
  communication/reporting.
- Treat Operations Hub `Volunteer`, volunteer-team `Team`, memberships, and Frivillig header data as
  projections when linked to Frivillig. Keep only explicitly Ops Hub-local fields writable here.
- Treat an Operations Hub planning event as distinct from a Frivillig event. Bind them through an explicit
  mapping; never share a permanent write master.
- Link planning tasks to Frivillig shifts by stable contract IDs and publish only a read-only briefing back
  to Frivillig. Either product must degrade gracefully when the add-on data is unavailable.
- Never infer Frivillig administrator or scoped access from an Operations Hub role, or the reverse.

## Field-ownership test

For each mutation, write down:

| Question | Required answer |
|---|---|
| What business fact changes? | Name the fact, not only the API or table. |
| Which product owns it? | Exactly one write owner. |
| Is this row mixed? | Separate source-owned and Ops Hub-local fields. |
| How does the other product learn it? | Versioned projection/mapping, not dual-write. |
| What if projection/add-on delivery fails? | The owning mutation remains valid and retryable. |

Stop if ownership is described only as “shared”, “reused”, or “already in this table”.

## Validation checklist

- [ ] Every mutated field has a named product owner.
- [ ] Frivillig domain contract was read for every overlapping person/team/event/shift concern.
- [ ] Operations Hub writes only its core or explicitly local fields on mixed projection entities.
- [ ] Incident and order state transitions use the canonical server-side state machines.
- [ ] Public reporting is neutral, rate-limited/abuse-protected, and cannot bypass server routing.
- [ ] Incident deduplication and priority monotonicity are preserved.
- [ ] Planning dependencies, dates, scopes, and destructive sync/recalculation are validated and audited.
- [ ] Cross-product mappings use stable IDs and idempotent projection flows.
- [ ] Projected team/member/leader state derives from the Frivillig organization and membership model, not
      event-owned copies or role-store writes.
- [ ] Operations Hub event roles and mappings do not grant Frivillig event scope.
- [ ] Ops Hub roles authorize only Ops Hub capabilities unless an explicit federation says otherwise.
