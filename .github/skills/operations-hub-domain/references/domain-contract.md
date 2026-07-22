# Operations Hub domain contract

Use this reference after the main skill classifies a change as Operations Hub core, add-on, support, or
projection work.

For overlapping concepts, the Frivillig domain contract controls source ownership; this contract controls
only Operations Hub-local behavior and projection consumption. If an explicit owner decision changes that
boundary, update both domain skills in the same change before implementation.

## Contents

1. Product scope and ownership
2. Cross-product projection boundaries
3. Public reporting and incidents
4. Orders and operational logistics
5. Planning and year-wheel add-on
6. Shared communication infrastructure
7. Roles, administration, and operational safeguards

## 1. Product scope and ownership

Operations Hub is a mobile-first operations product with two primary modules:

- **Operations:** public QR reporting, incidents, team handling, operational status, order/logistics flows,
  notifications, abuse protection, and operational master data.
- **Planning add-on:** planning events/templates, date groups, tasks/subtasks/dependencies, assignments,
  boards, checklists, documents, team requests, year-wheel, readiness/workload, export/import, and archive.

Operations Hub owns these business concepts:

| Capability | Authoritative Ops Hub concepts |
|---|---|
| Public intake | Reporting point, problem type, location/area classification, error report |
| Operational cases | Incident, incident timeline, priority, routing, SLA/escalation, public-board state |
| Orders/logistics | Order location, product/catalog, order and lines, delivery/refill workflow |
| Planning | Planning event, date group, template, task, subtask, dependency, assignment, checklist, team request |
| Local authorization | Ops Hub administrator, drift coordinator, tester, explicitly Ops Hub-local coordination/planning access |
| Operational support | Ops Hub message intent/templates, abuse decisions, admin audit, app settings, documents, analytics |

Operations Hub does not own volunteer-team leadership merely because that relation is used in operational
flows. Shared communication transport is also not an Operations Hub business domain; Operations Hub consumes
that infrastructure for its own authorized messages.

Do not interpret physical co-location in one database/deployment as shared business ownership.

## 2. Cross-product projection boundaries

Always read the sibling Frivillig domain contract before changing an overlapping concept.

### People

- A Frivillig-linked person has Frivillig-owned identity/lifecycle/contact source fields projected into
  Operations Hub. Update the source first through the Frivillig contract; reconcile the projection later.
- Operations Hub may own local display/preferences/device/operational-role fields where explicitly defined.
- An internal Entra-only staff person without a Frivillig identity may remain wholly Operations Hub-local.
- A migrated or administratively provisioned identity may be projected with provenance and partial
  verification state. Do not treat projection existence as proof of native active access.
- Never use projection availability as a prerequisite for Frivillig registration or login.

### Teams, memberships, and leadership

- A volunteer/vagt team, its membership, and leadership are Frivillig-owned projections in Operations Hub.
- CrewNet supervisor/team-leader relations migrate into the Frivillig membership/leadership authority first;
  Operations Hub receives them only as Frivillig projections after migration/cutover.
- Project one organization-level Frivillig team and represent event participation with explicit associations;
  never create an event-owned copy of the team.
- Treat projected `team_member` and `team_owner` as derived views of Frivillig membership and leadership
  authority, not as writes to an Operations Hub or general role store.
- Operations Hub may use projected volunteer-team leadership for incident/team scope, filtering, phonebook,
  planning views, reporting points, ordering, and similar operational flows.
- Operations Hub must not grant, revoke, or maintain a competing local team-lead record for the same business
  relationship.
- A genuinely Operations Hub-only coordination scope may be locally owned, but name and model it distinctly,
  for example as an operational coordinator role rather than “team lead”. Do not hide both meanings behind
  one ambiguous role or team write path.
- Authorization may consume projected relations, but only Frivillig can grant/revoke Frivillig scope.

### Events and shifts

- A planning event is Operations Hub-owned and carries planning templates, dates, tasks, dependencies, and
  year-wheel state.
- A Frivillig event is separately Frivillig-owned and carries volunteer/shift lifecycle.
- Bind them through an explicit mapping and projected header fields. Keep one write path per event.
- An Operations Hub planning owner or administrator role does not authorize creation or management of the
  mapped Frivillig event. Frivillig administrators bootstrap new event scope; `event_owner` operates only
  within scope granted by Frivillig.
- Creating an event mapping does not grant Frivillig event scope.
- Reference Frivillig shifts by stable contract IDs. Store planning-owned task links locally and publish a
  compact read-only briefing to Frivillig; do not copy the planning task master.
- Consume Frivillig projections locally and asynchronously. Normal Operations Hub requests must not require a
  live synchronous Frivillig call when a local projection exists.
- Surface projection version, last successful sync, stale/degraded status, and reconciliation health where
  operationally relevant. Never write Frivillig-owned fields back through the projection.

## 3. Public reporting and incidents

Use this flow:

```text
Public QR token + report
  -> neutral validation, rate limit, abuse scoring
  -> ErrorReport
  -> Allow: create or merge Incident
     Quarantine: retain report without Incident
     Block: neutral rejection
```

- Never expose abuse scores, internal decisions, raw IPs, or whether a reporter is specifically blocked.
- Route incidents from reporting point/problem type on the server. Frontend choices cannot broaden scope.
- For the same reporting point and problem type, merge into an existing open incident instead of creating a
  duplicate: increment report count, update last-report time, append context, and never lower priority.
- Append every meaningful transition to the incident timeline/audit trail.
- Apply only valid state transitions:
  - `New` -> assigned/on-the-way/in-progress/cancelled
  - `Assigned` -> on-the-way/in-progress/blocked/resolved/cancelled
  - `OnTheWay` -> in-progress/blocked/resolved/cancelled
  - `InProgress` -> blocked/resolved/cancelled
  - `Blocked` -> assigned/in-progress/resolved/cancelled
  - resolved/cancelled are terminal
- Scope incident reads/actions to the responsible teams and explicit Ops Hub roles. Treat public-board
  publication, reassignment, destructive image handling, and overrides as separate authorized actions.

## 4. Orders and operational logistics

- Keep an order location distinct from a reporting point even when UI patterns look similar.
- Distinguish order flows by location kind and fulfillment owner; display access must not imply administrative
  create/update/delete rights.
- Store orders and lines as one business aggregate and validate product activity/quantity at creation.
- Apply only valid order transitions:
  - `New` -> picking/rejected/cancelled
  - `Picking` -> on-the-way/rejected/cancelled
  - `OnTheWay` -> delivered/cancelled
  - delivered/rejected/cancelled are terminal
- Record actor, time, reason, and source for manual and automatic order transitions.
- Keep catalog seeding a UI/read-model concern. Refill automation must remain activity/balance-driven so an
  item never stocked does not appear as zero stock and trigger an order for the entire catalog.
- Make automatic refill idempotent, opening-hours aware, auditable, and subject to rejection cooldown.
  Clearly distinguish automatic proposals/orders from manual ones.

## 5. Planning and year-wheel add-on

- Model a planning master template separately from a concrete planning event; protect locked concrete events
  from unintended master overwrite.
- Keep tasks/subtasks, assignments, dependencies, checklists, labels, materials, documents, and activity
  attached to the planning event—not to the Frivillig event master.
- Validate dependency cycles, date rules, assignment totals, and state transitions server-side.
- Treat recalculation and sync-from-master as explicit, destructive event-wide operations. Preview impact
  where practical and audit actor, time, operation, and changed count.
- Resolve team/user data needed by planning from Operations Hub-local data or Frivillig projections according
  to field ownership. Do not turn a planning relationship into a Frivillig authorization grant.
- Publish shift briefings through a versioned, idempotent contract keyed by stable Frivillig shift IDs.
  Frivillig remains usable without a briefing; Operations Hub planning remains usable if publishing is
  temporarily unavailable.

## 6. Shared communication infrastructure

- Operations Hub owns the business decision to send messages for Ops Hub-owned workflows: incident/order/
  escalation triggers, template semantics, audience, recipient resolution, authorization, fallback policy,
  and business audit context.
- Frivillig independently owns the corresponding business decisions for shift/event/team communication.
- Both products use a shared communication gateway for SMS, push, and e-mail transport.
- The shared gateway owns provider integration, sender-ID registry, balance/usage retrieval, delivery receipts,
  retries, backoff, idempotency, channel rate limits, Review Mode suppression, and the technical delivery log.
- The gateway must not interpret Frivillig or Operations Hub business roles and must not independently decide
  recipients. The owning domain authorizes the actor and resolves the intended recipients before submission.
- Every send command must identify the owning product/domain, message type, actor and effective actor,
  recipient identifiers or an immutable resolved-recipient snapshot, sender identity, idempotency key,
  correlation/audit ID, fallback policy, and relevant incident/order/event/team/shift references.
- Keep a domain audit log explaining why the message was sent and a separate shared technical delivery log
  explaining provider/channel attempts and outcomes.
- Sender IDs, balances, and technical delivery logs may be administered through a shared operational surface,
  but permissions to send a specific business message remain owned by the sending product.
- A failure in the shared gateway must be visible and retryable, but must not corrupt the owning business
  transaction. Use an outbox or equivalent post-commit delivery model for non-security messages.
- OTP and security reverification follow their dedicated security-channel policy and must not be delayed by
  ordinary campaign/operational fallback rules.

## 7. Roles, administration, and operational safeguards

- Keep Ops Hub roles local: administrator, drift coordinator, tester, and explicitly Ops Hub-local
  coordination/planning roles. Never derive Frivillig administrator/event/team scope from them.
- Do not model the projected Frivillig volunteer-team leader as an Ops Hub-local role. Consume the projected
  leadership relation for documented Ops Hub capabilities.
- Enforce authorization server-side. Navigation/filtering is only a convenience and cannot grant access.
- Let drift coordinators act across operational teams only within their documented operational powers;
  do not let that role assign roles or inherit full administrator settings access.
- Audit important mutations in the correct append-only trail. Keep actor/effective actor distinct during
  any impersonation or delegated flow.
- Keep public intake endpoints explicitly public and hardened; keep all other routes consciously guarded.
- Run review accounts against isolated data and suppress real external effects. Operational projections and
  integration failures must degrade visibly without corrupting the owning domain.
