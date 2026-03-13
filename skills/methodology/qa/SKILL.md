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

## Rules

- Do NOT fix issues — only report them. QA observes and documents.
- If a check fails, still continue with remaining checks.
- Be specific about what failed and why.
- For browser validation, only check pages/routes that were actually changed.
- If the dev server is not running, skip browser validation and note it in the report.
