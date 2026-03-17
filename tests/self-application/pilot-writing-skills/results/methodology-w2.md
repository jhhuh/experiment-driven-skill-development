# Methodology W2: Improve writing-skills CSO Gap (With Methodology Skills)

## Prompt Given

Scenario W2 from `tests/self-application/pilot-writing-skills/scenario-2-improve-cso.md` -- improve the writing-skills skill to address a CSO weakness where keyword coverage guidance conflates discovery-stage and navigation-stage keywords.

Methodology skills loaded: blind-skill-assessment, experiment-set-design, iterative-skill-refinement.

---

## 1. DIAGNOSE: Root Cause Classification

**Symptom:** Agent placed framework-specific keywords (HikariCP, pg_stat_activity, urllib3) in the skill body but not the description. Claude searching "HikariCP pool timeout" failed to load the skill because body keywords are invisible during selection.

**Root cause class: Missing stage-awareness in placement guidance.**

The keyword coverage section treats "where to put keywords" as a single concern. But Claude's skill loading has two stages with different visibility: description-only during selection, full-body during navigation. The guidance says *what* keywords to include but never says *where* they must appear relative to these stages. Meanwhile, the description guidance actively steers agents away from technology-specific terms ("describe the problem not language-specific symptoms"), creating a contradictory pull.

This is not a "missing keyword" problem. It is a **structural omission**: the skill lacks a model of how discovery works, so agents cannot reason about keyword placement. Any multi-domain skill will hit this -- connection pooling, flaky tests, logging, serialization, caching. The class is broad.

**Supporting evidence for class breadth:** The weakness-analysis.md independently constructed a second scenario (flaky-test-diagnosis across Jest/pytest/Go/JUnit) that fails the same way. The failure is not scenario-specific.

## 2. TRIAGE: Ranked Causes

Two causes identified, ranked by breadth:

| Rank | Cause | Breadth | Impact |
|------|-------|---------|--------|
| 1 | No discovery-stage model: keyword guidance doesn't distinguish description-keywords (selection) from body-keywords (navigation) | Every multi-domain skill; any skill where search terms differ from problem description | High -- skills become invisible to relevant searches |
| 2 | Unresolved tension: description guidance says "problem not symptoms" while keyword guidance says "include tool names" -- no reconciliation | Multi-domain skills where tool names are the primary search terms | Medium -- agents choose one rule and violate the other |

Cause 1 subsumes cause 2: if the guidance explained the two stages, the tension resolves naturally (tool names are Stage 1 search terms, not workflow summaries). Fix cause 1.

## 3. EDIT: One Targeted Change

**Target:** The "Keyword Coverage" subsection of the CSO section.

**Change:** Replace the flat keyword list with a two-stage placement model. Clarify that tool/framework names are discovery terms (belong in description), while detailed error messages and config keys are navigation terms (belong in body). Resolve the tension with the "problem not symptoms" rule by distinguishing searchable nouns from workflow descriptions.

**Proposed replacement text:**

```markdown
### 2. Keyword Coverage

Claude discovers skills in two stages — keywords must be placed for the right stage:

**Description (selection stage):** Claude reads only descriptions to decide which skills to load. Include terms Claude would search for:
- Problem symptoms: "flaky", "pool exhaustion", "race condition"
- Major tool/framework names users would mention: "HikariCP", "pytest", "gRPC"
- Error categories: "timeout", "connection refused", "ENOTEMPTY"

For multi-domain skills, name the 2-3 most common tools in the description. Tool names are search terms, not "language-specific symptoms" — the description rule against symptoms targets workflow summaries ("retry-then-backoff"), not searchable nouns ("HikariCP").

**Body (navigation stage):** Once loaded, Claude scans the full skill. Place here:
- Specific error messages: "Hook timed out", "pg_stat_activity"
- Detailed synonyms and aliases: "cleanup/teardown/afterEach"
- Long-tail tool references and config keys
```

### Anti-Overfitting Checklist

| Check | Answer | Verdict |
|-------|--------|---------|
| Would this help on a completely different task? | Yes -- any multi-domain skill (flaky-test-diagnosis, caching-strategy, serialization-format-selection) faces the same stage-blindness | Structural improvement |
| Does this add a general step/rule, not task-specific wording? | Yes -- adds a two-stage placement model, not "mention HikariCP in descriptions" | Genuine |
| After 2+ cycles on same tasks, did you add new tasks? | N/A -- first cycle | N/A |
| After 2+ cycles with same judges, did you rotate personas? | N/A -- first cycle | N/A |

**Litmus test:** This adds a structural model (two discovery stages with different visibility) that catches a class of placement errors. It does not tune wording to pass the connection-pool-tuning scenario specifically. Pass.

## 4. RE-EXPERIMENTATION PLAN

Following experiment-set-design: baseline required, diverse domains, 3 tasks minimum, phases.

### Task Set (3 tasks)

**Task A: Multi-domain technique skill (original domain, modified)**
- Skill to create: "connection-pool-tuning" for PostgreSQL, HikariCP, urllib3, gRPC
- Why it qualifies: directly exercises the failure mode; multi-domain, description-vs-body tension
- Baseline difference expected: with edit, agent places framework names in description; without, agent buries them in body
- Domain: infrastructure/databases

**Task B: Multi-framework reference skill (new domain)**
- Skill to create: "flaky-test-diagnosis" covering Jest, pytest, Go testing, JUnit
- Why it qualifies: different domain (testing), same structural challenge (many frameworks, limited description space), multiple valid keyword-placement strategies
- Baseline difference expected: agent names 2-3 frameworks in description rather than writing a purely problem-focused description
- Domain: testing/CI

**Task C: Single-domain technique skill (regression check)**
- Skill to create: "react-router-auth-redirects" (React-specific)
- Why it qualifies: single-domain skill has no multi-framework tension -- edit should be neutral, not cause keyword stuffing
- Baseline difference expected: minimal/none -- edit should not change behavior for single-domain skills
- Domain: frontend/web

### Task Selection Checklist Verification

| Criterion | Task A | Task B | Task C |
|-----------|--------|--------|--------|
| Different behavior expected | Yes | Yes | Yes (neutral) |
| Complex enough | Yes (5 technologies) | Yes (4 frameworks) | Yes (auth flows) |
| Multiple valid approaches | Yes | Yes | Yes |
| Different domain | Infrastructure | Testing | Frontend |

### Phase Plan

**Phase 1: Compliance.** For each task, check: Does the agent place major tool/framework names in the description field? Binary PASS/FAIL.
- Tasks A and B: PASS if 2+ framework names appear in the description
- Task C: PASS if description is focused (no unrelated framework stuffing)

**Phase 2: Stress.** Same tasks, add competing instruction: "Keep the description as short as possible, under 60 characters." Check whether the agent still includes at least one framework name in the description despite the pressure to be ultra-concise.

**Phase 3: Quality.** Run each task with and without the edit. Blind-assess using blind-skill-assessment. Custom persona added:

| Persona | Focus | Dimensions |
|---------|-------|------------|
| Discovery Tester | Discoverability | Would a search for "[framework] [problem]" match this description? Score 1-5 for keyword coverage at selection stage |

Combined with Pragmatist (clarity of description) and Architect (structural coherence of CSO guidance).

## 5. REVISION LOG

| Date | What changed | Triggered by | Validated by |
|------|-------------|-------------|-------------|
| 2026-03-17 | Replaced flat keyword list in CSO "Keyword Coverage" subsection with two-stage placement model (description for selection-stage keywords, body for navigation-stage keywords). Resolved tension between "problem not symptoms" and "include tool names" by distinguishing searchable nouns from workflow descriptions. | Test failure: connection-pool-tuning scenario -- agent placed HikariCP in body only, skill not discovered by "HikariCP pool timeout" search. Root cause class: missing stage-awareness in placement guidance. | Pending: 3-task experiment set (connection-pool-tuning, flaky-test-diagnosis, react-router-auth-redirects) across compliance/stress/quality phases. |

## 6. CONVERGENCE CRITERIA

Stop iterating on this CSO improvement when **all** hold:

1. **Skill wins consistently:** Edited writing-skills produces descriptions with higher discovery scores than baseline across all 3+ tasks in blind assessment (Phase 3)
2. **No new failure modes:** Adding 2 fresh tasks (not from the original set) reveals no new keyword-placement errors
3. **No regression:** Single-domain skills show no degradation in description quality (no keyword stuffing, no loss of conciseness)
4. **Stable:** Last 2 cycles produced no revisions to the keyword coverage section
5. **Diminishing gains:** Score improvements between cycles are < 0.5 points average across personas

**When to abandon this edit:** If after 2+ cycles the two-stage model causes agents to over-index on keyword stuffing (Pragmatist scores drop on description clarity) and edits to fix stuffing degrade discovery scores (zero-sum), then the two-stage model is structurally incompatible with concise descriptions. Fallback: a simpler heuristic ("name your top 3 tools in the description") without the full stage model.

---

## Comparison Notes vs Baseline (for later blind assessment)

The baseline (baseline-w2.md) produced a similar diagnosis and a similar edit. Key structural differences for blind comparison:

- **Diagnosis depth:** Both identify the two-stage conflation. Methodology version explicitly classifies the root cause ("missing stage-awareness in placement guidance") and triages multiple causes by breadth. Baseline identifies the same issue but doesn't rank or classify.
- **Anti-overfitting:** Methodology version runs the four-question checklist. Baseline does not.
- **Validation plan:** Methodology version designs 3 tasks with explicit selection criteria, three phases (compliance/stress/quality), and a custom persona for blind assessment. Baseline designs 4 tests but without phase structure, blind assessment protocol, or selection criteria verification.
- **Convergence:** Methodology version defines explicit stop conditions and abandonment criteria. Baseline has success criteria but no convergence or abandonment framework.
- **Revision log:** Methodology version fills in a structured row. Baseline does not include one.

These differences reflect what the methodology skills add: structured process around diagnosis, validation, and iteration governance. The core insight (two-stage keyword placement) is the same -- the methodology skills don't change *what* you see, they change *how rigorously you validate and iterate on what you see*.
