---
name: review-maintainability
description: Review a code change for structural regressions that make code harder to change, delete, or reason about, including complexity moved rather than removed, missed simplifications, shared-path special cases, oversized files, wrong-layer logic, thin wrappers, premature abstraction, dead code, coupling, vague names, data-locality smells, stale comments, and type-safety holes. Use when reviewing for maintainability, simplicity, code structure, or design quality.
---

# Review lens: Maintainability

Review whether the change makes the codebase harder to change, delete, or reason about, and whether it deletes complexity rather than rearranging it.

## Scope

- **Complexity moved, not removed** The same logic spread across more files, helpers, or modes without reducing the concepts a reader must hold; or a simpler reframe that would eliminate whole branches, flags, wrappers, or orchestration layers.
- **Spaghetti growth** Ad-hoc conditionals, one-off booleans, or feature checks bolted into shared paths instead of a dedicated abstraction or policy object.
- **File-size regression** A touched file pushed past 1000 lines by this change, or one already over 1000 lines growing materially without being split.
- **Wrong layer** (*Information Leakage*) Feature-specific behavior in general-purpose modules, a bespoke helper duplicating an existing canonical utility, or implementation details exposed through a public API.
- **Thin wrappers** (*Pass-Through Method*, *Shallow Module*) Pass-through helpers, identity abstractions, or generic handlers that add indirection without clarity; more than two delegation hops to reach logic.
- **Premature abstraction** (*Speculative Generality*) Interfaces with one implementor, factories for one type, extension points with no consumers, base classes with a single subclass.
- **Dead code** Commented-out code, unused exports, unreachable branches, compatibility shims for unreleased paths.
- **Coupling** Circular dependencies, shared mutable state, imports of another module's internals.
- **Vague names** (*Vague Name*, *Mysterious Name*) `data`, `handler`, `process`, `manager`, `utils` as standalone names; booleans without `is`, `has`, or `should`.
- **Comments** New comments that restate the adjacent code; comments in sibling helpers that now falsely claim "same behavior" or "all other cases are identical" after one branch changed.
- **Data locality** *Feature Envy*, *Data Clumps*, *Primitive Obsession* (a raw string or number newly carrying domain rules), and *Repeated Switches* (another branch-set over a discriminator already switched on elsewhere), only when this change introduces or worsens the shape.
- **Type holes** New `any`, `@ts-ignore`, unchecked or `unknown as Foo` casts, un-narrowed nullable flows where the invariant is knowable, and loosely typed records where a shared contract would simplify control flow.

## Method

Prioritize structural simplification over classic smells. For each new branch, helper, or layer, ask what it would take to delete it and whether behavior would change.

When a change adds a branch to one helper in a paired classifier or mapper flow, read the sibling helpers and their comments. Where a canonical name from Ousterhout or Fowler applies, use it alongside the evidence; the detection condition, not the name, decides whether to report.

What counts as the canonical utility, the right layer, or an acceptable file size is often a written project rule. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report regressions visible in the change: a wrapper with no added behavior, a special case in a busy shared function, indirection that reduces no concepts, a cast bypassing a check you can point to, or a data-locality smell where every occurrence can be quoted. Do not report naming or boundary judgments you cannot ground in the code.

Do not report complexity that mirrors genuine domain rules, abstractions with several real consumers, framework-mandated patterns, formatting or naming taste, design philosophy without a concrete structural fix, or lookup tables and registries requested only because more cases might arrive later.

## Reporting

- Quote the shape: the wrapper, the duplicated helper and its canonical counterpart, every occurrence of the repeated switch or clump.
- Name the cost to the next person who changes this code.
- Give a concrete reframe: what to delete, split, move, or collapse, not "consider refactoring". For a comment that repeats code, suggest deleting it.
