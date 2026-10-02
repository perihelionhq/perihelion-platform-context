---
name: shepherd
description: Carry a factory pull request through review by resolving review comments and failed checks on its existing branch, until it is ready for a person to merge or needs one to decide. Use as the action for review, review-comment, `@factory` conversation and failed-check triggers on `factory/` branches.
---

# Shepherd a factory pull request

Answer the feedback on one existing factory pull request: Tessl Code Review findings, comments from people, and failed checks. Push fixes to the same branch. Never open a new branch or a new pull request.

## Inputs

- The triggering events at `.cloud-launch/trigger-events.json`. Several events can be folded into one run; read them all and require that they name one pull request on a `factory/` branch.
- An environment with `githubAccess: write`, for pushes, replies and resolving review threads.

## Procedure

1. **Count attempts.** Every top-level comment this skill posts ends with `<!-- perihelion-factory:shepherd -->`. If the pull request already has three, add `factory:needs-human`, comment that the budget is spent with a summary of what is still open, and stop. Tessl has no run budget of its own, so this count is the only thing that stops a loop between review and fix.
2. **Gather the current state.** Fetch the pull request, its unresolved review threads, its conversation and its check runs. Work from that state, not only from the events; earlier feedback may still be open.
3. **Read the standards.** Read the `AGENTS.md` or `CLAUDE.md` chain and the team standards, as the review lenses do. A comment that cites a standard is judged against the standard's text.
4. **Resolve the feedback.** Follow this plugin's `resolve-feedback` skill for each unresolved item: verify it against the code, fix what is right, and decline what is wrong with a reason. For a failed check, read its log, reproduce it locally, and fix the cause the change introduced.
5. **Verify and push.** Run the build, the tests and `tessl change verify` where configured, commit, and push to the existing branch.
6. **Reply.** Reply in each thread you acted on, saying what changed or why nothing did, and resolve the threads you fixed. Post one top-level summary ending with the marker from step 1.

## Stop and hand back

Add `factory:needs-human` and say why, without pushing, when feedback items contradict each other, a fix would go beyond what the issue asked for, or a failed check is infrastructure rather than code. Approving and merging always stay with a person.
