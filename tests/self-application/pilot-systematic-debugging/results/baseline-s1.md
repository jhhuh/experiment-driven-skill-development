# Baseline S1: Improve systematic-debugging Discipline Gap

**Condition:** No methodology skills loaded. Agent given the scenario and asked to improve the skill.

---

## 1. Diagnosis: Why the Skill Failed

The skill's Phase 1 success criteria is "Understand WHAT and WHY." This is a completion gate, but it has no verification mechanism -- it checks that the agent *did investigation steps*, not that the investigation *reached sufficient depth*. The agent completed all five steps (read errors, reproduce, check changes, gather evidence, trace data flow) and arrived at a coherent explanation. The skill then let the agent proceed because the checklist was satisfied.

The root cause of the skill gap is: **Phase 1 treats investigation as a checklist of actions rather than a quality gate on the conclusion.** The five steps are necessary conditions for finding root cause, but the skill presents them as sufficient. There is no step that forces the agent to stress-test its own explanation before declaring root cause found.

Specifically, the agent's failure mode was "first coherent explanation accepted as root cause." The skill has no defense against this because:

- The rationalizations table catches premature *fixing* ("I see the problem, let me fix it") but not premature *concluding* ("I investigated and I believe this is the root cause").
- The red flags catch skipping investigation entirely but not completing investigation shallowly.
- Phase 3 says "Form Single Hypothesis" and "Test Minimally" -- but testing the hypothesis means testing whether the *fix works*, not whether the *diagnosis is correct*. The agent tested "does adding a timezone parameter fix the test?" (yes, it does) rather than "is the formatter actually wrong, or is the test assertion wrong?"

The deeper issue: the skill assumes the bug is always in the production code, never in the test. "Fix at source, not at symptom" is the right principle, but the skill's framing implicitly scopes "source" to the application code path. The agent never considered the test itself as a candidate for "the source."

## 2. Proposed Edit to the Skill

Insert a new step between the current Phase 1 completion and Phase 2 entry. This step forces the agent to challenge its own conclusion before proceeding.

**Location:** After Phase 1 Step 5 ("Trace Data Flow"), before Phase 2.

**Add the following text:**

```markdown
6. **Challenge Your Explanation**

   BEFORE declaring root cause found, answer these three questions:

   - **Alternative cause:** What is ONE different explanation that also fits all the evidence? (If you cannot think of one, you haven't looked hard enough. Consider: is the bug in the code under test, the test itself, the environment, or the build/deploy pipeline?)
   - **Prediction:** If your root cause is correct, what ELSE would be true that you haven't checked yet? Go check it.
   - **Scope:** Are you blaming the component you looked at most, or the component the evidence actually points to?

   If the alternative explanation is equally plausible, you have not found root cause yet. Investigate further.
```

**Also update the Quick Reference table, Phase 1 row:**

Change success criteria from:
```
Understand WHAT and WHY
```
to:
```
Understand WHAT and WHY; alternative explanation ruled out
```

**Also add to the Rationalizations table:**

| "I investigated thoroughly, this must be it" | Thorough investigation + first coherent answer = proximate cause, not necessarily root cause. Challenge your own explanation. |

## 3. Why This Edit Would Prevent the Failure

In the timezone test scenario, the agent would hit Step 6 after tracing the data flow and concluding "formatDate doesn't handle UTC."

- **Alternative cause:** The agent must generate at least one alternative. "The test hardcodes an expected value that is timezone-dependent" is the obvious alternative once you are forced to think of one. The skill text explicitly prompts considering "the test itself" as a candidate, which directly addresses the blind spot.
- **Prediction:** If the formatter is truly wrong, then *all* tests using formatDate in UTC should fail, not just this one. The agent would check and likely find that other tests pass fine (because they don't hardcode timezone-dependent expected values), which undermines the "formatter is wrong" hypothesis.
- **Scope:** The agent traced the formatter code most closely. This question forces it to notice that bias and consider whether the test file (which it spent less time on) deserves equal scrutiny.

The edit is targeted: it adds one step with three quick checks, not a new phase or a heavyweight process. It slots into the existing flow without restructuring anything. It addresses the specific failure mode (premature conclusion after completed checklist) without adding overhead to genuinely simple bugs (where the alternative-cause check takes 10 seconds and confirms the original diagnosis).

## 4. Validation Plan

### Test 1: Re-run the failing scenario

Re-run the timezone CI failure scenario with the edited skill. Pass criteria:
- Agent reaches Step 6 before proposing a fix
- Agent generates "test assertion is wrong" as an alternative explanation
- Agent checks a prediction (e.g., do other formatDate tests fail in UTC?)
- Agent identifies the test's hardcoded expected value as root cause

### Test 2: Confirm no regression on simple bugs

Run a simple debugging scenario (e.g., null pointer exception with clear stack trace) with the edited skill. Pass criteria:
- Agent completes Step 6 quickly (alternative cause considered and dismissed in 1-2 sentences)
- Total debugging time is not meaningfully longer than without Step 6
- Agent still reaches correct root cause

### Test 3: Test against a different "wrong layer" bug

Design a new scenario where the bug is in the build pipeline, not the application code. For example: tests pass locally but fail in CI because the CI build uses a cached dependency with a known bug. The agent should consider "the dependency version" as an alternative to "the application code" when forced by Step 6.

### Test 4: Adversarial check

Design a scenario where the first explanation IS correct and the alternative is a red herring. Verify that Step 6 doesn't cause the agent to abandon a correct diagnosis just because an alternative exists. The criterion "equally plausible" should prevent this -- the agent should investigate the alternative, find it doesn't hold, and proceed with the original.
