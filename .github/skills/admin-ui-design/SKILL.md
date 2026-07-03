---
name: admin-ui-design
description: |
  Admin interface patterns with feature gates, destructive operation guards, audit trail visibility,
  and permission-based UI customization. Use when: designing admin dashboards, adding admin features,
  implementing destructive operations (delete, reset, purge), adding configuration UI, or deciding
  what UI elements to display based on user role. Covers: Admin property pane pattern, feature
  toggles, confirmation dialogs for destructive ops, audit log viewer, role-based UI visibility,
  admin-only form fields, batch operation safeguards, approval workflows.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Admin UI Design Skill

Feature-rich admin interfaces with fine-grained control: toggle settings, perform destructive operations
with confirmation, view audit trail, all while maintaining security and auditability.

## DO NOT USE FOR

- Non-admin end-user UX flows
- Backend-only API design without UI implications
- Infrastructure deployment or runtime operations

## Fallback Workflow

- If Fluent UI components are unavailable, keep the same guard semantics with minimal equivalent controls.
- If audit-log UI cannot be delivered immediately, still enforce server-side audit writes and expose a temporary read endpoint.
- If confirmation dialog components fail, use a typed confirmation input pattern before destructive operations.

## Principle: "Enable Much, Guard Destructive"

**Philosophy:**
- ✅ **Read-heavy**: Admins can view anything (with audit trail)
- ✅ **Mutation-light**: Most settings changeable, but with confirmation dialog
- ✅ **Destructive-locked**: Delete, purge, reset operations require explicit multi-step approval
- ✅ **Audit-first**: Every action logged; admins see their own audit trail

---

## Layer 1: Admin Property Pane Pattern

Admin feature toggles in property pane with sensible defaults (mutations disabled).

## Layer 2: Feature Toggling in Components

Render features conditionally based on property flags.

## Layer 3: Destructive Operation Confirmation

Multi-step confirmation with reason capture and checkbox agreement.

## Layer 4: Bulk Delete Panel

Preview count, execute with confirmation, audit each record.

## Layer 5: Audit Log Viewer (Read-Only)

Searchable, paginated, displays reason, result, error messages.

---

## Admin UI Checklist

- ✅ Feature flags in property pane (enable/disable mutations)
- ✅ Read-only features always visible; mutations hidden by default
- ✅ Destructive operations have red UI warning + confirmation dialog
- ✅ Confirmation requires reason + checkbox agreement
- ✅ Affected count previewed before deletion
- ✅ Audit trail visible (read-only, searchable, paginated)
- ✅ All admin actions logged with reason + result
- ✅ Batch size limits enforced
- ✅ Error handling + retry capability
- ✅ Admin can see their own audit trail
