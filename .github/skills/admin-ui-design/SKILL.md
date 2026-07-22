---
name: admin-ui-design
version: 1.3.0
releaseDate: 2026-07-23
description: |
  Admin interface patterns for navigation shells, information architecture, feature gates, destructive
  operation guards, audit visibility, and permission-based UI. Use when creating or changing any admin
  route, admin page, dashboard, settings area, configuration UI, role-management surface, or destructive
  operation. Requires a persistent desktop sidebar, responsive mobile navigation, active-state and grouped
  menu entries, and placement of every admin page in the existing product admin shell. Also covers feature
  toggles, confirmation dialogs, audit viewers, role-based visibility, batch safeguards, and approvals.
owner: TimeWinder IT
lastReviewed: 2026-07-23
---

# Admin UI Design Skill

Admin functionality is not a collection of isolated pages. It is one coherent administration product with
stable information architecture, predictable navigation, consistent permissions, and visible auditability.

## DO NOT USE FOR

- Non-admin end-user UX flows
- Backend-only API design without UI implications
- Infrastructure deployment or runtime operations

## Mandatory workflow

Before implementing an admin feature:

1. Inspect the existing admin shell, navigation groups, route structure, design tokens, responsive behavior,
   and scroll model in the target product.
2. Identify the correct existing menu group and route location. Do not invent a detached admin surface.
3. Reuse or extend the existing admin-shell component. Do not replace it with a centered standalone page.
4. Add the route to the admin navigation and active-state logic in the same change.
5. Verify desktop, tablet, mobile, dark mode, keyboard navigation, authorization, and empty states.
6. Add tests that prove the route is discoverable from the admin navigation.

## Mandatory admin shell and navigation contract

### Desktop

Every admin route must render inside the product's persistent admin shell.

- A left sidebar remains visible even when the admin center currently contains only one or two features.
- The sidebar and main content scroll independently when content exceeds the viewport.
- The active route has a clear selected state using the product accent treatment.
- Menu entries are grouped under stable domain headings such as Drift, Planning, Users, Communication,
  Integrations, or the equivalent groups in the target product.
- The sidebar includes the established admin search/filter when the product already has one.
- Main content uses the available admin workspace width. Do not force dense admin pages into a narrow,
  centered public-page container.
- The product-level burger menu is not a substitute for admin-local navigation.

### Tablet and mobile

- Preserve the same information architecture and labels.
- Convert the persistent sidebar to the product's established drawer, switcher, or compact admin navigator.
- Do not remove admin destinations merely because the viewport is narrow.
- The active destination and current group remain understandable.

### Route and menu completeness

- No admin page may be orphaned: every admin route must have a visible navigation entry for every role that
  can access it.
- No navigation entry may lead to an unrelated page or a silent placeholder.
- If a destination is planned but not implemented, omit it or show an explicit disabled/planned state only
  when the product convention allows this.
- Labels describe the managed domain, not the technical implementation. For example, a user-management
  destination is labelled "Volunteers"/"Frivillige", not after an internal table or role-grant type.
- Permission checks filter entries, but must not collapse the entire admin shell for authorized admins.

### Existing-interface parity rule

When another administration surface in the same product family already defines the visual pattern, treat it
as the reference implementation. Match its:

- sidebar width and hierarchy,
- group labels,
- selected-state treatment,
- search placement,
- spacing and density,
- content/sidebar scroll behavior,
- responsive transformation.

Having little content is never a reason to omit the shell. The shell must be established before the admin
area grows, otherwise each feature becomes an incompatible standalone page.

## Principle: Enable much, guard destructive

- Read-heavy: admins can inspect relevant data with appropriate audit visibility.
- Mutation-light: ordinary settings are editable with validation and feedback.
- Destructive-locked: delete, purge, reset, revoke, and bulk actions require explicit confirmation.
- Audit-first: important actions record actor, time, target, reason, and result.

## Feature gating

- Render features conditionally from server-resolved permissions or product feature flags.
- UI visibility is convenience only; backend authorization remains authoritative.
- Do not infer one product's administrator role from another product's role area.
- Keep empty states, unauthorized states, and unavailable-feature states visually distinct.

## Destructive operation confirmation

Destructive or privilege-reducing operations must include:

- a clear destructive title and consequence,
- the affected person/entity/count,
- reason capture when useful for audit,
- explicit confirmation rather than a single accidental click,
- disabled/busy state while executing,
- visible success or recoverable error feedback.

## Bulk operations

- Derive selection, preview, counts, and mutation targets from the same currently filtered scope the user
  can see. “Select all” means all matching rows, not the unfiltered backing collection.
- Keep hidden/non-matching rows out of the mutation unless the UI explicitly offers and names a broader
  scope (for example “all 12,430 records”).
- Reconcile selection when filters change, and state clearly whether selection persists across pages.
- Preview the affected count before execution.
- Cap batch sizes server-side.
- Make retries idempotent where possible.
- Audit the operation and its result.
- Never hide partial failures.

## Audit viewer

An admin audit viewer should be:

- read-only,
- searchable/filterable,
- paginated or virtualized,
- explicit about actor, target, reason, time, result, and errors,
- permission-scoped to the current product/domain.

## Canonical admin CRUD page anatomy (BINDING)

An entity-administration page is not a free-form layout. Every normal admin CRUD surface in a product family
follows ONE anatomy, and a second product in the family (e.g. Volunio next to Operations Hub) must match it —
not merely share color tokens. When another surface in the family already defines the pattern, it is the
reference implementation; reuse its primitives rather than re-inventing per page.

**Page anatomy (top to bottom):**

1. **Page header** — title, a short plain-language description, and the primary action (e.g. "Create X") at
   the top-right. No permanent create-form stacked above the list.
2. **List toolbar** — full-width search; status/domain filters; a sort field + direction; a "show archived/
   deactivated" toggle where relevant; a visible result count; clearly-resettable active filters. Use the
   product's searchable single-select for option lists — never a raw id/GUID text field.
3. **Entity list/grid** — a compact, responsive card/row grid that uses the full admin workspace width
   (multi-column on wide desktop, single column on phones). Each card shows the primary name, secondary
   identifiers, status/role/relation **chips**, and per-row edit + (authorized) destructive actions.
4. **View / New / Edit** — a real detail view (header with name + status + actions, grouped read-only
   sections, relations/counts, audit/provenance) and a shared form shell for create AND edit (same fields,
   validation, busy state, submit/cancel). New opens from the primary action; it is NOT permanently open.
5. **Detail placement** — the detail/edit must never be stacked as a column far below a long list. Open it in
   the canonical modal/sheet (bottom sheet on mobile) or navigate to a dedicated detail route.
6. **Dialogs** — every destructive/privilege-reducing action uses the canonical confirm dialog with a clear
   consequence and the affected entity — never `window.confirm`/`window.prompt`, never a hand-rolled overlay.

**Data-presentation rules:**

- Raw technical ids/GUIDs are never primary content — put them in a "Technical information" section or a
  copyable support detail. Timestamps render through localized formatting, never raw ISO.
- Internal enum values (status, kind, role) are shown as localized user-facing labels, never the raw token.
- Relations to people/teams/events use searchable pickers (name/email/phone for people; name/parent/active
  for teams); the internal id is the hidden value behind the label. Model the administrator's task
  ("Add member", "Make team lead"), not the backing table ("grant", "kind: member/lead").

**Reuse + extraction:**

- Do NOT repeat a local `rowStyle`/card/form-layout object per page. Layout that is shared belongs in the
  shared UI package.
- If the reference product's list/form/detail pattern is not yet packaged reusably, extract the pure visual +
  interaction part into the shared UI package (`@ops-hub/ui`) so both products consume ONE implementation.
  Do NOT copy the reference product's business logic or its domain-specific fields into the other product,
  and do NOT move authoritative entities across the product boundary.
- Prefer extending an existing shared primitive over adding a parallel variant.

**Visual regression evidence:** a CRUD migration is not done on typecheck + unit tests alone. Capture
screenshots (desktop ~1920×1080, laptop ~1366×768, mobile ~390×844; light + dark) and compare density,
working width, action hierarchy and form rhythm against the reference product — clearly the same family.

## Admin UI checklist

- [ ] Every admin route is inside the persistent/responsive admin shell.
- [ ] Desktop sidebar remains visible even with very few menu items.
- [ ] Sidebar and main content have the established independent scroll behavior.
- [ ] Every authorized route has a correctly labelled menu entry.
- [ ] Active route and group are visually clear.
- [ ] Existing product-family admin visual boundaries are reused rather than approximated.
- [ ] Admin content is not incorrectly constrained to a narrow public-page layout.
- [ ] Mobile/tablet navigation preserves the same destinations and hierarchy.
- [ ] Role filtering uses server-resolved permissions and backend guards remain authoritative.
- [ ] Destructive actions require appropriate confirmation and audit reason.
- [ ] Bulk actions preview counts, cap batch size, and report partial failure.
- [ ] Bulk selection, displayed count, preview, and submitted IDs use the same visible/filter scope.
- [ ] Audit trail is visible where operationally relevant.
- [ ] Loading, empty, unauthorized, success, and error states are implemented.
- [ ] Navigation discoverability and responsive shell behavior are covered by tests.
- [ ] CRUD pages follow the canonical anatomy (header+primary action, toolbar, entity grid, view/new/edit, modal/sheet detail).
- [ ] No permanent create-form above the list; new/edit share one form shell; detail is not stacked below a long list.
- [ ] No raw ids/enums/ISO as primary content (technical info section; localized labels + dates).
- [ ] People/team relations use searchable pickers with business-action labels; no raw-id text field; no `window.confirm`.
- [ ] Shared list/card/form layout is extracted to the shared UI package, not duplicated per page; no local `rowStyle` copies.
- [ ] Product-family visual parity verified with light/dark desktop+laptop+mobile screenshots.
