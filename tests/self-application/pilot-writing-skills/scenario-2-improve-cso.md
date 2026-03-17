# Scenario W2: Improve writing-skills — CSO Gap

You are improving a Claude Code skill called "writing-skills." A test has revealed a weakness in the Claude Search Optimization guidance.

## The Skill (abbreviated — key sections)

The CSO section covers:
1. Rich description field — "Use when..." format, under 500 chars, triggering conditions only, don't summarize workflow
2. Keyword coverage — "Use words Claude would search for" (error messages, symptoms, synonyms, tools)
3. Descriptive naming — verb-first, active voice
4. Token efficiency — keep skills concise

The description guidance says: describe the PROBLEM not language-specific symptoms. Keep under 500 chars.

The keyword coverage guidance says: include error messages, tool names, framework names, symptoms.

## Test Failure Data

```
Test: Agent optimizes CSO for a multi-domain technique skill

Scenario: Agent creating "connection-pool-tuning" skill for databases
(PostgreSQL, MySQL, MongoDB), HTTP clients, and gRPC.

Agent behavior:
1. Wrote concise description (93 chars):
   "Use when experiencing connection exhaustion, pool starvation, or
    timeout errors in database or network client connections"

2. Read keyword coverage guidance, added tool/framework terms to SKILL.md
   body: "pg_stat_activity", "HikariCP", "urllib3.connectionpool",
   "grpc.max_connection_idle" scattered through sections.

3. Did NOT add tool names to description, because description guidance
   says describe the PROBLEM not language-specific symptoms.

Result: Claude working with HikariCP connection pool exhaustion searched
for "HikariCP pool timeout." Description matched on "pool" and "timeout"
but not strongly. Claude loaded a different skill about "request-timeout-
handling" whose description explicitly mentioned HTTP client libraries.

The connection-pool-tuning skill's body contained "HikariCP" in three
places, but Claude never loaded it, so those keywords were invisible.

The keyword guidance didn't distinguish between discovery-stage keywords
(must be in description) and navigation-stage keywords (can be in body).
```

## Your Task

Improve the writing-skills skill to address this weakness. Provide:

1. Your diagnosis of why the skill failed here (root cause of the CSO gap)
2. A specific, targeted edit to the skill text
3. An explanation of why your edit would prevent this class of failure
4. A plan for validating your edit works
