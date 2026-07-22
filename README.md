# TimeWinder Skills

**AI coding agent skills for TimeWinder IT projects** — backend architecture, frontend design, security, testing, observability, and resilience patterns.

**10 skills**, owner: **TimeWinder IT**, last reviewed: **2026-07-22**

---

## Skills Overview

| Skill | Purpose | When to Use |
|-------|---------|------------|
| **code-architecture** | Backend 4-layer (handler → domain → repository → persistence) | New backend features, handlers, business logic |
| **security-hardening** | Auth, validation, review-mode, CORS, rate limiting, audit trails | Auth guards, API endpoints, user input, feature gates |
| **frontend-design** | SPFx webparts, React hooks, Fluent UI, AAD client, caching | New webparts, components, APIs, state management |
| **admin-ui-design** | Admin dashboards, feature toggles, destructive operation guards, audit viewers | Admin interfaces, toggles, bulk operations, auditing |
| **integration-resilience** | Timeouts, retries, idempotency, circuit breakers, fallback handling | HTTP/API integrations, background jobs, notifications, sync flows |
| **observability-ops** | Structured logging, metrics, tracing, SLOs, runbooks, release gates | Backend flows, integrations, background jobs, production rollout |
| **testing-strategy** | Test pyramid, fixtures, mocking, contract tests, CI gates | Features, refactoring, API contracts, flaky stabilization |
| **extract-coding-patterns** | Extract patterns from any codebase into skills | Team conventions, onboarding, standardization, playbooks |
| **frivillig-system-domain** | Frivillig product ownership and business invariants | Identity, authorization, teams, events, shifts, staffing, migration |
| **operations-hub-domain** | Operations Hub ownership and business invariants | Incidents, orders, planning, roles, and Frivillig projections |

---

## Installation

### Via npx (if skills CLI enabled)

```bash
npx skills add timewinder-dk/skills
```

Then select skills:

```
✓ code-architecture
✓ security-hardening
✓ frontend-design
✓ admin-ui-design
✓ integration-resilience
✓ observability-ops
✓ testing-strategy
✓ extract-coding-patterns
✓ frivillig-system-domain
✓ operations-hub-domain
```

### Manual

1. Clone this repo
2. Copy `.github/skills/*` to your project's `.github/skills/` directory
3. Copilot will auto-discover skills with `name:` frontmatter

---

## Quick Start

1. **Planning a new feature?** Check [skills-matrix.md](https://github.com/timewinder-dk/TimeWinder-Operations-Hub/blob/main/docs/ai/skills-matrix.md) to find the right skill(s)
2. **Read the skill** (`.github/skills/<skill>/SKILL.md`)
3. **Use the checklist** at the end to validate your implementation

### Example: Adding a New API Endpoint

- Primary: **code-architecture** (handler pattern, domain service, repository)
- Secondary: **security-hardening** (auth wrapper, input validation, audit)
- Optional: **integration-resilience** (if calling external services)
- Optional: **observability-ops** (logging, metrics, runbook)
- Validate: **testing-strategy** (unit + integration tests)

---

## File Structure

```
.github/
├── skills/
│   ├── admin-ui-design/SKILL.md
│   ├── code-architecture/SKILL.md
│   ├── extract-coding-patterns/SKILL.md
│   ├── frivillig-system-domain/SKILL.md
│   ├── frontend-design/SKILL.md
│   ├── integration-resilience/SKILL.md
│   ├── operations-hub-domain/SKILL.md
│   ├── observability-ops/SKILL.md
│   ├── security-hardening/SKILL.md
│   └── testing-strategy/SKILL.md
└── workflows/
    └── deploy-pages.yml
```

---

## Metadata

- **Owner:** TimeWinder IT
- **Last Reviewed:** 2026-07-03
- **Technology Stack:** TypeScript, React, SPFx, Azure Functions, Azure SQL, SharePoint
- **Constraints:** ES5 target, multi-provider architecture, review-mode isolation

---

## Contributing

To update a skill:

1. Edit the `.github/skills/<skill>/SKILL.md` file
2. Update the `lastReviewed` date in frontmatter
3. Submit a PR with a clear description of changes
4. Merge after review

To add a new skill:

1. Follow the [extract-coding-patterns](./github/skills/extract-coding-patterns/SKILL.md) workflow
2. Create `.github/skills/<skill>/SKILL.md` with YAML frontmatter
3. Include checklist for validation
4. Submit PR

---

## License

MIT License — See [LICENSE](LICENSE)

---

**More information:** [TimeWinder Operations Hub](https://github.com/timewinder-dk/TimeWinder-Operations-Hub)
