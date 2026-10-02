---
name: review-testing
description: Review a code change for untested branches and error paths, tests that do not assert behavior, mirror tests that miss the source of truth, test-only production seams, duplicate coverage, brittle or nondeterministic tests, and behavior changes with no test changes. Use when reviewing tests, test coverage, test quality, or whether the tests prove the code works.
---

# Review lens: Testing

Review whether the tests in a change prove the code works, separating tests that catch real regressions from tests that give false confidence.

## Scope

- **Untested branches** New `if/else`, `switch`, `try/catch`, or conditional logic that changes behavior with no test exercising it.
- **Lifecycle branches** "Already loaded" guards and early returns after setup or global mutation in code that adds effect cleanup, script loading, event listeners, timers, or DOM append and remove, not only the production and non-production happy paths.
- **Sentinel semantics** An existing sentinel (`null`, `undefined`, empty array or object, fallback enum) reused for a new meaning, without tests that consumers render, log, measure, or act on it truthfully rather than merely not crashing.
- **Mirror tests** A test comparing a file to a hardcoded expected array or fixture without checking the executable source of truth. If the source script changes but the expected array does not, does the test fail?
- **No behavioral assertion** Tests that would pass with the code broken: asserting only no throw or truthiness; expected values computed by the helper under test; mocks or fixtures supplying the result, ordering, or side effect the code should produce; negative cases rejected by a different guard than the one named.
- **Test-only seams** An export, flag, wrapper, global, or injection hook no production caller uses, added so a test can reach an internal the real entry point could have exercised. Seams for time or randomness are exempt.
- **Duplicate coverage** A new test asserting a contract an existing test already owns, at another layer or as a near-copy, with no distinct risk such as a transport or lifecycle failure.
- **Brittle tests** Exact mock call counts, tests of private methods, snapshots of internal data structures, or order assertions where order does not matter.
- **Nondeterminism** Sleeps, `Date.now` without a fake clock, real network, shared mutable fixtures or module state, or reliance on test run order.
- **Error paths** Catch blocks, error returns, or fallback branches with no test that the error path fires correctly.
- **Behavior change without tests** New branches, state mutations, changed API contracts, altered control flow, or error behavior with zero test files added or modified. Formatting, comments, type-only annotations, and metadata that does not alter runtime behavior are excluded.

## Method

Trace each new branch and error path in the change to at least one test that exercises it. For each test, ask whether it would fail if the code under test were broken; where Kent Beck's test desiderata fit, name the one violated (behavior-sensitive, structure-insensitive, deterministic, isolated).

If you mutate production code to check whether tests catch it, do so only in an isolated worktree or scratch copy whose HEAD equals the reviewed commit and that includes any staged or unstaged changes under review. Never mutate a shared checkout.

Required coverage and test conventions are often written project rules. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report gaps a normal future code path will hit. When you infer a gap from file structure or naming alone, such as a new module with no matching test file, say that coverage may exist elsewhere, for example in an integration test.

Do not report missing tests for trivial getters and setters, test style preferences (`describe/it` vs `test()`, AAA, co-location), coverage percentage targets, missing tests for code the change did not touch unless the change makes it riskier, or gaps that depend on test infrastructure you cannot see.

## Reporting

- Name the untested branch or the weak test, with its location.
- For a weak test, state the break it would let through or the specific dependency that makes it flaky.
- State what would fix it: the case to add, the assertion to tighten, the existing test to extend or table-drive, the production entry point to test through.
