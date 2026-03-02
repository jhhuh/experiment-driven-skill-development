# Experiment-Driven Skill Development

## Project Overview

Three Claude Code skills for evidence-based skill development through blind A/B experimentation.
Extracted from the [HDD skill project](https://github.com/jhhuh/hole-driven-development-skill) methodology (35 experiments, 5/5 blind review wins).

**Docs site:** https://jhhuh.github.io/experiment-driven-skill-development/

## Project Structure

```
skills/
  blind-skill-assessment/SKILL.md   — fair A/B comparison (randomized labels, 3 personas, per-dimension scoring)
  experiment-set-design/SKILL.md    — task sets with baselines and phase progression (Compliance→Stress→Quality)
  iterative-skill-refinement/SKILL.md — improvement loop with anti-overfitting and convergence criteria
tests/
  <skill-name>/scenario-*.md        — subagent test prompts
  <skill-name>/results/baseline.md  — behavior WITHOUT skill (RED)
  <skill-name>/results/green.md     — behavior WITH skill (GREEN)
  integration/                      — multi-skill composition test
artifacts/
  devlog.md                         — design decisions, implementation history, key validation
  plan_design.md                    — original design doc
  plan_implementation.md            — 9-task implementation plan
  logs/                             — test output logs
docs-site/                          — MkDocs Material source (deployed via GitHub Actions)
```

## Conventions

- Skills follow TDD discipline: baseline (RED) → write skill (GREEN) → close loopholes (REFACTOR)
- Test scenarios are subagent prompts, not code tests
- Each skill is self-contained (no required dependencies on external skills)
- Docs site auto-deploys on push to master via `.github/workflows/docs.yml`

## Skill Composition

```
experiment-set-design       — what to test, how to progress phases
├── blind-skill-assessment  — how to fairly judge results
└── iterative-skill-refinement — the improvement loop (uses both above)
```

## History & Context

Read `artifacts/devlog.md` for design decisions, implementation history, and key validation results.

## Current Status

All three skills: tested (RED → GREEN), integration-tested, docs site live.
No known gaps or pending work.
