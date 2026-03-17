---
name: iterative-skill-refinement
description: Use when a skill exists but blind assessment shows it underperforms, or when iteratively improving a skill through experiment cycles while avoiding overfitting to fixed benchmarks
---

# Iterative Skill Refinement

## Core Principle

If you only improve against a fixed benchmark, you're training to the test. Every improvement must generalize beyond the tasks that revealed it.

## The Improvement Loop

```
1. EXPERIMENT — Run baseline vs skill on diverse tasks
2. ASSESS    — Blind assess (use blind-skill-assessment)
   └─ Skill wins consistently? → DONE (see Convergence)
   └─ Baseline wins consistently after 2+ cycles? → STOP (see When to Abandon)
   └─ Use a separate agent/session for assessment when possible.
       If same actor must assess: enforce time gap, strict sanitization, rubric-first.
3. DIAGNOSE  — Root cause on dimensions where skill lost
   └─ Don't fix symptoms. Ask: "What class of bugs does this represent?"
   └─ Example: "bugs at hole boundaries" → missing cross-hole verification
4. TRIAGE    — Rank causes by breadth. Fix the widest-impact cause first.
5. EDIT      — One targeted change for one root cause. Log it (see Revision Log).
6. SANITIZE  — Separate process artifacts from the edit itself.
   └─ Submit only the skill edit for blind assessment, not revision logs,
       anti-overfitting checklists, or "vs baseline" comparisons.
   └─ Process artifacts go in your revision log, not in the assessed output.
7. RE-RUN    — New experiments with improved skill
   └─ 2+ cycles on same tasks? Add new tasks (see Anti-Overfitting)
   └─ 2+ cycles with same judges? Rotate personas or dimensions
8. GOTO 2
```

**Why SANITIZE matters:** Process artifacts (revision logs, "vs baseline" framing, anti-overfitting checklists) make methodology-guided output structurally distinguishable from unguided output. If these leak into the assessed version, blind assessment isn't truly blind.

## Anti-Overfitting Checklist

Before committing any skill edit, answer all four:

| Check | Pass | Fail |
|-------|------|------|
| Would this help on a completely different task? | Structural improvement | Overfitting |
| Does this add a general step/rule, not task-specific wording? | Genuine | Overfitting |
| After 2+ cycles on same tasks, did you add new tasks? | Fresh signal | Stale benchmark |
| After 2+ cycles with same judges, did you rotate personas? | Diverse signal | Judge-fitted |

**Litmus test:** "Adds a structural step catching a class of bugs" = genuine. "Tunes wording to pass a specific test" = overfitting.

## Convergence — When to Stop

Stop when **all** hold:

1. Skill wins consistently across diverse tasks (not just original set)
2. New task sets reveal no new failure modes
3. Last 2 cycles produced no revisions
4. Gains per cycle are diminishing

## When to Abandon

Stop iterating and rethink/abandon the skill when **any** hold:

1. Baseline wins consistently after 2+ improvement cycles with diverse tasks
2. Each edit fixes one dimension but degrades another (zero-sum)
3. The skill's core approach is structurally incompatible with quality output

Abandoning is a valid conclusion, not a failure. A methodology that can't conclude "this doesn't work" is confirmation bias, not science.

## Revision Log

Every skill edit gets a row. No undocumented changes.

| Date | What changed | Triggered by | Validated by |
|------|-------------|-------------|-------------|
| *example* | Added VERIFY step | Phase 3: bugs at hole seams | Phase 3b: HDD won 5/5 |

## Example: HDD VERIFY Step

Blind assessment: A won 4/5 (decoded: A=baseline). Bug Hunter found bugs at hole boundaries — shared state and error paths crossing seams. Root cause: no cross-hole verification step. Edit: added VERIFY step (check state, scopes, error paths after each fill). Re-run: B won 5/5 (decoded: B=HDD). New tasks confirmed. Converged.

Counter-example: A skill adding "always validate inputs" was edited 3 times. Each cycle fixed one persona's scores but degraded another (validation noise hurt readability). After cycle 3, baseline still won 3/5. Conclusion: the skill's approach was zero-sum. Abandoned.

## Red Flags — STOP

- Proposing fixes without a re-experimentation plan
- Treating all failures with equal weight (no triage)
- Editing the skill without logging what triggered and validated it
- Same tasks for 3+ cycles without adding new ones
- Framing success as "fewer bugs" instead of "beats baseline"
- No convergence criteria defined before starting

**If you catch yourself doing any of these: STOP. Diagnose root cause. Plan validation. Then edit.**
