---
name: check-ci
type: methodology
description: Checks the PR for CI failures, runs local validation, reviews automated code review comments, and fixes issues. This is a sequential workflow — follow these steps in order.
---

# Check CI — Fix Pipeline Failures

Check the PR for CI failures, run local equivalents, and fix issues.

Do not commit — only fix. The user decides when to commit.

## Process

1. **Check PR status** — Run `gh pr checks` to see any failures on the PR
2. **If there are errors** — Make a plan to fix them
3. **Run local equivalents** — Check `.github/workflows/pr.yml` and `.github/workflows/reusable-security.yml` for what commands are run, then run locally: `npm run validate` (lint, type-check, test) and `npm run format:check`
4. **If there are local errors** — Add to the plan to fix them
5. **Check review comments** — Look at the PR for any comments from automated code review workflows
6. **If there are review issues** — Add to the plan to fix them

If fixing an error requires changing functionality in any way, ask the user if they want to proceed with that change.
