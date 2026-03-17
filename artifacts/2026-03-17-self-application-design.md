# Design: Self-Application of Experiment-Driven Skill Development

## Goal

Apply the experiment-driven skill improvement methodology to its own three skills (`blind-skill-assessment`, `experiment-set-design`, `iterative-skill-refinement`), validating that the methodology produces better skill improvements than unguided effort.

## Approach

Approach B: "Audit First, Then Pilot" — internal audit catches cheap structural gaps, then two contrasting pilot skills validate the methodology end-to-end.

## Phase 1 — Internal Audit (session 1)

Use each skill's own quality criteria as a rubric to audit all three skills.

### Cross-audit matrix

| Auditor Rubric | Audits |
|---|---|
| `blind-skill-assessment`'s anti-bias rules & red flags | Are the other two skills' examples/processes free of bias toward specific outcomes? |
| `experiment-set-design`'s task selection checklist & red flags | Do the skills' own test scenarios (in `tests/`) satisfy the checklist? Same domain? Same difficulty? Missing baselines? |
| `iterative-skill-refinement`'s anti-overfitting checklist & convergence criteria | Were the skills themselves developed with overfitting safeguards? Do they define when to stop? |

### Output

Gap report listing structural issues, ranked by breadth of impact. Fix structural issues (not cosmetic). No edits without documenting what triggered them.

## Phase 2 — Pilot Experiments (sessions 2-3)

### Pilot skills

| Pilot | Type | Why chosen |
|---|---|---|
| `systematic-debugging` | Discipline-enforcing | Tests whether methodology improves rule-enforcing skills. 296 lines, well-structured, room for improvement. |
| `writing-skills` | Technique/process hybrid | The meta-skill that built these three. 655 lines, complex. Tests whether methodology handles large, multi-concern skills. |

### Experiment design (per pilot)

1. **Task:** "Improve this pilot skill." Provide pilot skill text and known weaknesses or synthetic test failures.

2. **Two conditions:**
   - **Baseline:** Agent improves the pilot with no methodology guidance
   - **With-methodology:** Agent improves the pilot using all three methodology skills

3. **Assessment:** Blind-assess both outputs using `blind-skill-assessment`. Judge both the *process* and the *result*.

4. **Custom personas:**

| Persona | Focus | Dimensions |
|---|---|---|
| Methodologist | Process rigor | Root cause diagnosis, revision tracking, anti-overfitting, convergence criteria |
| Skill Architect | Output quality | Structural clarity, coverage of failure modes, actionability |
| Pragmatist | Efficiency | Time to useful improvement, unnecessary complexity, over-engineering |

5. **Experiments:** 2 per pilot (different weaknesses), 4 total.

## Phase 3 — Self-Edit (sessions 3-4)

### Process

Follow `iterative-skill-refinement` literally on the methodology skills themselves:

1. **DIAGNOSE** — Where did the methodology fail to help or actively hinder? Did baseline beat methodology on any dimension?
2. **TRIAGE** — Rank by breadth (gap hurting both pilots > one pilot).
3. **EDIT** — One targeted change per root cause. Log in revision table.
4. **RE-RUN** — Re-run 1 experiment from Phase 2 with edited skills. After 2+ cycles on same experiments, add new experiments.
5. **CONVERGE** — Stop when methodology wins consistently, re-audit shows no new gaps, and last cycle produced no revisions.

### Anti-overfitting safeguard

After 2+ edit cycles, add a third pilot skill (`brainstorming` — short, flexible-type, very different from the other two) to prevent tuning the methodology to only work on discipline/technique skills.

### Deliverables

- Revised skill files committed with revision log entries
- Gap report and assessment results in `tests/`
- Convergence evidence documented

## Decisions

- **Two pilots, not one:** Diversity across skill types (discipline vs technique) prevents overfitting conclusions.
- **Internal audit first:** Nearly free — catches structural gaps before spending tokens on subagent runs.
- **Custom personas:** Default Bug Hunter/Architect/Pragmatist don't fit methodology assessment. Methodologist/Skill Architect/Pragmatist cover process rigor, output quality, and efficiency.
- **4 experiments total:** Enough signal without being exhaustive. Can add more if overfitting suspected.

## Non-Goals

- Improving the pilot skills themselves (they're instruments, not targets)
- Full Phase 1-2-3 progression from `experiment-set-design` (overkill for first pass)
- Automating subagent runs (manual is fine for this scale)
