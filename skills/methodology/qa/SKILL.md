---
name: qa
type: methodology
description: Runs comprehensive QA checks on the current branch. This is a sequential workflow — follow these steps in order. Run after implementation to validate before merging.
---

# QA — Quality Assurance Evidence Report

Run a comprehensive QA pass on the current branch and post evidence to the PR.

## Phase 1: CLI Validation (always runs)

Run these checks and capture output:

1. `npm run validate` — runs lint, type-check, and tests
2. `npm run test:coverage` — capture coverage summary
3. Check for any new `any` types introduced: `git diff origin/develop...HEAD -- '*.ts' '*.tsx' | grep '+.*: any'`

Record pass/fail and output for each.

## Phase 2: Browser Validation (requires chrome-devtools MCP)

If chrome-devtools MCP tools are available, perform browser-based QA:

1. **Navigate** to the app at `http://localhost:80` (or the dev server URL)
2. **Identify affected pages** by checking which files changed: `git diff --name-only origin/develop...HEAD`
3. For each affected page/route:
   a. Navigate to the page
   b. **Take a screenshot** — save with a descriptive filename
   c. **Check console** — `list_console_messages` — verify zero errors/warnings
   d. **Check network** — `list_network_requests` — verify no failed requests (4xx/5xx)
   e. **Take accessibility snapshot** — `take_snapshot` — verify the page structure is sound
   f. **Run JavaScript checks** — `evaluate_script` — verify no hydration errors, check key elements exist
4. If forms are involved, fill and submit test data to verify form flows
5. If the change is visual, take before/after screenshots if possible

If chrome-devtools MCP tools are NOT available, skip this phase and note in the report that browser validation was skipped. Suggest the developer install the chrome-devtools MCP server.

## Phase 3: Generate Evidence Report

Build a structured markdown QA report:

```markdown
## QA Evidence Report

**Branch:** `{branch}` | **Date:** {date} | **PR:** #{pr_number}

### CLI Validation

| Check | Result | Details |
|-------|--------|---------|
| TypeScript | {pass/fail} | {error count or "No errors"} |
| ESLint | {pass/fail} | {warning/error count} |
| Jest Tests | {pass/fail} | {passed}/{total} |
| Coverage | {pass/fail} | Stmts: {n}% Branch: {n}% Funcs: {n}% Lines: {n}% |
| New `any` types | {pass/fail} | {count found} |

<details>
<summary>Test Output</summary>

{test output here}

</details>

<details>
<summary>Coverage Report</summary>

{coverage table here}

</details>

### Browser Validation

| Page | Screenshot | Console Errors | Network Failures | A11y |
|------|-----------|----------------|-------------------|------|
| {route} | {pass/fail} | {count} | {count} | {pass/fail} |

<details>
<summary>Console Log</summary>

{console output}

</details>

<details>
<summary>Network Requests</summary>

{network summary}

</details>

<details>
<summary>Accessibility Snapshot</summary>

{a11y tree summary — key landmarks, headings, form labels}

</details>

### Summary

- **Overall:** {PASS/FAIL}
- **Issues Found:** {list any issues}
- **Notes:** {any observations}

---
*QA performed by Claude Code with chrome-devtools-mcp*
```

## Phase 4: Post to PR

1. Get the current PR number: `gh pr view --json number -q '.number'`
2. If a PR exists, post the report as a PR comment: `gh pr comment {number} --body "{report}"`
3. If no PR exists, output the report to the terminal instead

## Review Ordering

When reviewing a PR, evaluate in this order and **stop at the first blocking failure**:

### Stage 1: Spec Compliance (check this first)
Does the code do what the issue says it should do?

- [ ] Every acceptance criterion from the GitHub Issue is addressed
- [ ] No AC is partially implemented or skipped without explanation
- [ ] The approach matches what was specified (not a creative reinterpretation)

**If any AC is missing or wrong, block here.** Do not continue to Stage 2.
Code quality issues are irrelevant if the feature is wrong.

### Stage 2: Code Quality (only after Stage 1 passes)
Is the implementation well-built?

- [ ] Tests exist and pass for new behavior
- [ ] No debug artifacts (`console.log`, `TODO`, commented-out code)
- [ ] No out-of-scope files modified
- [ ] No obvious anti-patterns (see tdd/SKILL.md anti-patterns section)
- [ ] Commit history is reasonably clean

**Soft flags** — issues worth noting but not blocking — go in the report even if they don't block the merge.

## Rules

- Do NOT fix issues — only report them. QA observes and documents.
- If a check fails, still continue with remaining checks within the same stage.
- Stage 1 failures always block. Stage 2 failures block by default; use judgment on soft flags.
- Be specific about what failed and why.
- For browser validation, only check pages/routes that were actually changed.
- If the dev server is not running, skip browser validation and note it in the report.
