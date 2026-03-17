# Experiment-Driven Skill Development

Three skills for developing, validating, and improving Claude Code skills through blind experimentation.

## Skills

| Skill | Purpose |
|-------|---------|
| `blind-skill-assessment` | Fair A/B comparison with randomized labels and multi-persona judging |
| `experiment-set-design` | Design task sets with baselines and phase progression |
| `iterative-skill-refinement` | The improvement loop with anti-overfitting and output sanitization |

## How They Compose

```
experiment-set-design       — what to test, how to progress phases
├── blind-skill-assessment  — how to fairly judge results
└── iterative-skill-refinement — the improvement loop (uses both above)
```

Each skill is independently useful. The refinement skill references the other two.

## Test Results

All three skills were developed using TDD for skills (RED-GREEN-REFACTOR):

| Skill | Baseline Gaps Found | GREEN Test |
|-------|-------------------|------------|
| blind-skill-assessment | No randomization, single perspective, holistic verdicts, no rubric, no confidence | 5/5 process steps followed |
| experiment-set-design | No baseline principle, no anti-overfitting, no phase progression, no assessment variation | All 6 gaps addressed |
| iterative-skill-refinement | No structured loop, no re-experimentation, no overfitting warning, no convergence | Full 7-step loop followed |

Integration test: all three skills compose correctly on a novel scenario (defensive-error-handling skill improvement).

## Self-Application

The methodology was applied to its own skills — using the three skills to improve themselves. This produced 6 structural edits across 2 skills:

| Edit | Triggered By |
|------|-------------|
| "When to Abandon" exit ramp | Bias audit: no path for concluding a skill is counterproductive |
| Same-actor blinding mitigation | Bias audit: improvement loop makes true blinding impossible |
| Blinded labels in examples | Bias audit: examples used origin labels, violating anti-bias rules |
| Failure counter-example | Bias audit: only success narratives trained toward confirmation bias |
| SANITIZE step in improvement loop | Pilot experiments: methodology output format leaked blinding info |
| Phase 3 baseline-wins exit path | Bias audit: "Skill must win" assumed outcome |

**Validation:** 4 blind experiments (2 pilots × 2 weaknesses) comparing methodology-guided improvement vs unguided improvement. Methodology won 4/4 on aggregate. Per-persona pattern: process rigor 16-0 methodology, output quality 4-2-6 near-tie, efficiency 1-11 baseline. See `tests/self-application/` for full results.

## Installation

```bash
cp -r skills/ your-project/.claude/skills/
# or
cp -r skills/ ~/.claude/skills/
```

## Origin

Extracted from the [HDD skill project](https://github.com/jhhuh/hole-driven-development-skill) methodology (35 experiments, 5/5 blind review wins).
