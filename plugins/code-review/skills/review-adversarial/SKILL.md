---
name: review-adversarial
description: Review a code change for violated assumptions, cross-component composition failures, multi-step failure cascades, abuse through normal use, and verification mechanisms that can pass while production fails. Use when reviewing for adversarial failure scenarios, emergent misbehavior, or green-while-red CI, test, and deploy guards.
---

# Review lens: Adversarial

Review a change by constructing specific scenarios that make it fail: if this happens, then that happens, which causes this to break.

## Scope

- **Assumption violation** What the code assumes about data shape (an API always returns JSON, a config key is set, a list is never empty), timing (an operation finishes before a timeout, a lock is held for the whole block), ordering (initialization completes before the first request) and value ranges (IDs are positive, strings non-empty, counts small), and the input or condition that breaks each one.
- **Composition failures** Components that are each correct alone but fail together: a caller passing a value the callee does not expect, two components writing the same row, cache key or global without coordination, A assuming B has already run with nothing enforcing it, A throwing an error type that B does not catch.
- **Cascades** Multi-step failure chains: a timeout that triggers retries that cause more timeouts, partial data that one component writes and another acts on, recovery paths that create new failures such as a retry that duplicates, a rollback that orphans state, or a circuit breaker that blocks recovery.
- **Abuse cases** Legitimate-seeming use that misbehaves: the same action submitted rapidly or the thousandth time, requests that arrive mid-deploy or between cache invalidation and repopulation, two users or processes mutating or claiming the same resource, inputs at the maximum size, the exact rate limit, or valid but nonsensical values.
- **Silent-pass verification** When the change is a guard standing in for the real thing (a CI or merge-blocking check, build or deploy step, coverage or lint gate, test harness or mock), the scenario where it goes green while production is red: a different build context, working directory, prepared directories, environment or command sequence than production, a mock that removes the path that actually breaks, or an assertion on a proxy instead of the real output.

## Method

Scale depth to the change. For a small diff without risk signals, check two or three environmental assumptions. For a larger diff, add composition failures and abuse cases. For a large diff or one touching authentication, authorization, payments, billing, data migration or backfill, external APIs, webhooks, cryptography, sessions, personal data or compliance, also construct cascades and make several passes over complex interaction points.

Treat any change to a verification mechanism as high risk regardless of size, and always check its fidelity to the thing it protects.

For each assumption or interaction, construct the concrete input or condition, then trace it through the code to its consequence.

## Threshold

Report scenarios you can construct step by step from the change and the surrounding code. Report a scenario that depends on one condition you can see but cannot confirm, such as a real timing window or an external format, only when the consequence is severe.

Do not report speculation about runtime state, cascades without traceable steps, or failures that need several unlikely conditions at once.

Do not report single-pattern issues that stand alone: an isolated logic bug, a known vulnerability class, missing error handling on one I/O boundary, a performance anti-pattern, style, test coverage gaps, API contract breakage or migration safety. This lens covers what emerges from combinations, assumptions and sequences. A test harness or mock that could mask a production failure is in scope.

## Reporting

- Title the finding by the constructed failure, not the pattern: "Cascade: payment timeout triggers unbounded retry loop", not "Missing timeout handling".
- Give the scenario step by step: the trigger, the execution path, and the failure state it ends in.
- Describe a concrete fix only when you have one; otherwise present the finding as a risk for a person to judge.
