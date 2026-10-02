# platform-team/code-review

Review lenses for Tessl Code Review, derived from the reviewer personas in Every's [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) `ce-code-review` skill (v3.30.1, MIT; see `THIRD-PARTY-LICENSE`).

Each persona is rewritten as a Tessl lens: Scope (what the persona hunts for), Method, Threshold (its confidence calibration and "what you don't flag", without CE's numeric anchors or severities) and Reporting. CE's orchestration, JSON findings schema and output format are dropped.

| Lens | Derived from |
| --- | --- |
| `review-adversarial` | `adversarial-reviewer` |
| `review-agent-native` | `agent-native-reviewer` |
| `review-api-contract` | `api-contract-reviewer` |
| `review-correctness` | `correctness-reviewer` |
| `review-data-migration` | `data-migration-reviewer` |
| `review-frontend-races` | `julik-frontend-races-reviewer` |
| `review-maintainability` | `maintainability-reviewer` |
| `review-performance` | `performance-reviewer` |
| `review-project-standards` | `project-standards-reviewer` |
| `review-reliability` | `reliability-reviewer` |
| `review-security` | `security-reviewer` |
| `review-swift-ios` | `swift-ios-reviewer` |
| `review-testing` | `testing-reviewer` |

Not converted: `learnings-researcher`, `previous-comments-reviewer` and `deployment-verification-agent`, which gather context or write checklists rather than report findings on a diff.

## Using the lenses

Reference them from a repository's `.tessl-code-review.yml`. A review takes at most 8 lenses, so pick per repository and narrow with `globs`:

```yaml
schemaVersion: 1
lenses:
  - ref: platform-team/code-review@1.0.0#review-correctness
  - ref: platform-team/code-review@1.0.0#review-security
  - ref: platform-team/code-review@1.0.0#review-frontend-races
    globs: ['components/**', 'pages/**']
```

Hosted review currently fails when a profile references a lens from a private registry plugin. Until that is fixed, copy the lens into the repository and reference it by path (`ref: ./review-lenses/review-correctness/SKILL.md`).
