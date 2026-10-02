# Canonical Workflow role evals

Skill-neutral scenarios, written by hand from Perihelion's repos rather than generated from any skill, so no candidate is favoured. Each scenario judges one role; run it once per candidate skill and compare. Reviewed by the Tessl agent for format, judgeability and neutrality (2026-10-02).

| Scenario | Role | Repo at commit | Candidates |
|---|---|---|---|
| `plan-checkout-discount-codes` | Plan | `perihelion-checkout` `a90d88e` | `assistant/agent-workflow#ce-plan`, `storefront/frontend-workflow#writing-plans`, `commerce/checkout-workflow#to-spec` |
| `build-agent-order-status-tool` | Build, test-first | `perihelion-agent` `2cc6157` | `storefront/frontend-workflow#test-driven-development`, `commerce/checkout-workflow#tdd`, `assistant/agent-workflow#ce-work` |
| `resolve-search-api-review` | Resolve review feedback | `perihelion-frontend` `4045eb5` (PR #1 head) | `assistant/agent-workflow#ce-code-review` and `#ce-commit-push-pr` are poor fits; compare `storefront/frontend-workflow#requesting-code-review` and any receiving-review skill the variants gain |
| `remediate-hardcoded-colours` | Remediate a finding | `perihelion-frontend` `48e9ffc` | the build-role candidates |

Each fixture excludes the repo's vendored agent context (`.claude/`, `.agents/`, `.tessl/`, `AGENTS.md` and so on) so only the injected candidate is in play.

Example, one candidate for one role:

    tessl eval run evals/plan-checkout-discount-codes --context assistant/agent-workflow@1.0.0 --skill ce-plan --quality-check

Unverified: whether the eval sandbox can clone these private `perihelionhq` repos.
