---
name: admin-ui-design
version: 1.1.0
releaseDate: 2026-07-22
description: |
  Admin interface patterns for navigation shells, information architecture, feature gates, destructive
  operation guards, audit visibility, and permission-based UI. Use when creating or changing any admin
  route, admin page, dashboard, settings area, configuration UI, role-management surface, or destructive
  operation. Requires a persistent desktop sidebar, responsive mobile navigation, active-state and grouped
  menu entries, and placement of every admin page in the existing product admin shell. Also covers feature
  toggles, confirmation dialogs, audit viewers, role-based visibility, batch safeguards, and approvals.
owner: TimeWinder IT
lastReviewed: 2026-07-22
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
- [ ] Audit trail is visible where operationally relevant.
- [ ] Loading, empty, unauthorized, success, and error states are implemented.
- [ ] Navigation discoverability and responsive shell behavior are covered by tests.
