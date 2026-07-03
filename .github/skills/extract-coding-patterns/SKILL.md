---
name: extract-coding-patterns
version: 1.0.0
releaseDate: 2026-07-03
description: |
  Generic workflow for extracting reusable coding patterns from any codebase and turning them into shareable skills.
  Use when: documenting team conventions, onboarding developers, standardizing implementation approaches, or creating reusable engineering playbooks.
  Covers: Scope selection, exploratory review, pattern extraction, skill authoring, validation criteria, maintenance guidance, and when-not-to-use rules.
owner: TimeWinder IT
lastReviewed: 2026-07-03
---

# Extract Coding Patterns Skill

This skill is repository-agnostic. It converts observed implementation patterns into actionable, reviewable skill documents.

## DO NOT USE FOR

- Writing production feature code directly
- Full architecture redesign without code evidence
- Audits that require legal/compliance interpretation beyond engineering patterns

## Fallback Workflow

- If only partial code access exists, produce provisional patterns marked as "confidence: low" and list missing evidence.
- If no recurring pattern is found, document anti-patterns and decision criteria instead of forcing a pattern.
- If scope is too broad, split into multiple pattern extraction passes per domain.

---

## Layer 1: Define Scope

Choose 1-2 high-impact domains and avoid full-repo review in one pass.

---

## Layer 2: Collect Observations

Capture repeated structures, contracts, invariants, and failure modes from complete files.

---

## Layer 3: Extract Reusable Patterns

Distill observations into 3-5 concrete patterns with clear trade-offs.

---

## Layer 4: Author Skill Document

Convert patterns into a practical, enforceable skill file.

---

## Quality Scoring Rubric (0-2 each)

- Scope clarity: 0 unclear, 1 partial, 2 explicit and bounded
- Evidence quality: 0 assumptions, 1 mixed, 2 code-backed findings
- Pattern actionability: 0 abstract, 1 partially actionable, 2 directly implementable
- Trade-off coverage: 0 none, 1 implicit, 2 explicit use/not-use guidance
- Enforceability: 0 no checks, 1 weak checks, 2 checklist is testable

**Target score: 8-10 before publishing.**

---

## Validation Checklist

- ✅ Frontmatter has `name`, `description`, `Use when:`, `Covers:`.
- ✅ Scope is explicit and bounded.
- ✅ Includes 3-5 concrete layers/patterns.
- ✅ Includes gotchas with mitigations.
- ✅ Includes use/not-use guidance.
- ✅ Ends with action-oriented checklist.
- ❌ No repository paths or product IDs in generic examples.

---

## Maintenance Guidance

- Review at least quarterly.
- Keep deprecated patterns visible temporarily with migration notes.
- Split into smaller skills when one document exceeds one clear problem area.
