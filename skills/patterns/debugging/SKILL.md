---
name: debugging
type: pattern
description: Lightweight 4-phase protocol for root cause analysis. Use when a test is failing unexpectedly, a Vex block is unclear, or a bug is proving hard to reproduce.
---

# Debugging Protocol

When something is broken and the cause isn't obvious, follow these four phases in order.
Do not guess. Do not skip to "try something and see."

## Phase 1: Reproduce

Confirm the failure before doing anything else.

- Can you reproduce it consistently?
- Which test, command, or flow triggers it?
- Does it reproduce on a clean checkout?

If you can't reproduce it reliably, stop here. An unreliable reproduction means you don't understand the problem yet. Note what conditions seem to trigger it and gather more information.

**Output:** A single reproducible case — ideally a failing test or a minimal command sequence.

---

## Phase 2: Isolate

Narrow the failure to the smallest possible scope.

- Which file or function is the actual source?
- Comment out, remove, or simplify surrounding code until the failure is isolated
- Check: is this failing because of the code you wrote, or something it depends on?
- Is the failure environmental (test setup, stale state, ordering dependency)?

**Useful questions:**
- Does this fail in isolation, or only in combination with something else?
- What changed most recently that's relevant?
- Is there a simpler version of this that still fails?

**Output:** The exact line, function, or precondition that causes the failure.

---

## Phase 3: Fix

With the cause isolated, apply the minimal fix.

- Fix only the identified cause — do not refactor surrounding code in the same change
- If the fix requires a broader change, note it and address it separately
- Prefer correctness over elegance; you can refactor after tests pass
- Write or update the test to verify the fix

**Anti-patterns:**
- Don't fix the symptom instead of the cause
- Don't change multiple things at once to "try something"
- Don't delete tests to make the build pass

---

## Phase 4: Verify

Confirm the fix actually works and didn't break anything else.

- The failing test now passes
- All other tests still pass
- The fix is reproducible — run the test suite a second time if there's any doubt
- If the failure was environmental (ordering, stale state), confirm it doesn't reappear under normal conditions

**Output:** Clean test run. Commit only after verification.

---

## When to Escalate

If you've completed all four phases and the root cause is still unclear, stop and report what you found — don't keep guessing. Post your reproduction case, what you isolated, and where you're stuck. It's faster for the team to debug with that information than to iterate blindly.
