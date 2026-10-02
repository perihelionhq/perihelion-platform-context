---
name: review-performance
description: Review a code change for production-observable performance problems, including N+1 queries, unbounded memory growth, missing pagination, hot-path allocations, and blocking I/O in async contexts. Use when reviewing for runtime performance, query efficiency, memory use, or scalability at expected load.
---

# Review lens: Performance

Review what happens when the change runs ten thousand times or against a table with a million rows, focusing on measurable problems rather than micro-optimizations.

## Scope

- **N+1 queries** A database query inside a loop that should be one batched query or an eager load.
- **Unbounded memory growth** Loading a whole table or collection without pagination or streaming, caches without eviction, and string concatenation in loops building unbounded output.
- **Missing pagination** Endpoints or fetches that return all results without a limit, cursor, or stream.
- **Hot-path allocations** Object creation, regex compilation, or expensive computation inside a loop or per-request path that could be hoisted, memoized, or precomputed.
- **Blocking I/O in async contexts** Synchronous file reads, blocking HTTP calls, or CPU-heavy work on an event-loop thread or async handler that stalls other requests.

## Method

Find the loops and per-request paths the change touches, and what each iteration does.

Count iterations against the expected data size: a loop over three config items is not an N+1. For an unbounded fetch, trace whether the consumer handles the full result set or will run out of memory on large data.

Expected scale and which tables are large are often written down. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down, along with any schema or docs that describe data size.

## Threshold

The bar is higher than for other lenses: performance problems are easy to measure and fix later, and false positives waste time on premature optimization. Report only problems real users will hit under normal load, where the loop and the per-iteration cost, or the blocking call on the async path, are visible in the code. Drop findings that depend on data sizes you cannot confirm.

Do not report micro-optimizations in cold paths such as startup, migrations, admin tools, or one-time initialization; caching suggestions without evidence the uncached path is slow or frequent; scale concerns beyond the expected near-term load of clearly early-stage code; or style-based opinions such as `for` versus `forEach` or `Map` versus a plain object.

## Reporting

- Name the loop or path, what runs per iteration or request, and what scales it.
- State the realistic data size or call rate and why it applies here.
- Give the fix: the batched query, eager load, limit or cursor, hoisted computation, or non-blocking call.
