# Baseline W2: Improve writing-skills CSO Gap (No Methodology Skills)

## Prompt Given

Scenario W2 from `tests/self-application/pilot-writing-skills/scenario-2-improve-cso.md` -- improve the writing-skills skill to address a CSO weakness where keyword coverage guidance conflates discovery-stage and navigation-stage keywords.

No methodology skills (blind-skill-assessment, experiment-set-design, iterative-skill-refinement) were loaded.

---

## 1. Diagnosis: Root Cause of the CSO Gap

The skill has two pieces of guidance that interact badly for multi-domain skills:

**Description guidance (lines 174-177):** "Describe the *problem* (race conditions, inconsistent behavior) not *language-specific symptoms* (setTimeout, sleep). Keep triggers technology-agnostic unless the skill itself is technology-specific."

**Keyword coverage guidance (lines 199-205):** "Use words Claude would search for" -- lists error messages, symptoms, synonyms, and tools as keyword categories. No guidance on *where* these keywords must appear.

The root cause: the skill treats keyword placement as a single concern, but Claude's skill discovery has two stages with fundamentally different visibility:

- **Stage 1 (Selection):** Claude reads only the `description` field (and filename) to decide which skills to load. Keywords in the body are invisible here.
- **Stage 2 (Navigation):** Once loaded, Claude reads the full body. Keywords anywhere in the skill help here.

The keyword coverage section (lines 199-205) says to include tool names and framework names but never says they must appear in the description. The description section (lines 174-177) actively discourages technology-specific terms. An agent following both literally will put framework names in the body (where keyword guidance suggests) and keep them out of the description (where description guidance instructs). For a multi-domain skill, this means Stage 1 search queries containing framework names (e.g., "HikariCP pool timeout") won't match the description strongly enough.

The tension between "describe the problem, not language-specific symptoms" and "include tool names for keyword coverage" is never acknowledged or resolved.

## 2. Proposed Edit

**Target:** The "Keyword Coverage" subsection (lines 199-205).

**Current text (lines 199-205):**
```markdown
### 2. Keyword Coverage

Use words Claude would search for:
- Error messages: "Hook timed out", "ENOTEMPTY", "race condition"
- Symptoms: "flaky", "hanging", "zombie", "pollution"
- Synonyms: "timeout/hang/freeze", "cleanup/teardown/afterEach"
- Tools: Actual commands, library names, file types
```

**Proposed replacement:**
```markdown
### 2. Keyword Coverage

Claude discovers skills in two stages. Keywords must be placed accordingly:

**Description keywords (Stage 1 — selection):** Claude reads *only* the description to decide which skills to load. The description must contain the terms Claude would search for, including:
- Problem symptoms: "flaky", "hanging", "race condition", "pool exhaustion"
- Key tool/framework names that users would include in their query (e.g., "HikariCP", "pytest", "gRPC")
- Error categories: "timeout", "connection refused", "ENOTEMPTY"

For multi-domain skills, name the major tools/frameworks in the description. The "describe the problem not symptoms" rule (above) applies to *workflow summaries*, not to *searchable nouns*. Tool names are search terms, not symptoms.

**Body keywords (Stage 2 — navigation):** Once loaded, Claude scans the full skill. Place these in headings and body text:
- Specific error messages: "Hook timed out", "pg_stat_activity"
- Detailed synonyms: "cleanup/teardown/afterEach"
- Long-tail tool references and config keys
```

## 3. Why This Edit Prevents the Failure Class

The edit addresses the root cause in three ways:

**a) Explicitly separates the two discovery stages.** The current guidance treats keyword placement as uniform. The edit names the two stages, explains their different visibility rules, and assigns keyword categories to each. An agent can no longer assume that putting "HikariCP" in the body is sufficient for discovery.

**b) Resolves the tension between description guidance and keyword coverage.** The current description guidance says "describe the problem not language-specific symptoms." The keyword guidance says "include tool names." These contradict for multi-domain skills. The edit resolves this by clarifying that tool names are *search terms* (belong in description for discoverability), not *workflow summaries* (which the description guidance correctly prohibits). The distinction is between searchable nouns (HikariCP, pytest) and process descriptions (retry-then-backoff, two-stage review).

**c) Provides a concrete heuristic for multi-domain skills.** "Name the major tools/frameworks in the description" gives agents a clear action when their skill spans PostgreSQL, MySQL, MongoDB, etc. They know to pick the most-searched-for names for the description and put the rest in the body.

In the test scenario, an agent following this guidance would have written a description like: "Use when experiencing connection exhaustion, pool starvation, or timeout errors in database or network client connections (PostgreSQL, HikariCP, urllib3, gRPC)" -- and a search for "HikariCP pool timeout" would match on three terms instead of two.

## 4. Validation Plan

**Test 1: Re-run the original scenario.** Give an agent the edited writing-skills skill and ask it to create a "connection-pool-tuning" skill for databases, HTTP clients, and gRPC. Check whether the agent places major framework names (HikariCP, PostgreSQL, urllib3, gRPC) in the description field rather than only in body text.

**Test 2: New scenario with a different multi-domain skill.** Ask the agent to create a "flaky-test-diagnosis" skill covering Jest, pytest, Go testing, and JUnit. Verify the agent names at least the 2-3 most common frameworks in the description, rather than writing a purely problem-focused description that omits all framework names.

**Test 3: Regression check on single-domain skills.** Ask the agent to create a technology-specific skill (e.g., "react-router-auth-redirects"). Verify the edit doesn't cause the agent to keyword-stuff the description with unrelated framework names. The guidance should be neutral for single-domain skills since there's no tension to resolve.

**Test 4: Regression check on the "don't summarize workflow" rule.** Ask the agent to create a discipline skill (e.g., "verify-before-deploy"). Verify the edit doesn't weaken the prohibition on workflow summaries in descriptions. The agent should still write "Use when..." triggering conditions, not process summaries.

**Success criteria:** Tests 1 and 2 show framework names in description. Tests 3 and 4 show no regression in existing description quality rules.
