# Self-Application Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Apply the experiment-driven skill improvement methodology to its own three skills, validating that the methodology produces better skill improvements than unguided effort.

**Architecture:** Three phases — internal audit (rubric-based self-check), pilot experiments (baseline vs methodology-guided improvement of two external skills), self-edit (iterative refinement loop on the methodology skills themselves).

**Tech Stack:** Markdown skills, subagent experiments, blind assessment rubrics. No code — all outputs are structured documents.

---

## Session 1: Phase 1 — Internal Audit

### Task 1: Audit using blind-skill-assessment rubric

**Files:**
- Read: `skills/blind-skill-assessment/SKILL.md` (the rubric source)
- Read: `skills/experiment-set-design/SKILL.md` (audit target)
- Read: `skills/iterative-skill-refinement/SKILL.md` (audit target)
- Create: `tests/self-application/phase1-audit.md`

**Step 1: Extract audit criteria from blind-skill-assessment**

Pull the anti-bias rules and red flags from `blind-skill-assessment/SKILL.md`:
- Anti-Bias Rules (lines 37-43): label randomization, independent dimension scoring, sanitize identifying info
- Red Flags (lines 87-95): evaluating without randomizing, holistic verdicts, skipping rubric, single perspective

**Step 2: Audit experiment-set-design against these criteria**

Check: Do the examples and processes in `experiment-set-design` exhibit bias toward specific outcomes? Does the HDD example (lines 67-76) present results neutrally or with embedded judgment? Does the skill's own Phase 3 description (lines 46-52) correctly reference blind assessment or shortcut it?

**Step 3: Audit iterative-skill-refinement against these criteria**

Check: Does the HDD VERIFY example (lines 59-61) present the improvement as a foregone conclusion? Does the improvement loop (lines 14-27) risk confirmation bias by having the same person who edits the skill also assess it? Does the convergence criteria (lines 44-49) include safeguards against declaring victory prematurely?

**Step 4: Record findings in audit document**

Write findings to `tests/self-application/phase1-audit.md` with structure:

```markdown
# Phase 1: Internal Audit — blind-skill-assessment rubric

## Criteria Applied
[list from Step 1]

## Findings: experiment-set-design
| Finding | Severity | Lines | Category |
|---------|----------|-------|----------|
| ... | structural/cosmetic | ... | bias/red-flag |

## Findings: iterative-skill-refinement
[same table]

## Summary
[ranked by breadth of impact]
```

### Task 2: Audit using experiment-set-design rubric

**Files:**
- Read: `skills/experiment-set-design/SKILL.md` (the rubric source)
- Read: `tests/blind-skill-assessment/scenario-1-compare-versions.md`
- Read: `tests/experiment-set-design/scenario-1-design-test-set.md`
- Read: `tests/iterative-skill-refinement/scenario-1-improve-skill.md`
- Read: `tests/integration/scenario-1-full-loop.md`
- Modify: `tests/self-application/phase1-audit.md` (append)

**Step 1: Extract audit criteria from experiment-set-design**

Pull the task selection checklist (lines 18-24) and red flags (lines 77-85):
- Task selection: different behavior, complex enough, multiple valid approaches, different domains
- Red flags: no baseline, same difficulty, jumping phases, same tasks for 3+ cycles, no version tracking

**Step 2: Audit existing test scenarios against these criteria**

For each of the 4 test scenarios, check:
1. Does the task satisfy all 4 selection criteria?
2. Are all tasks at the same difficulty or domain?
3. Is there baseline comparison?
4. Is there version tracking?

**Step 3: Append findings to audit document**

Append to `tests/self-application/phase1-audit.md`:

```markdown
# Phase 1: Internal Audit — experiment-set-design rubric

## Criteria Applied
[list from Step 1]

## Findings: Test Scenario Quality
| Scenario | Criterion | PASS/FAIL | Notes |
|----------|-----------|-----------|-------|
| blind-skill-assessment/scenario-1 | Different behavior | ... | ... |
| ... | ... | ... | ... |

## Red Flag Check
| Red Flag | Present? | Evidence |
|----------|----------|----------|
| No baseline comparison | ... | ... |
| ... | ... | ... |

## Summary
[ranked findings]
```

### Task 3: Audit using iterative-skill-refinement rubric

**Files:**
- Read: `skills/iterative-skill-refinement/SKILL.md` (the rubric source)
- Read: git log for skill development history
- Modify: `tests/self-application/phase1-audit.md` (append)

**Step 1: Extract audit criteria from iterative-skill-refinement**

Pull the anti-overfitting checklist (lines 29-41) and red flags (lines 63-71):
- Anti-overfitting: generalizes beyond task, adds general step, new tasks after 2 cycles, rotate judges
- Red flags: fixes without re-experimentation, no triage, undocumented edits, 3+ cycles same tasks, "fewer bugs" framing, no convergence criteria

**Step 2: Check the skills' own development history**

Review git log to check:
- Were the skills developed with overfitting safeguards?
- Were new test scenarios added after the GREEN phase?
- Were different judge personas used across tests?
- Was there a convergence definition before the project was declared "done"?

**Step 3: Check each skill for convergence criteria**

Do the skills themselves define when to stop improving? Or is the development just "RED passed, GREEN passed, ship it"?

**Step 4: Append findings to audit document**

Append to `tests/self-application/phase1-audit.md`:

```markdown
# Phase 1: Internal Audit — iterative-skill-refinement rubric

## Criteria Applied
[list from Step 1]

## Findings: Development Process
| Check | Pass | Fail | Evidence |
|-------|------|------|----------|
| Overfitting safeguards during dev | ... | ... | ... |
| ... | ... | ... | ... |

## Findings: Convergence Gaps
[per skill]

## Summary
[ranked findings]
```

### Task 4: Compile gap report and rank findings

**Files:**
- Read: `tests/self-application/phase1-audit.md`
- Create: `tests/self-application/phase1-gap-report.md`

**Step 1: Compile all findings across three audits**

Merge findings from all three audit sections. Deduplicate — some gaps will surface from multiple rubrics.

**Step 2: Rank by breadth of impact**

Score each finding:
- Affects all 3 skills → high breadth
- Affects 2 skills → medium
- Affects 1 skill → low

Also score by type:
- Structural (missing process step, logical gap) → fix now
- Cosmetic (wording, formatting) → skip

**Step 3: Write gap report**

```markdown
# Phase 1 Gap Report

## Ranked Findings (structural only)

| Rank | Finding | Breadth | Skills Affected | Triggered By |
|------|---------|---------|-----------------|--------------|
| 1 | ... | high | all 3 | [audit rubric] |
| ... | ... | ... | ... | ... |

## Deferred (cosmetic)
[list]

## Recommended Edits
[one edit per structural finding, with rationale]
```

### Task 5: Apply structural edits

**Files:**
- Modify: `skills/blind-skill-assessment/SKILL.md` (if gaps found)
- Modify: `skills/experiment-set-design/SKILL.md` (if gaps found)
- Modify: `skills/iterative-skill-refinement/SKILL.md` (if gaps found)

**Step 1: For each structural finding, make one targeted edit**

Follow iterative-skill-refinement's own rule: one change per root cause. Document what triggered each edit.

**Step 2: Verify edits don't break existing GREEN tests**

Re-read each skill after editing. Confirm the changes don't contradict the behavior that passed in GREEN tests (documented in `tests/*/results/green.md`).

**Step 3: Commit with revision log**

```bash
git add tests/self-application/ skills/
git commit -m "refactor: Phase 1 audit — fix structural gaps found by self-application

Gap report: tests/self-application/phase1-gap-report.md
[list edits and what triggered each]"
```

---

## Session 2-3: Phase 2 — Pilot Experiments

### Task 6: Design experiment scenarios for systematic-debugging pilot

**Files:**
- Read: `~/.claude/plugins/cache/claude-plugins-official/superpowers/4.3.1/skills/systematic-debugging/SKILL.md` (pilot skill)
- Create: `tests/self-application/pilot-systematic-debugging/weakness-analysis.md`
- Create: `tests/self-application/pilot-systematic-debugging/scenario-1-improve-discipline.md`
- Create: `tests/self-application/pilot-systematic-debugging/scenario-2-improve-coverage.md`

**Step 1: Analyze systematic-debugging for real weaknesses**

Read the pilot skill and identify 2 genuine weaknesses that could be improved. These must be real — not fabricated for testing. Look for:
- Missing edge cases the skill doesn't cover
- Rules that could be clearer or more actionable
- Gaps between what the skill teaches and what real debugging requires

Document in `weakness-analysis.md`.

**Step 2: Design Scenario 1 — Discipline Improvement**

Write a scenario prompt that gives the agent:
- The systematic-debugging skill text
- A specific weakness related to discipline enforcement
- Test failure data showing the weakness matters
- Task: "Improve this skill to address this weakness"

Two versions of this prompt:
- **Baseline version:** Just the task and data, no methodology skills
- **Methodology version:** Same task and data, plus all three methodology skills loaded

**Step 3: Design Scenario 2 — Coverage Improvement**

Same structure, different weakness (coverage gap rather than discipline).

**Step 4: Commit scenarios**

```bash
git add tests/self-application/pilot-systematic-debugging/
git commit -m "test: design pilot experiment scenarios for systematic-debugging"
```

### Task 7: Design experiment scenarios for writing-skills pilot

**Files:**
- Read: `~/.claude/plugins/cache/claude-plugins-official/superpowers/4.3.1/skills/writing-skills/SKILL.md` (pilot skill)
- Create: `tests/self-application/pilot-writing-skills/weakness-analysis.md`
- Create: `tests/self-application/pilot-writing-skills/scenario-1-improve-testing-guidance.md`
- Create: `tests/self-application/pilot-writing-skills/scenario-2-improve-cso.md`

**Step 1: Analyze writing-skills for real weaknesses**

Same approach as Task 6. This skill is 655 lines — look for sections that are verbose, unclear, or missing important guidance.

**Step 2: Design Scenario 1 — Testing Guidance Improvement**

Focus on a weakness in how the skill teaches testing (e.g., the "Testing All Skill Types" section).

**Step 3: Design Scenario 2 — CSO Improvement**

Focus on a weakness in the Claude Search Optimization section.

**Step 4: Commit scenarios**

```bash
git add tests/self-application/pilot-writing-skills/
git commit -m "test: design pilot experiment scenarios for writing-skills"
```

### Task 8: Run baseline experiments (no methodology)

**Files:**
- Create: `tests/self-application/pilot-systematic-debugging/results/baseline-s1.md`
- Create: `tests/self-application/pilot-systematic-debugging/results/baseline-s2.md`
- Create: `tests/self-application/pilot-writing-skills/results/baseline-w1.md`
- Create: `tests/self-application/pilot-writing-skills/results/baseline-w2.md`

**Step 1: Run each baseline scenario as a subagent**

For each of the 4 scenarios, launch a subagent with:
- The baseline version of the prompt (no methodology skills)
- Record the full output

**Step 2: Save baseline outputs**

Save each subagent's output verbatim in the results file. Include:
- The exact prompt given
- The full response
- Timestamp and model info

**Step 3: Commit baselines**

```bash
git add tests/self-application/pilot-*/results/baseline-*.md
git commit -m "test: baseline experiment results — improvement without methodology"
```

### Task 9: Run methodology-guided experiments

**Files:**
- Create: `tests/self-application/pilot-systematic-debugging/results/methodology-s1.md`
- Create: `tests/self-application/pilot-systematic-debugging/results/methodology-s2.md`
- Create: `tests/self-application/pilot-writing-skills/results/methodology-w1.md`
- Create: `tests/self-application/pilot-writing-skills/results/methodology-w2.md`

**Step 1: Run each methodology scenario as a subagent**

For each of the 4 scenarios, launch a subagent with:
- The methodology version of the prompt (all three skills loaded)
- Record the full output

**Step 2: Save methodology outputs**

Same format as baselines.

**Step 3: Commit methodology results**

```bash
git add tests/self-application/pilot-*/results/methodology-*.md
git commit -m "test: methodology-guided experiment results"
```

### Task 10: Blind assess all 4 experiment pairs

**Files:**
- Create: `tests/self-application/assessment/experiment-s1.md`
- Create: `tests/self-application/assessment/experiment-s2.md`
- Create: `tests/self-application/assessment/experiment-w1.md`
- Create: `tests/self-application/assessment/experiment-w2.md`
- Create: `tests/self-application/assessment/aggregate.md`

**Step 1: For each experiment pair, run blind assessment**

Follow `blind-skill-assessment` exactly:
1. Randomly assign A/B labels (coin flip per experiment)
2. State rubric with custom personas (Methodologist, Skill Architect, Pragmatist) before reading
3. Three personas score both versions (1-5 per dimension)
4. Decode labels after all scoring complete
5. Aggregate dimension wins

**Step 2: Aggregate across all 4 experiments**

Write `aggregate.md` with:
- Per-experiment winners
- Per-persona averages across experiments
- Per-pilot summary (systematic-debugging vs writing-skills)
- Overall: methodology won N/4 experiments

**Step 3: Commit assessments**

```bash
git add tests/self-application/assessment/
git commit -m "test: blind assessment of baseline vs methodology — N/4 methodology wins"
```

---

## Session 3-4: Phase 3 — Self-Edit

### Task 11: Diagnose methodology failures

**Files:**
- Read: `tests/self-application/assessment/aggregate.md`
- Read: `tests/self-application/phase1-gap-report.md`
- Create: `tests/self-application/phase3-diagnosis.md`

**Step 1: Identify where methodology lost or underperformed**

For each experiment where baseline won or tied:
- Which personas scored baseline higher?
- Which dimensions specifically?
- What did the baseline do that the methodology-guided version didn't?

**Step 2: Correlate with Phase 1 gaps**

Did Phase 1 structural gaps predict Phase 2 failures? If yes, the audit was useful but the fixes insufficient. If no, there are gaps the audit missed.

**Step 3: Root cause analysis**

For each failure dimension, ask: "What class of problems does this represent?" Don't fix symptoms.

**Step 4: Write diagnosis**

```markdown
# Phase 3 Diagnosis

## Experiments Where Methodology Lost
| Exp | Persona | Dimension | Baseline Score | Methodology Score | Root Cause |
|-----|---------|-----------|----------------|-------------------|------------|

## Root Cause Classes
| Class | Breadth | Experiments Affected |
|-------|---------|---------------------|

## Phase 1 Correlation
[did audit predict failures?]
```

### Task 12: Triage and edit methodology skills

**Files:**
- Read: `tests/self-application/phase3-diagnosis.md`
- Modify: `skills/*/SKILL.md` (targeted edits)
- Create: `tests/self-application/revision-log.md`

**Step 1: Rank root causes by breadth**

Widest-impact cause first. A cause affecting both pilots > one pilot.

**Step 2: One targeted edit per root cause**

Follow iterative-skill-refinement: one edit, one cause. Log each edit.

**Step 3: Anti-overfitting check per edit**

Complete the 4-check table:
- Would this help on a completely different pilot skill?
- Does this add a general step, not pilot-specific wording?
- (If cycle 2+) Did you add new experiments?
- (If cycle 2+) Did you rotate personas?

**Step 4: Write revision log and commit**

```markdown
# Revision Log — Self-Application

| Date | What Changed | Triggered By | Validated By |
|------|-------------|-------------|-------------|
| ... | ... | Exp S1: Pragmatist scored ... | Pending re-run |
```

```bash
git add skills/ tests/self-application/revision-log.md
git commit -m "refactor: edit methodology skills based on Phase 3 diagnosis

[list edits with triggers]"
```

### Task 13: Re-run experiments and check convergence

**Files:**
- Create: `tests/self-application/assessment/rerun-*.md`
- Modify: `tests/self-application/revision-log.md` (fill "Validated by")
- Create: `tests/self-application/convergence.md`

**Step 1: Pick 1-2 experiments to re-run**

Choose experiments where methodology lost. Re-run both baseline and methodology conditions with edited skills.

**Step 2: Blind assess re-run results**

Same process as Task 10.

**Step 3: Check convergence criteria**

- Methodology wins consistently? (across original + re-run)
- Phase 1 re-audit shows no new structural gaps?
- Last cycle produced no revisions?
- If NO on any: plan next cycle (add third pilot if 2+ cycles done)

**Step 4: Write convergence document**

```markdown
# Convergence Assessment

| Criterion | Met? | Evidence |
|-----------|------|----------|
| Methodology wins consistently | ... | N/M experiments |
| No new structural gaps | ... | re-audit result |
| Last cycle: no revisions | ... | revision log |
| Diminishing gains | ... | delta between cycles |

## Decision
[CONVERGED / NEXT CYCLE NEEDED — with plan]
```

**Step 5: Final commit**

```bash
git add tests/self-application/
git commit -m "test: convergence assessment — [CONVERGED/CONTINUING]"
```

---

## Anti-Overfitting Safeguard

If Task 13 triggers a second cycle:
- Add `brainstorming` skill as third pilot (short, flexible-type, different from systematic-debugging and writing-skills)
- Design 1 new experiment scenario for it
- Rotate personas: replace Pragmatist with a domain-specific persona (e.g., "Workflow Designer" focused on process flow efficiency)
