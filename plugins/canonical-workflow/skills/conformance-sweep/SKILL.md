---
name: conformance-sweep
description: Scheduled sweep that checks a repository against its team standards and reports one Tessl finding per standard it drifts from, for the `remediate` action to fix. Use as the skill of the weekly conformance-sweep schedule.
---

# Conformance sweep

Report drift; do not fix it. Each finding raises `tessl:finding.created`, which starts `remediate`, so the fix goes through the same review path as any other factory pull request.

## Inputs

- `TESSL_PROJECT_ID`, supplied as a schedule input: this repository's Tessl project.
- An environment with read access to the repository.

## Procedure

1. **Read the standards.** Read the team standards installed with the team's workflow plugin (its `rules/*-standards.md`) and the `AGENTS.md` or `CLAUDE.md` chain. Only written rules count.
2. **Scan.** Check the default branch against each standard. Skip generated code (`genproto/`, `pb/`), vendored files and agent context (`.claude/`, `.agents/`, `.tessl/`).
3. **Group by standard.** Keep the standards with at least one confirmed violation. A violation is confirmed when you can quote the rule and point to the line.
4. **Deduplicate.** List existing findings with `tessl finding list --project $TESSL_PROJECT_ID --limit 100 --json`, without `--state`, following `links.next` with `--after`. Match on the standard and the files, not on title alone, and view likely matches before deciding. Skip a standard whose matching finding is `ignored`. For one that is `open`, the drift is still there: append this run's evidence as a new observation with `tessl finding update <id> --file changes.json --json`, sending only `observations` (no `loopId` or `runId`), and do not create another.
5. **Report.** For each remaining standard, write a finding and create it with `tessl finding create --project $TESSL_PROJECT_ID --file finding.json --json`:

```json
{
  "schemaVersion": 1,
  "title": "Conformance: <standard, in a few words>",
  "observations": [
    {
      "observedAt": "<now, ISO 8601>",
      "evidence": ["<path>:<line> <what violates the rule>", "..."]
    }
  ],
  "analysis": "<the rule, quoted, and how these lines break it>",
  "recommendation": "<the specific change that makes them conform>",
  "expectedImprovement": "<what conforming gives the team, without inventing a measured benefit>"
}
```

Leave out `loopId` and `runId`; the service derives them from the scheduled run. Each evidence string is one observed fact.

6. **Finish.** If nothing drifts, create nothing. Open no branches, commits or pull requests.
