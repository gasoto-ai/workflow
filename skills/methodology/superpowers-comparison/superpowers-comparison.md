# obra/superpowers vs. Fish Tank Workflow — Comparison & Gap Analysis

> Research only. No installation. No changes to current stack.
> Source: https://github.com/obra/superpowers (read 2026-03-16)

---

## What Superpowers Is

Superpowers is a composable skills framework for Claude Code / Cursor. Skills are Markdown files that the agent loads when triggered — they function as mandatory workflow protocols, not suggestions. The core philosophy: **no code before design; no design without approval; no implementation without a plan**.

---

## Skill-by-Skill Comparison

### 1. `brainstorming` → Their equivalent of our `plan-feature`

**What it does:**
- Hard gate: zero code until design is presented AND approved (enforced with a `<HARD-GATE>` tag)
- Questions asked one at a time — never a list dump
- Proposes 2–3 approaches with trade-offs and a recommendation
- Writes design doc to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- Dispatches a `spec-document-reviewer` subagent in a loop (max 5 iterations) before showing the spec to the human
- Terminal state: invoke `writing-plans`

**Our equivalent:** `plan-feature/SKILL.md`

**Gaps vs. ours:**
- ✅ Our `plan-feature` has the same design-before-code intent
- ❌ We don't have a hard gate enforced with a structured tag — it's a soft instruction
- ❌ We don't dispatch a spec-reviewer subagent before presenting to the human
- ❌ We don't persist the spec to a file at this stage (we rely on GitHub Issues)
- 🟡 Their "one question at a time" constraint is stronger than ours

**Verdict: cherry-pick**
The `<HARD-GATE>` tag pattern + subagent spec review loop is worth borrowing. Not for our current `plan-feature` (which runs pre-implementation on Ren's side), but if we ever add a Forge-side design phase.

---

### 2. `writing-plans` → Their detailed implementation planning step

**What it does:**
- Requires an approved spec as input
- Maps all files to be created/modified before writing tasks
- Each task is a single 2–5 minute action (write failing test → run test → implement → run test → commit)
- Saves plan to `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
- Dispatches `plan-document-reviewer` subagent before handing to execution

**Our equivalent:** GitHub Issue body (AC list) + Ren's handoff pattern

**Gaps vs. ours:**
- ❌ Our tasks are ACs (acceptance criteria), not step-level implementation checklists
- ❌ We don't have a plan file on disk — GitHub Issue is our plan artifact
- ❌ We don't dispatch a plan reviewer subagent
- ✅ Our GitHub Issue pattern is simpler and requires less agent coordination overhead

**Verdict: skip (with note)**
Their granularity (write failing test → step, run test → step, implement → step) is valuable for junior-mode execution but adds overhead for our setup. Ren's Issues already specify AC coverage. If we ever want Forge to operate more autonomously on complex tasks, the step-level plan format is worth revisiting.

---

### 3. `subagent-driven-development` → Their execution engine; maps to our `spawn-team`

**What it does:**
- Dispatches a fresh subagent per task (no context pollution from parent session)
- Two-stage review per task: spec compliance reviewer first, then code quality reviewer
- If reviewer flags issues, loops back to implementer
- Final full-implementation code review before branch close

**Our equivalent:** `spawn-team/SKILL.md` + Vex as the external gate

**Gaps vs. ours:**
- ✅ We have Vex as hard gate — equivalent function, different mechanism
- ❌ Their review subagents are purpose-built and isolated; Vex reviews the whole PR
- ❌ We don't do task-level review loops — Vex gates per PR, not per task
- 🟡 Their "fresh subagent per task" principle is good; our Forge spawns are typically single-session

**Verdict: cherry-pick concept**
The "spec compliance before code quality" two-stage review is a good mental model for Vex's gate criteria. We could document this ordering in our `qa/SKILL.md` or Vex's review protocol. The subagent-per-task pattern is valid but adds complexity we don't currently need.

---

### 4. `test-driven-development` → Maps to our `tdd/SKILL.md`

**What it does:**
- Enforces RED-GREEN-REFACTOR with explicit steps
- Deletes any code written before tests (nuclear enforcement)
- Has a list of testing anti-patterns (test the behavior, not the implementation)
- Step-level: write test → confirm FAIL → write minimal code → confirm PASS → commit

**Our equivalent:** `tdd/SKILL.md`

**Gaps vs. ours:**
- 🟡 Their nuclear enforcement ("delete code written before tests") is stronger than ours
- ❌ We don't have an explicit anti-patterns reference section
- ✅ Our TDD skill covers the same cycle

**Verdict: cherry-pick**
Worth adding a testing anti-patterns section to our `tdd/SKILL.md`. Their "delete code written before tests" enforcement is also worth formalizing — we currently rely on convention.

---

### 5. `requesting-code-review` → No direct equivalent in our stack

**What it does:**
- Pre-review checklist before sending to human/reviewer
- Checks: tests pass, no TODOs left, no debug artifacts, spec AC coverage, commit history clean

**Our equivalent:** Vex's AC gate (partial)

**Gaps vs. ours:**
- ❌ We don't have a self-review checklist Forge runs before opening a PR
- ✅ Vex catches the AC and test gaps on our behalf

**Verdict: adopt**
A lightweight `pre-pr-checklist` section in `tdd/SKILL.md` or a new `self-review/SKILL.md` would reduce Vex block rate. Forge could run it before every PR open, catching obvious issues without needing Vex's full gate.

---

### 6. `using-git-worktrees` → No equivalent in our stack

**What it does:**
- Creates an isolated worktree on a new branch after design approval
- Runs project setup in the worktree
- Verifies clean test baseline before implementation starts

**Our equivalent:** We just `git checkout -b feat/<slug>` from main

**Gaps vs. ours:**
- ❌ We don't use worktrees — Forge works in the main repo checkout
- ✅ For our current scale (one Forge, sequential tasks) this overhead isn't justified

**Verdict: skip**
Relevant if we ever have true parallel agent builds on the same repo. Not worth the complexity now.

---

### 7. `systematic-debugging` → No equivalent in our stack

**What it does:**
- 4-phase root cause process (reproduce → isolate → fix → verify)
- Explicit "don't guess" protocol
- References: root-cause-tracing, defense-in-depth, condition-based-waiting techniques

**Our equivalent:** Nothing formal

**Verdict: cherry-pick**
A lightweight debugging protocol in `patterns/` would be useful when Vex blocks a PR and Forge needs to investigate a root cause rather than guess. Simple 4-step format is enough.

---

## Summary Table

| Superpowers Skill | Our Equivalent | Gap | Recommendation |
|---|---|---|---|
| brainstorming | plan-feature | No hard-gate tag; no spec-reviewer subagent | cherry-pick (hard-gate pattern) |
| writing-plans | GitHub Issue ACs | No step-level plan file | skip for now |
| subagent-driven-development | spawn-team + Vex | No task-level review loop | cherry-pick (two-stage review model for Vex) |
| test-driven-development | tdd/SKILL.md | No anti-patterns section; no nuclear enforcement | cherry-pick (anti-patterns + enforcement) |
| requesting-code-review | Vex gate (partial) | No self-review checklist pre-PR | adopt (lightweight self-review step) |
| using-git-worktrees | none | none needed at current scale | skip |
| systematic-debugging | none | no formal debug protocol | cherry-pick (lightweight patterns/ entry) |
| finishing-a-development-branch | none formal | PR creation is manual | skip (Ren + George handle this) |

---

## Top 3 Recommendations (if George approves phase 2)

1. **Add `<HARD-GATE>` pattern to `plan-feature/SKILL.md`** — explicit tag that enforces design approval before any code starts. Zero overhead, meaningful guardrail.

2. **Add pre-PR self-review step to `tdd/SKILL.md`** — Forge runs a quick checklist (tests pass, no debug artifacts, ACs covered) before opening every PR. Reduces Vex block rate.

3. **Add testing anti-patterns section to `tdd/SKILL.md`** — "test behavior, not implementation" and 5–6 specific patterns to avoid. Directly applicable to our current test quality.

---

*Researched and written by Forge — 2026-03-16. Read-only analysis. No superpowers files loaded or installed.*
