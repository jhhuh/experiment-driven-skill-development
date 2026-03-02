# Devlog: Experiment-Driven Skill Development

## Origin (2026-03-01)

These skills were extracted from the workflow used to develop the [HDD skills](https://github.com/jhhuh/hole-driven-development-skill). During HDD development, we ran 35 experiments across 5 languages. The key insight: **compliance doesn't prove quality.** The HDD skill passed all compliance and stress tests but lost 4/5 blind quality reviews because cross-hole interaction bugs piled up. Only after adding a VERIFY step (and later monolithic-algorithm guidance and REVIEW-ALL) did the skill win 5/5.

The user observed this was a reusable pattern and asked to capture it as skills.

## Design Decisions

**Three skills, not one.** The user said "it is a big workflow so it may need several small skills as a building block." We decomposed into:
- `blind-skill-assessment` — the judging methodology
- `experiment-set-design` — what to test and how to progress
- `iterative-skill-refinement` — the improvement loop (orchestrates the other two)

**Self-contained.** No dependency on the HDD project's `writing-skills` or `superpowers` skills. Each skill stands alone.

**Baseline is non-negotiable.** The user emphasized: "any assessment of experiment we need a baseline. Otherwise the experiment is meaningless." This became a core principle in all three skills.

**Anti-overfitting.** The user cautioned: "this feedback loop should not fine-tune the skill description in the way that it's effective only to those fixed set of task and way of assessment." This became the anti-overfitting checklist in `iterative-skill-refinement`.

**Default personas + customization.** Three proven judge personas (Bug Hunter, Architect, Pragmatist) ship as defaults, but the skill teaches how to add domain-specific ones.

## Implementation (TDD for Skills)

Followed RED → GREEN → REFACTOR:

### RED: Baselines (what agents do WITHOUT skills)

Ran each test scenario with a fresh subagent, no skills loaded. Documented gaps:

- **blind-skill-assessment baseline:** No label randomization, single perspective, holistic verdicts only, no upfront rubric, no confidence markers.
- **experiment-set-design baseline:** Baseline concept was implicit (not stated as principle), no anti-overfitting, no phase progression, single assessment method, impractical sample sizes.
- **iterative-skill-refinement baseline:** Jumped straight to fixes (no loop structure), no re-experimentation plan, no overfitting warning, no convergence criteria, no triage, no revision tracking.

### GREEN: Skills address every baseline gap

Each skill was written to directly address the gaps found in RED. Then re-tested with skill loaded:

- **blind-skill-assessment GREEN:** 5/5 process steps followed. Agent randomized labels, used 3 personas, scored per-dimension with `[h]/[m]/[l]` confidence, decoded only after scoring, aggregated correctly.
- **experiment-set-design GREEN:** All 6 gaps addressed. Baselines stated as NON-NEGOTIABLE. Three-phase progression designed. Anti-overfitting rules included. Agent even identified 7 HDD rules (more than the 4 listed in the scenario).
- **iterative-skill-refinement GREEN:** Full 7-step loop followed. Agent independently proposed the exact same three improvements (VERIFY, monolithic guidance, REVIEW-ALL) that were actually implemented in the real HDD project — strong validation that the skill guides to the right diagnosis.

### REFACTOR: No loopholes found

All GREEN tests passed cleanly — no shortcuts, rationalizations, or skipped steps. No REFACTOR changes needed.

### Integration Test

Novel scenario: improving a "defensive-error-handling" skill. All three skills composed correctly. `iterative-skill-refinement` orchestrated the loop, `experiment-set-design` governed task planning, `blind-skill-assessment` governed judgment. Clear handoff points, no conflicts.

## Key Validation

The strongest evidence that `iterative-skill-refinement` works: when given the same Phase 3 failure data from the real HDD project, a fresh agent (with only the skill loaded) independently arrived at the same root cause diagnosis and proposed the same structural fixes that were actually implemented. The skill doesn't just teach a process — it guides agents to the right answer.

## Repo Extraction & Standalone Setup (2026-03-02)

This project started as a subdirectory (`experiment-driven-skill-development/`) inside the [HDD skill repo](https://github.com/jhhuh/hole-driven-development-skill). All implementation happened there because the sandbox couldn't persist files outside that repo.

### Extraction steps

1. **`git subtree split --prefix=experiment-driven-skill-development`** — created a branch with only this subdirectory's commits (9 commits, full history preserved).
2. **Created GitHub repo** via `gh repo create jhhuh/experiment-driven-skill-development --public`.
3. **Pushed split branch** as `master` to the new repo.
4. **Added repo metadata** — description, homepage URL, 9 topics (`claude-code`, `claude-code-skills`, `ai-agent-skills`, `skill-development`, `blind-assessment`, `experiment-design`, `tdd`, `prompt-engineering`, `llm-evaluation`).
5. **Set up GitHub Pages** — MkDocs Material site with GitHub Actions workflow (`.github/workflows/docs.yml`), enabled via `gh api repos/.../pages`.
6. **Wrote docs-site** — index page, 3 skill pages (with admonitions for principles/red flags), results page, origin story page.
7. **Removed subdirectory** from parent repo (`git rm -r experiment-driven-skill-development/`).
8. **Enriched CLAUDE.md** — added project overview, skill composition diagram, structure with descriptions, pointer to devlog.
9. **Added artifacts/devlog.md** — this file, capturing design decisions and implementation history from the original conversation.
10. **Restructured to artifact-first convention** — moved `docs/plans/` to `artifacts/plan_*.md`, added `artifacts/logs/`.

### Why this matters

The original conversation context (in the HDD repo's Claude session) is not accessible from a fresh clone of this repo. The devlog, CLAUDE.md, plans, and test results together reconstruct the essential reasoning without needing the original session transcript.
