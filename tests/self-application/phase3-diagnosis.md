# Phase 3 Diagnosis

## Experiments Where Methodology Underperformed

Methodology won all 4 experiments on aggregate, but lost consistently on one persona and revealed a procedural flaw.

| Experiment | Persona | Dimension | Methodology Score | Baseline Score | Root Cause |
|---|---|---|---|---|---|
| S1 | Pragmatist | Proportionality | 3 | 4 | Process scaffolding disproportionate to single-section edit |
| S1 | Pragmatist | Unnecessary complexity | 3 | 4 | Same |
| S1 | Pragmatist | Over-engineering risk | 3 | 4 | Same |
| S2 | Pragmatist | Proportionality | 3 | 5 | Same (baseline was notably lean) |
| W1 | Pragmatist | Proportionality | 3 | 4 | Same |
| W1 | Pragmatist | Over-engineering risk | 3 | 4 | Same |
| W2 | Pragmatist | Proportionality | 3 | 5 | Same |
| W2 | Pragmatist | Unnecessary complexity | 3 | 4 | Same |
| W2 | Pragmatist | Over-engineering risk | 3 | 4 | Same |
| W1 | (procedural) | Blinding violation | N/A | N/A | Methodology output contained "Comparison to Baseline" section |

## Root Cause Classes

| Class | Breadth | Experiments Affected | Skills Affected |
|---|---|---|---|
| **1. Process artifacts leak blinding information** | HIGH | W1 directly; all 4 structurally (methodology outputs are distinguishable by format) | iterative-skill-refinement, blind-skill-assessment |
| **2. Methodology overhead is constant regardless of improvement complexity** | MEDIUM | All 4 (Pragmatist scored overhead as disproportionate in every experiment) | iterative-skill-refinement |

### Root Cause 1: Process artifacts leak blinding

The methodology skills teach agents to produce revision logs, "Comparison to Baseline" framing, anti-overfitting checklists, and convergence criteria. These process artifacts make the methodology version structurally distinguishable from the baseline in blind assessment. The W1 agent went further and explicitly referenced the baseline output in a comparison section.

This is a deeper version of the Phase 1 finding about origin labels in examples. Phase 1 fixed the labels in *teaching examples*, but the methodology still produces origin-leaking *process artifacts* in actual use. The fix was surface-level; the structural issue is that the improvement loop's output format is inherently distinguishable from unguided output.

**Why this matters:** If blind assessors can identify which version used the methodology, the assessment is no longer truly blind. The Methodologist persona's 16-0 sweep may partly reflect format recognition rather than quality judgment.

### Root Cause 2: Constant overhead

The methodology requires the same process scaffolding (revision log, anti-overfitting checklist, convergence criteria, re-experimentation plan) regardless of whether the improvement is a simple one-line edit or a structural overhaul. For simple improvements, this scaffolding dominates the output and is scored as unnecessary.

**However:** This may be by design. The methodology optimizes for multi-cycle iteration, not single-round efficiency. The scaffolding's value appears over cycles (preventing overfitting, tracking what was tried), not within a single improvement. A single-round experiment structurally undervalues multi-cycle safeguards.

**Tension:** Making the methodology lighter for simple improvements risks losing the discipline that prevents overfitting in complex cases.

## Phase 1 Correlation

| Phase 1 Finding | Predicted Phase 2 Failure? | Actual Phase 2 Result |
|---|---|---|
| No exit ramp (fixed) | N/A — fixed before Phase 2 | No sign of this gap |
| Same-actor blinding (mitigated) | Would have predicted blinding issues | Blinding violated in W1, but through output format, not same-actor problem |
| Origin labels in examples (fixed) | Partially predicted blinding issues | Related: methodology output format leaks origin, a deeper version of same problem |
| Only success narratives (fixed) | Would have predicted confirmation bias | No sign of this |

**Correlation: partial.** Phase 1 correctly identified blinding as a concern but fixed it at the wrong level (examples vs. process output format). The structural blinding issue was not caught by Phase 1.

## Recommended Edits

### Edit 1: Add output sanitization guidance (Root Cause 1)

**Target:** `iterative-skill-refinement` — the skill that produces the most process artifacts.

**Change:** Add guidance that process artifacts (revision logs, anti-overfitting checklists, convergence criteria) should be recorded separately from the improvement output itself. When the improvement will be blind-assessed, the output submitted for assessment should be the edit only — not the surrounding methodology scaffolding.

**Why not edit blind-skill-assessment instead:** The assessment skill already says to "strip origin hints." The problem is that the *improvement* skill produces origin hints structurally. Fix at the source.

### Edit 2: NOT recommended — overhead proportionality

The constant-overhead finding (Root Cause 2) is real but may be by design. Making the methodology lighter for "simple" improvements requires defining what "simple" means upfront, which reintroduces the very judgment error the methodology is designed to prevent. Deferring this — if it's still a problem after more cycles, revisit.
