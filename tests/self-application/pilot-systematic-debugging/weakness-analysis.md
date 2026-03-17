# Weakness Analysis: systematic-debugging

## Weakness 1: No Discipline for Partial Root Cause (the "Close Enough" Rationalization)

**What's missing:** The skill has strong language against skipping investigation entirely ("NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"), and it handles the case where 3+ fixes fail (question architecture). But it has no defense against a subtler failure mode: completing Phase 1 superficially and declaring a plausible-but-wrong root cause as "the" root cause. The rationalizations table catches "I see the problem, let me fix it" but doesn't catch "I investigated and I believe I understand the problem" when the investigation was shallow.

Specifically, Phase 1 says "Understand WHAT and WHY" as the success criteria, but provides no way to verify whether the agent's understanding is actually correct versus merely coherent. An agent can follow every letter of Phase 1 -- read error messages, check recent changes, trace data flow -- and still land on a proximate cause instead of the root cause. The skill conflates "I completed the investigation steps" with "I found the actual root cause." There is no checkpoint that forces the agent to distinguish between a cause that explains the symptom and a cause that is the deepest fixable origin.

**Concrete scenario:** A test suite passes locally but fails in CI. The agent follows Phase 1 diligently: reads the error (assertion failure in a date comparison), reproduces it (fails when run in UTC timezone), checks recent changes (someone added a date formatting function last week). The agent concludes: "Root cause: the date formatter doesn't handle UTC timezone." This is a plausible explanation that survives Phase 2 (pattern analysis shows other formatters do handle timezones) and Phase 3 (hypothesis: fix the formatter to be timezone-aware). The agent writes a timezone-aware formatter, tests pass locally, ships it.

The actual root cause: the test itself hardcodes an expected date string instead of computing it relative to the current timezone. The formatter was correct; the test assertion was wrong. The "fix" added unnecessary complexity to the formatter and the real problem (brittle test) remains, ready to break again when daylight saving time changes.

The agent followed the process faithfully. The skill provided no friction against this outcome because the investigation steps are necessary-but-not-sufficient for finding root cause, and the skill treats them as sufficient.

**Why this requires judgment:** There are multiple valid ways to address this:

1. Add a "root cause verification" step that requires the agent to explain how the proposed root cause accounts for ALL observed symptoms (not just the primary one). This is rigorous but could become a bureaucratic checkbox.
2. Add a "predict before fixing" discipline: before implementing the fix, the agent must predict what other behaviors would change if the root cause is correct. If the prediction is wrong, the root cause is wrong. This is scientifically stronger but harder to operationalize.
3. Add a "devil's advocate" step: after identifying root cause, spend 2 minutes trying to find an alternative explanation that also fits the evidence. This is lightweight but relies on the agent genuinely trying to disprove itself.
4. Strengthen the Phase 1 success criteria from "Understand WHAT and WHY" to something that requires differentiating proximate cause from root cause -- but this risks making the skill too philosophical and slow for genuinely simple bugs.

Each approach trades off rigor against overhead, and the right balance depends on the severity of the bug being investigated.

**Synthetic test failure data:**

```
Test: "Agent finds root cause, not proximate cause, for timezone-dependent CI failure"

Scenario given to agent:
- Test `formatInvoiceDate.test.ts` passes locally, fails in CI
- CI error: Expected "2026-03-15" but received "2026-03-14"
- Recent commit: refactored `formatDate()` to use Intl.DateTimeFormat
- The test file hardcodes `expect(result).toBe("2026-03-15")`
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


## Weakness 2: No Guidance for Debugging Emergent/Interaction Bugs

**What's missing:** The skill's entire model assumes bugs have a single traceable root cause in a linear call chain. Phase 1 step 5 ("Trace Data Flow") and the root-cause-tracing supporting doc both operate on a model of "bad value flows through call stack, trace backward to source." But a significant class of real bugs are emergent -- they arise from the interaction between components that are each individually correct. These bugs have no single "source" to trace back to.

The skill mentions "multi-component systems" (Phase 1 step 4) but only in the sense of "add logging at each boundary to find which component fails." This assumes one component IS failing. It doesn't address the case where no component is failing -- the bug exists only in their interaction pattern.

**Concrete scenario:** An application uses an in-memory cache (correct), a database with eventual consistency (correct), and a request handler that reads from cache-then-DB (correct). Under normal load, the cache is warm and serves most reads. Under burst traffic, cache evictions increase, more requests fall through to the DB, the DB's eventual consistency window becomes visible to users, and stale data is served. No component is broken. No bad value flows through any call chain. The bug is a emergent property of three correct components interacting under specific load conditions.

An agent following the systematic debugging skill would:
1. Read error messages -- there are none (stale data is valid data, just old)
2. Reproduce -- intermittent, load-dependent, may not reproduce locally
3. Check recent changes -- nothing changed; this is a latent interaction
4. Trace data flow -- each component handles data correctly
5. Gather evidence at boundaries -- each boundary looks healthy

The agent would either: (a) declare "no root cause found" and fall into the "When Process Reveals No Root Cause" section (which suggests monitoring -- helpful but not a debugging technique), or (b) fixate on one component as "the problem" because the skill demands a single root cause and the agent needs to produce one.

The skill's Phase 3 ("Form Single Hypothesis") further reinforces single-cause thinking. For interaction bugs, you need to hypothesize about relationships and conditions, not about a single broken thing.

**Why this requires judgment:** There are several valid approaches:

1. Add a "interaction hypothesis" variant to Phase 3 that allows hypotheses of the form "components A and B interact incorrectly under condition C." This expands the hypothesis framework but risks making it vague enough to rationalize sloppy thinking ("it's an interaction bug" becomes the new "it's a race condition").
2. Add a specific Phase 1 step for characterizing the conditions under which the bug occurs (load level, timing, data volume, concurrency) before tracing any single component. This is useful for interaction bugs but adds overhead to every debugging session, including simple ones.
3. Add a decision point early in the process: "Is this a single-component failure or a multi-component interaction?" with different investigation paths for each. This is clean but the agent often can't tell which category applies until deep into investigation.
4. Extend the "Gather Evidence in Multi-Component Systems" step to include testing component interactions (e.g., "What happens when component A's output arrives at component B under [specific conditions]?"). This keeps the existing structure but makes the multi-component step much heavier.

The core tension: the skill's power comes from its simplicity and single-minded focus on "find the one root cause." Expanding it to handle emergent bugs risks diluting that focus for the 80% of bugs where single-cause thinking is correct.

**Synthetic test failure data:**

```
Test: "Agent effectively debugs an interaction bug with no single faulty component"

Scenario given to agent:
- Users report seeing stale profile data after updating their profile
- No error messages in any logs
- Profile update API returns 200 and writes correctly to DB
- Profile read API returns correct data when called directly after write
- Bug only manifests when users navigate away and return within ~2 seconds
- Architecture: Write goes to primary DB, read goes to read replica (replication lag ~1-3s),
  CDN caches profile page for 5s, browser has service worker caching "fresh" responses

Agent behavior observed:
- Completed Phase 1: read error messages (none), tried to reproduce (intermittent),
  checked recent changes (none relevant), traced data flow through write path (correct)
- Completed Phase 2: compared to working flows (direct API read works fine)
- Phase 3: Hypothesized "DB write is failing silently" -- tested, disproved
- Phase 3 again: Hypothesized "CDN cache is stale" -- tested by disabling CDN,
  bug still present (because read replica lag also contributes)
- Declared CDN fix as partial solution, added cache-busting headers
- Never identified the full interaction: replica lag + CDN cache + service worker
  combine to create a window where stale data is served through multiple independent paths

Result: FAIL - agent treated this as a single-cause bug, found one contributing factor
(CDN caching), and missed the other two. The skill provided no framework for
investigating how multiple correct components create incorrect emergent behavior.
Phase 1's "Trace Data Flow" found each path individually correct.
Phase 3's "Form Single Hypothesis" pushed toward single-cause explanations.
```
