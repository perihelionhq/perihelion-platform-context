# platform-team/canonical-workflow

The workflow every Perihelion team converges on for the factory. It has two layers:

* **Role skills** do the engineering. Each role is filled by **slotting** the strongest skill from the team workflows, chosen with the skill-neutral evals in `evals/`, then improved with `tessl review fix` against the same evals. The names are fixed, so the actions do not change when a slot does.
* **Action skills** are thin wrappers that Tessl triggers run. They read the triggering event, call the role skills, and do the GitHub and Tessl bookkeeping.

## Role skills (to be slotted)

| Skill | Role | Eval |
| --- | --- | --- |
| `plan` | Plan a change from an issue | `plan-checkout-discount-codes` |
| `build` | Build test-first from a plan | `build-agent-order-status-tool`, `remediate-hardcoded-colours` |
| `resolve-feedback` | Verify and resolve review feedback | `resolve-search-api-review` |

Review isn't a role here. Tessl Code Review runs the lenses from `platform-team/code-review` on every pull request.

## Action skills

| Skill | Started by | Does |
| --- | --- | --- |
| `implement` | `github:issues.labeled` (`factory`); `@factory` on an issue | issue → `plan` → `build` → pull request on `factory/<n>-<slug>` |
| `shepherd` | review, review comment, `@factory` on the PR, failed check, on `factory/` branches | `resolve-feedback` on the same branch; three attempts, then `factory:needs-human` |
| `close-out` | `github:pull_request.closed`, merged, `factory/` branch | summarises and closes the issue; links the finding to the merged PR |
| `remediate` | `tessl:finding.created` from the conformance-sweep loop | finding → `build` → pull request on `factory/finding-<id>` |
| `fan-out` | tag `push` in perihelion-protos | opens a `factory` issue in each `demo.proto` consumer |
| `conformance-sweep` | weekly schedule (`TESSL_PROJECT_ID` input) | reports one finding per drifted standard; opens no PRs |

Conventions the actions share:
* Events are read from `.cloud-launch/trigger-events.json`.
* Factory branches start with `factory/`, which is what the shepherd and close-out triggers filter on.
* `factory:needs-human` hands the work to a person.
* A person always merges.

## Open questions

* Can a run started by a GitHub event read and link findings (`close-out`)? The findings CLI reference only grants linking to the producing run and to runs started by `tessl:finding.created`.
* Who resolves a finding once its fix merges? Run keys can't change state, so for now the sweep keeps adding observations while drift remains, and a person resolves it.
* Does `fan-out`'s organisation-wide issue access need its own environment, or can it be minted?
