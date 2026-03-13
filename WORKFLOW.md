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

## Task Convention (TASK.md)
- **Template:** `TASK-TEMPLATE.md`
- **Active:** lives in project repo root as `TASK.md`
- **Archive:** move to `.workflow/memory/tasks/YYYY-MM-DD-<task-slug>.md` on completion
- Forge fills in Decisions and Blockers as work progresses
- Status field: `in-progress` → `blocked` → `complete`
- Tag @Ren AI in Blockers when input is needed

## Handoff Format (Ren → Forge)
```
## Task: [title]
**Repo/path:** [where to work]

### Context
[what exists, what needs to change, why]

### Acceptance Criteria
- [ ] ...

### Constraints
[things not to do]

### Out of Scope
[explicit boundaries]

### Skills to Read
- .workflow/skills/[relevant]/SKILL.md
```

## Environment
- Platform: OpenClaw/Ren on Raspberry Pi 5
- Shell: bash
- Agent spawning: `sessions_spawn` (runtime: subagent)
- Max concurrent agents: 4

## Workflow Loop
```
plan-feature → [user approves] → spawn-team → qa → check-ci → reflect
```
