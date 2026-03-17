# Phase 1 Audit: experiment-set-design rubric

Applied the experiment-set-design skill's own criteria (SKILL.md lines 18-85) as a rubric against every test scenario in this project.

## Criteria Applied

**Task Selection Checklist (4 criteria per task):**
1. Skill's guidance should produce different behavior on this task
2. Complex enough that architecture decisions matter (not a one-liner)
3. Multiple valid approaches exist (skill's approach is distinguishable)
4. Covers a different domain or language than other tasks in the set

**Red Flags (5 checks):**
1. Designing experiments with no baseline comparison
2. All tasks at the same difficulty or in the same domain
3. Jumping to Phase 3 without passing Phase 1
4. Same task set for 3+ improvement cycles without adding tasks
5. No record of which skill version produced which results

**Three-Phase Progression:** Phase 1 (Compliance) -> Phase 2 (Stress) -> Phase 3 (Quality)

**Scenario Count Target:** 3-5 tasks per phase

## Findings: Test Scenario Quality

### blind-skill-assessment/scenario-1-compare-versions

| Criterion | PASS/FAIL | Notes |
|-----------|-----------|-------|
| Different behavior | PASS | Baseline produced single-perspective holistic verdict; skill produced randomized labels, 3-persona scoring, per-dimension aggregation. Clear behavioral difference demonstrated. |
| Complex enough | PASS | Evaluating a 460-line three-way merge algorithm requires genuine architectural judgment (algorithm decomposition, correctness analysis, design tradeoffs). |
| Multiple valid approaches | PASS | Agent could assess holistically, per-function, by test case, by persona -- skill's approach (blind + multi-persona + scored rubric) is distinguishable from all of these. |
| Different domain/language | N/A | Only 1 scenario exists, so domain diversity cannot be evaluated within this skill's test set. This is itself a problem (see Scenario Count Analysis). |

### experiment-set-design/scenario-1-design-test-set

| Criterion | PASS/FAIL | Notes |
|-----------|-----------|-------|
| Different behavior | PASS | Baseline produced a fixed 12-test matrix with no phase progression, no anti-overfitting, no revision tracking. Skill produced phased plan with all of these. Clear difference. |
| Complex enough | PASS | Designing a test plan for a skill requires multi-level reasoning (what to test, how to measure, how to iterate). Architecture decisions matter. |
| Multiple valid approaches | PASS | Could design ad-hoc tests, statistical experiments, A/B tests, user studies. Skill's approach (3-phase + baseline + diversity) is distinguishable. |
| Different domain/language | N/A | Only 1 scenario. Same note as above. |

### iterative-skill-refinement/scenario-1-improve-skill

| Criterion | PASS/FAIL | Notes |
|-----------|-----------|-------|
| Different behavior | PASS | Baseline jumped to fixes with no loop, no re-experimentation, no overfitting check, no convergence criteria. Skill produced 7-step structured loop with all of these. Strongest behavioral delta of all scenarios. |
| Complex enough | PASS | Diagnosing cross-cutting bug patterns, triaging by breadth, planning re-experimentation, and defining convergence criteria is inherently multi-step. |
| Multiple valid approaches | PASS | Could fix bugs one-by-one, rewrite the skill, add testing, add reviews. Skill's approach (diagnose root cause class -> triage by breadth -> targeted edit -> re-run -> anti-overfit check) is distinguishable. |
| Different domain/language | N/A | Only 1 scenario. |

### integration/scenario-1-full-loop

| Criterion | PASS/FAIL | Notes |
|-----------|-----------|-------|
| Different behavior | PASS | Tests whether all three skills compose. Different from individual skill scenarios. |
| Complex enough | PASS | Requires coordinating three skills on a novel problem (defensive-error-handling). |
| Multiple valid approaches | PASS | Could apply skills sequentially, selectively, or in different orderings. |
| Different domain/language | PASS | Uses a different target skill (defensive-error-handling) than the unit scenarios (which all use hole-driven-development). |

## Red Flag Check

| Red Flag | Present? | Evidence |
|----------|----------|----------|
| No baseline comparison | **NO** (good) | All three unit skills have baseline.md and green.md results. Baselines were run first, gaps identified, then skill tested. The integration test is the one exception -- it has no baseline run (results.md only, no results/baseline.md). |
| All same difficulty/domain | **YES** | All three unit scenarios operate at the same difficulty level: a single well-specified prompt with clear instructions, no time pressure, no competing instructions, no ambiguity. There is no Phase 2 (stress) scenario in any skill's test set. The domain is also narrow: all scenarios involve skill-development-for-LLM-coding-agents. There is no cross-domain variation (e.g., testing these meta-skills on a data analysis skill vs. a UI skill vs. an infrastructure skill). |
| Jumped phases | **YES** | The project performed Phase 1 (compliance) testing only. No Phase 2 (stress) testing was done for any skill. Yet the project has already shipped and been documented as complete. The three-phase progression was not followed -- Phase 2 and Phase 3 were skipped entirely. Note: the integration test is not a substitute for Phase 2/3; it tests composition, not stress or quality. |
| Same tasks for 3+ cycles | **NO** (good) | Only 1 cycle has been run (baseline -> GREEN). This red flag has not triggered yet, but it will if improvement cycles are run without adding tasks. |
| No version tracking | **PARTIAL** | Results files do not record: git hash of the skill version tested, model used, timestamp of the run, or temperature setting. The results reference "baseline" and "GREEN" but there is no way to reproduce the exact conditions. The git log does show commits (e.g., d8d7e73 "test: GREEN tests pass for all three skills"), which provides implicit version tracking, but the results files themselves lack this metadata. |

## Phase Progression Analysis

The project **did not follow the three-phase progression** prescribed by the experiment-set-design skill.

**What was done:**
- Phase 1 (Compliance): Baseline run, then GREEN run checking rule-by-rule compliance. All three skills passed. This is a valid Phase 1.

**What was skipped:**
- Phase 2 (Stress): No scenario includes competing instructions, time pressure, conflicting edge cases, or partial solutions. The skill says "Same checks + competing instructions, time pressure, conflicting edge cases." None of this was tested.
- Phase 3 (Quality): No blind comparison of skill-guided output vs. baseline output on the same task. The blind-skill-assessment scenario tests whether the agent *follows the blind assessment process*, but it does not test whether the *output quality* of any skill-guided work is better than baseline. Phase 3 requires running the blind-skill-assessment skill as a judge on paired outputs.

**The phase-skipping is the most significant finding.** The experiment-set-design skill explicitly says: "All PASS before advancing. Any FAIL: improve skill, re-run current phase." The project correctly stayed in Phase 1 until all passed, but then stopped instead of advancing to Phase 2. The skill was declared complete (committed, documented, MkDocs site deployed) without ever running stress or quality tests.

This is exactly the pattern the skill warns against: "Jumping to Phase 3 without passing Phase 1." The project did the inverse -- it stopped at Phase 1 without ever attempting Phase 2 or 3.

## Scenario Count Analysis

The experiment-set-design skill says: **"Aim for 3-5 tasks per phase."**

Actual count per skill:

| Skill | Scenarios | Target | Gap |
|-------|-----------|--------|-----|
| blind-skill-assessment | 1 | 3-5 | Missing 2-4 scenarios |
| experiment-set-design | 1 | 3-5 | Missing 2-4 scenarios |
| iterative-skill-refinement | 1 | 3-5 | Missing 2-4 scenarios |
| integration | 1 | 3-5 | Missing 2-4 scenarios |

**Every skill has only 1 scenario.** This is the minimum possible coverage. With 1 scenario per skill, there is no way to verify that the skill works across different domains, complexities, or languages. A skill could pass its single scenario by coincidence or because the scenario happens to align with the model's default behavior.

The experiment-set-design skill's own example table shows 5 tasks across Python, Haskell, and Go. The project's actual tests use 0 languages (all scenarios are natural-language meta-tasks, not coding tasks) and 1 domain (skill development).

This is a serious gap. The Task Selection Checklist criterion "Covers a different domain or language than other tasks in the set" is untestable with only 1 scenario per skill.

## Summary

Findings ranked by severity:

1. **Phase progression not followed (HIGH).** Only Phase 1 was executed. Phase 2 (stress) and Phase 3 (quality) were never run. The project was declared complete after Phase 1 compliance tests. This directly violates the skill's three-phase model and its advancement rule.

2. **Only 1 scenario per skill (HIGH).** The skill prescribes 3-5 tasks per phase. Every skill has exactly 1. Domain diversity, complexity variation, and language coverage are all untested. With N=1, any single scenario could be an outlier.

3. **No version tracking in results (MEDIUM).** Results files lack git hash, model name, temperature, and timestamp. The skill says "For each run, record: skill version (git hash), tasks run, phase, results, what changed since last run." This metadata is absent from all results files.

4. **All scenarios at same difficulty/domain (MEDIUM).** Every scenario is a well-structured, unambiguous prompt about LLM skill development. No scenario tests a different domain (e.g., applying these meta-skills to a web development skill, a data science skill, or an infrastructure skill). No scenario introduces competing instructions or time pressure.

5. **Integration test has no baseline (LOW).** The integration scenario has results.md but no baseline.md. The skill says "Every experiment needs a baseline -- same prompt, same task, no skill loaded." The integration test violates this. However, this is less severe because the individual skill baselines partially cover the gap.

**Meta-observation:** The experiment-set-design skill is the most prescriptive of the three skills about experimental rigor. The project that developed it violates its own prescriptions at multiple points. This is not unusual -- the skill was developed through TDD (write baseline, identify gaps, write skill to fill gaps), and the TDD cycle stopped after the GREEN phase. The skill's own guidance about what comes after GREEN (Phase 2 stress testing, Phase 3 quality comparison, task set expansion) was never applied to itself.
