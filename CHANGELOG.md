# Changelog

All notable changes to TimeWinder Skills are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Changed

- **billetto-platform / billetto-ops-hub (2026-11-10 sync from Operations Hub)** — ticket types in 2026708 read via
  the API (ids, prices, hidden flags, which types are price sources for personaleforeningsbilletter), how Billetto
  packages are built (items are ordinary types; first child paid, further children 0 kr; 2+2 = 2+3 price),
  canceled duplicate events still listed; the price feed as a price source (standard price only, rename ⇒ 503).
  Also brings over the Ops Hub notes from 2026-11-05…09 (code `quantity` semantics, personal `/select/<CODE>`
  link, exhibitor import target moved to 2026708, ticket banner owner decision).

### Added

- **economic-api 1.0.0 (2026-11-09)** — Visma e-conomic REST API (`restapi.e-conomic.com`) for TimeWinder:
  two-token model (app secret + agreement grant) and how an agreement admin grants the app "Timewinderudtræk",
  Key Vault secret names, resource model (customers, products, draft → booked invoices, master data), pagination/
  filter/sort/idempotency/error conventions verified against the schemas and the demo agreement, app roles, the
  hard limits (no versioning, GET-only demo, booked = immutable, no published rate limit), payment verification on
  `remainder == 0` of the invoice matched by `references.other`, Operations Hub wiring pattern, and a time-boxed
  spike plan that must run before any build task (owner decision).

- **billetto-platform / billetto-ops-hub (2026-11-05 sync)** — Synced with the Operations Hub overlays:
  own barcodes + CSV import as the supported way to issue exhibitor tickets (verified end to end, incl. the
  import UI's mapping pitfalls), terminal restriction requirement for scanning, Billetto's final answer that
  `tickets/generate` cannot be opened for organiser key pairs, account update 2026-11-05, and the key *pair*
  (`BILLETTO-API-KEY-ID` / `-SECRET`) instead of a single `BILLETTO_API_KEY`. Overview of everything:
  Operations Hub `docs/billetto/README.md`.
- **billetto-platform 1.0.0** — Shared Billetto knowledge for TimeWinder's 2027 vendor switch: event → ticket
  type → package/group model, Basket, campaigns/codes, scanning (Access Control, Multi Scanner, terminal
  restrictions, exit codes), money/plan facts, the hard limits (no external check-in, whole-ticket refunds,
  one PDF per order), organiser API v3 + webhook signature reference, and a dated snapshot of TimeWinder's
  account (events, decisions, open items, contacts).
- **billetto-web-sales 1.0.0** — timewinder.dk (WordPress/Inspiro) as the sales channel: buy-button → Smart
  Link → event page as default (Basket), widget rules, sales-page blueprint, measurement (Smart Links, UTM,
  GA4), conversion levers (drops, early bird, codes, waiting list, chat widget), consent/headers hygiene.
- **billetto-tw-tickets 1.0.0** — Vendor swap inside tw-tickets behind the unchanged `TicketsArrivals`
  contract: `BillettoService` mapping (entrance scans only, admission states, ticket type names), vendor
  switch setting, optional HMAC-verified webhook route, `/diag` additions, dual-vendor period rules.
- **billetto-ops-hub 1.0.0** — Operations Hub consumption paths (read tw-tickets table by default, direct
  adapter or webhook when justified), wristband rules on real ticket types (family multiplier retired for
  scans), pre-day ordering signal, cached public counter, deposit refund list, route-guard/registry/Key Vault
  wiring and test fixtures.
- **billetto-own-ticket-app 1.0.0** — Decision table and architecture for an own gate app: Billetto cannot
  receive external check-ins, so the app owns a scan ledger; projection minimisation, local validation,
  offline conflicts, terminal identity = Ops Hub entrance, reconciliation and rehearsal plan.

- **frivillig-system-domain** — Added the authoritative Frivillig product boundary, identity and
  authorization separation, event/shift hierarchy, staffing invariants, standalone behavior, and
  CrewNet-to-Frivillig-to-Operations-Hub migration/projection contract.
- **operations-hub-domain** — Added Operations Hub ownership for public reporting/incidents,
  order/logistics flows, planning/year-wheel, local roles and administration, with a mandatory
  cross-check against `frivillig-system-domain` for every overlapping person/team/event/shift concern.

### Changed

- **frontend-design 1.2.0** — Added cross-stack admin CRUD parity invariants: products in one family share
  `@ops-hub/ui` (Operations Hub, Volunio) and their list/detail/form patterns must match; never present raw
  ids/enums/ISO as primary UI (technical-info section, localized labels/formatters, searchable relation
  pickers with the id hidden behind the label); compose the shared form-shell/list-toolbar/entity-card/detail
  primitives instead of a per-page layout.
- **frontend-design 1.1.0** — Added cross-stack invariants for canonical UI primitives, native link
  behavior, locale-aware controls, auth/public clients, semantic version comparisons, exhaustive UI maps,
  verified package exports, and safe generated `calc()` expressions.
- **admin-ui-design 1.3.0** — Added the BINDING canonical admin CRUD page anatomy (header + primary action,
  list toolbar, full-width entity grid, view/new/edit in a modal/sheet, technical-info for raw ids, localized
  enums/dates, business-action searchable pickers, shared-layout extraction, and light/dark desktop+laptop+
  mobile visual-parity screenshots), with matching checklist items.
- **admin-ui-design 1.2.0** — Required bulk selection, counts, previews, and submitted targets to share the
  same visible/filter scope.
- **frivillig-system-domain** — Added Volunio admin-UI design boundaries: Volunio admin surfaces write the
  Volunio master (`frivillig.*`) and never Operations Hub masters; projected Ops Hub data is a separate
  read-only view; team membership/leadership is administered team-centrically with the team-lead ⊆ member
  invariant enforced in the UI.
- **code-architecture 1.1.0** — Added explicit custom-repository mapping/filter rules, provider scalar
  normalization, reserved identifier handling, concrete-provider tests, and production entry-point wiring.
- **integration-resilience 1.1.0** — Added durable cross-invocation coordination and adapter-level
  review/kill-switch suppression for HTTP, timer, and queue entry points.

## [1.0.0] — 2026-07-03

### Initial Release

**8 core skills** for TimeWinder IT development teams:

#### Added

- **code-architecture** (v1.0.0) — Backend 4-layer pattern (handler → domain → repository → persistence). Multi-provider DataContext for SQL/SharePoint/memory.
- **security-hardening** (v1.0.0) — Auth chain, role-based authorization, input validation, review-mode isolation, CORS, rate limiting, audit trails.
- **frontend-design** (v1.0.0) — SPFx webparts, React hooks, Fluent UI, AAD authentication, caching service layer, session storage utilities.
- **admin-ui-design** (v1.0.0) — Admin dashboards, feature toggles, destructive operation guards, confirmation dialogs, audit log viewers.
- **integration-resilience** (v1.0.0) — Timeouts, retries with jitter, idempotency, circuit breakers, fallback handling, observable failures.
- **observability-ops** (v1.0.0) — Structured logging, metrics, distributed tracing, SLOs, alerts, runbooks, release gates.
- **testing-strategy** (v1.0.0) — Test pyramid, fixtures, mocking rules, contract testing, regression protection, CI quality gates.
- **extract-coding-patterns** (v1.0.0) — Workflow for extracting reusable patterns from any codebase into shareable, maintainable skills.

#### Infrastructure

- GitHub Pages documentation site (live at https://timewinder-dk.github.io/skills)
- GitHub Actions workflow for automatic Pages deployment
- Repository structure with `.github/skills/` convention
- MIT License
- README with quick start and skills matrix link

### Skills in This Release

All 8 skills are at version **1.0.0** and ready for production use.

### Repository

- **Repository**: https://github.com/TimeWinder-dk/skills
- **Owner**: TimeWinder IT
- **License**: MIT
- **Documentation**: See [README.md](./README.md) and [Skills Matrix](https://github.com/timewinder-dk/TimeWinder-Operations-Hub/blob/main/docs/ai/skills-matrix.md)

---

## Version History

### Versioning Strategy

We follow [Semantic Versioning](https://semver.org/):

- **PATCH** (1.0.x) — Bug fixes, clarifications, typo corrections
- **MINOR** (1.x.0) — New sections, extra guidance, non-breaking improvements
- **MAJOR** (x.0.0) — Breaking pattern changes, major refactors (rare)

### Update Workflow

When updating a skill:

1. Update the `version` field in SKILL.md frontmatter
2. Update the `releaseDate` field to today's date
3. Add a new section to CHANGELOG.md under `[Unreleased]`
4. Create a git commit with message like `feat(security-hardening): v1.1.0 - add MFA pattern`
5. Create a git tag: `git tag -a v1.1.0 -m "security-hardening v1.1.0 + observability-ops patch"`
6. Push: `git push origin main --tags`
7. Create a GitHub Release with CHANGELOG excerpt

### Breaking Changes

If a skill has breaking changes (major version bump):

1. Clearly document migration path in a new "Migration" section
2. Link to old version for reference
3. Add deprecation notice at top of skill if needed
4. Announce in project communications

---

## Downstream Users

If you're using these skills:

1. **Track versions**: Pin to specific skill versions if your codebase depends on specific patterns
2. **Check CHANGELOG**: Before pulling in new versions, review what changed
3. **Test locally**: Always test imported skills in your project before shipping
4. **Feedback**: Report issues or suggest improvements via GitHub Issues

---

**Last Updated**: 2026-07-22
