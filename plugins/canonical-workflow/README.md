# platform-team/canonical-workflow

The workflow every Perihelion team converges on. It holds the **role skills**: the engineering steps the factory's agents follow. Each role is filled by **slotting** the strongest skill from the team workflows, chosen with the skill-neutral evals in `evals/`, then improved with `tessl review fix` against the same evals. The names are fixed, so nothing that calls a role changes when its slot does.

The factory itself runs Tessl's stock actions from `tessl-labs/factory` (`implement-github-issue`, `shepherd`, `follow-ticket-changes`). A repo installs this plugin and points those actions at the role skills through each action's `instructions`.

## Role skills

| Skill | Role | Slotted from | Eval evidence (sonnet-5, 5 runs) |
| --- | --- | --- | --- |
| `plan` | Plan a change from an issue | Matt Pocock's `to-spec`, via `commerce/checkout-workflow` | `plan-checkout-discount-codes`: 100 on every run against a baseline of 92.8, but it activated in only 1 of 5 runs. `ce-plan` scored 11 because it never wrote the plan where the task asked; `writing-plans` timed out on every run. Provisional. |
| `build` | Build test-first | Superpowers' `test-driven-development`, via `storefront/frontend-workflow` | `build-order-status-under-pressure`: 94.4 against a baseline of 78.4, activated 5 of 5. `ce-work` scored 81.2 and `tdd` 69.0. |
| `resolve-feedback` | Verify and resolve review feedback | not slotted | `resolve-search-api-review` was rejected by the quality check for both candidates. The stock `shepherd` handles review feedback until a harder scenario picks one. |

`plan` keeps `to-spec`'s instruction to publish the spec to the issue tracker. Factory runs override that in the action's `instructions`: the spec goes in the pull request, not a new issue.

Review isn't a role here. Tessl Code Review runs the lenses from `platform-team/code-review` on every pull request.

The drafted custom actions (`implement`, `shepherd`, `close-out`, `remediate`, `fan-out`, `conformance-sweep`) were removed when the factory moved to the stock actions. They are in commit `caafac3` if `remediate`, `fan-out` or `conformance-sweep` are wanted later.
