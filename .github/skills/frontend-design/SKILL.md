---
name: frontend-design
description: |
  SPFx webparts, React components, Fluent UI, service pattern with AAD auth, session storage,
  and TypeScript+ES5 constraints. Use when: building new webparts, creating React components,
  integrating APIs, managing component state, or working with SharePoint/Teams frontend code.
  Covers: WebPart base class pattern, React functional components with hooks, Fluent UI design
  system, AAD-authenticated API client, caching data service, session storage utilities,
  responsive dialog patterns, ES5 constraints.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Frontend Design Skill

Modern React + SPFx pattern with AAD authentication, Fluent UI design system, caching service layer,
and clean component hierarchy.

## DO NOT USE FOR

- Backend authorization or persistence design
- Infrastructure or deployment pipeline decisions
- Non-SPFx stacks where the assumptions do not hold

## Fallback Workflow

- If Fluent UI components are unavailable, keep the same interaction and accessibility semantics using equivalent components.
- If AAD client initialization fails in local/dev contexts, provide a deterministic sample-data mode without changing production auth flow.
- If hooks-based refactor is too large for one change, isolate side effects first, then migrate state/event handlers incrementally.

---

## Layer 1: WebPart Base Class

SPFx webpart extends `BaseClientSideWebPart` with property pane configuration, React rendering, and AAD setup.

---

## Layer 2: React Component Pattern

Functional components with `useState`, `useCallback`, `useEffect`, `useMemo` hooks.

---

## Layer 3: Reusable Components (Composition)

Build shared components with clear interfaces and prop management.

---

## Layer 4: AAD-Authenticated API Client

Lazy-load and cache `AadHttpClient` with automatic token handling.

---

## Layer 5: Caching Data Service

Service layer with TTL-aware cache and business logic isolation.

---

## Layer 6: Session Storage Utilities

TTL-aware read/write without quota errors.

---

## Layer 7: Dialog Components (Portal Pattern)

Render dialogs at document root via `ReactDOM.createPortal`.

---

## Layer 8: Fluent UI Design System

Consistent use of Fluent components, icons, and theming.

---

## Frontend Checklist

- ✅ WebPart uses property pane for config (never hardcoded)
- ✅ React components are functional (hooks, not class)
- ✅ `useCallback` wraps event handlers (stable references)
- ✅ `useEffect` used only for side effects (API calls, cleanup)
- ✅ `useMemo` for expensive calculations or service instances
- ✅ API client cached via `httpClientFactory` lazy-load pattern
- ✅ Data service handles caching with TTL
- ✅ Session storage reads/writes fail silently (no quota errors)
- ✅ Dialogs use portal pattern (ReactDOM.createPortal)
- ✅ Fluent UI components used consistently
- ✅ Error states displayed to user (no silent failures)
- ✅ No ES5 forbidden patterns (check Heft build)
