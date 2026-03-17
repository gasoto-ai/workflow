# Workflow Guide

How tasks move through the system, from George's request to shipped code.

---

## Full Task Lifecycle

```
1. George approves idea (or asks for something)
2. Ren creates GitHub Issue (spec, ACs, branch name)
3. Ren pings @Forge AI with issue link
4. Forge reads issue, comments "On it. Branch: <branch>"
5. Forge builds on feature branch (TDD: test → implement → pass → commit)
6. Forge opens PR (branch → main, body: "Closes #N")
7. Forge posts "PR #N up" in Discord
8. Ren pings @Vex AI with PR link
9. Vex reviews (Stage 1: spec compliance → Stage 2: code quality)
10. Vex posts result to @Ren AI
11. Ren relays to Forge: PASS → merge, or BLOCKED → specific fix
12. If BLOCKED: Forge fixes, updates PR, loop back to step 8
13. If PASS: Ren pre-reviews, then merges or escalates to George
14. George merges (or auto-merge for dungeon-cards)
```

---

## GitHub Issues as the Source of Truth

Every task starts with a GitHub Issue. No code before an issue.

**Issue structure:**
```markdown
## Context
Why this exists.

## Acceptance Criteria
- [ ] Specific, testable outcomes
- [ ] One per line

## Files likely affected
- Which files change

## Out of scope
- What we're NOT doing
```

**Issue location:**
- `gasoto-ai/workflow` — methodology, process, system docs
- `gasoto-dev/<repo>` — code changes to that repo

---

## Branch Naming

| Type | Convention | Example |
|---|---|---|
| Feature | `feat/<slug>` | `feat/stripe-checkout` |
| Bug fix | `fix/<slug>` | `fix/cart-quantity-bug` |
| Chore | `chore/<slug>` | `chore/ci-workflow` |
| Research | `research/<slug>` | `research/mcp-audit` |

Always branch from `main`. Push to origin. Open PR against `main`.

---

## PR Conventions

**PR body minimum:**
```
Closes #N

<one paragraph: what changed and why>
```

- Always include `Closes #N` to auto-close the issue on merge
- No "wip" commits in history — squash before opening
- One PR per issue; no bundling unrelated changes
- No direct push to `main` — ever

---

## TDD Workflow (Mandatory)

Every implementation follows Red-Green-Refactor:

1. Write a failing test that describes the behavior
2. Run it — confirm it fails
3. Write the minimal code to make it pass
4. Run tests — confirm pass
5. Commit
6. Refactor if needed (tests must still pass)

**Before opening any PR:**
- [ ] All tests pass locally
- [ ] No `console.log`, `TODO`, or commented-out code left behind
- [ ] Every AC from the issue has at least one test
- [ ] Commit history is clean
- [ ] No files modified outside the task scope

---

## Task Ownership

`memory/active-tasks.md` (at `/home/georges/.openclaw/workspace/memory/active-tasks.md`) is the single source of truth for who owns what right now.

Format:
```markdown
## Active
- <repo>/<task>: OWNER=<Ren|Forge>, STATUS=<assigned|in-progress|blocked|review>
```

Forge checks this before acting. If a task is owned by Ren, Forge stands down.

---

## ClickUp Integration

ClickUp (workspace "George Soto's Workspace") tracks project-level work alongside GitHub Issues.

| Tool | Purpose |
|---|---|
| GitHub Issues | Code-level: ACs, implementation details, PR tracking |
| ClickUp | Project-level: priorities, deadlines, blocked items |

Ren manages ClickUp tickets and keeps them in sync with GitHub.

Fish Tank space lists: Backlog, In Progress, Blocked, Done.

---

## CI Requirements

Every repo has `.github/workflows/ci.yml` that runs on every PR and push to `main`:
- `npm ci` + `npm test` for Next.js repos
- GUT headless for dungeon-cards (Godot 4)

CI is a required check on all repos — PRs cannot merge if CI fails.

---

## Repo Structure Conventions

Every repo includes:
- `.devcontainer/devcontainer.json` — Dev container config
- `.github/workflows/ci.yml` — CI workflow
- `README.md` — Project overview, setup, architecture
- Tests in `__tests__/` (Next.js) or `tests/` (GDScript)

---

## Memory Maintenance

At the end of significant sessions, agents write to `memory/YYYY-MM-DD.md`. Periodically, Ren distills daily logs into `MEMORY.md` (long-term memory).

Forge creates `memory/projects/<repo>.md` when working on a repo — stack, routes, DB schema, test setup, gotchas. This file persists between sessions.
