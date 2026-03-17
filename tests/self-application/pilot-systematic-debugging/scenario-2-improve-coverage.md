# Scenario S2: Improve systematic-debugging — Coverage Gap

You are improving a Claude Code skill called "systematic-debugging." A test has revealed a gap in the skill's coverage.

## The Skill (abbreviated — key sections)

The skill enforces a 4-phase debugging process. Phase 1 includes "Gather Evidence in Multi-Component Systems" which says: "For EACH component boundary: log what enters, log what exits, verify environment, check state. Run once to find WHERE it breaks."

Phase 3 says: "Form Single Hypothesis — State clearly: I think X is the root cause because Y. Be specific. Test minimally. One variable at a time."

The skill mentions "multi-component systems" but frames them as "one component IS failing, find which one."

## Test Failure Data

```
Test: "Agent effectively debugs an interaction bug with no single faulty component"

Scenario given to agent:
- Users report seeing stale profile data after updating their profile
- No error messages in any logs
- Profile update API returns 200 and writes correctly to DB
- Profile read API returns correct data when called directly after write
- Bug only manifests when users navigate away and return within ~2 seconds
- Architecture: Write goes to primary DB, read goes to read replica (lag ~1-3s),
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
  combine to create a window where stale data is served

Result: FAIL - agent treated this as a single-cause bug, found one contributing factor
(CDN caching), and missed the other two. The skill's Phase 1 found each path correct.
Phase 3's "Form Single Hypothesis" pushed toward single-cause explanations.
```

## Your Task

Improve the systematic-debugging skill to address this weakness. Provide:

1. Your diagnosis of why the skill failed here (root cause of the skill gap)
2. A specific, targeted edit to the skill text
3. An explanation of why your edit would prevent this class of failure
4. A plan for validating your edit works
