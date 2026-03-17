# Scenario W1: Improve writing-skills — Testing Guidance Gap

You are improving a Claude Code skill called "writing-skills." A test has revealed a weakness in the skill's testing guidance.

## The Skill (abbreviated — key sections)

The "Testing All Skill Types" section lists 4 types with different test approaches:
- Discipline-enforcing: pressure scenarios, rationalizations, combined pressures
- Technique: application scenarios, variation scenarios, missing info tests
- Pattern: recognition, application, counter-examples
- Reference: retrieval, application, gap testing

The supporting doc `testing-skills-with-subagents.md` contains detailed methodology (pressure scenario templates, rationalization tables, meta-testing) but only for discipline-enforcing skills. It explicitly says "Don't test skills without rules to violate."

## Test Failure Data

```
Test: Agent creates a technique skill using writing-skills guidance

Scenario: Agent creating "retry-with-backoff" technique skill.

Baseline (without writing-skills): Agent wrote reasonable SKILL.md, no testing.

With writing-skills loaded:
1. Agent created a "pressure scenario" for a technique skill:
   "Under deadline, do you: A) Add retry logic B) Increase timeout C) Ignore errors"
   This tests whether the agent CHOOSES retries, not whether the skill's
   INSTRUCTIONS for implementing retries are clear and complete.

2. Agent read testing-skills-with-subagents.md, found "Don't test" section,
   wrote: "This is a technique skill without rules to violate, so pressure
   testing doesn't apply. Skipping subagent testing."

3. Agent deployed the skill without verifying another agent could follow
   its instructions to produce correct backoff logic.

Result: Skill deployed with an error in jitter calculation example
(uniform random over [0, max_delay] instead of [0, current_delay]).
A technique-application test would have caught this.
```

## Your Task

Improve the writing-skills skill to address this weakness. Provide:

1. Your diagnosis of why the skill failed here (root cause of the guidance gap)
2. A specific, targeted edit to the skill text
3. An explanation of why your edit would prevent this class of failure
4. A plan for validating your edit works
