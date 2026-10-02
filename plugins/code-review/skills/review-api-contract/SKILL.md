---
name: review-api-contract
description: Review a code change for breaking changes to public interfaces, breaking changes shipped without versioning or migration paths, inconsistent error shapes, undocumented behavior changes, overloaded sentinel values, and backward-incompatible type changes. Use when reviewing for API contract stability, backward compatibility, or consumer impact.
---

# Review lens: API Contract

Review a change through every consumer of the current interface: what breaks when a client sends yesterday's request to today's server, and whether anyone would know before production.

## Scope

- **Breaking changes** Renamed fields, removed endpoints, changed response shapes, narrowed accepted input, or altered status codes that existing clients depend on.
- **Unversioned breaks** A breaking change shipped without a version bump, deprecation period or migration path. Under SemVer, a break in a stable API (1.0.0 or later) is a major bump; a 0.y.z package follows the project's declared versioning policy.
- **Inconsistent error shapes** New endpoints returning errors in a different format from existing ones, such as `{ error: string }` alongside `{ errors: [{ message }] }`, so clients need per-endpoint error parsing.
- **Undocumented behavior changes** Under Hyrum's Law, every observable behavior is depended on by someone. Watch for a field whose meaning silently changes (`count` used to include deleted items and now does not), changed defaults, and shifted sort order.
- **Sentinel overloads** A new `null`, `undefined`, empty collection or object, or fallback enum value that reuses an existing value for a new state, so clients cannot tell "no data" from "data exists but cannot be summarized".
- **Incompatible type changes** A return type widened (`string` to `string | null`) without updating consumers, an input narrowed (any string to UUID only), or a field switched between required and optional.

## Method

Trace each change to the interface and classify it as additive, which is safe, or subtractive or mutative, which breaks.

For sentinel and semantic changes, audit the visible consumers for how they interpret the value, not only whether it still type-checks.

What counts as the public interface and how breaking changes are versioned is often a written project rule. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Report contract changes you can point to on a specific line. Report a change that depends on how consumers use the API, such as a field whose meaning changes while its type stays the same, only when the impact is severe.

Do not report internal refactors behind an unchanged interface, naming style preferences unless they are inconsistent within the same API, slower responses, or additive changes such as new optional fields, new endpoints, or new query parameters with defaults.

Do not report internal changes when you are only guessing that they surface to consumers.

## Reporting

- Name the line where the contract changes and the consumers it affects.
- State what an old client now gets: an error, wrong data, or a value it cannot distinguish from another state.
- For an undocumented behavior change, name Hyrum's Law in the title, but base the finding on the observed change in meaning.
- State the fix: a version bump, deprecation period, migration path, richer response shape, or explicit discriminator.
