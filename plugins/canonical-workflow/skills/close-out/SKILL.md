---
name: close-out
description: Close the loop after a factory pull request merges by summarising it on its issue, closing the issue, and recording the remediated finding's pull request. Use as the action for a `github:pull_request.closed` trigger where the pull request merged from a `factory/` branch.
---

# Close out a merged factory pull request

Finish the bookkeeping for one merged factory pull request, so that the issue, and the finding when there is one, say what happened.

## Inputs

- The triggering event at `.cloud-launch/trigger-events.json`. Require one `pull_request.closed` event with `payload.pull_request.merged` true and a head branch starting `factory/`; otherwise stop without changes.
- An environment with `githubAccess: write`.

## Procedure

1. **Find the work item.** From the pull request body, read the issue it closes (`Closes #<number>`) and any finding it remediates (`Finding: <id>`). A pull request can have either or both.
2. **Summarise on the issue.** Comment on the issue with the merged pull request, the merge commit, what changed in one or two sentences, and anything the pull request deferred. Close the issue if GitHub has not already closed it, and remove the `factory` and `factory:needs-human` labels.
3. **Record the finding.** For a remediated finding, read it with `tessl finding view <id> --json`. If it has no `pull_request` link matching this pull request, add one with `tessl finding link add <id> --type pull_request --url <url> --json`. Leave the finding's state alone; the next conformance sweep confirms whether the drift is gone.
4. **Note deferred work.** If the pull request or its review deferred work, open one follow-up issue that lists it and links back, without the `factory` label. A person decides whether it goes to the factory.

## Do not

Do not reopen, revert or edit the merged code, and do not post more than one summary comment per issue.
