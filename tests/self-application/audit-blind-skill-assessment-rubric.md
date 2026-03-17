# Phase 1 Audit: blind-skill-assessment rubric

## Criteria Applied

From blind-skill-assessment SKILL.md:

**Anti-Bias Rules (lines 37-43):**
1. Judges never see origin labels ("baseline", "skill-version", "v1", "v2")
2. Randomize which version is A -- do not always assign first-listed as A
3. Score each dimension independently -- no holistic verdicts
4. Sanitize identifying information (file paths, naming conventions that leak origin)

**Red Flags (lines 87-95):**
1. Evaluating without randomizing labels
2. Reading origin labels before scoring is complete
3. Giving only a holistic verdict with no per-dimension scores
4. Skipping the upfront rubric
5. Using a single perspective instead of multiple personas

---

## Findings: experiment-set-design

| # | Finding | Severity | Lines | Category |
|---|---------|----------|-------|----------|
| 1 | Phase 3 declares "Skill must win on aggregate" -- assumes desired outcome | structural | 50 | bias |
| 2 | HDD example shows "Skill won 4/5" using origin label in results | structural | 75 | anti-bias rule violation |
| 3 | Phase 3 flow lacks explicit sanitization/randomization step | structural | 50 | red-flag (omission) |
| 4 | "baseline vs skill" phrasing throughout privileges the skill as the treatment | cosmetic | 15, 50, 75 | bias |

### Detail

**Finding 1 (line 50):** "Skill must win on aggregate." This is the most consequential problem. The skill's own Phase 3 description pre-judges the outcome: a valid experiment should determine which version is better, not require a predetermined winner. If the baseline genuinely produces better output for a given task, the correct conclusion is "this skill is not helpful here," not "the skill failed." By framing baseline-wins as failure, the methodology pressures users to keep editing the skill until it beats baseline, even when baseline superiority is the honest result. This directly violates the anti-bias principle of scoring each dimension independently without holistic verdicts that presuppose a winner.

**Finding 2 (line 75):** The HDD example table shows "Skill won 4/5" in the Result column. This uses the origin label ("Skill") in the result presentation. Under blind-skill-assessment rules, results should be reported as "A won 4/5" until the decode step, then translated. Showing "Skill won" in a teaching example normalizes seeing origin labels in results, which is exactly what the anti-bias rules forbid. A reader learning from this example would naturally replicate the pattern.

**Finding 3 (line 50):** Phase 3 says "Run with and without skill. Blind-assess using blind-skill-assessment." This correctly references the blind assessment skill, but omits the critical steps: randomize label assignment, sanitize identifying information, define rubric before reading code. A user following experiment-set-design alone (without also reading blind-skill-assessment) would know to "blind-assess" but not how to avoid the specific biases. Since the skill positions itself as self-contained ("use when designing experiments"), this omission is a gap.

**Finding 4 (lines 15, 50, 75):** The consistent framing of "baseline vs skill" (always baseline first, skill second) establishes a linguistic pattern where baseline is the control and skill is the treatment expected to be better. This is a subtle framing bias. Neutral phrasing would be "version A vs version B" or at minimum alternate the ordering.

---

## Findings: iterative-skill-refinement

| # | Finding | Severity | Lines | Category |
|---|---------|----------|-------|----------|
| 1 | "Skill wins consistently" as convergence criterion assumes skill should win | structural | 17, 46 | bias |
| 2 | HDD VERIFY example presents improvement as a narrative arc with foregone conclusion | structural | 59-61 | bias |
| 3 | Same agent edits AND assesses -- no separation of concerns for bias | structural | 14-27 | red-flag (single perspective) |
| 4 | No convergence path for "baseline is genuinely better" | structural | 42-49 | bias |
| 5 | "baseline won 4/5" uses origin labels in example assessment results | cosmetic | 61 | anti-bias rule violation |

### Detail

**Finding 1 (lines 17, 46):** The improvement loop (line 17) checks "Skill wins consistently? -> DONE" and convergence criterion #1 (line 46) says "Skill wins consistently across diverse tasks." This encodes the assumption that the skill should eventually win. There is no branch for "baseline wins consistently -> the skill is harmful, remove or fundamentally rethink it." The methodology only models the happy path where iteration leads to skill superiority. This is confirmation bias baked into the process structure.

**Finding 2 (lines 59-61):** The HDD VERIFY example reads as a success story: "baseline won 4/5 -> found root cause -> added VERIFY step -> HDD won 5/5 -> New tasks confirmed -> Converged." This narrative arc presents improvement as inevitable -- diagnose, fix, win. There is no example of a cycle that failed, a fix that made things worse, or a conclusion that the skill approach was wrong. As a teaching example, it trains users to expect and seek this arc, which is a form of confirmation bias. A balanced example set would include at least one case where the fix did not help or where the conclusion was to revert.

**Finding 3 (lines 14-27):** The improvement loop has the same agent (or person) performing steps 1-7: designing experiments, assessing results, diagnosing failures, editing the skill, and re-assessing. This violates the spirit of the blind-skill-assessment red flag "Using a single perspective instead of multiple personas." While step 2 references blind-skill-assessment (which uses multiple judge personas), the overall loop has a single actor who knows which version is the skill throughout the entire process. They perform the edit at step 5, then assess at step 2 -- they cannot be blind to which version they just edited. The loop structure makes true blinding impossible without an external assessor.

**Finding 4 (lines 42-49):** The convergence criteria are exclusively about the skill winning:
- "Skill wins consistently" (line 46)
- "No new failure modes" (line 47)
- "No revisions needed" (line 48)
- "Diminishing gains" (line 49)

There is no criterion for "baseline wins consistently -> stop iterating, the skill is counterproductive." The only modeled outcome is eventual skill victory or indefinite iteration. A rigorous methodology needs an explicit exit ramp for "this skill does not help."

**Finding 5 (line 61):** "baseline won 4/5" and "HDD won 5/5" both use origin labels in the assessment results. Per blind-skill-assessment anti-bias rules, results should use blinded labels (A/B) until decode. The example normalizes reporting results with origin labels visible, undermining the very skill it references.

---

## Summary

Ranked by breadth of impact:

1. **No exit ramp for "baseline is better"** (experiment-set-design line 50, iterative-skill-refinement lines 17, 46, 42-49). Both skills assume the skill-version should eventually win. This is the widest-impact bias: it contaminates the entire methodology by framing every experiment as "how do we make the skill win" rather than "which version is better." A user following these skills has no sanctioned path to conclude that a skill is harmful and should be abandoned.

2. **Same actor edits and assesses** (iterative-skill-refinement lines 14-27). The improvement loop makes true blinding structurally impossible. The person who edited the skill at step 5 cannot be blind to which version is theirs at step 2. This is the "single perspective" red flag from blind-skill-assessment, elevated to a process-level concern.

3. **Origin labels in teaching examples** (experiment-set-design line 75, iterative-skill-refinement line 61). Both examples show results using origin labels ("Skill won 4/5", "baseline won 4/5") rather than blinded labels. Since these are the examples users will imitate, they normalize exactly the behavior that blind-skill-assessment flags as an anti-bias violation.

4. **Phase 3 omits operational blind-assessment steps** (experiment-set-design line 50). The reference to blind-skill-assessment is present but the critical operational steps (randomize, sanitize, rubric-first) are not surfaced, leaving a gap for users who treat experiment-set-design as self-sufficient.

5. **Teaching examples show only success narratives** (iterative-skill-refinement lines 59-61). The sole example is a clean diagnose-fix-win arc. No failed iterations, no reversions, no "skill was wrong" conclusions. This trains users toward confirmation bias in their own improvement cycles.
