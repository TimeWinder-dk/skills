# Git recipes and enforcement snippets for multi-agent repos

Concrete command sequences and scripts referenced from [SKILL.md](../SKILL.md). Everything here is
copy-adaptable; replace paths and handles per repo.

## 1. Worktree per session (R5a)

```bash
git fetch origin
git worktree add .claude/worktrees/T-100-081 -b t-100-081 origin/main
cd .claude/worktrees/T-100-081
# ... claim, implement, push ...
cd ../../..
git worktree remove .claude/worktrees/T-100-081   # after the branch has landed
```

Notes:

- `.claude/worktrees/` must be in `.gitignore`.
- Branch from `origin/main`, never from the local `main` — the local one may be stale or dirty.
- If the delivery lands on main by fast-forward push from inside the worktree, remember the main
  checkout still points at the old commit; other sessions must `git pull --rebase` as usual.
- Leftover worktrees are noise that hides real ones: `git worktree list` during cleanup, and
  `git worktree prune` after deleting directories manually.

## 2. Atomic claim commit (alone and first)

```bash
git fetch origin && git pull --rebase origin main
# edit ONLY the task file: status: in_progress, claimed_by: <handle>, claim_at: <iso>
git add -- docs/tasks/T-100-081.md
git commit --only -- docs/tasks/T-100-081.md -m "chore(T-100-081): claim (<handle>)"
git push origin main   # or the shared branch; the claim must reach origin NOW
```

If the push is rejected (non-fast-forward), someone else just pushed — `git pull --rebase` and check
whether the task got claimed under you before retrying.

## 3. ID allocation (push the file alone)

```bash
git fetch origin && git pull --rebase origin main
npm run validate:tasks          # prints "Next free ID: T-NNN" — never eyeball the directory
# create docs/tasks/T-NNN[-slug].md
git add -- docs/tasks/T-NNN*.md
git commit --only -- docs/tasks/T-NNN*.md -m "docs(tasks): allocate T-NNN <title>"
git push origin main
```

## 4. Scoped staging in a shared tree (R6)

```bash
# ALWAYS list paths explicitly:
git add -- src/feature/thing.ts src/feature/thing.test.ts
git commit --only -- src/feature/thing.ts src/feature/thing.test.ts -m "..."

# When one FILE is shared and dirty with foreign work, commit only your hunks by staging
# an explicit blob built on origin/main's version — or take the file to a worktree instead.
```

Pre-commit review gate (R5c), before every commit:

```bash
git fetch origin
git diff origin/main -- <your files>
# Every hunk must be a change you made on purpose. A hunk that REMOVES code you never
# touched means your base is stale: rebase, re-apply, re-diff. Do not commit.
```

## 5. Backlog validator invariants (rung 2)

The validator (Node script, no dependencies) must check at minimum:

1. Filename matches the documented pattern; a non-matching file is an error (it would be invisible
   to generated status views — a silent failure of its own).
2. Frontmatter parses and contains `id`, `title`, `status`, `priority`.
3. `id` matches the ID grammar, and filename prefix == frontmatter `id`.
4. **ID prefix is unique across the directory** — report *all* colliding files by name.
5. Every `depends_on` target exists; no self-dependency.
6. `status` and `priority` are from the documented vocabulary.
7. Claim consistency: no duplicate `claimed_by` on one task, no claim on a terminal-status task,
   (optional, warning) `claim_at` older than N days with no commits touching the task file.
8. CLI prints the **next free ID** on success.

Wiring — the check must be unavoidable in normal work and blocking in CI:

```jsonc
// package.json
"scripts": {
  "predev":  "npm run validate:tasks",
  "prebuild":"npm run validate:tasks",
  "prelint": "npm run validate:tasks",
  "pretest": "npm run validate:tasks",
  "validate:tasks": "node scripts/validate-tasks.mjs"
}
```

```yaml
# ci.yml — blocking step, before anything else
- run: npm run validate:tasks
```

A working reference implementation lives in `norge-companion-pwa/scripts/validate-tasks.mjs`
(covers invariants 1–6 + 8, with the I4 incident documented in its header comment).

## 6. Demonstrating the collision guard (bootstrap checklist item)

Simulate two sessions and prove the second destructive attempt is rejected or loudly visible:

```bash
# Session B lands a feature:
git -C worktree-b commit -am "feat: landed" && git -C worktree-b push

# Session A (stale main tree, dirty with its own edits) tries the I5 failure:
git status --porcelain          # NOT empty → R5b applies
git reset --hard origin/main    # this is the forbidden op — the guard must catch it
```

Minimal guard as an agent-harness hook (Claude Code `PreToolUse` on Bash/PowerShell), pseudo-logic:

```
if command matches (reset --hard | checkout -- | restore | clean -f | stash)
   and `git status --porcelain` is non-empty
then deny with: "Dirty tree may contain another session's work (R5b).
     Move to a worktree from origin/main instead."
```

This is rung 3 — belt and braces. It does not replace the rung-2 validator or the rule text; a
different harness (Copilot, Codex CLI) won't run the hook, but it still reads the rule file.

## 7. Handling the reverse case (your stale copy vs. their newer file)

You edited `MapView.tsx` based on Monday's version; another agent landed a change to it on Tuesday.
Your working copy on disk now *predates* origin. Committing it as-is deletes their feature.

Detection is the R5c diff: `git diff origin/main -- MapView.tsx` shows their feature being
*removed*. Recovery:

```bash
git stash push -- MapView.tsx          # park your version (yours alone — R5b allows this)
git checkout origin/main -- MapView.tsx
git stash pop                           # conflict markers show exactly what must merge
# re-apply your intent onto their version by hand; re-run their tests AND yours
```

Tests that fail on a feature you never touched are this failure announcing itself — treat that
signal as "my base is stale", never as "flaky test".
