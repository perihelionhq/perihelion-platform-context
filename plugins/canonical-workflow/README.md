# platform-team/canonical-workflow

The workflow every Perihelion team converges on. It has two layers:

* **Role skills** do the engineering. Each role is filled by **slotting** the strongest skill from the team workflows, chosen with the skill-neutral evals in `evals/`, then improved with `tessl review fix` against the same evals. The names are fixed, so nothing that calls a role changes when its slot does.
* **Loop skills** run in the factory. They are our own copies of Tessl's `tessl-labs/factory` skills (0.6.1), renamed so it's clear they aren't the defaults, with the role skills written into their steps rather than suggested through `instructions`.

## Loop skills

| Skill | Copied from | Started by | What we changed |
| --- | --- | --- | --- |
| `ship-ticket` | `implement-github-issue` | the `tessl` label on an open issue | Plans with `plan` when the issue needs a design, builds every change with `build`; the spec goes in the PR body, never the tracker. Invariants 12 and 13 make both rules hard. |
| `land-pr` | `shepherd` | reviews, comments and checks on `tessl/` branches | Code fixes follow `build`: a test reproduces the finding before the fix. Invariant 14. |
| `follow-ticket` | `follow-ticket-changes` | the label removed, the issue closed, a reply, a Tessl PR closed | Cancels `ship-ticket` and `land-pr` runs; the stock skill only cancels skills named `implement` or `shepherd`. |

The copies keep the stock `tessl/` branches, the `tessl` label and the `<!-- tessl-factory -->` markers, so they work with the stock triggers and with each other. They no longer get fixes from `tessl-labs/factory`: compare against each new release and carry over what applies.

## Role skills

| Skill | Role | Slotted from | Eval evidence (sonnet-5, 5 runs) |
| --- | --- | --- | --- |
| `plan` | Plan a change from an issue | Matt Pocock's `to-spec`, via `commerce/checkout-workflow` | `plan-checkout-discount-codes`: 100 on every run against a baseline of 92.8, but it activated in only 1 of 5 runs. `ce-plan` scored 11 because it never wrote the plan where the task asked; `writing-plans` timed out on every run. Provisional. |
| `build` | Build test-first | Superpowers' `test-driven-development`, via `storefront/frontend-workflow` | `build-order-status-under-pressure`: 94.4 against a baseline of 78.4, activated 5 of 5. `ce-work` scored 81.2 and `tdd` 69.0. |
| `resolve-feedback` | Verify and resolve review feedback | not slotted | `resolve-search-api-review` was rejected by the quality check for both candidates. `land-pr` handles review feedback on its own judgment until a harder scenario picks one. |

`plan` keeps `to-spec`'s instruction to publish the spec to the issue tracker and to check test seams with the user. `ship-ticket` overrides both for factory runs; interactive use still gets the upstream behaviour.

Review isn't a role here. Tessl Code Review runs the lenses from `platform-team/code-review` on every pull request.

The earlier drafted wrappers (`implement`, `shepherd`, `close-out`, `remediate`, `fan-out`, `conformance-sweep`) are in commit `caafac3` if `remediate`, `fan-out` or `conformance-sweep` are wanted later.
