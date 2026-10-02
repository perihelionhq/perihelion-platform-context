---
name: review-correctness
description: Review a code change for off-by-one and boundary errors, null propagation, changed sentinel meanings, tooling and provisioning drift, race conditions, invalid state transitions, React effect cleanup gaps, and broken error propagation. Use when reviewing for logic bugs, behavioral correctness, or edge cases that tests miss.
---

# Review lens: Correctness

Review a change by mentally executing it, tracing inputs through branches and state across calls, to find bugs that pass tests because nobody tested that input.

## Scope

- **Boundaries** Loop bounds that skip the last element, slices that take one too many, pagination that misses the final page when the total is an exact multiple of the page size.
- **Null propagation** A function returns null on error, the caller does not check, and downstream code dereferences it; or an unguarded optional field becomes `"undefined"` in a string or `NaN` in arithmetic.
- **Sentinel meaning** A new return path that reuses an existing `null`, `undefined`, empty array or object, or fallback enum, so one value now means several states. Check call sites and user-visible rendering, metrics and actions for the new path; not crashing is not enough if the message or action is false.
- **Tooling and provisioning** In shell and setup scripts, CI, agent config, generated shims and provisioner tests: `PATH` and exported variable propagation, child and background process inheritance, consistency between paired local and cloud fallbacks, quoting and interpolation in generated scripts, and docs or config lists that drift from the executable source of truth. For a check, build or deploy step, confirm it reproduces the real thing's build context, working directory, prepared directories and environment, not merely that it runs.
- **Races and ordering** Operations that assume sequential execution but can interleave, shared state without synchronization, async completion order that matters but is not enforced, and time-of-check-to-time-of-use gaps.
- **State transitions** A state machine that can reach an invalid state, a flag set on success but not cleared on error, partial updates that change some fields but not related ones, and half-updated state after an error.
- **React effect lifecycle** When a change moves a component's mount point, alters cleanup, or manages a third-party script or global, enumerate every `useEffect` exit path and check that each mutation before return has matching cleanup, including "already loaded" guards, early returns after touching `window`, script injection, listeners, timers, and DOM append and remove pairs.
- **Error propagation** Errors swallowed, re-thrown without context, mapped to the wrong handler, or masked by fallbacks, such as returning an empty array so the caller reads "no results" instead of "query failed".

## Method

Trace boundary math with concrete values at the edges. Follow each changed return value to its callers and each mutation to its cleanup.

## Threshold

Report bugs you can trace from an input, through the branch it takes, to the line that produces the wrong result, where a normal caller will hit it. Report a bug that depends on a condition you cannot confirm, such as whether a caller outside the change passes null, only when it is severe.

Do not report bugs that need timing, input shapes or external state you have no evidence for. Do not report style, naming, or missing optimizations in correct code. Do not report duplicate `PATH` exports or repeated environment setup unless they change process resolution, shadow an executable, or make paired scripts behave differently. Do not suggest null checks for values that cannot be null on the current path.

## Reporting

- Give the trace: the input, where it enters, the branch it takes, and the line that produces the wrong result.
- State the wrong outcome a caller or user sees.
- State the fix: the corrected bound, the guard, the cleanup, or the richer return shape that keeps states distinct.
