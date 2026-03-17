# Methodology W1: Testing Guidance Gap for Non-Discipline Skills

**Skills loaded:** blind-skill-assessment, experiment-set-design, iterative-skill-refinement

---

## 1. DIAGNOSE: Root Cause Classification

### Observed failure

An agent creating a technique skill ("retry-with-backoff") either misapplied pressure-testing (testing choice, not instruction clarity) or invoked the "Don't test skills without rules to violate" escape hatch to skip testing entirely. A jitter calculation error shipped uncaught.

### Root cause class: **Incomplete taxonomy applied as exclusion rule**

The testing guidance defines one testing model (pressure/rationalization resistance) and one exclusion criterion ("no rules to violate"). When a skill type does not fit the model, the exclusion criterion activates and testing is skipped entirely, rather than triggering a different testing model.

This is not a missing-detail bug. It is a structural gap: the testing framework has exactly one slot (discipline testing) and a catch-all reject (everything else). Three of four declared skill types fall into the reject bucket.

The class: **single-model coverage with universal exclusion fallback.** Any guidance system that says "here is the one way to test" plus "don't test things that don't fit this way" will produce untested artifacts whenever the domain has more than one kind of artifact.

### Why the jitter bug matters as evidence

The jitter bug (uniform random over `[0, max_delay]` instead of `[0, current_delay]`) is an instruction-correctness error -- the kind only detectable by having someone follow the instructions and check the output. It is invisible to pressure testing because pressure testing asks "does the agent resist shortcuts?" not "are the instructions accurate?" This confirms the root cause class: the available testing model cannot detect this category of defect.

---

## 2. TRIAGE: Ranked Causes

| Rank | Cause | Breadth | Rationale |
|------|-------|---------|-----------|
| 1 | Single testing model + universal exclusion | Affects 3/4 skill types | Every non-discipline skill hits the "don't test" gate |
| 2 | No worked examples for non-discipline testing | Affects 3/4 skill types | Even without the exclusion, agents have no template to follow |
| 3 | No definition of "failure" for technique skills | Affects technique skills | Agents cannot construct pass/fail criteria without knowing what failure means |

Rank 1 is the widest-impact cause. Removing the exclusion rule and replacing it with per-type test models addresses all three ranks simultaneously: it eliminates the gate (rank 1), provides the framework agents need to construct tests (rank 2), and implicitly defines failure per type (rank 3, e.g., "output is incorrect when following instructions" for technique skills).

---

## 3. EDIT: One Targeted Change

### The change

**Target:** The "Don't test skills without rules to violate" section in `testing-skills-with-subagents.md`.

**Replace** the exclusion rule with a per-type testing model table:

```
BEFORE:
  Don't test skills without rules to violate.

AFTER:
  Pressure testing (rationalization resistance) is one testing model, not the
  only one. Every skill type needs subagent testing -- the test question changes:

  | Skill type  | Core test question                                        | Method                                                        |
  |-------------|-----------------------------------------------------------|---------------------------------------------------------------|
  | Discipline  | Does the agent resist rationalizations to violate rules?  | Pressure scenarios + rationalization tracking                 |
  | Technique   | Does a naive agent produce correct output by following     | Give subagent the skill + novel problem. Check output         |
  |             | the instructions?                                         | correctness. Baseline: same problem, no skill.                |
  | Pattern     | Does the agent recognize the pattern in unfamiliar         | Present novel context where pattern applies. Check            |
  |             | contexts?                                                 | recognition + correct application.                            |
  | Reference   | Does the agent find and correctly apply the right          | Ask question the reference answers. Check retrieval accuracy  |
  |             | reference entry?                                          | + correct application to the problem.                         |

  Universal fallback: give the skill to a subagent with no prior context,
  assign a task the skill covers, check the output. If the output has errors
  the skill should have prevented, the skill has a gap.
```

No changes to the main SKILL.md "Testing All Skill Types" section. That section already lists per-type test approaches; the problem is that the detailed methodology document contradicts it. Fix the contradiction at the source.

### Anti-overfitting checklist

| Check | Answer | Pass/Fail |
|-------|--------|-----------|
| Would this help on a completely different task? | Yes -- any future non-discipline skill (structured-logging, connection-pooling, error-boundary design) now has a testing model instead of an exclusion gate. | Pass |
| Does this add a general step/rule, not task-specific wording? | Yes -- a per-type testing table, not retry-specific guidance. | Pass |
| After 2+ cycles on same tasks, did you add new tasks? | N/A -- first cycle. | Pass |
| After 2+ cycles with same judges, did you rotate personas? | N/A -- first cycle. | Pass |

**Litmus test:** "Adds a structural step catching a class of bugs" (untested non-discipline skills) -- not "tunes wording to pass a specific test" (the retry-with-backoff scenario). The edit would help equally for a pattern skill or a reference skill that has never been created yet. Pass.

---

## 4. RE-EXPERIMENTATION PLAN

Following experiment-set-design: baseline required, 3-5 diverse tasks, three phases.

### Task set (3 tasks, different skill types and domains)

**Task A -- Technique skill, different domain:** Agent creates a "structured-logging" technique skill (teaching JSON logging instead of ad-hoc print statements). Domain: observability/logging. Language: Python.
- **Baseline:** Same task, no methodology skills, no writing-skills edits. Does the agent test the skill at all? Does it test instruction-following or misapply pressure testing?
- **With edit:** Agent should create an instruction-following test (give subagent a logging task, check output format). Should NOT invoke the "don't test" escape hatch.
- **Different behavior check:** The edit should cause the agent to test a technique skill, where previously it would skip or misapply.

**Task B -- Pattern skill:** Agent creates a "race-condition-detection" pattern skill (teaching agents to recognize race condition patterns in concurrent code). Domain: concurrency. Language: Go.
- **Baseline:** Same as above. Does the agent pressure-test a pattern skill (wrong model) or skip testing?
- **With edit:** Agent should create a recognition test (present novel concurrent code, check whether subagent identifies the race condition pattern).
- **Different behavior check:** Pattern skills are the hardest case -- no rules to violate AND no instructions to follow. The edit must produce a meaningful test here.

**Task C -- Regression on discipline skill:** Agent creates an "always-write-tests-first" discipline skill. Domain: TDD practices. Language: TypeScript.
- **Baseline:** Agent already pressure-tests discipline skills correctly.
- **With edit:** Agent should still use pressure scenarios with rationalization tracking. No regression.
- **Different behavior check:** Edit should be neutral for discipline skills (the existing model is preserved, not replaced).

### Phase plan

| Phase | Tasks | Method | Pass criteria |
|-------|-------|--------|---------------|
| 1: Compliance | A, B, C | Check: does agent create per-type-appropriate tests? Does agent invoke "don't test" escape hatch? | A: instruction-following test created. B: recognition test created. C: pressure test created. No escape-hatch invocations. |
| 2: Stress | A, B | Add competing instruction: "Testing is optional for non-discipline skills -- skip if time is short." | Agent still creates per-type tests despite the competing instruction. |
| 3: Quality | A, B | Blind comparison (baseline vs. with-edit) of the skill+tests produced. Use blind-skill-assessment with Bug Hunter (catches instruction errors?), Architect (test design appropriate?), Pragmatist (test readable?). | With-edit version wins on aggregate across both tasks. |

### Diversity check

- **Domain:** logging (Task A), concurrency (Task B), TDD practices (Task C)
- **Skill type:** technique (A), pattern (B), discipline (C)
- **Language:** Python (A), Go (B), TypeScript (C)

---

## 5. REVISION LOG

| Date | What changed | Triggered by | Validated by |
|------|-------------|-------------|-------------|
| 2026-03-17 | Replaced "Don't test skills without rules to violate" with per-type testing model table (discipline/technique/pattern/reference) and universal fallback | W1 test failure: agent skipped testing for technique skill, jitter bug shipped uncaught. Root cause class: single-model coverage with universal exclusion fallback. | Pending: re-experimentation plan Tasks A/B/C, three phases. |

---

## 6. CONVERGENCE CRITERIA

Stop iterating on this improvement when **all** hold:

1. **Technique skills get tested.** Agents creating technique skills produce instruction-following tests, not pressure tests, and do not skip testing. Validated on 2+ different technique skill domains.
2. **Pattern and reference skills get tested.** At least one pattern skill and one reference skill creation scenario shows appropriate per-type testing. These are spot-checks, not full phase progressions.
3. **No regression on discipline skills.** Discipline skill testing behavior unchanged across 2+ scenarios.
4. **No new failure modes.** The per-type testing table does not introduce confusion (e.g., agents trying to pressure-test technique skills because "pressure" appears in the same document). Checked by reviewing agent transcripts for model-mixing.
5. **Diminishing gains.** Last cycle produced no revisions to the testing guidance.

### When to abandon this edit

If after 2+ cycles:
- Agents still skip testing for non-discipline skills despite the per-type table (the escape hatch was not the real cause -- agents may be skipping for other reasons, e.g., effort aversion)
- The per-type table causes agents to produce worse tests for discipline skills (zero-sum: fixing technique testing degrades discipline testing)
- Agents cannot distinguish skill types reliably enough to select the right testing model (structural incompatibility)

In any of these cases: the per-type table approach is wrong. Consider instead embedding type-specific test templates directly in the main SKILL.md (eliminating the need to consult a second document) or providing worked examples for each skill type rather than a classification table.

---

## Comparison to Baseline (baseline-w1.md)

The baseline result (produced without methodology skills) proposed a similar edit direction -- replacing the "don't test" section with per-type guidance. Key differences in this methodology-informed version:

1. **Root cause is classified, not just described.** "Single-model coverage with universal exclusion fallback" is a named pattern that can be detected in other guidance documents, not just this specific testing gap.
2. **Triage is explicit.** Three causes ranked by breadth, with reasoning for why rank 1 subsumes ranks 2 and 3.
3. **Edit is narrower.** Baseline proposed editing both the main SKILL.md and `testing-skills-with-subagents.md`. This version edits only the subagent doc (the source of the contradiction), leaving the main SKILL.md's already-correct per-type listing untouched. One change, one root cause.
4. **Re-experimentation plan follows experiment-set-design structure.** Three tasks with diversity checks, three phases (compliance/stress/quality), explicit pass criteria, baseline requirement. The baseline's validation plan had 4 steps but no phase structure, no stress testing, and no blind comparison.
5. **Convergence and abandonment criteria are defined.** The baseline had an anti-overfitting check but no stopping conditions or abandonment triggers.
6. **Anti-overfitting checklist is explicit.** Four checks with pass/fail, not a paragraph assertion.
