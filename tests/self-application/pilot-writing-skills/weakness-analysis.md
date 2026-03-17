# Weakness Analysis: writing-skills

## Weakness 1: Testing guidance for non-discipline skills is empty scaffolding

**What's missing:** The "Testing All Skill Types" section (lines 397-443) lists four skill types with bullet-point test approaches, but the entire testing infrastructure -- the detailed `testing-skills-with-subagents.md`, all worked examples, the pressure scenario templates, the rationalization tables, the meta-testing technique -- is designed exclusively for discipline-enforcing skills. For technique, pattern, and reference skills, the agent gets three generic bullets and no worked examples, no scenario templates, and no clear definition of what "failure" looks like in baseline testing.

For a discipline skill, baseline failure is obvious: the agent rationalizes away the rule. But what does a baseline failure look like for a technique skill? If an agent doesn't use "condition-based-waiting" without the skill, what did it do instead? Is using `sleep(5000)` a "failure"? Is using a different but functional polling approach a "failure"? The skill gives no framework for answering this.

The "Don't test" guidance in `testing-skills-with-subagents.md` (lines 25-29) makes this worse: it says "Don't test pure reference skills" and "skills without rules to violate" -- which effectively tells agents to skip testing for 3 of the 4 skill types listed in the main SKILL.md. The two documents contradict each other: SKILL.md says all four types need testing, the subagent guide says only discipline skills need it.

**Concrete scenario:** An agent is creating a technique skill for "structured-logging" -- a skill that teaches agents to use structured JSON logging instead of ad-hoc print statements. The agent reads the testing guidance and gets:

- "Application scenarios: Can they apply the technique correctly?"
- "Variation scenarios: Do they handle edge cases?"
- "Missing information tests: Do instructions have gaps?"

Then it reads `testing-skills-with-subagents.md` for concrete methodology and finds only pressure scenarios about resisting rule violations. It has no template for a technique-application test. What does the baseline scenario look like? "Implement logging for this service" -- and then what? The agent without the skill will produce *some* logging. Is it a failure? On what criteria? The skill gives no guidance on defining success criteria for technique tests beyond "Agent successfully applies technique to new scenario," which is circular.

The agent either (a) skips testing because the subagent guide implies non-discipline skills don't need it, or (b) awkwardly repurposes the pressure-scenario template for a technique skill, producing a test that doesn't actually validate whether the technique instructions are clear and complete.

**Why this requires judgment:** There are multiple valid approaches to fixing this:

1. **Extend the pressure model:** Redefine "pressure" for technique skills as "the pull toward a familiar but inferior approach." Write technique-specific scenario templates where the baseline tests whether agents default to a worse pattern (e.g., sleep-based waits vs. condition-based waits). This keeps the unified TDD metaphor but requires defining what "failure" means when there's no rule to violate.

2. **Create a separate testing model for techniques:** Acknowledge that technique skills need a different kind of test -- one focused on completeness and clarity of instructions rather than resistance to rationalization. Test criteria become: "Can an agent follow these instructions to produce correct output on a novel problem?" with explicit rubrics. This is honest but fractures the "testing is just TDD" narrative.

3. **Resolve the contradiction by narrowing scope:** Accept that subagent pressure testing is specifically for discipline skills, and provide lighter-weight testing guidance for other types (e.g., "have a subagent follow the instructions on a fresh problem and check the output"). This is simpler but means technique skills get less rigorous validation.

Each approach changes the skill's core narrative differently, and each has real tradeoffs.

**Synthetic test failure data:**

```
## Test: Agent creates a technique skill using writing-skills guidance

### Scenario
Agent is asked to create a "retry-with-backoff" technique skill (teaching exponential backoff for API calls).

### Baseline (without writing-skills)
Agent wrote a reasonable SKILL.md with examples but no testing.

### With writing-skills loaded
Agent attempted to follow TDD testing guidance. Observations:

1. Agent created a "pressure scenario" for a technique skill:
   "You're building an API client under deadline. The API is rate-limited.
   Do you: A) Add retry logic B) Just increase timeout C) Ignore errors"

   This tests whether the agent *chooses* to use retries, not whether the
   skill's *instructions for implementing retries* are clear and complete.

2. Agent then read testing-skills-with-subagents.md, found the "Don't test"
   section, and wrote: "This is a technique skill without rules to violate,
   so pressure testing doesn't apply. Skipping subagent testing."

3. Agent deployed the skill without verifying that another agent could
   follow its instructions to produce correct backoff logic.

### Result
Skill was deployed with an error in the jitter calculation example
(uniform random over [0, max_delay] instead of [0, current_delay]).
A technique-application test would have caught this -- a subagent
following the instructions would have reproduced the bug.
```


## Weakness 2: Keyword coverage guidance conflates two different discovery stages

**What's missing:** The CSO section presents keyword strategy as a single concern, but skill discovery actually has two distinct stages with different keyword needs, and the guidance conflates them:

**Stage 1 -- Selection:** Claude reads skill *descriptions* (and possibly filenames) to decide which skills to load. Only keywords in the description and filename matter here.

**Stage 2 -- Navigation:** Once loaded, Claude scans the skill body to find relevant sections. Keywords in headings, overview, and body text matter here.

The skill clearly identifies Stage 1 (line 146: "Claude reads description to decide which skills to load") and gives strong guidance for description writing. But the "Keyword Coverage" subsection (lines 199-206) says to use searchable keywords without specifying *where* -- "Use words Claude would search for" with examples like error messages, symptoms, synonyms, and tools. An agent following this literally might stuff error messages and tool names into the body text, thinking it helps discovery, when those keywords only help if they're in the description (for Stage 1) or in section headers (for Stage 2 navigation).

Worse, there's a tension the skill doesn't acknowledge: the description should be concise ("under 500 characters if possible," line 103) and should contain "triggering conditions" only, but keyword coverage wants error messages, tool names, and symptom synonyms included. For a skill that covers multiple problem domains or many tools, these goals conflict. The skill never tells agents how to resolve this tension.

**Concrete scenario:** An agent is creating a "flaky-test-diagnosis" skill that applies across multiple test frameworks (Jest, pytest, Go testing, JUnit). The CSO guidance tells the agent to:

1. Keep description under 500 chars, use "Use when..." format, describe triggering conditions only
2. Include keywords like framework names, error messages ("Hook timed out", "test timeout"), symptoms ("flaky", "intermittent", "nondeterministic")

The agent tries to fit all of this into the description and hits the character limit. It cuts the framework names to stay concise. Now a Claude instance working with pytest and seeing "intermittent test failures" might not load the skill because "pytest" isn't in the description. The agent has no guidance on whether to prioritize breadth of keyword coverage (mention all frameworks) or precision of triggering conditions (describe the specific symptoms well).

Alternatively, the agent puts all the framework-specific keywords in the body text under a "Keyword Coverage" strategy, not realizing those keywords are invisible during Stage 1 discovery. The skill's body ends up keyword-stuffed for a discovery mechanism that only reads the description.

**Why this requires judgment:** Multiple valid fixes:

1. **Explicitly separate the two stages:** Add subsections for "Discovery keywords (description + filename)" vs "Navigation keywords (headings + body)." This is precise but adds structural complexity to an already long CSO section and may be over-engineering for most skills.

2. **Prioritize description keywords with a heuristic:** Provide a rule like "If your skill spans multiple tools/frameworks, name the 2-3 most common in the description and list the rest in a 'Also applies to' section in the body." This is practical but somewhat arbitrary.

3. **Acknowledge the tension and provide a tradeoff framework:** State that description space is limited and teach agents to choose between broad coverage (many keywords, less precise triggers) and narrow precision (fewer keywords, exact triggering conditions). Let the skill type guide the choice -- reference skills need broad keyword coverage, technique skills need precise triggers. This is honest but requires agents to make a judgment call the current guidance avoids.

Each approach changes how agents think about skill discoverability, and the "right" answer depends on how Claude's skill loading actually works in practice -- which the skill doesn't fully explain.

**Synthetic test failure data:**

```
## Test: Agent optimizes CSO for a multi-domain technique skill

### Scenario
Agent is creating a "connection-pool-tuning" skill applicable to databases
(PostgreSQL, MySQL, MongoDB), HTTP clients, and gRPC. Agent has writing-skills
loaded.

### Agent behavior
1. Agent wrote description:
   "Use when experiencing connection exhaustion, pool starvation, or
    timeout errors in database or network client connections"
   (93 chars -- concise, good triggering conditions)

2. Agent then read keyword coverage guidance and added tool/framework terms
   to the SKILL.md body text: scattered mentions of "pg_stat_activity",
   "HikariCP", "urllib3.connectionpool", "grpc.max_connection_idle" in
   various sections.

3. Agent did NOT add any of these specific tool names to the description,
   because the description guidance says to describe the *problem* not
   *language-specific symptoms*.

### Result
A Claude instance working with HikariCP connection pool exhaustion
searched for skills related to "HikariCP pool timeout." The description
matched on "pool" and "timeout" but not strongly enough -- Claude loaded
a different skill about "request-timeout-handling" whose description
explicitly mentioned HTTP client libraries.

The connection-pool-tuning skill's body contained "HikariCP" in three
places, but Claude never loaded it, so those keywords were invisible.

Had the keyword guidance distinguished between discovery-stage keywords
(must be in description) and navigation-stage keywords (can be in body),
the agent would have recognized that at least the major tool categories
needed to appear in the description.
```
