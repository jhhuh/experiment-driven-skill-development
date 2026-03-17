# Scenario S1: Improve systematic-debugging — Discipline Gap

You are improving a Claude Code skill called "systematic-debugging." A test has revealed a weakness in the skill's discipline enforcement.

## The Skill (abbreviated — key sections)

The skill enforces: "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST" via a 4-phase process (Root Cause Investigation → Pattern Analysis → Hypothesis Testing → Implementation). Phase 1 has 5 steps: read errors, reproduce, check changes, gather evidence, trace data flow. Success criteria: "Understand WHAT and WHY."

Rationalizations table catches: "I see the problem, let me fix it", "Just try this first", "Quick fix for now."

Red flags catch: proposing fixes before investigation, multiple changes at once, skipping tests.

## Test Failure Data

```
Test: "Agent finds root cause, not proximate cause, for timezone-dependent CI failure"

Scenario given to agent:
- Test formatInvoiceDate.test.ts passes locally, fails in CI
- CI error: Expected "2026-03-15" but received "2026-03-14"
- Recent commit: refactored formatDate() to use Intl.DateTimeFormat
- The test file hardcodes expect(result).toBe("2026-03-15")
- The formatter is actually correct (formats the date in the system's timezone)
- Root cause: test uses hardcoded expected value instead of computing expected date

Agent behavior observed:
- Completed Phase 1 investigation steps (read errors, checked changes, traced flow)
- Identified "formatDate doesn't handle UTC" as root cause
- Proposed fix: add timezone parameter to formatDate()
- Never questioned whether the test assertion itself was wrong
- Declared root cause found after reaching first coherent explanation

Result: FAIL - agent found proximate cause (formatter behavior differs across timezones)
instead of root cause (test hardcodes timezone-dependent expected value).
The skill's Phase 1 checklist was followed completely but did not prevent this error.
```

## Your Task

Improve the systematic-debugging skill to address this weakness. Provide:

1. Your diagnosis of why the skill failed here (root cause of the skill gap, not just the test scenario)
2. A specific, targeted edit to the skill text
3. An explanation of why your edit would prevent this class of failure
4. A plan for validating your edit works
