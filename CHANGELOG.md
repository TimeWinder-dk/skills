# Changelog

All notable changes to TimeWinder Skills are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added

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
