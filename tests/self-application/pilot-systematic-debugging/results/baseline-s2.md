# Baseline Result: S2 — Interaction Bug Coverage Gap

## 1. Diagnosis: Why the Skill Failed

The skill has two structural assumptions that jointly cause this failure:

**Assumption A: Bugs have a single root cause.** Phase 3 says "Form Single Hypothesis" and the entire flow is designed to converge on one thing that is broken. The multi-component guidance in Phase 1 says to check each boundary to find "WHERE it breaks" -- presupposing that exactly one place breaks. When the agent tested CDN caching and found the bug persisted after disabling it, the skill gave no framework for interpreting that result as "CDN is one contributing factor among several" rather than "CDN is not the cause."

**Assumption B: Correct components produce correct behavior.** Phase 1's boundary logging checks what enters and exits each component. If each component transforms its inputs correctly, Phase 1 declares the path healthy. But in this scenario, every component is individually correct. The bug lives in the timing and layering of three independent caching/consistency mechanisms. Phase 1 has no step that asks "even though each component is correct, could their combination produce incorrect behavior under specific conditions?"

The root cause of the skill gap is not a missing checklist item -- it is a missing *category of hypothesis*. The skill's entire hypothesis vocabulary is "component X is broken." It needs a second form: "components X, Y, Z interact to produce incorrect behavior under condition C."

## 2. Proposed Skill Edit

### Edit location: Phase 3 (Hypothesis Testing)

**Current text (paraphrased):**
> Form Single Hypothesis -- State clearly: I think X is the root cause because Y. Be specific. Test minimally. One variable at a time.

**Replace with:**

> Form Hypothesis -- State clearly in one of two forms:
>
> - **Single-cause:** "Component X is broken because Y." Test by isolating X.
> - **Interaction:** "Components X, Y, Z combine to cause the bug under condition C, even though each works correctly alone." Test by disabling each contributing factor independently -- if the bug *reduces but persists* when you remove one factor, you likely have an interaction bug with multiple contributors.
>
> **Key signal for interaction bugs:** You disabled a suspect (e.g., CDN cache) and the bug *changed* (reduced frequency, different timing) but didn't disappear. This means you found a *contributing factor*, not *the* cause. List all contributing factors before proposing a fix. A partial fix that addresses one factor while leaving others will produce a partial reduction in symptoms, not a resolution.
>
> Be specific. Test minimally. One variable at a time.

### Edit location: Phase 1 (Gather Evidence in Multi-Component Systems)

**Current text (paraphrased):**
> For EACH component boundary: log what enters, log what exits, verify environment, check state. Run once to find WHERE it breaks.

**Append after existing text:**

> If no component boundary shows incorrect behavior, check for interaction effects: map every layer that can serve stale/cached/delayed data between the write path and the read path. For each layer, note its staleness window (cache TTL, replication lag, retry delay). If multiple layers have overlapping staleness windows, the user-visible staleness is the *longest* of these windows, not any single one. This is the signature of an interaction bug.

## 3. Why This Edit Prevents the Failure Class

The agent failed at two specific decision points:

1. **Phase 1 ended prematurely.** The agent found each component correct and moved on. The new Phase 1 text adds a specific follow-up when no single component is broken: map staleness/caching layers and their windows. In the test scenario, this would have surfaced three layers (replica lag 1-3s, CDN cache 5s, service worker cache) before hypothesis formation even begins.

2. **Phase 3 misinterpreted a partial result.** When disabling CDN didn't eliminate the bug, the agent had no framework for "contributing factor vs. sole cause." The new Phase 3 text names this exact signal ("bug changed but didn't disappear") and prescribes the response: list all contributing factors before fixing. The agent would have been directed to continue testing replica lag and service worker caching as additional factors.

The edit is structural, not scenario-specific. It adds a *category of hypothesis* (interaction) alongside the existing category (single-cause). This applies to any bug where multiple correct components produce incorrect emergent behavior: cache stampedes, race conditions, cascading timeouts, consistency window overlaps, etc.

**Anti-overfitting check:**
- Would this help on a completely different task? Yes -- any multi-layer caching, consistency, or timing interaction bug. The edit names the pattern class, not the specific scenario.
- Does this add a general step/rule, not task-specific wording? Yes -- "interaction hypothesis" and "contributing factor vs. sole cause" are general debugging concepts.
- Does it avoid degrading the common case? Yes -- the interaction path only activates when Phase 1 finds no single broken component, so simple bugs still follow the fast path.

## 4. Validation Plan

### Re-run original scenario (regression)

Run the same stale-profile-data scenario against the edited skill. Success criteria: the agent identifies all three contributing factors (replica lag, CDN cache, service worker) and proposes a fix that addresses the interaction, not just one layer.

### Novel interaction-bug scenario (generalization)

Design a new scenario where the bug is an interaction between correct components in a different domain. Example:

> A payment system occasionally double-charges. The payment gateway is idempotent (correct). The retry logic uses exponential backoff (correct). The load balancer has sticky sessions (correct). But when a node fails mid-request, the load balancer routes the retry to a new node, which doesn't share the idempotency key cache with the failed node, so the gateway sees it as a new request. No component is broken. The interaction of failover + per-node idempotency creates the double-charge.

Success criteria: the agent forms an interaction hypothesis rather than blaming a single component.

### Regression on simple bugs (no degradation)

Run 2-3 single-cause bug scenarios to confirm the edit doesn't add overhead or confusion when the bug genuinely has one root cause. The interaction path should not activate when Phase 1 finds a broken component.
