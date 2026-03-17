# Agent Guide

How to work with Ren, Forge, and Vex.

---

## Ren (Main Agent)

**Role:** Planner, orchestrator, George's main interface.

Ren is who George talks to. She reads everything, makes decisions, and coordinates Forge and Vex. She also does research, manages calendar/email on George's behalf, runs cron jobs, and handles strategic planning.

**What George says to Ren:**
- Natural language requests: "Build me X", "What's on my calendar?", "Research Y"
- High-level direction: Ren figures out the details
- Approval decisions: Ren will ask "should I proceed?" for anything with real-world consequences

**Workspace:** `/home/georges/.openclaw/workspace/`

**Discord:** `Ren AI` — `@1481802596443357196`

---

## Forge (Builder Agent)

**Role:** Implementation — code, PRs, TDD, CI. Does not initiate conversations with George.

Forge picks up tasks from GitHub Issues and builds them. He follows TDD, opens PRs, and pings Vex. He does not respond to general conversation in Discord. He only acts when Ren assigns a task.

**How to get Forge to build something:**
1. Tell Ren what you want
2. Ren creates a GitHub Issue with full spec
3. Ren pings `@Forge AI` with the issue link
4. Forge builds and opens a PR

**When Forge goes quiet:** That's correct behavior. He's idle until Ren assigns a task.

**Workspace:** `/home/georges/.openclaw/workspace-forge/`

**Discord:** `Forge AI` — `@1481893856633950248`

---

## Vex (QA Gate)

**Role:** Hard gate on every PR. Pass or fail — no redesign, no suggestions beyond what blocks the merge.

Vex reviews PRs after Forge opens them. Her verdict is binary: ✅ PASS (clear for merge) or ❌ BLOCKED (with specific fix). When blocked, the fix goes to Ren — who relays it to Forge. Vex does not respond to general conversation.

**What Vex checks:**
- Stage 1 (spec compliance): Do ACs from the issue match what's in the PR?
- Stage 2 (code quality): Tests pass? No debug artifacts? No out-of-scope changes?

**Workspace:** `/home/georges/.openclaw/workspace-vex/`

**Discord:** `Vex AI` — `@1482113890610446449`

---

## Coordination Protocol

The crew follows a hub-and-spoke model where Ren is the hub:

```
George → Ren → Forge → Vex → Ren → George
```

1. **George** tells Ren what to do
2. **Ren** creates a GitHub Issue and pings `@Forge AI` with the task
3. **Forge** builds, opens a PR, posts "PR #N up" in Discord (for visibility only)
4. **Ren** pings `@Vex AI` to gate the PR
5. **Vex** posts gate result addressed to `@Ren AI` — then stands down
6. **Ren** relays to Forge: "PASS — merging" or "BLOCKED — [specific fix]"
7. **Forge** acts only on Ren's relay

**Key rules:**
- Forge does not respond to Vex's Discord messages directly
- Vex does not respond to Forge's Discord messages or George's messages
- When Forge has contradictory data, he states it once and waits for Ren to adjudicate
- `memory/active-tasks.md` shows who owns what — check before touching a task

---

## Who Talks to Whom

| Conversation | How |
|---|---|
| George → crew | Discord `#fish-tank`, natural language to Ren |
| Ren → Forge | Discord `@Forge AI` mention + GitHub Issue link |
| Ren → Vex | Discord `@Vex AI` mention + PR link |
| Forge → Ren | PR comment + brief Discord post |
| Vex → Ren | Discord gate result addressed to `@Ren AI` |
| Ren → George | Discord `#fish-tank`, plain language |

---

## PR Lifecycle

```
GitHub Issue (spec) 
  → Forge branch (feat/<slug> or fix/<slug>)
    → PR opened (Closes #N in body)
      → Vex gate (PASS or BLOCKED)
        → Ren pre-review
          → George merges (or Ren for eligible PRs)
```

**Auto-merge eligible (no George required):**
- `dungeon-cards` PRs after Vex PASS + Ren pre-review
- George explicitly granted this for game dev

**All other repos:** George merges.
