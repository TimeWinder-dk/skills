---
name: frontend-design
version: 1.0.2
releaseDate: 2026-07-08
description: |
  Frontend patterns for TWO stacks. FIRST-CLASS: TimeWinder Operations Hub — Vite + React 18 + Capacitor
  PWA (Fluent UI v9 + theme tokens, shared primitives, API client facade, i18n; NOT SPFx, target ES2021).
  LEGACY (SPFx only, CrewNet/tw-tickets): SPFx webparts, Fluent UI, service pattern with AAD auth, session
  storage, ES5 constraints. Use when: building React components, integrating APIs, managing state, styling,
  or (legacy) SharePoint/Teams webpart code. Pick the section matching your repo — SPFx rules are not universal.
owner: TimeWinder IT
lastReviewed: 2026-07-08
---

# Frontend Design Skill

This skill covers **two** frontend stacks. Pick the section for the repo you are in — the SPFx patterns
below are NOT universal.

> ## Applicability / scope
>
> | Stack | Repos | What applies |
> |---|---|---|
> | **Vite + React 18 + Capacitor PWA** (first-class) | **TimeWinder Operations Hub** | Functional React + hooks; **Fluent UI v9** (`<FluentProvider>` at the root + `@fluentui/react-icons`) **alongside** a CSS-variable theme-token system; Vite build (`target: ES2021`). **Not** SPFx — no WebPart / property pane / AAD-client machinery, no ES5 constraint. See `TimeWinder-Operations-Hub/AI.md`. |
> | **SPFx webpart** (legacy) | CrewNet (`src/`), tw-tickets (`src/`) | Everything below: WebPart base class, Fluent UI / Griffel, `httpClientFactory` AAD client, session-storage utils, ES5 constraints. |
>
> Fluent UI v9 + Griffel (`makeStyles`) are used in **both** stacks. The SPFx WebPart / AAD-client /
> session-storage machinery and the ES5 forbidden-pattern list apply to the **legacy SPFx repos only**.

Below this scope note is the **legacy SPFx** guidance: modern React + SPFx pattern with AAD authentication,
Fluent UI design system, caching service layer, and clean component hierarchy.

## DO NOT USE FOR

- Backend authorization or persistence design
- Infrastructure or deployment pipeline decisions
- Non-SPFx stacks where the assumptions do not hold

## Fallback Workflow

- If Fluent UI components are unavailable, keep the same interaction and accessibility semantics using equivalent components.
- If AAD client initialization fails in local/dev contexts, provide a deterministic sample-data mode without changing production auth flow.
- If hooks-based refactor is too large for one change, isolate side effects first, then migrate state/event handlers incrementally.

## Brand Asset Rule (Required)

- Never create ad-hoc logos, favicons, or brand marks for TimeWinder properties.
- For public/docs usage, always reference brand assets hosted in this public `skills` repo (e.g. `docs/favicon.svg`).
- Keep those assets synchronized from official Operations Hub brand files (`frontend/public/favicon.svg` and `frontend/public/favicon.png`).
- If an asset is missing, add the official file first and then reference it — do not invent a replacement.

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
- ✅ Favicon/logo uses official TimeWinder asset (no generated substitute)
- ✅ Error states displayed to user (no silent failures)
- ✅ No ES5 forbidden patterns (check Heft build)
