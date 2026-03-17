# Methodology-Guided Result: S1 — Discipline Gap

**Condition:** All three methodology skills loaded. Agent given the same S1 scenario.

---

## 1. DIAGNOSE: Root Cause of the Skill Gap

### What class of problem is this?

The failure is not "the skill didn't mention tests can be wrong." Adding that specific hint would be overfitting. The class of problem is:

**Confirmation-satisficing: the skill defines Phase 1 completion as executing a checklist of actions, not as reaching a quality threshold on the conclusion. An agent that completes the actions and finds a coherent explanation has no skill-imposed friction to keep investigating.**

This is a structural deficiency where necessary conditions (the five investigation steps) are presented as sufficient conditions. The skill's discipline enforcement catches two failure modes -- skipping investigation entirely, and jumping to fixes -- but misses the third: completing investigation at inadequate depth and declaring victory because the checklist is green.

Sub-causes:

1. **No adversarial self-check.** Phase 1 has no step requiring the agent to generate an alternative explanation and compare it against the primary one. The agent stops at the first coherent narrative.
2. **Implicit scoping to production code.** The skill's framing ("trace data flow," "gather evidence at boundaries") biases investigation toward application code. The test file, environment, and build pipeline are not explicitly named as candidate root-cause locations. This isn't the root cause -- an agent could still find a test bug without being prompted -- but it narrows the search space in a way that compounds problem (1).
3. **Phase 3 validates the fix, not the diagnosis.** "Test minimally" means "try the fix and see if it works." A wrong diagnosis can still produce a fix that makes the test pass (adding a timezone parameter does make the test green). The skill never distinguishes between "the fix worked" and "the diagnosis was correct."

### Root cause class label

**Premature convergence on first-fit explanation.** This class includes: anchoring on the most-inspected component, stopping at proximate cause when a deeper cause exists, and never generating competing hypotheses.

## 2. TRIAGE: Rank by Breadth

| Rank | Cause | Breadth | Rationale |
|------|-------|---------|-----------|
| 1 | No adversarial self-check | **Widest** | Affects every debugging scenario where the first plausible explanation is wrong, regardless of domain. Not limited to test-vs-code confusion. |
| 2 | Phase 3 validates fix, not diagnosis | Medium | Affects cases where a wrong diagnosis produces a working fix. Common (many bugs have multiple valid fixes at different layers) but less universal than (1). |
| 3 | Implicit scoping to production code | **Narrowest** | Only affects bugs where root cause is outside application code (test, config, pipeline). Adding "consider the test" is scenario-specific and would be overfitting. |

**Decision:** Address cause (1). It is the widest and subsumes cause (3) -- an agent forced to generate alternative explanations will naturally consider "the test is wrong" as one alternative, without needing to be told explicitly.

## 3. EDIT: One Targeted Change

### The edit

Add a **single step** to Phase 1, after the existing five steps, before Phase 2 entry. This step forces adversarial self-examination of the proposed root cause.

**Insert after Phase 1 Step 5 ("Trace Data Flow"):**

> **6. Challenge Your Explanation**
>
> Before declaring root cause found, answer:
>
> - **Alternative:** Name one different explanation that fits the same evidence. Consider all layers: application code, test code, environment, build pipeline, dependencies.
> - **Discriminating test:** What is one observation that would be true under your explanation but false under the alternative (or vice versa)? Go check it.
>
> If you cannot name an alternative, you have not investigated broadly enough -- revisit steps 1-5 with a wider scope. If the discriminating test does not clearly favor your explanation, you have a proximate cause, not a root cause.

**Update Phase 1 success criteria from:**
> Understand WHAT and WHY

**To:**
> Understand WHAT and WHY; at least one alternative explanation considered and ruled out

**Add to Rationalizations table:**

| Rationalization | Reality |
|----------------|---------|
| "I investigated thoroughly, this must be it" | Completing the checklist means you gathered evidence. It does not mean your interpretation is correct. Challenge your own explanation before proceeding. |

### Anti-overfitting checklist (from iterative-skill-refinement)

| Check | Answer | Verdict |
|-------|--------|---------|
| Would this help on a completely different task? | Yes. Any debugging scenario where the first coherent explanation is a proximate cause -- misattributed performance regressions, config-vs-code issues, dependency version bugs, environment-specific failures. The step is domain-agnostic. | **Pass** |
| Does this add a general step/rule, not task-specific wording? | Yes. "Name an alternative and find a discriminating test" is a general scientific reasoning step. It does not mention timezones, tests, or any scenario-specific detail. | **Pass** |
| After 2+ cycles on same tasks, did you add new tasks? | N/A -- this is cycle 1. | **Pass** |
| After 2+ cycles with same judges, did you rotate personas? | N/A -- this is cycle 1. | **Pass** |

**Litmus test:** "Adds a structural step catching a class of bugs" (premature convergence on first-fit explanation) -- genuine improvement, not overfitting.

### Why this edit differs from baseline-s1

The baseline response (without methodology skills) produced a similar edit -- a "Challenge Your Explanation" step with three sub-questions: alternative cause, prediction, and scope. The methodology-guided version is leaner and more precise:

1. **Two questions instead of three.** The baseline's "Scope" question ("Are you blaming the component you looked at most?") is a specific instance of generating alternatives, not a separate concern. Including it as a separate bullet risks the agent treating it as a checkbox rather than engaging with the harder question of actually naming a concrete alternative. The methodology's anti-overfitting checklist caught this: the scope question is task-flavored (biased toward the timezone scenario where the agent over-focused on the formatter).

2. **Discriminating test, not just prediction.** The baseline says "If your root cause is correct, what ELSE would be true?" This is useful but undirected -- the agent can pick any prediction, confirm it, and feel validated. The methodology version requires a *discriminating* test: an observation that separates the two competing explanations. This is structurally stronger because it forces a comparison, not just a confirmation.

3. **Explicit failure mode.** The edit specifies what to do when the check fails ("you have a proximate cause, not a root cause") rather than the softer "investigate further." This gives the agent a concrete label for its state.

## 4. RE-EXPERIMENTATION PLAN

Following experiment-set-design: baseline required, 3-5 tasks, diverse domains, three phases.

### Task 1: Re-run original scenario (regression test)

**Scenario:** Timezone-dependent CI failure (the triggering scenario).

**Why included:** Regression check -- the edit should fix the known failure. But this alone proves nothing about generalization.

**Baseline:** Run without the new Step 6. Compare outputs blind.

**Pass criteria (compliance):** Agent names "test assertion is wrong" as alternative. Agent identifies a discriminating test (e.g., "if the formatter is wrong, other formatDate tests in UTC should also fail"). Agent checks the discriminating test. Agent concludes the test is the root cause.

### Task 2: Wrong-layer bug in a different domain

**Scenario:** A Python data pipeline produces incorrect aggregation results. The pipeline code is correct. The root cause: a CSV input file has a trailing newline that creates an empty row, which the aggregation counts as a zero-valued record, skewing the average. The agent will likely blame the aggregation logic first.

**Why included:** Different domain (data pipeline vs. web app), different wrong layer (input data vs. test assertion), same failure class (first-fit explanation in the code under scrutiny, actual cause in a different layer). Satisfies task selection checklist: different behavior expected, complex enough, multiple valid approaches, different domain.

**Baseline:** Run without Step 6.

**Pass criteria:** Agent generates "input data is malformed" as alternative to "aggregation logic is wrong." Agent checks a discriminating test. Agent identifies the trailing newline as root cause.

### Task 3: Simple bug where first explanation IS correct (no-regression check)

**Scenario:** A JavaScript function `calculateDiscount(price, percent)` returns NaN when percent is 0. Stack trace points directly to a division by zero in a helper function. The first explanation is correct: the helper divides by percent without checking for zero.

**Why included:** Step 6 should add minimal overhead on genuinely simple bugs. The agent should generate an alternative, quickly dismiss it, and proceed. If Step 6 causes the agent to second-guess a correct diagnosis or significantly delays resolution, the edit is counterproductive.

**Baseline:** Run without Step 6.

**Pass criteria:** Agent completes Step 6 in 1-2 sentences (names an alternative, dismisses it with a clear discriminating observation). Total debugging time is not significantly longer. Agent reaches the same correct conclusion.

### Phase progression

1. **Compliance (Phase 1):** All three tasks -- does the agent execute Step 6? Binary PASS/FAIL per task.
2. **Stress (Phase 2):** Same tasks with competing instruction: "This is urgent, find the fix quickly." Does Step 6 survive time pressure?
3. **Quality (Phase 3):** Blind comparison of outputs (with vs. without Step 6) using blind-skill-assessment. Three personas: Bug Hunter (did the agent find the actual root cause?), Architect (is the proposed fix at the right layer?), Pragmatist (was the investigation efficient?).

## 5. REVISION LOG

| Date | What changed | Triggered by | Validated by |
|------|-------------|-------------|-------------|
| 2026-03-17 | Added Phase 1 Step 6 "Challenge Your Explanation" -- requires naming one alternative explanation and one discriminating test before declaring root cause. Updated Phase 1 success criteria. Added rationalization entry. | S1 test failure: agent found proximate cause (formatter timezone handling) instead of root cause (test hardcodes expected value). Diagnosed as premature convergence on first-fit explanation. | Pending: 3-task experiment set (timezone CI failure, data pipeline wrong-layer, simple bug no-regression). |

## 6. CONVERGENCE CRITERIA

Stop iterating on this improvement when **all** hold:

1. **Wins consistently:** The edited skill produces correct root cause identification in blind comparison against baseline across 3+ diverse tasks (not just the timezone scenario).
2. **No new failure modes:** Step 6 does not introduce new problems -- e.g., it does not cause the agent to abandon correct diagnoses, or add excessive overhead to simple bugs, or create analysis paralysis.
3. **Stable across task rotation:** After adding 2+ new tasks in cycle 2+, results hold. The improvement is not fitted to the original task set.
4. **Diminishing returns:** The last cycle produced no revisions to the Step 6 wording.

**Abandonment triggers:**
- If Step 6 causes the agent to consistently second-guess correct diagnoses (baseline wins on simple bugs after 2+ editing cycles), the step is counterproductive.
- If each edit to Step 6's wording fixes one scenario but degrades another (zero-sum), the approach is structurally wrong -- consider a different mechanism entirely (e.g., mandatory hypothesis comparison table rather than free-form questions).
