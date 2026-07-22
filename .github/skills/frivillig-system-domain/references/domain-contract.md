# Frivillig domain contract

Use this reference after the main skill routes a change into the Frivillig domain.

## Contents

1. Product and ownership model
2. Identity, lifecycle, and authorization
3. Organization, events, and shift structure
4. Staffing and eligibility rules
5. Offline, communication, and impersonation
6. Projection, migration, and cutover

## 1. Product and ownership model

The permanent data direction is:

```text
CrewNet -- one-time migration --> Frivillig master -- versioned projection --> Operations Hub add-on
```

Frivillig must be product-logically standalone even while products share deployment or database
infrastructure. Lack of an add-on may remove add-on information, but must not break registration,
authorization, shifts, signup, attendance, or other core flows.

| Concept | Authoritative owner | Other-product role |
|---|---|---|
| Volunteer identity and lifecycle | Frivillig | Optional person projection |
| Frivillig roles and administrators | Frivillig | Projection only when needed |
| Teams, memberships, and leadership | Frivillig | Team/membership projection |
| Frivillig event | Frivillig | Explicit mapping to a distinct planning event |
| Workplaces and volunteer opportunities | Frivillig | Read model when needed |
| Shift plan, shift type, and shift | Frivillig | Read model/projection |
| Applications, waitlists, and assignments | Frivillig | Read model/projection |
| Qualifications and attendance | Frivillig | Projection when needed |
| Shift-related communication/reporting | Frivillig | Add-on consumption when needed |
| Operations Hub admin and operations roles | Operations Hub | No implicit Frivillig access |
| Planning tasks, annual planning, incidents, orders | Operations Hub | Optional shift briefing into Frivillig |

Keep distinct concepts distinct even when they describe the same real-world team or event. Bind them with
stable external IDs or mapping tables instead of sharing a write master.

## 2. Identity, lifecycle, and authorization

- Create the permanent identity only after both normalized email and normalized mobile have been verified
  with separate OTP challenges. Do not grant active access to a partially verified registration.
- Resolve native Frivillig sessions from the Frivillig identity and Frivillig-owned role/membership stores.
  Do not require an Operations Hub person projection, Entra identity, or planning role.
- Keep role namespaces separate. Frivillig roles include global administrator/volunteer roles and scoped
  event/planner access; team membership and leadership come from the membership authority.
- Prevent a team owner from extending their own scope. Record grantor, time, scope, active/revoked state,
  reason, and revoker for grants and revocations.
- Verify a new email or mobile channel before replacing the old value.
- Treat `blocked`, `dormant`, and `deleted` as identity lifecycle states, not projection flags.
  - Person blocking prevents registration, login, and participation and supports audited unblock/expiry.
  - Dormancy is inactivity cleanup: the user may self-reactivate through a verified channel before the
    retention deadline; only dormancy is self-unlockable.
  - Preserve append-only audit obligations when erasing/anonymizing personal data.
- Return neutral registration/recovery responses where a specific response would reveal whether a person,
  contact channel, or block entry exists.

## 3. Organization, events, and shift structure

Use this hierarchy:

```text
Frivillig event
├── team
│   ├── workplace
│   └── volunteer opportunity
└── shift plan
    ├── shift type
    └── concrete shift
        ├── application/waitlist
        ├── assignment
        └── attendance
```

- Model a master event separately from a concrete event. Master data may provide templates/defaults;
  concrete events own dates, assignments, applications, attendance, and event-specific communication.
- Make event timezone an IANA zone. Store instants in UTC and use the event zone for wall-clock meaning;
  test daylight-saving gaps, duplicates, midnight crossings, and user display preferences.
- Treat a volunteer opportunity as onboarding/discovery, not as a shift type.
- Treat a shift plan as its own entity, not as a child team. A team may have multiple shift plans.
- Support draft/published/locked/archived plan lifecycle and draft/open/full/cancelled/completed shift
  lifecycle as explicit state transitions.
- Let an add-on attach a read-only, versioned shift briefing by stable shift ID. Frivillig must continue
  normally when no briefing exists.

## 4. Staffing and eligibility rules

- Support direct signup, application/approval, and planner-only assignment as distinct modes.
- Approve an application and create its assignment atomically.
- Reject overlapping active assignments globally across teams, workplaces, plans, and events in the same
  environment. Adjacent intervals where one ends exactly as another begins do not overlap.
- Enforce capacity atomically. Two concurrent requests for the last place must not both succeed.
  Require explicit permission, reason, and audit for overbooking.
- Apply the strictest relevant minimum age across opportunity, workplace, shift type, and concrete shift.
- Enforce required qualifications, licenses, certificates, and allowed self-declaration on the server.
  Do not treat an arbitrary tag as verified competence.
- Block signup/assignment for blocked volunteers and respect leave at its global/event/team/workplace scope.
  Never change existing assignments silently; show planners the consequences.
- Before the self-unsignup deadline, allow direct withdrawal. After it, create a request and release the
  place only when an authorized planner approves, rejects, or reassigns it.
- Apply normal overlap, capacity, qualification, and lifecycle rules during impersonation. Model an
  administrative override as a separate authorized and audited action.

## 5. Offline, communication, and impersonation

- Allow read-only “my shifts” data to come from cache with an explicit last-sync/staleness indicator.
- Queue suitable offline attendance mutations with client time, enqueue time, server receipt time,
  idempotency key, actor, effective actor, and impersonation session. Reject conflicts visibly rather than
  overwriting newer server state.
- Require online server validation for signup/application; never confirm capacity, overlap, qualification,
  or approval solely on the client.
- Use push-first delivery for operational shift messages with acknowledgement-based fallback according to
  product policy. Send OTP and security reverification directly through their required channels.
- Scope operational messages to concrete shift, shift type, plan, workplace, team, or tag segment; do not
  turn this capability into an unrelated campaign platform.
- Make impersonation explicitly authorized, time-limited, visibly bannered, and auditable with both actual
  and effective principals. Never cross production/review boundaries.

## 6. Projection, migration, and cutover

- Publish Frivillig data through a versioned contract. Maintain Operations Hub projections with an
  idempotent reconciler/outbox, retries, backoff, health visibility, and source-wins semantics.
- Keep projection processing outside the critical transaction for registration, profile, blocking, event,
  or other Frivillig mutations.
- Import CrewNet records into Frivillig-owned entities first. Build Operations Hub projections only from
  Frivillig after that; never use Operations Hub as an intermediate master.
- Preserve CrewNet IDs only as optional provenance/mapping values.
- Make imports repeatable and difference-reportable. An empty, partial, or failed source response must
  never trigger destructive prune.
- Use dual-read/diff and a time-limited rollback window for cutover validation, not permanent dual-write or
  permanent provider selection.
- Finish cutover by removing CrewNet credentials, jobs, runtime reads, provider branches, and cache reliance
  from normal operation. The system must be able to run an event without CrewNet data.
