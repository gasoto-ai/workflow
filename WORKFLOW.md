# WORKFLOW.md — Ren's Development System

A portable AI-native software delivery workflow adapted from George's Claude Code system.

## Skills

### Methodology (sequential workflows — follow in order)
| Skill | When to Use |
|-------|-------------|
| `plan-feature` | Any non-trivial work — plan before building |
| `spawn-team` | After plan approved — parallel agent execution |
| `qa` | After implementation — validate before shipping |
| `check-ci` | After PR — investigate and fix failures |
| `reflect` | End of session — capture learnings |

### Patterns (reference — consult during implementation)
| Skill | When to Consult |
|-------|-----------------|
| `writing-tests` | Test patterns and conventions |
| `writing-react` | React component patterns |
| `writing-typescript` | TypeScript patterns |
| `organizing-files` | File/directory structure decisions |

## Memory
- Location: `.workflow/memory/`
- `MEMORY.md` — index, keep under 200 lines
- Topic files: `patterns.md`, `debugging.md`, `preferences.md`, `skill-gaps.md`
- Task archive: `.workflow/memory/tasks/YYYY-MM-DD-<task-slug>.md`

## Task Convention

Tasks are tracked as **GitHub Issues** in the relevant repo. Implementation notes live in a local `TASK.md` during the build (see `TASK-TEMPLATE.md`).

### Why GitHub Issues over TASK.md only
- Linkable, commentable, searchable — full history in one place
- No Discord truncation — issue bodies have no practical length limit
- PRs auto-close issues via `Closes #N` in the PR description
- George can read the full task thread without digging through Discord
- Cross-project process issues live in `gasoto-ai/workflow`; implementation tasks live in the relevant app repo

### Issue lifecycle
1. Ren opens a GitHub Issue in the relevant repo with the full task spec (context, AC, constraints, out of scope, skills)
2. One-liner in #fish-tank Discord: "Forge — [task name], see [issue link] <@FORGE_ID>"
3. Forge drops any concerns as issue comments before starting
4. Forge creates a local `TASK.md` for working notes during the build
5. PR description includes `Closes #N` to auto-close the issue on merge
6. George merges → issue closes, Forge deletes `TASK.md`
7. Discord gets brief status pings only: "started", "PR up", "pre-review passed", "merged"

### Discord message rules (Ren ↔ Forge)
- **Task handoffs:** GitHub Issue + one-line Discord ping (never post full spec in Discord)
- **Concerns/blockers:** GitHub Issue comment + @mention in Discord if urgent
- **Status updates:** Discord only, keep to one line
- **Pre-review results:** Discord (brief summary of findings + approve/reject)

## Handoff Format (GitHub Issue body)
```
## Context
[What exists, what needs to change, why]

## Acceptance Criteria
- [ ] ...

## Constraints
[Things not to do — "don't break X", "don't touch Y"]

## Out of Scope
[Explicit boundaries — refactors, unrelated fixes, etc.]

## Skills to Read
- `.workflow/skills/methodology/tdd/SKILL.md`
- `.workflow/skills/patterns/[relevant]/SKILL.md`
```

## PR Convention
- Branch: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`
- PR target: `playground` (never directly to `main`)
- PR description must include `Closes #N` linking back to the task issue
- Ren pre-reviews before flagging George
- George has final merge authority

## Environment
- Platform: OpenClaw/Ren on Raspberry Pi 5
- Forge workspace: `/home/georges/.openclaw/workspace-forge/`
- Shared workflow skills: `.workflow/skills/` (in Ren's workspace, symlinked/copied to Forge's)

## TDD Policy

TDD is mandatory by default. The only exception:

### Static/presentational sites
Sites with no business logic (no API routes, no state machines, no data transformations) are exempt from the full TDD cycle. They still require **render smoke tests**:
- Configure Jest + React Testing Library
- One test per component: renders without throwing + asserts at least one key piece of content
- `npm test` must pass

The TDD mandate (write failing test → implement → pass → refactor) applies to any code with behavior. "Out of Scope: tests" must be explicitly declared in the issue spec when tests are intentionally skipped — omission is not acceptable.
