---
name: plan-feature
type: methodology
description: Structures feature requirements into a complete implementation plan, then executes it with spawn-team. This is a sequential workflow — follow these steps in order. Do not skip planning or user approval.
---

# Plan Feature

Take a raw feature description and produce a structured, actionable implementation plan ready for team execution. This skill bridges the gap between "what we want" and "how we build it."

## First Action: Switch to Plan Mode

**Immediately** switch to Plan mode using `SwitchMode(target_mode_id: "plan")` before doing anything else. All planning work happens in Plan mode — do not start exploring or structuring in Agent mode.

## When to Use

- Feature requests that touch 3+ files
- Work requiring multiple agents or parallel execution
- Cross-cutting changes (API + UI + tests)
- Architectural changes or migrations
- Any migration phase step (e.g., UI library swap, state management refactor)
- Adding new API routes or server actions
- Any work where the wrong approach wastes significant time

## When NOT to Use

- Single-file bug fixes or simple config changes
- Typo corrections
- Adding a test for existing code
- Tasks with obvious implementation (just do them)
- Pure research or exploration (use Explore agent instead)

## Planning Process

### Phase 1: Clarify Ambiguity

Before planning, eliminate unknowns. Ask the user targeted questions:

**Requirements questions:**
- What is the expected user-facing behavior?
- What are the edge cases and error states?
- Are there performance or accessibility requirements?
- Which users/environments are affected?

**Scope questions:**
- What is explicitly out of scope?
- Is this a full implementation or a proof of concept?
- Are there time or complexity constraints?

**Technical questions:**
- Are there existing patterns in the codebase to follow?
- Are there dependencies on external services or APIs?
- Should this be behind a feature flag for gradual rollout?

**Stop and ask** if any of these are unclear. Do not guess at requirements.

### Phase 2: Explore Codebase

Systematically discover the relevant code:

1. **Find entry points** — Search for files related to the feature area
2. **Trace dependencies** — Follow imports to understand the dependency graph
3. **Check patterns** — Read existing similar features to match conventions
4. **Discover skills** — Run `ls .workflow/skills/` and read relevant skill files
5. **Check docs** — Look for related documentation in `docs/`
6. **Identify tests** — Find existing test patterns for the affected areas

**Output:** A list of files that will be read, modified, or created.

### Phase 3: Structure Requirements

Produce a structured plan document with these sections:

#### 3a. Summary
One paragraph: what changes, why, and what success looks like.

#### 3b. User Stories
```
As a [role], I want [capability] so that [benefit].
Acceptance criteria:
- [ ] Given [context], when [action], then [result]
- [ ] Given [context], when [action], then [result]
```

#### 3c. Technical Breakdown
For each logical unit of work:
```
### [Component Name]
- **Files:** list of files to create/modify
- **Depends on:** other components that must be done first
- **Approach:** brief description of the implementation
- **Risk:** what could go wrong
```

#### 3d. Task List with Dependencies
Structure tasks for parallel execution:

```
Task 1: [Description] (no dependencies)
Task 2: [Description] (no dependencies)
Task 3: [Description] (blocked by: Task 1)
Task 4: [Description] (blocked by: Task 1, Task 2)
Task 5: [Description] (blocked by: Task 3, Task 4)
```

Rules for task creation:
- **TDD first:** Every implementation task should follow the `tdd` skill — agents write one test, implement to pass, repeat. Task descriptions should open with "Follow TDD workflow (`.workflow/skills/workflow/tdd/SKILL.md`)" so agents read it before writing any code.
- **Pattern skills:** Each task description must list the relevant pattern skill paths agents should consult (e.g., "Consult `.workflow/skills/standards/writing-react/SKILL.md` and `.workflow/skills/standards/writing-forms/SKILL.md`"). This ensures spawn-team includes them in agent prompts.
- **Size:** Each task should take one agent 10-30 minutes
- **Independence:** Maximize tasks that can run in parallel
- **Completeness:** Each task should be self-contained and testable
- **Clarity:** Task descriptions must be specific enough for an agent to execute without further questions
- **Testing:** Tests are written as part of TDD cycles within implementation tasks, not as separate tasks

#### 3e. Team Composition
Recommend the number and type of agents:

```
Agents needed: [N]
- Agent 1: [name] — [responsibility] (tasks: 1, 3)
- Agent 2: [name] — [responsibility] (tasks: 2, 4)
- Agent 3: [name] — [responsibility] (task: 5)
```

Guidelines:
- **2-4 agents** for most features (more agents = more coordination overhead)
- **Max 6 agents** even for large features
- **1 agent** if tasks are strictly sequential
- Group related tasks to minimize cross-agent dependencies
- Name agents by responsibility (e.g., "api-agent", "ui-agent", "test-agent")

#### 3f. Risk Assessment

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| [What could go wrong] | High/Med/Low | High/Med/Low | [How to prevent or recover] |

### Phase 4: User Review

Present the complete plan to the user before execution. The plan should be:

- **Readable** — No jargon; anyone on the team can understand it
- **Actionable** — Each task has enough detail to start immediately
- **Reviewable** — User can approve, modify, or reject any part

**Always include an "Execution" section** at the end of the plan:

```
## Execution

Once approved, I will read `.workflow/skills/workflow/spawn-team/SKILL.md` and use it
to create the agent team, assign tasks with dependencies, and coordinate
parallel execution automatically.
```

Present the plan for user approval. Do NOT ask "does this look good?" — present it and wait.

---

**⛔ HARD GATE — Do not proceed past this point until the user explicitly approves the plan.**

Valid approvals: "go", "looks good", "approved", "build it", "execute", "ship it", or any clear affirmative.

**Silence does not count as approval.** If the user hasn't responded, wait.
If they ask questions or request changes, address them and re-present before proceeding.

---

### Phase 5: Execute with spawn-team (MANDATORY)

**This phase is not optional.** When the user approves the plan (or says "go", "looks good", "build it", "execute", "approved", etc.):

1. **Immediately** read `.workflow/skills/workflow/spawn-team/SKILL.md`
2. **Follow every step** in spawn-team to create the team, create tasks, set dependencies, spawn agents, and monitor execution
3. **Do NOT** implement the plan yourself task-by-task
4. **Do NOT** ask the user how they want to execute — always use spawn-team
5. **Do NOT** skip this phase or suggest the user run spawn-team manually

The entire point of plan-feature is to produce a plan that spawn-team executes. If you finish the plan without spawning agents to build it, you have not completed the skill.

## Task Sizing Guidelines

| Size | Duration | Files | Example |
|------|----------|-------|---------|
| Small | 5-10 min | 1-2 | Add a utility function with tests |
| Medium | 10-20 min | 2-5 | Add an API endpoint with service + tests |
| Large | 20-30 min | 5-8 | Refactor a component with new patterns + tests |
| Too Large | 30+ min | 8+ | Split into multiple tasks |

## Dependency Chain Guidelines

- **Critical path** — Identify the longest chain of dependent tasks; this determines minimum completion time
- **Parallel width** — Maximize the number of tasks that can run simultaneously
- **Minimize depth** — Prefer wide, shallow dependency graphs over deep chains
- **Foundation first** — Types, configs, and utilities should be early tasks (they unblock others)
- **Tests alongside** — Include testing in implementation tasks rather than as separate blocking tasks

## Output Format

The final plan should be a single markdown document that can be passed to the `spawn-team` skill for execution. Structure:

```markdown
# Plan: [Feature Name]

## Summary
[One paragraph]

## User Stories
[Stories with acceptance criteria]

## Tasks

### Task 1: [Title]
- **Agent:** [agent-name]
- **Blocked by:** none
- **Files:** [list]
- **Pattern skills:** [list of `.workflow/skills/standards/` paths relevant to this task]
- **Description:** Follow TDD workflow (`.workflow/skills/workflow/tdd/SKILL.md`). [Detailed instructions an agent can follow]

### Task 2: [Title]
- **Agent:** [agent-name]
- **Blocked by:** Task 1
- **Files:** [list]
- **Pattern skills:** [list of `.workflow/skills/standards/` paths relevant to this task]
- **Description:** Follow TDD workflow (`.workflow/skills/workflow/tdd/SKILL.md`). [Detailed instructions an agent can follow]

## Team
- [agent-name]: Tasks 1, 3 (general-purpose)
- [agent-name]: Tasks 2, 4 (general-purpose)

## Risks
[Risk table]

## Execution
Once approved, I will read `.workflow/skills/workflow/spawn-team/SKILL.md` and use it
to create the agent team, assign tasks with dependencies, and coordinate
parallel execution automatically.
```

## Checklist

- [ ] Switched to Plan mode before starting
- [ ] Ambiguities resolved with the user
- [ ] Relevant codebase files explored
- [ ] Relevant skills discovered and read
- [ ] User stories with acceptance criteria written
- [ ] Technical breakdown with file lists
- [ ] Tasks sized correctly (10-30 min each)
- [ ] Dependencies minimize critical path length
- [ ] Team composition recommended (2-6 agents)
- [ ] Risks identified with mitigations
- [ ] Execution section included (use `spawn-team`)
- [ ] Plan presented to user for review
- [ ] After approval, executed with `spawn-team` skill
