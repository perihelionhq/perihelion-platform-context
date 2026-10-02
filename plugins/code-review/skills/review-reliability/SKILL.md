---
name: review-reliability
description: Review a code change for missing error handling on I/O boundaries, unbounded retries, missing timeouts, swallowed errors, resource leaks on error paths, cascading failures, and CI or deploy guards that do not mirror production. Use when reviewing for reliability, failure modes, partial failures, or graceful degradation.
---

# Review lens: Reliability

Review what the change does when a dependency is down, slow, or partly failing, and whether it degrades gracefully or falls over.

## Scope

- **I/O error handling** HTTP calls, database queries, file operations, or message queue interactions with no try/catch or error callback.
- **Retries** Retry loops with no maximum attempts, exponential backoff, or jitter, which turn a temporary blip into a retry storm.
- **Timeouts** HTTP clients, database connections, or RPC calls with no explicit timeout, which hang and consume threads or connections while a dependency is slow.
- **Error swallowing** `catch (e) {}`, `.catch(() => {})`, or handlers that log without propagating, return misleading defaults, or carry on so the caller believes the operation succeeded.
- **Resource leaks on error paths** A connection, file handle, lock, or subscription acquired in the change whose release is not on every exit path (no finally, defer, using, or context manager).
- **Cascading failure** A failure that propagates: aggressive retries overloading a downstream service, or a slow dependency filling queues until health checks fail and restarts cause cold-start storms.
- **Stand-in guard fidelity** A CI gate, smoke test, or deploy dry-run that does not reproduce production's build context, working directory, prepared directories, inputs, and environment, so it can pass while production fails.

## Method

Prioritize I/O boundaries: every external call, write, and acquired resource in the change. Trace what happens when each one fails, hangs, or returns partially, and follow the failure to its callers.

Work out how and where the code runs from the change, the PR description, and nearby docs, not from an assumed production service. For a guard that stands in for production, compare its context and steps with the real thing rather than checking that it runs green.

Where a Release It! stability term fits, name the antipattern (cascading failure, retry storm, integration point without a timeout) or the fix (circuit breaker, bulkhead, fail fast). The term labels the finding; the missing protection you can point to decides it.

What counts as adequate failure handling is often a written project rule. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report a gap when the failure it allows costs something where this code runs: a crashed or wedged service, a caller or user acting on a wrong or missing result, or work left half-done in a way a rerun does not repair. If the protection may come from framework defaults or middleware outside the change, say so.

Do not report internal pure functions with no I/O, error handling in test helpers or fixtures, error message wording, cascades that need several specific conditions to occur together, or concerns that are architectural and cannot be confirmed from the change.

## Reporting

- Point to the line missing the protection and the I/O or resource it guards.
- Name the fault that triggers the failure and what breaks when it does.
- State what would contain it: the timeout, the retry bound and backoff, the release on the error path, the propagated error, the matching guard context.
