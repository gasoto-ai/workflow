---
name: reflect
type: methodology
description: End-of-session reflection that captures learnings to persistent memory. This is a sequential workflow — follow these steps in order. Use at the end of significant sessions.
---

# Reflect — Session Learning Capture

Review the current session and persist useful learnings to memory files so future sessions benefit.

## Step 1: Review the Session

Look back at the conversation history and identify:
1. **What work was done** — features built, bugs fixed, refactors, investigations
2. **What was discovered** — codebase patterns, gotchas, unexpected behaviors
3. **What went wrong** — mistakes, false starts, debugging dead ends
4. **What the user prefers** — workflow choices, communication style, tool preferences
5. **What could be done better** — approaches that should be tried first next time

## Step 2: Read Existing Memory

Read all files in the project's memory directory to avoid duplicating existing knowledge. Look for a `memory/` folder in the project root or Claude project directory.

Read `MEMORY.md` and any topic files that exist (e.g., `patterns.md`, `debugging.md`, `preferences.md`).

## Step 3: Categorize New Learnings

Sort findings into these categories. Only include genuinely new or updated insights — skip anything already captured:

### Codebase Patterns
Structural patterns confirmed through actual code interaction (not just reading docs). Examples:
- "Country config X requires Y special handling"
- "Component Z uses pattern A instead of the documented pattern B"
- "Service function for X lives at unexpected path Y"

### Debugging Insights
Problems encountered and their root causes. Examples:
- "Error X is caused by Y, fix by doing Z"
- "When tests fail with error X, check Y first"
- "Build failures related to X are usually caused by Y"

### User Preferences
How this user likes to work. Examples:
- "Prefers detailed plans before implementation"
- "Wants tests written alongside features, not after"
- "Likes concise commit messages"

### Skill Gaps
Things the current skills don't cover well that came up during the session. Examples:
- "writing-forms skill doesn't cover dynamic field arrays"
- "No guidance on handling LaunchDarkly flags in tests"

### Architecture Notes
Structural decisions or constraints discovered. Examples:
- "Module X depends on Y in a non-obvious way"
- "Migration from A to B is blocked by C"

## Step 4: Update Memory Files

### MEMORY.md (Index File)
- Keep under 200 lines (it's always loaded into context)
- Contains a brief summary of the most important patterns and a table of contents linking to topic files
- Update existing entries if the session refined understanding
- Remove entries that turned out to be wrong

### Topic Files
Create or update topic files as needed:
- `patterns.md` — Confirmed codebase patterns and conventions
- `debugging.md` — Debugging insights and solutions
- `preferences.md` — User workflow and communication preferences
- `skill-gaps.md` — Areas where skills need improvement
- `architecture.md` — Non-obvious architectural decisions and dependencies

### Update Rules
- **Verify before writing:** Only write patterns confirmed by actual code interaction, not assumptions
- **No duplicates:** Check if the insight already exists before adding
- **Update > append:** If a learning refines an existing entry, update it rather than adding a new one
- **Remove if wrong:** If the session revealed a previous learning was incorrect, remove or correct it
- **Be specific:** Include file paths, function names, and concrete details — not vague observations
- **Date entries:** Add `(learned YYYY-MM-DD)` after new entries so we can track freshness

## Step 5: Summarize to User

Tell the user:
1. How many new learnings were captured (by category)
2. How many existing entries were updated or corrected
3. Any skill gaps identified that might warrant a skill update
4. A one-line summary of the most valuable learning from this session

## Guidelines

- **Be selective.** Not every session produces learnings. If nothing genuinely new was discovered, say so — don't manufacture insights.
- **Prefer specificity over generality.** "The `getProducts` service needs market code in the header for JP/KR" is better than "Some services need special headers."
- **Don't duplicate CLAUDE.md.** Memory is for things discovered through experience, not things already documented in project instructions.
- **Keep MEMORY.md lean.** It's loaded into every conversation. Put details in topic files and link from MEMORY.md.
