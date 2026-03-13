# MEMORY.md — Workflow System Learnings

_Keep under 200 lines. Details in topic files._

## Key Learnings

- CLAUDE.md should be minimal/stable (conventions only) — task details belong in the spawn prompt (learned 2026-03-12)
- Splitting CLAUDE.md from spawn prompt reduces agent context bloat and keeps each source of truth focused
- Claude Code follows TDD reliably when explicitly mandated in the task prompt
- Agents make smart out-of-scope decisions silently — need `discovered.md` convention to surface these
- Auto-notify via `openclaw system event` must be in the spawn prompt template — agents won't add it themselves
- Skipping `plan-feature` and writing spawn prompts by hand reintroduces the problems the system was designed to prevent

## Topic Files
- `patterns.md` — confirmed workflow patterns
- `skill-gaps.md` — gaps in current skills

## Overnight Work (2026-03-13)
- Stripped Enroll-specific content from `plan-feature` — skill is now generic/portable
- Wrote `tdd` skill from scratch — RED→GREEN→REFACTOR cycle, covers what to test, what not to test, signs of bad tests
- Wrote 3 new pattern skills: `writing-tests`, `writing-typescript`, `organizing-files`
- Pattern skills are reference-style — agents consult them during implementation, not sequential workflows
- Updated `WORKFLOW.md` to list `organizing-files` in patterns table
- Marked plan-feature Enroll gap as fixed in `skill-gaps.md`

## Stats
- Sessions run: 1
- Projects built: 1 (outreach-cli)
- Features added: 1 (delete command)
- Tests: 16 passing
