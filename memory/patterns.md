# patterns.md — Confirmed Workflow Patterns

## CLAUDE.md Structure
- Keep it short: what the project is, stack, file structure, key types, code style, TDD mandate
- Do NOT include task descriptions — those go in the spawn prompt
- Target: under 30 lines of actual content (learned 2026-03-12)

## Spawn Prompt Structure
- Always include: task description, files to touch, acceptance criteria, TDD reminder
- Always end with: `discovered.md` logging instruction + `openclaw system event` notification
- Source of truth for *what to build*; CLAUDE.md is source of truth for *how to build*

## Agent Behavior
- Claude Code reads CLAUDE.md reliably when told to at session start
- TDD is followed when "TDD is Mandatory" appears in both CLAUDE.md and spawn prompt
- Agents will make good out-of-scope decisions (e.g. making CLI testable via exports) but won't document them without instruction
- Agents execute faster and more precisely with shorter CLAUDE.md

## Multi-Agent
- Sequential tasks (Task 2 blocked by Task 1) work well with a single agent
- For parallel work, max 4 concurrent Claude Code agents
- Task files in `.claude/tasks/` needed for coordination
