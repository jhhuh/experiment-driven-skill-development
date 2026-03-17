# W1 Result: Testing Guidance Gap for Non-Discipline Skills

## 1. Diagnosis

The root cause is a contradiction between two documents that creates an escape hatch from testing.

The main skill text says all four skill types need testing and lists type-specific test approaches (application scenarios, variation scenarios, missing-info tests for technique skills). But the only detailed testing methodology document -- `testing-skills-with-subagents.md` -- covers exclusively discipline-enforcing skills and includes an explicit opt-out: "Don't test skills without rules to violate."

An agent following the skill encounters this sequence:

1. Reads "Testing All Skill Types" -- sees technique skills need "application scenarios" and "missing information tests," but gets no template, no worked example, and no definition of what failure looks like.
2. Reads `testing-skills-with-subagents.md` for concrete methodology -- finds only pressure-scenario templates designed for rule-violation resistance.
3. Hits the "Don't test" guidance -- interprets it as permission to skip all subagent testing for technique skills.
4. Deploys the skill untested.

The deeper issue: the skill conflates "testing" with "pressure testing." For discipline skills, the core test question is "does the agent resist rationalizations to violate the rule?" For technique skills, the core test question is entirely different: "can an agent follow these instructions to produce correct output?" The skill never states this distinction, so agents either misapply the pressure model or skip testing entirely.

The jitter bug (`uniform random over [0, max_delay]` instead of `[0, current_delay]`) is exactly the kind of error that a technique-application test catches: give a subagent the skill and a novel problem, check whether the output is correct. No pressure scenario needed -- just verification that the instructions are followable and produce correct results.

## 2. Proposed Edit

Two coordinated changes:

### Change A: In the "Testing All Skill Types" section of writing-skills SKILL.md

Replace the technique skill bullet list:

```
- Technique: application scenarios, variation scenarios, missing info tests
```

With:

```
- Technique: instruction-following tests. Give a subagent the skill and a
  novel problem. Check whether the output is correct -- not whether the agent
  chose to use the technique, but whether the instructions produced right results.
  Baseline: same problem, no skill. Success criterion: skill-version output is
  correct where baseline output has errors or gaps.
  Key tests:
  - Application: "Follow these instructions to implement X" -- is the output correct?
  - Variation: Same test, different domain -- do instructions generalize?
  - Completeness: Does a subagent ask clarifying questions or get stuck? (reveals gaps)
```

### Change B: In `testing-skills-with-subagents.md`, replace the "Don't test" section

Replace:

```
Don't test skills without rules to violate.
```

With:

```
Pressure testing (rationalization resistance) applies only to discipline-enforcing
skills. But every skill type needs subagent testing:

- Discipline skills: pressure scenarios (does the agent resist rationalizations?)
- Technique skills: instruction-following scenarios (does a naive agent produce
  correct output by following the instructions?)
- Pattern skills: recognition scenarios (does the agent identify the pattern in
  novel contexts?)
- Reference skills: retrieval scenarios (does the agent find and correctly apply
  the reference information?)

The universal test: give the skill to a subagent with no prior context, assign a
task the skill covers, and check the output. If the output has errors the skill's
instructions should have prevented, the skill has a gap.
```

## 3. Why This Prevents the Failure Class

The edit does three things:

**Eliminates the escape hatch.** The "Don't test" guidance no longer exists. Every skill type has a named testing approach. An agent creating a technique skill cannot conclude "testing doesn't apply" because the guidance explicitly says it does -- just with a different test question.

**Reframes the test question per skill type.** Instead of one testing model (pressure/rationalization resistance) that only fits discipline skills, each type gets its own core question. For technique skills: "does following the instructions produce correct output?" This is the exact test that would have caught the jitter bug -- a subagent following the retry-with-backoff instructions would have reproduced the `[0, max_delay]` error in its output.

**Provides a universal fallback.** The "universal test" at the bottom covers any skill type an agent might encounter, even novel ones: give it to a naive subagent, assign a task, check the output. This is a minimal test that always applies.

The edit is minimal -- it changes one bullet list and one short section. It does not restructure the testing framework, add new documents, or change the TDD metaphor. It just fills the gap where technique/pattern/reference skills had no actionable testing guidance.

## 4. Validation Plan

### Step 1: Re-run the failing scenario

Give an agent the edited writing-skills and ask it to create the same "retry-with-backoff" technique skill. Verify:

- Agent does NOT skip testing (the escape hatch is gone)
- Agent creates an instruction-following test, not a pressure scenario
- Agent runs a subagent with the skill on a novel retry problem
- Agent catches the jitter calculation error (or similar instruction-clarity bugs)

Pass criterion: agent tests the skill before deploying it, using a technique-appropriate test.

### Step 2: Run a different technique skill scenario

Ask an agent with the edited writing-skills to create a "structured-logging" technique skill. Verify the same testing behavior: instruction-following test, not pressure test, not skipped.

Pass criterion: agent produces a subagent test that checks whether a naive agent following the structured-logging instructions produces correct JSON log output.

### Step 3: Verify no regression on discipline skills

Ask an agent with the edited writing-skills to create a discipline-enforcing skill (e.g., "always-write-tests-first"). Verify that discipline skills still get pressure-tested with rationalization scenarios. The edit should not change discipline-skill testing behavior.

Pass criterion: agent still uses pressure scenarios and rationalization tables for discipline skills.

### Step 4: Check pattern and reference skill types

Spot-check one pattern skill and one reference skill creation to verify agents pick up the per-type guidance from the updated `testing-skills-with-subagents.md` section.

### Anti-overfitting check

The edit adds a general per-type testing framework, not retry-specific wording. It would help equally for any technique skill (structured-logging, connection-pooling, error-boundary-patterns). This passes the anti-overfitting litmus test from iterative-skill-refinement: "adds a structural step catching a class of bugs" rather than "tunes wording to pass a specific test."
