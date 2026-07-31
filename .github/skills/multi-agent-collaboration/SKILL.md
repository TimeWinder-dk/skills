---
name: multi-agent-collaboration
version: 1.0.0
releaseDate: 2026-07-31
description: |
  Run several AI coding agents (and humans) against the same repository without losing work.
  Use when: bootstrapping a new repo that will have more than one concurrent agent session, designing a task
  backlog, reviewing an agent workflow, or after any incident where agents overwrote each other's work.
  Covers: one-file-per-work-item backlog structure, collision-safe task numbering (epics + global bugs),
  ID allocation against origin, the claim protocol (atomic claim push), the handoff.json lesson (no shared
  mutable committed files), working-tree discipline (worktree-per-session, scoped staging, forbidden git
  operations, diff review), shared-resource number reservation, and the enforcement ladder from advisory
  text to blocking validators.
owner: TimeWinder IT
lastReviewed: 2026-07-31
---

# Multi-Agent Collaboration

Every rule in this skill exists because a real repository lost — or nearly lost — real work.
The incidents are listed first so a new repo can recognise the failure before it happens, not after.

## Incident registry (why each rule exists)

| # | Incident (repo, date) | What went wrong | Rule that now prevents it |
|---|---|---|---|
| I1 | Operations Hub, 2026-07 | Shared `handoff.json` held claims, eligibility and update log in one file; parallel agents conflicted constantly, incl. a rebase mid-merge of the file | R1: one file per work item, one writer per file |
| I2 | Operations Hub, 2026-07 | Committed generated board (`state.generated.json`, `BOARD.md`) — a regen clobbered a parallel agent's freshly added bug | R2: generated aggregations are local and gitignored, never committed |
| I3 | Operations Hub, pre-V3 | `T-<epic><seq2>` IDs were ambiguous: `T-10101` parsed as both EPIC-100/101 and EPIC-101/01 | R3: delimited ID scheme `T-<epic>-<seq>` |
| I4 | Norge Companion, 2026-07-31 | Two agents each allocated `T-065`/`T-066` the same day. Filenames were `T-NNN-slug.md`, so the four files merged cleanly; `depends_on: [T-066]` pointed at two different tasks and nothing noticed | R4: filename **is** the ID, or a blocking validator enforces prefix uniqueness; allocate IDs against origin, push the ID file alone and first |
| I5 | Norge Companion, 2026-07 (T-074) | Two sessions shared one working tree. Session B committed, then ran `git reset` to `origin/main` mid-way through session A's work; A's next commit nearly deleted B's just-landed feature. Caught only because unrelated tests failed and the diff was read line by line | R5: one session per worktree; destructive git ops are forbidden in a dirty shared tree; diff against origin before commit |
| I6 | Operations Hub, recurring | Agents staging with `git add .` in a shared tree swept up other agents' uncommitted files | R6: scoped staging only (`git add -- <paths>` + `git commit --only`) |
| I7 | Operations Hub, recurring | Two in-flight branches picked the same SQL migration number | R7: reserve sequential shared resources above the highest on origin, skipping known in-flight numbers, and record the reservation in the claim commit |

If your repo hits a failure not in this table, add it here (new incident number, new or amended rule)
before patching the local symptom. This registry is the shared memory across all TimeWinder repos.

## Core model

Three principles generate every rule below:

1. **One writer per file.** Any file that two agents can plausibly write in the same week is a defect
   in the structure, not a coordination problem to be solved with care. Split it, generate it, or gitignore it.
2. **`origin/main` is the only registry.** A claim, an ID allocation, or a number reservation that is
   not visible on origin does not exist. Local state coordinates nothing.
3. **Text rules decay; validators don't.** Every rule should name its enforcement level (see the
   enforcement ladder) and an upgrade path. A rule that stays advisory after it has been violated once
   is a scheduled repeat incident.

---

## 1. Backlog structure: one file per work item

- One Markdown file per task and per bug, with YAML frontmatter
  (`id`, `title`, `status`, `priority`, `depends_on`, `claimed_by`/`claim_by`, `claim_at`).
- Two agents on two different tasks must never touch the same backlog file. No shared status board,
  no shared claims array, no shared changelog file is ever committed.
- Everything derivable (boards, indexes, `/dev_status` feeds, handoff summaries) is **generated,
  local, and gitignored** (R2). If a human needs it rendered, generate it in CI *for display only*
  (artifact or Pages), never written back to the repo — CI-as-writer was explicitly rejected
  (race + latency + `[skip ci]` hole).
- Status vocabulary must distinguish "code exists" from "verified": at minimum
  `planned / ready / in_progress / blocked / implemented / tested_with_mocks / live_verified /
  completed / dropped`, plus explicit `awaiting_<dependency>` states. Never mark `completed` on
  mock-only evidence.

### Migration note for existing repos

Moving from a shared `handoff.json`-style file: freeze it, move each live claim into the task file's
frontmatter, and delete the shared file in the same change. Leaving both alive means agents disagree
about which one is authoritative — that intermediate state caused conflicts of its own.

## 2. Task numbering: collision-safe IDs

- **Scheme:** `T-<epic>-<seq>` for tasks inside an epic (e.g. `T-100-081`), a single global sequence
  `B-<nnnn>` for bugs. Always delimit the epic from the sequence (I3). Repos without epics use a
  single `T-<nnn>` sequence — but must then be strict about R4.
- **Filename is the ID** (`T-100-081.md`). This is the structural guarantee: two agents choosing the
  same ID write to the same path, and git turns that into a loud conflict instead of a silent merge.
- **If you keep readable slugs** (`T-074-concurrent-agent-worktree-collision.md`), you have given up
  the structural guarantee and MUST replace it with a blocking validator that enforces:
  - the `T-NNN` prefix is unique across the directory,
  - the filename prefix equals the frontmatter `id`,
  - every `depends_on` target exists, no self-dependency,
  - status and priority come from the documented vocabulary.
  See [references/git-recipes.md](references/git-recipes.md) §5 for the invariant list and wiring.
- **Legacy IDs are frozen.** When a scheme changes, old IDs keep their meaning forever; never reuse
  or renumber a terminal item.

### ID allocation protocol (R4)

1. `git fetch origin && git pull --rebase origin main`.
2. Ask the validator/CLI for the next free ID — never eyeball a directory listing (that is exactly
   where I4 started).
3. Create the file and **push it alone, immediately**, before any other work exists. An ID that only
   exists locally is not allocated; the next agent will take the same number.
4. Never batch several new task files into a later commit.

## 3. Claim protocol

- **Claim fields live in the task file** (`claimed_by: <handle>`, `claim_at: <iso>`), nowhere else.
- **Claim before you work, and push the claim.** Set `status: in_progress` + `claimed_by`, commit
  **only that file**, push at once — as the first commit, before the work exists. Other agents must
  see the claim on origin in real time, not when the delivery lands.
- **A claim commit is atomic and alone.** Never mix a claim with implementation code. Code only after
  the claim is confirmed on origin.
- **Pull before claiming.** If the task already carries another agent's `claimed_by`, do not take it —
  pick another or ask the owner.
- **Release explicitly.** Clear `claimed_by` when handing a task back (e.g. to `blocked` awaiting the
  owner), so the task is visibly free.
- **Agent handles are a fixed, documented set** per repo (e.g. `HjemmePC`, `HAA`, `ATP-PC`), each with
  its capability boundary (network access, filesystem access) written next to the handle. An agent
  that does not know which handle it is must ask, never invent one.
- **Staleness:** the protocol has no lease/timeout. A human owner (or a validator warning on
  `claim_at` older than N days with no matching commits) is the garbage collector. Add the validator
  warning when a stale claim first blocks someone — don't wait for the third time.

## 4. Working-tree discipline (the T-074 rules)

The claim protocol prevents two agents taking the same *task*. It does nothing about two sessions in
the same *working tree* — which is how one agent's `git reset` destroyed another's in-flight work (I5).

### R5a — one session per worktree (default, not exception)

Every concurrent agent session works in its own `git worktree` branched from `origin/main`, under a
gitignored directory (e.g. `.claude/worktrees/<task-id>`). The main checkout belongs to the human
owner. Recipe in [references/git-recipes.md](references/git-recipes.md) §1.

Exception: an agent may work directly in the main tree only when it has checked that the tree is
clean (`git status --porcelain` empty) **and** no other session is registered there — and it accepts
that a human may still appear; scoped staging (R6) still applies.

### R5b — forbidden operations in a tree you share

When `git status` shows *any* change you did not make yourself, the following are forbidden without
explicit owner approval, because each one destroys other people's uncommitted or just-landed work:

- `git reset --hard` / `git reset <ref>` that moves the index over foreign changes
- `git checkout -- <file>` / `git restore <file>` on files you did not modify
- `git clean -f` in any form
- `git stash` / `git stash drop` of changes you did not create
- branch switches that would touch dirty foreign files

The safe move is always: leave the foreign changes alone, take your own work to a fresh worktree
from `origin/main`, and continue there.

### R5c — diff review against origin before every commit

Before committing, read `git diff origin/main -- <your files>` line by line and answer one question:
**does this diff contain only changes I made on purpose?** If a hunk removes code you never touched —
especially code that landed on origin after your edit session began — stop: your base is stale, and
committing would delete someone's just-landed work (the reverse case of I5). Rebase, re-apply your
edit onto the current version, and diff again. This review is a hard gate, not a courtesy: the
informal version of it is the only thing that saved T-074's near-miss.

### R6 — scoped staging only

In any tree that might be shared: `git add -- <explicit paths>` and `git commit --only -- <paths>`.
Never `git add .`, `git add -A`, or `git commit -a`. An instruction to "commit and push the task" is
**not** permission to include the owner's or another agent's existing changes; in a mixed tree,
unclear files require explicit confirmation before staging.

### R7 — reserving sequential shared resources

SQL migration numbers, port numbers, enum ordinals: reserve by picking **above the highest on
origin, skipping known in-flight numbers**, and write the reservation into the claim commit or task
file so it is visible on origin. Two branches that both pick "highest + 1" locally will collide at
merge time.

## 5. Enforcement ladder

Every rule sits on one of four rungs. Name the rung explicitly in the repo's agent rule file, and
promote a rule one rung the first time it is violated:

1. **Advisory text** in the always-on agent rule file (`AI.md` or equivalent). Cheapest; decays fastest.
2. **Validator script** (`npm run validate:tasks` style) wired as a `pre` hook on `dev`/`build`/
   `lint`/`test` so it is unavoidable in normal work, **and** as a blocking CI step. This is the
   workhorse rung — I4's fix lives here.
3. **Git hooks / tool hooks** (pre-commit guard against unscoped staging; agent-harness PreToolUse
   hook denying `reset --hard` in a dirty tree). Stronger, but per-machine; treat as belt-and-braces
   over rung 2, never a substitute.
4. **Structural impossibility** — the failure has no representation (filename-is-ID, gitignored
   generated state, worktree isolation). Prefer this rung whenever the cost is acceptable.

What must be at least rung 2 in every multi-agent repo from day one:
ID/filename uniqueness, `depends_on` integrity, status vocabulary, duplicate/stale claim detection.

## 6. Agent rule file requirements

The repo's always-on rule file (`AI.md`, pointed to by thin `CLAUDE.md` / `AGENTS.md` /
`.github/copilot-instructions.md` stubs — one source of truth, never parallel rule sets) must state,
verbatim or adapted:

- the claim protocol incl. "an unpushed claim is not a claim",
- the ID allocation protocol incl. "allocate against origin, push alone and first",
- the worktree rule and the forbidden-operations list (R5a/R5b),
- scoped staging and scope-limited push permission (R6),
- the diff-review gate (R5c),
- the known agent handles and their boundaries,
- the standing order: **when you fix an agent-caused mistake that could recur, record it in the
  incident registry / relevant skill before moving on** — never let the next agent rediscover it by
  losing code.

## 7. Bootstrap checklist for a new repo

- [ ] `docs/tasks/` (or `docs/ai/work/`) with one file per work item; README documents frontmatter
      schema and status vocabulary
- [ ] ID scheme chosen and documented: `T-<epic>-<seq>` + `B-<nnnn>`, or single sequence; filename
      is the ID **or** validator enforces prefix uniqueness
- [ ] Validator script exists, runs as `pre` hook on dev/build/lint/test, blocks CI, and prints the
      next free ID
- [ ] Generated boards/status files are gitignored from the first commit
- [ ] `AI.md` contains the claim protocol, worktree rules, forbidden ops, scoped staging, diff gate,
      agent handles; `CLAUDE.md`/`AGENTS.md`/copilot file are pointers only
- [ ] Worktree directory convention decided and gitignored (`.claude/worktrees/`)
- [ ] Incident registry section exists (may start empty) with the instruction to append before fixing
- [ ] A demonstrated collision test: simulate two sessions, verify the second one's destructive
      attempt is rejected or loudly visible (see [references/git-recipes.md](references/git-recipes.md) §6)

## DO NOT USE FOR

- Single-agent repos with no concurrency (the overhead buys nothing — but keep R5c's diff review;
  it is good practice everywhere)
- Runtime concurrency control (locks, leases, idempotency in application code) — see
  `integration-resilience`
- Human-only teams on PR-based flows; branch protection + review already covers most of this

## Validation checklist

- ✅ No committed file is written by more than one concurrent agent by design
- ✅ Claims and ID allocations are visible on `origin/main` before any implementation commit
- ✅ A duplicate ID, dangling `depends_on`, or duplicate claim fails a build locally and in CI
- ✅ Concurrent sessions use separate worktrees; the forbidden-operations list is in the rule file
- ✅ All staging in shared trees is path-scoped; `git add .` appears nowhere in agent instructions
- ✅ The diff-against-origin gate is stated as a hard rule, not a habit
- ✅ Every past incident maps to a rule, and every rule names its enforcement rung
