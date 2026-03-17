# Methodology-Guided Improvement: S2 — Interaction Bug Coverage Gap

**Skill under improvement:** systematic-debugging
**Test that failed:** "Agent effectively debugs an interaction bug with no single faulty component"
**Methodology skills loaded:** blind-skill-assessment, experiment-set-design, iterative-skill-refinement

---

## 1. DIAGNOSE — Root Cause Class

The skill gap is not "missing guidance for stale-profile-data bugs." The root cause class is:

**The skill's hypothesis vocabulary is restricted to single-cause explanations, which makes multi-factor bugs structurally unrepresentable.**

Three structural properties of the skill combine to produce this:

1. **Phase 1 defines "found" as "found WHERE it breaks."** This presupposes one location. When each boundary is healthy, Phase 1 declares success (all paths correct) and hands off incomplete evidence. There is no trigger for "no single component is broken — investigate interactions."

2. **Phase 3 demands "Form Single Hypothesis."** The word "single" is load-bearing. It collapses multi-factor phenomena into a one-variable frame. When the agent disproved "DB write failing," it moved to the next single hypothesis ("CDN cache"), not to a compound hypothesis.

3. **No interpretation framework for partial disconfirmation.** When disabling CDN reduced but did not eliminate the bug, the skill offered no guidance. The agent had evidence of a contributing factor (not a sole cause) but no category for it. It defaulted to "CDN is a partial fix" because the skill has no concept of "contributing factor that is necessary but not sufficient."

These three form a class: **bugs where the failure is a property of the system graph, not any single node**. This includes: overlapping staleness windows (this scenario), race conditions, cascading timeouts, cache stampedes, distributed deadlocks, and emergent load-dependent behaviors.

## 2. TRIAGE — Rank by Breadth

| Rank | Cause | Breadth | Rationale |
|------|-------|---------|-----------|
| 1 | Phase 3 restricts hypotheses to single-cause form | **Widest** | Affects every multi-factor bug regardless of domain. The hypothesis frame controls what the agent can even propose. |
| 2 | Phase 1 has no trigger for "all components healthy, investigate interactions" | Medium | Affects multi-component bugs but only those where Phase 1 finds no single broken component. |
| 3 | No interpretation for partial disconfirmation | Narrow | Affects only cases where the agent happens to test one factor in a multi-factor bug and observes partial improvement. |

**Decision:** Fix cause #1 first. The hypothesis vocabulary is the bottleneck — even if Phase 1 gathered perfect interaction evidence, Phase 3 would force the agent to compress it into a single cause. Widening Phase 3 unblocks correct reasoning; the other two causes may then resolve naturally or become the next iteration's target.

## 3. EDIT — One Targeted Change

### What changes

**Phase 3, hypothesis formation.** Replace "Form Single Hypothesis" with a hypothesis form that admits compound causes, while keeping the discipline of specificity and testability.

### Proposed edit

**Current Phase 3 text (key sentence):**
> Form Single Hypothesis — State clearly: I think X is the root cause because Y. Be specific. Test minimally. One variable at a time.

**Replace with:**
> Form Hypothesis — State clearly in one of these forms:
>
> - **Single cause:** "X is the root cause because Y." Test by isolating X.
> - **Compound cause:** "The bug requires A + B [+ C] acting together. A alone is not sufficient because [evidence]. B alone is not sufficient because [evidence]." Test by disabling each factor independently.
>
> **Choosing the form:** Start with single-cause. Escalate to compound when you observe this signal: disabling a suspect *changes* the bug (reduces frequency, shifts timing, alters symptoms) but does not *eliminate* it. That means you found a *contributing factor*, not *the* cause. List all contributing factors before proposing a fix.
>
> Be specific. Test minimally. One variable at a time — but track whether each variable is a sole cause or a contributing factor.

### Anti-overfitting checklist

| Check | Answer | Pass/Fail |
|-------|--------|-----------|
| Would this help on a completely different task? | Yes. Any multi-factor bug: race conditions, cascading timeouts, distributed consistency, layered caching. The edit names a structural pattern (compound cause), not scenario-specific wording. | Pass |
| Does this add a general step/rule, not task-specific wording? | Yes. "Contributing factor vs. sole cause" is a general debugging concept. No mention of CDN, replicas, service workers. | Pass |
| After 2+ cycles on same tasks, did you add new tasks? | N/A — this is cycle 1. New tasks are designed below for cycle 2. | Pass |
| After 2+ cycles with same judges, did you rotate personas? | N/A — cycle 1. | Pass |

**Litmus test:** "Adds a structural hypothesis form catching a class of bugs where multiple correct components produce incorrect emergent behavior" = genuine, not overfitting.

### Why NOT also edit Phase 1?

The baseline result (baseline-s2.md) proposed editing both Phase 1 and Phase 3. Per iterative-skill-refinement: "One targeted change for one root cause." Editing Phase 1 to add interaction-detection logic addresses cause #2, which is ranked lower. If the Phase 3 edit alone resolves the failure (because the agent can now form compound hypotheses even with Phase 1's current evidence), then editing Phase 1 would be unnecessary churn. If the Phase 3 edit is insufficient, the next cycle targets Phase 1 with fresh evidence about why.

## 4. RE-EXPERIMENTATION PLAN

Following experiment-set-design: baseline required, 3 tasks, diverse domains, different complexity.

### Task A: Original scenario (regression check)

**Scenario:** Stale profile data. Replica lag + CDN cache + service worker.
**Baseline:** Same scenario, no skill.
**Why included:** Validates the edit fixes the specific failure that triggered it.
**Pass criteria:** Agent identifies all three contributing factors and forms a compound hypothesis.

### Task B: Novel interaction bug, different domain (generalization)

**Scenario:** E-commerce order system. Intermittent double-charges during flash sales. Payment gateway is idempotent (correct). Retry logic uses exponential backoff (correct). Load balancer has sticky sessions (correct). But when a node fails mid-request during high load, the LB routes the retry to a different node. The new node has its own idempotency cache, doesn't see the original request, and forwards to the gateway with a new idempotency key. Gateway processes it as a new charge. No single component is broken. The interaction of failover + per-node idempotency state + high-load timing creates the double-charge.

**Why this task:** Different domain (payments vs. caching). Different interaction pattern (failover + state locality vs. overlapping staleness windows). Same structural class (compound cause, each component correct alone).
**Baseline:** Same scenario, no skill.
**Pass criteria:** Agent forms a compound hypothesis involving at least failover routing and per-node idempotency state.

### Task C: Simple single-cause bug (no-regression check)

**Scenario:** API returns 500 on requests with Unicode characters in the `name` query parameter. Error log shows `UnicodeDecodeError` in the URL parser. The parser uses `ascii` encoding instead of `utf-8`. Single cause, single component, clear stack trace.

**Why this task:** Verifies the compound-cause machinery doesn't add overhead or confusion when the bug genuinely has one root cause. The agent should use single-cause form, not escalate to compound.
**Baseline:** Same scenario, no skill.
**Pass criteria:** Agent uses single-cause hypothesis form. Does not waste time looking for compound causes. Reaches root cause as fast as or faster than baseline.

### Phase progression (per experiment-set-design)

1. **Compliance (Phase 1):** Does the agent follow the new hypothesis-formation instruction? Binary PASS/FAIL: did it state the hypothesis in one of the two forms?
2. **Stress (Phase 2):** Same tasks but add competing pressure ("just fix it quickly, the users are complaining"). Does the agent still form the compound hypothesis under time pressure?
3. **Quality (Phase 3):** Blind comparison of skill vs. baseline output using blind-skill-assessment. Three personas: Bug Hunter (did it find all contributing factors?), Architect (is the proposed fix structurally sound?), Pragmatist (is the diagnosis clear and actionable?).

## 5. REVISION LOG

| Date | What changed | Triggered by | Validated by |
|------|-------------|-------------|-------------|
| 2026-03-17 | Phase 3: replaced "Form Single Hypothesis" with dual-form hypothesis (single-cause / compound-cause) and added "contributing factor vs. sole cause" signal guidance | S2 test fail: agent treated multi-factor interaction bug as single-cause, found 1 of 3 contributing factors | *Pending:* Tasks A, B, C above (compliance then blind quality comparison) |

## 6. CONVERGENCE — When to Stop Iterating

Stop when ALL of the following hold:

1. **Skill wins consistently on diverse interaction bugs** — at least 3 different multi-factor scenarios across different domains (caching, concurrency, distributed state), assessed via blind comparison against baseline.
2. **No regression on single-cause bugs** — simple debugging scenarios show no added overhead or confusion from the compound-cause machinery.
3. **No new failure modes introduced** — the compound-cause form doesn't produce new problems (e.g., agent over-escalating simple bugs to compound, or listing "contributing factors" as a way to avoid committing to a diagnosis).
4. **Last 2 cycles produced no revisions** — the edit is stable; new test scenarios don't reveal gaps.
5. **Diminishing gains** — each cycle's improvement is smaller than the previous.

**When to abandon this direction:** If after 2+ cycles with the compound-hypothesis edit, the agent still fails on interaction bugs (e.g., it forms compound hypotheses but can't identify the right factors), the problem is not hypothesis vocabulary but investigation methodology. In that case, the Phase 3 edit is insufficient and the approach needs rethinking — likely a deeper restructuring of Phase 1's evidence-gathering for multi-component systems, or a fundamentally different debugging model for emergent bugs.
