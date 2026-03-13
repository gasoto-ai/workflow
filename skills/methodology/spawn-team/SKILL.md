---
name: spawn-team
type: methodology
description: Creates a team of agents from a structured plan. This is a sequential workflow — follow these steps in order. Only use after plan-feature produces an approved plan.
---

# Spawn Team

Create and coordinate a team of agents to execute a structured implementation plan. This skill handles the full lifecycle: validation, creation, execution, monitoring, and cleanup.

## Platform

This skill works in **two environments** with different tooling:

| Capability | Claude Code | Cursor IDE |
|---|---|---|
| Create team | `TeamCreate` | Not needed |
| Track tasks | `TaskCreate` / `TaskUpdate` / `TaskGet` / `TaskList` | `TodoWrite` tool |
| Spawn agents | `Agent` tool | `Task` tool (`subagent_type: "generalPurpose"`) |
| Message agents | `SendMessage` | `Task` with `resume` parameter |
| Background work | `run_in_background: true` | `run_in_background: true` |
| File isolation | `isolation: "worktree"` | Not available — avoid overlapping files |
| Max concurrent agents | 6 | 4 |
| Cleanup | `TeamDelete` | Not needed |

**How to detect:** If `TeamCreate` is available, you are in Claude Code. If `Task` tool with `subagent_type` is available, you are in Cursor IDE.

## When to Use

- After `plan-feature` produces an approved plan
- When the user provides a task list with dependencies
- When parallel execution would significantly speed up work

## When NOT to Use

- For single tasks (just do them directly — hand off to Forge)
- When the plan hasn't been approved by the user
- For research or exploration (use Explore agent)

## OpenClaw Branch Convention (Ren + Forge)

When operating in the OpenClaw environment with Forge as tech lead:

### Solo task (simple, clear scope)
- Ren hands off directly to Forge via #fish-tank
- Forge works on branch `feat/<task-slug>` or `fix/<task-slug>`
- Forge opens a PR when done, posts link in #fish-tank
- Ren does optional pre-review; George has final merge authority

### Team task (parallel agents)
Each spawned agent:
1. Creates and works on branch `feat/<task-slug>-<agent-id>` (e.g. `feat/auth-agent-1`)
2. Commits and pushes their branch when done
3. Posts completion status to Forge via #fish-tank mention

Forge (tech lead role):
1. Reviews each agent branch: `git diff main...feat/<task-slug>-<agent-id>`
2. Merges accepted work into consolidation branch: `feat/<task-slug>-consolidated`
3. Resolves conflicts, ensures architectural consistency
4. Opens a single PR from the consolidation branch to `main`
5. Posts PR link in #fish-tank for George's review

George: reviews and merges. Nothing reaches `main` without his approval.

### Branch naming
- `feat/<slug>` — new feature
- `fix/<slug>` — bug fix
- `chore/<slug>` — maintenance, docs, tooling
- `feat/<slug>-<agent-id>` — spawned agent branch (team tasks)
- `feat/<slug>-consolidated` — Forge's merge branch (team tasks)

## Prerequisites

The input plan MUST have:
1. Named tasks with clear descriptions
2. Dependency relationships between tasks (or explicit "no dependencies")
3. Agent assignments (which agent handles which tasks)
4. Enough detail in each task for an agent to work autonomously

## Execution Process

### Step 1: Validate Plan

Before creating anything, verify:

- [ ] Every task has a description detailed enough to execute
- [ ] Dependencies form a DAG (no circular dependencies)
- [ ] Agent assignments cover all tasks
- [ ] No task is assigned to more than one agent
- [ ] Critical path is reasonable (not unnecessarily sequential)

If validation fails, stop and fix the plan before proceeding.

### Step 2: Create Team / Track Tasks

**Claude Code:**
```
Use TeamCreate with:
- team_name: descriptive kebab-case name (e.g., "auth-feature", "api-migration")
- description: one-line summary of what the team is building
```

**Cursor IDE:**
No team creation needed. Use `TodoWrite` to create all tasks as todos for tracking:
```
TodoWrite with merge: false and todos:
- { id: "task-1", content: "Task 1: [description]", status: "pending" }
- { id: "task-2", content: "Task 2: [description]", status: "pending" }
- { id: "task-3", content: "Task 3: [description] (blocked by: task-1)", status: "pending" }
```
Include dependency info in the `content` field since TodoWrite has no dependency mechanism.

### Step 3: Create Tasks with Dependencies

**Claude Code:**

Create tasks in order, then set up dependency chains:

```
1. Create all tasks with TaskCreate (they start as "pending")
2. Set dependencies with TaskUpdate using addBlockedBy
3. Assign owners with TaskUpdate using owner parameter
```

**Important:** Create ALL tasks before setting dependencies, since you need task IDs for the `addBlockedBy` field.

**Cursor IDE:**

Tasks are already created via TodoWrite in Step 2. You manage dependencies yourself:
1. Identify which tasks have no dependencies — these launch first
2. Track which tasks are blocked and by what
3. After a blocking task completes, launch the newly unblocked tasks
4. Update todo status with `TodoWrite(merge: true)` as tasks progress

### Step 4: Spawn Agents

**Claude Code:**

For each agent in the plan, spawn using the Agent tool with this prompt template:

```markdown
You are "[agent-name]" on team "[team-name]". Your assigned tasks are listed below.

**Working directory:** [working directory path]

## Setup

1. Read the project instructions: `cat .claude/CLAUDE.md`
2. Read the TDD methodology (this is your implementation workflow — follow it step by step):
   ```
   cat .workflow/skills/methodology/tdd/SKILL.md
   ```
3. Read pattern skills relevant to your tasks (these are reference material — consult during implementation):
   [LIST RELEVANT PATTERN SKILL PATHS, e.g.:
   ```
   cat .workflow/skills/patterns/writing-react/SKILL.md
   cat .workflow/skills/patterns/writing-tests/SKILL.md
   ```
   ]

## Your Tasks

[List of task IDs and descriptions]

## Workflow

1. Call `TaskGet` for your first task ID to read the full description
2. Mark it `in_progress` with `TaskUpdate`
3. **Follow the TDD skill workflow for implementation:**
   - Plan: identify the behaviors to test
   - RED: write a failing test for the first behavior
   - GREEN: write the minimal code to make it pass
   - REFACTOR: clean up while tests stay green
   - Repeat RED→GREEN→REFACTOR for each behavior
4. Mark it `completed` with `TaskUpdate`
5. Call `TaskList` to find your next available task
6. If blocked, message the team lead explaining what you need
7. When all your tasks are done, message the team lead

## Rules

- **TDD is mandatory** — do not write implementation code before writing a failing test
- Follow the project's code style and patterns (read CLAUDE.md first)
- Consult pattern skills as reference during implementation
- Do not modify files outside your task scope
- If you discover issues, create new tasks with TaskCreate rather than scope-creeping
- Message the team lead if you're stuck or need a decision
```

**Agent configuration:**
- Use `subagent_type: "general-purpose"` for implementation tasks
- Use `team_name` parameter to join the team
- Use `name` parameter matching the agent name from the plan
- Run agents in background with `run_in_background: true` for parallel execution
- Use `isolation: "worktree"` when agents modify overlapping files

**Cursor IDE:**

For each agent, spawn using the `Task` tool. Key differences from Claude Code:
- Subagents have **no access** to conversation history — put ALL context in the prompt
- Max **4 concurrent** subagents (use a single message with multiple Task tool calls)
- Each subagent returns a single result when done
- Use `run_in_background: true` to run agents in parallel
- Use `resume` parameter with agent ID to send follow-up messages

**Launching parallel agents:** To run agents concurrently, call multiple `Task` tools in a **single message**. Do NOT launch them one at a time sequentially.

**Launching waves:** When tasks have dependencies, launch in waves:
- **Wave 1:** All tasks with no dependencies (up to 4 concurrent)
- **Wave 2:** Tasks unblocked by Wave 1 completion (up to 4 concurrent)
- Continue until all tasks are done

Use this prompt template for each Cursor subagent:

```markdown
You are "[AGENT_NAME]" working on [FEATURE_NAME].

**Working directory:** [WORKING_DIRECTORY]

## Project Context

Read these files before writing any code:
- `.cursorrules` — code style and component patterns
- `.claude/CLAUDE.md` — project architecture and conventions

Read the TDD methodology (this is your implementation workflow — follow it step by step):
- `.workflow/skills/methodology/tdd/SKILL.md`

Read pattern skills relevant to your tasks (reference material — consult during implementation):
[LIST RELEVANT PATTERN SKILL PATHS, e.g., .workflow/skills/patterns/writing-react/SKILL.md]

## Your Tasks

[FULL TASK DESCRIPTIONS — copy the complete task details from the plan,
including files to modify, approach, and acceptance criteria.
Do NOT reference task IDs the agent cannot look up.]

## Rules

- **TDD is mandatory** — follow the TDD skill as your implementation workflow (RED→GREEN→REFACTOR)
- Follow the project's code style from .cursorrules (double quotes, semicolons, 4-space indent)
- Consult pattern skills as reference during RED→GREEN cycles
- Do not modify files outside your task scope
- If you encounter an issue that blocks your work, describe it clearly in your response
- When done, return a summary of: files created/modified, what was implemented (including test-first evidence), and any issues found
```

**Task tool configuration for Cursor:**
```
Task({
  description: "[3-5 word summary]",
  prompt: "[full prompt from template above]",
  subagent_type: "generalPurpose",
  run_in_background: true
})
```

Use `model: "fast"` for simple tasks (utility functions, config changes, small tests). Omit `model` for complex implementation tasks.

**IMPORTANT — Avoid file conflicts in Cursor:** Since Cursor has no worktree isolation, do NOT assign overlapping files to different agents. If two tasks touch the same file, assign them to the same agent or make them sequential.

### Step 5: Monitor and Coordinate

**Claude Code:**

As team lead, your responsibilities:

**Reactive (respond to messages):**
- Answer agent questions promptly
- Unblock stuck agents by clarifying requirements
- Approve or reject agent plans (if agents use plan mode)
- Reassign tasks if an agent fails

**Proactive (check periodically):**
- Call `TaskList` to check progress
- Identify newly unblocked tasks and notify assigned agents
- Watch for agents that have been idle too long without completing tasks

**Decision-making:**
- If an agent is stuck for 2+ turns, consider reassigning the task
- If a task reveals unexpected complexity, split it and create sub-tasks
- If dependencies change, update them with TaskUpdate

**Cursor IDE:**

Background agents write output to terminal files. Monitor them:

1. When a `Task` with `run_in_background: true` returns, it provides an `output_file` path
2. Read that file to check progress — the header shows `pid` and `running_for_ms`
3. When complete, a footer with `exit_code` and `elapsed_ms` appears
4. Read the agent's final output from the file to see results

**Coordination loop in Cursor:**
1. Launch Wave 1 agents (all tasks with no dependencies, up to 4)
2. Poll output files to check for completion
3. When an agent finishes, update its todo status: `TodoWrite(merge: true, todos: [{ id: "task-X", status: "completed" }])`
4. Check if any blocked tasks are now unblocked
5. Launch Wave 2 agents for newly unblocked tasks
6. Repeat until all tasks complete
7. If an agent fails, read its output, fix the issue, and re-spawn with `Task` (include failure context in the new prompt)

**Sending follow-ups in Cursor:** If an agent needs additional input, use `Task` with the `resume` parameter set to the agent's ID. This sends a follow-up message preserving the agent's context.

### Step 6: Error Recovery

#### Agent is stuck
**Claude Code:** Read the agent's last message. Clarify via `SendMessage`, provide guidance, or reassign.

**Cursor IDE:** Read the agent's output file. If stuck, either:
- Resume with `Task(resume: agentId, prompt: "clarification...")` 
- Spawn a replacement `Task` with failure context included in the prompt

#### Task fails
1. Do not mark completed — keep as `in_progress` (Claude) or update todo (Cursor)
2. Understand why it failed
3. Either: fix the issue and have the agent retry, or reassign to another agent
4. If the failure affects downstream tasks, adjust the launch order

#### Agent crashes or disconnects
**Claude Code:** Check `TaskList` — tasks will still be `in_progress`. Spawn a new agent with the same name and team.

**Cursor IDE:** Check the output file for error information. Spawn a new `Task` with the same prompt plus context about what the previous agent completed (read its output file first).

#### Circular dependency discovered
1. This means the plan was invalid — stop and fix
2. **Claude Code:** Use `TaskUpdate` to remove the circular dependency
3. **Cursor IDE:** Update todo content via `TodoWrite(merge: true)` and adjust launch order
4. Re-order task execution as needed

### Step 7: Completion and Cleanup

When all tasks are complete:

**Claude Code:**
1. **Verify** — Call `TaskList` to confirm all tasks show `completed`
2. **Review** — Spot-check the work (read key files, run tests if applicable)
3. **Shutdown agents** — Send `shutdown_request` to each agent via SendMessage
4. **Report** — Summarize what was done to the user
5. **Clean up** — Call `TeamDelete` after all agents have shut down

**Cursor IDE:**
1. **Verify** — Check all todos show `completed` via TodoWrite
2. **Review** — Read key files modified by agents, run `npm run validate` or `npm test`
3. **Report** — Summarize to the user:
   - Tasks completed
   - Files created/modified
   - Any issues encountered and how they were resolved
   - Suggested follow-up actions

No cleanup needed — Cursor subagents terminate automatically.

## Agent Prompt Template (Full) — Claude Code

Use this template when spawning agents in Claude Code. Replace bracketed values:

```markdown
You are "[AGENT_NAME]" on team "[TEAM_NAME]".

**Working directory:** [WORKING_DIRECTORY]

## Before You Start

1. Read project instructions:
   ```
   cat .claude/CLAUDE.md
   ```
2. Read the TDD methodology (this is your implementation workflow — follow it step by step):
   ```
   cat .workflow/skills/methodology/tdd/SKILL.md
   ```
3. Read pattern skills relevant to your work (these are reference material — consult during implementation):
   [LIST RELEVANT PATTERN SKILL PATHS, e.g.:
   ```
   cat .workflow/skills/patterns/writing-react/SKILL.md
   cat .workflow/skills/patterns/writing-tests/SKILL.md
   ```
   ]

## Your Tasks

Read your tasks with TaskGet:
[LIST TASK IDS]

## Execution Rules

- **TDD is mandatory** — follow the TDD skill as your implementation workflow (RED→GREEN→REFACTOR)
- Start with TaskGet for your lowest-numbered task
- Mark tasks in_progress before starting, completed when done
- Follow CLAUDE.md conventions strictly
- Consult pattern skills as reference during RED→GREEN cycles
- Do not modify files outside your task scope
- If you need something from another agent's task, message the team lead
- If you discover additional work needed, create a new task with TaskCreate
- When all tasks are done, message "team-lead" that you are finished
```

## Agent Prompt Template (Full) — Cursor IDE

Use this template when spawning subagents in Cursor. Replace bracketed values:

```markdown
You are "[AGENT_NAME]" working on [FEATURE_NAME].

**Working directory:** [WORKING_DIRECTORY]

## Project Context

Read these files before writing any code:
- `.cursorrules` — code style and component patterns
- `.claude/CLAUDE.md` — project architecture and conventions

Read the TDD methodology (this is your implementation workflow — follow it step by step):
- `.workflow/skills/methodology/tdd/SKILL.md`

Read pattern skills relevant to your work (these are reference material — consult during implementation):
[LIST RELEVANT PATTERN SKILL PATHS, e.g.:
- `.workflow/skills/patterns/writing-react/SKILL.md`
- `.workflow/skills/patterns/writing-tests/SKILL.md`
]

## Your Tasks

### Task: [TITLE]
- **Files:** [list of files to create/modify]
- **Approach:** [implementation approach]
- **Acceptance criteria:**
  - [criterion 1]
  - [criterion 2]

[REPEAT FOR EACH TASK ASSIGNED TO THIS AGENT]

## Rules

- **TDD is mandatory** — follow the TDD skill as your implementation workflow (RED→GREEN→REFACTOR)
- Follow .cursorrules conventions (double quotes, semicolons, 4-space indent, 80 char width)
- Consult pattern skills as reference during RED→GREEN cycles
- Do not modify files outside your task scope
- If you encounter a blocking issue, describe it clearly in your response

## Deliverables

When done, return:
1. List of files created or modified
2. Summary of what was implemented (including test-first evidence: which tests were written before implementation)
3. Any issues found or decisions made
4. Anything that needs follow-up or review
```

**Key difference:** In Cursor, you MUST include the full task details in the prompt. Subagents cannot call TaskGet — they have no access to conversation history or external task trackers.

## Team Size Guidelines

| Feature Complexity | Claude Code | Cursor IDE | Example |
|-------------------|-------------|------------|---------|
| Small (3-5 tasks) | 2 agents | 2 agents | Add API endpoint + UI component |
| Medium (5-10 tasks) | 3-4 agents | 3-4 agents | New feature with API, UI, tests |
| Large (10-15 tasks) | 4-5 agents | 4 agents (max concurrent) | Migration or cross-cutting refactor |
| Very Large (15+ tasks) | 5-6 agents | 4 agents in waves | Break into phases |

**Claude Code max: 6.** Coordination overhead grows quadratically.
**Cursor IDE max concurrent: 4.** Hard limit on simultaneous subagents. For larger plans, use waves — launch 4, wait for completion, launch next 4.

## Checklist — Claude Code

- [ ] Plan validated (tasks, dependencies, assignments)
- [ ] Team created with TeamCreate
- [ ] All tasks created with TaskCreate
- [ ] Dependencies set with TaskUpdate addBlockedBy
- [ ] Tasks assigned to agents with TaskUpdate owner
- [ ] Agents spawned with proper prompts including skill discovery
- [ ] Agents configured with correct team_name and name
- [ ] All tasks completed
- [ ] Work reviewed
- [ ] Agents shut down
- [ ] Team deleted
- [ ] Summary reported to user

## Checklist — Cursor IDE

- [ ] Plan validated (tasks, dependencies, assignments)
- [ ] All tasks created as todos via TodoWrite
- [ ] No file overlaps between concurrent agents (or tasks made sequential)
- [ ] Wave 1 agents launched (up to 4 concurrent, all unblocked tasks)
- [ ] Agent prompts include FULL task details and relevant skill file paths
- [ ] Output files monitored for completion
- [ ] Todos updated as agents complete
- [ ] Subsequent waves launched as tasks unblock
- [ ] All tasks completed
- [ ] Work reviewed (read files, run validate)
- [ ] Summary reported to user
