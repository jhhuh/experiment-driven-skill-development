# Phase 1 Audit: iterative-skill-refinement rubric

## Criteria Applied

From `skills/iterative-skill-refinement/SKILL.md`:

**Anti-Overfitting Checklist (lines 29-41):**

| Check | Pass | Fail |
|-------|------|------|
| Would this help on a completely different task? | Structural improvement | Overfitting |
| Does this add a general step/rule, not task-specific wording? | Genuine | Overfitting |
| After 2+ cycles on same tasks, did you add new tasks? | Fresh signal | Stale benchmark |
| After 2+ cycles with same judges, did you rotate personas? | Diverse signal | Judge-fitted |

**Convergence Criteria (lines 44-49):**
Stop when ALL hold:
1. Skill wins consistently across diverse tasks (not just original set)
2. New task sets reveal no new failure modes
3. Last 2 cycles produced no revisions
4. Gains per cycle are diminishing

**Red Flags (lines 63-71):**
- Proposing fixes without a re-experimentation plan
- Treating all failures with equal weight (no triage)
- Editing the skill without logging what triggered and validated it
- Same tasks for 3+ cycles without adding new ones
- Framing success as "fewer bugs" instead of "beats baseline"
- No convergence criteria defined before starting

---

## Findings: Development Process

| Check | Pass | Fail | Evidence |
|-------|------|------|----------|
| Overfitting safeguards | | **FAIL** | Each skill was tested on exactly 1 scenario (baseline + GREEN), never varied. No anti-overfitting checklist was applied to the skills themselves. |
| New tasks added after initial set | | **FAIL** | The integration test used a novel scenario (defensive-error-handling), but this was a composition test, not a per-skill validation on new tasks. Each skill was only ever tested on its single original scenario. |
| Different assessment methods | | **FAIL** | All tests used the same method: human-authored transcript analysis checking for behavioral compliance. No blind review, no automated checks, no persona rotation. |
| Convergence defined before stopping | | **FAIL** | No convergence criteria were stated anywhere in the test results or commit messages. The project went RED -> GREEN -> integration -> done. There is no document defining when development would stop. |
| REFACTOR phase completed | | **FAIL** | Git history shows: `e131873` (RED scenarios) -> `42c2c63` (RED baselines) -> `ddcb8f1`/`a7be76b`/`1cfcb2e` (GREEN skills) -> `d8d7e73` (GREEN tests) -> `c5ce6fb` (integration). No REFACTOR commit exists. The CLAUDE.md states the convention is "baseline (RED) -> write skill (GREEN) -> close loopholes (REFACTOR)" but only RED and GREEN were executed. |
| Revision log exists | | **FAIL** | No revision log file exists anywhere in the repository (`find` for `revision*` returns nothing). The iterative-skill-refinement SKILL.md has a revision log template (line 55-57) with only an example row, never filled in with actual data. No `artifacts/` directory exists. |
| "Beats baseline" framing | **PASS** | | GREEN test results for all 3 skills include explicit "Improvement Over Baseline" comparison tables. The integration test also frames skill effectiveness in terms of baseline comparison. |

---

## Findings: Per-Skill Convergence

The iterative-skill-refinement skill defines 4 convergence criteria. Did each methodology skill meet ALL 4 before shipping?

### blind-skill-assessment

| Criterion | Met? | Evidence |
|-----------|------|----------|
| 1. Wins consistently across diverse tasks | **NO** | Tested on exactly 1 task (compare two merge3.py versions). N=1 is not "diverse tasks." |
| 2. New task sets reveal no new failure modes | **NO** | No new tasks were ever tried. |
| 3. Last 2 cycles produced no revisions | **NO** | There was only 1 cycle (RED -> GREEN). No iteration occurred. |
| 4. Gains per cycle diminishing | **NO** | Cannot assess with only 1 cycle. |

**Convergence: 0/4 criteria met.**

### experiment-set-design

| Criterion | Met? | Evidence |
|-----------|------|----------|
| 1. Wins consistently across diverse tasks | **NO** | Tested on exactly 1 task (design test plan for HDD skill). |
| 2. New task sets reveal no new failure modes | **NO** | No new tasks were ever tried. |
| 3. Last 2 cycles produced no revisions | **NO** | Only 1 cycle. |
| 4. Gains per cycle diminishing | **NO** | Cannot assess with only 1 cycle. |

**Convergence: 0/4 criteria met.**

### iterative-skill-refinement

| Criterion | Met? | Evidence |
|-----------|------|----------|
| 1. Wins consistently across diverse tasks | **NO** | Tested on exactly 1 task (improve HDD skill given Phase 3 results). |
| 2. New task sets reveal no new failure modes | **NO** | No new tasks were ever tried. |
| 3. Last 2 cycles produced no revisions | **NO** | Only 1 cycle. |
| 4. Gains per cycle diminishing | **NO** | Cannot assess with only 1 cycle. |

**Convergence: 0/4 criteria met.**

---

## Summary

### The central finding

**The skills teach a REFACTOR loop but were never themselves put through one.** This is confirmed by the git history: the last substantive commit is the integration test (`c5ce6fb`), followed only by documentation (`0ce12f8`, `59cd4f9`, `d69bf2a`). The CLAUDE.md convention explicitly calls for "baseline (RED) -> write skill (GREEN) -> close loopholes (REFACTOR)" but only the first two phases were executed.

### Ranked findings (structural to cosmetic)

1. **No iteration occurred (STRUCTURAL).** Each skill had exactly 1 development cycle: write baseline scenario, observe gap, write skill, confirm skill closes gap. The iterative-skill-refinement skill prescribes a multi-cycle loop (lines 14-27) with "GOTO 2" at step 7. This loop was never executed for any of the three skills. The development process was strictly linear.

2. **N=1 test coverage per skill (STRUCTURAL).** Each skill was validated on a single scenario. The anti-overfitting checklist requires adding new tasks after 2+ cycles, but the deeper problem is that 1 scenario per skill provides no diversity signal at all. A skill that passes 1 test could be overfitted to that test by construction.

3. **No convergence criteria defined before starting (STRUCTURAL / RED FLAG).** The iterative-skill-refinement skill lists "no convergence criteria defined before starting" as an explicit red flag (line 70). No document in the repository defines when development of the methodology skills would be considered complete. The project simply stopped after GREEN tests passed.

4. **No revision log (STRUCTURAL / RED FLAG).** "Editing the skill without logging what triggered and validated it" is a red flag (line 67). The skills were written in a single pass and committed. No revision history beyond git commits exists. The revision log template in the skill was never populated with real data.

5. **No assessment method variation (MODERATE).** All testing used the same approach: human-authored scenario -> agent transcript -> human evaluation of compliance. The experiment-set-design skill teaches three-phase progression (Compliance -> Stress -> Quality) and assessment method variation, but neither was applied to the methodology skills.

6. **"Beats baseline" framing is present (COSMETIC PASS).** The GREEN results consistently compare against baseline behavior. This is the one criterion the development process followed well.

### What this means

The methodology skills are internally consistent and well-designed -- the GREEN tests demonstrate clear improvement over baseline behavior, and the integration test shows they compose correctly. But the development process violated 5 of the 6 red flags the iterative-skill-refinement skill defines. The skills were built in a single pass without the iterative refinement loop they prescribe.

This does not mean the skills are bad. It means they are **unvalidated beyond their initial scenario**. They may be robust, but the evidence to support that claim has not been gathered.
