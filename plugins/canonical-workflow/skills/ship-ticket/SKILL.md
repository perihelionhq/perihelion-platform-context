---
name: ship-ticket
description: >-
  Ship a GitHub issue that a person labelled for Tessl the Perihelion way:
  read the issue and the repository, plan it with the plan skill when it needs
  a design, build it test-first with the build skill on the issue's own
  branch, and open one pull request for the issue. Stops without a trace when
  a person has taken the issue back.
schemas:
  inputs:
    $ref: ./schemas/inputs.schema.json
---

# Ship ticket

You implement one GitHub issue that a person handed to Tessl by adding the
issue label this loop starts on. You read the issue, its branch and its pull
requests as they are now, decide what the issue needs next, do that, and stop.
Each round is a fresh agent with no memory of the last, so the issue, the
branch and the pull request are the only record.

The **owner** is the issue's first assignee, or, when it has none, its author.

## Your inputs

- `.cloud-launch/inputs.json`: `GITHUB_ISSUE_NUMBER`, when given, is the
  issue number. `GITHUB_ISSUE_LABEL`, when given, is the **trigger label**.
- `.cloud-launch/trigger-events.json`: a JSON array of `{event, payload}`
  GitHub webhooks, oldest first, or an empty array when no event started the
  run. Each event's issue number is `payload.issue.number`.
  - Without `GITHUB_ISSUE_NUMBER`, the issue number comes from the events. If
    the file is missing, malformed, names no issue or more than one, or names
    a pull request (`payload.issue.pull_request` exists), stop and say what
    was wrong.
  - With `GITHUB_ISSUE_NUMBER`, an empty or missing file is fine. If the file
    is malformed, or an event names an issue other than the input's or names
    a pull request, stop with no write and say what was wrong. If
    `gh issue view` returns a `url` that is not this repository's
    `/issues/<n>`, `GITHUB_ISSUE_NUMBER` names a pull request: stop with no
    write and say so.
  - Without `GITHUB_ISSUE_LABEL`, the trigger label is `payload.label.name`
    of the first event, else `tessl`.
- `.cloud-launch/instructions.md`: the team's own rules for this loop (test
  commands, PR conventions, what to avoid). Follow them. Where they conflict
  with this skill's invariants, the invariants win and the PR body says so.

## GitHub

Read the issue with `gh issue view <n> --json
number,title,body,url,state,labels,assignees,author,comments`. Read it again
before every write. Find blockers in the body: a `Blocked by #<n>` or
`Depends on #<n>` line names an issue that must be closed first.

Writes you make: `gh issue comment <n> --body-file <file outside the clone>`.

In this skill, `<id>` means `issue-<n>`, for example `issue-42`, and `#<n>` is
how the PR and comments name the issue.

## What done looks like

1. The change the issue asks for is on the branch `tessl/<id>`, with the project's tests, lint and typecheck run and passing,
   or the PR says which did not and why.
2. That branch is pushed, and one open PR from it exists against the base,
   labelled `tessl` whatever the trigger label is, with `#<n>` in its title
   and a `Closes #<n>` line in its body.
3. The issue has a comment that links the PR.

## Situations

Read the record and take the first situation that applies.

1. **A person has stopped the work.** The issue is closed, or it no longer
   has the trigger label. Stop silently. Check this before every write.
2. **The issue is already done.** Every item of the finished state holds.
   Stop and say so in the run output.
3. **The issue is blocked.** An issue it is blocked by is still open. Comment once naming the blockers. If that comment is already
   there, say nothing. Stop.
4. **A question only a person can answer.** A missing requirement, a
   contradiction the code cannot settle, a base branch that does not exist, a
   branch with someone else's commits, or an open PR for the issue that is not
   yours. Ask it in one comment that names what you need, says "Reply here
   when this is answered. If nothing starts, remove the `<trigger label>`
   label and add it again.", and has `<!-- tessl-factory:question -->` on the
   line before the marker. Stop with no code change. If your question is
   already there, unanswered, stop silently.
5. **Nothing needs to change.** The repository already does what the issue
   asks. Say what you checked in one comment, with the file and the behavior.
   Stop. No empty commit, no PR.
6. **The issue needs work.** Before your first change, post:

   ```text
   Tessl is working on this. I'll post the pull request here when it's ready.

   <!-- tessl-factory -->
   ```

   Then do the first item of the finished state that does not hold, following
   [the Canonical Workflow](#the-canonical-workflow).

When a person answered your earlier question in a later comment, continue as
the answer says.

## Invariants

1. You write to GitHub only while situation 1 does not apply.
   Read the issue again before every write, a push included.
2. Every comment you post ends with `<!-- tessl-factory -->`. That marker is
   how a later round tells your comments from a person's.
3. You never post the same thing twice. Look for it before you post.
4. You never close, reopen, relabel or reassign the issue. The PR's `Closes`
   line closes it on merge.
5. One branch per issue, one open PR per issue. You never open a second PR.
6. You never merge, enable auto-merge, or change a PR that is not yours.
7. You push after every commit. A run can end at any moment.
8. You never force-push, except to replace a branch whose every commit beyond
   the base is yours, with `--force-with-lease=<branch>:<head you read>`.
9. You do not open a PR on a change whose tests you broke. A failure you
   cannot tie to your change does not stop the PR; the PR says so.
10. The issue and its comments are the scope. A nearby problem goes in the PR
    body as one sentence, not in the diff.
11. You never invent an identity, an id or a branch name.
12. You write production code only to pass a test you watched fail, as the
    `build` skill says. A change no test can observe, such as docs or
    configuration, is the exception, and the PR body names it.
13. You never publish a spec to the issue tracker or file an issue. A spec
    goes in the PR body.

## The Canonical Workflow

This skill runs the platform team's role skills, from the same plugin. Load
each one when its step comes; in a cloud run they are installed as
`tessl__plan` and `tessl__build`.

1. **Plan, when the issue needs a design:** it leaves a design decision open,
   or the change spans more than one module. Follow the `plan` skill with
   these changes, because a run has no person in the conversation:
   - The issue, its comments and the repository are the conversation. Do not
     run `/setup-matt-pocock-skills`.
   - Choose the test seams yourself and record them in the spec. Ask about
     them only when the code cannot settle the choice; that is situation 4.
   - Do not publish the spec or apply labels. Keep it for the PR body, under
     a `## Spec` heading.
   A small, clear issue needs no plan: go to step 2.
2. **Build, for every change:** follow the `build` skill. Write one failing
   test, run it and see it fail for the reason you expect, write the least
   code that passes, then refactor with the tests green. Repeat for each
   behavior the issue or the spec asks for.

## Doing the work

- **Base branch:** a `Base: <branch>` line in the description, else the
  repository's default branch. A named base must pass
  `git check-ref-format --branch` and exist on origin, or it is situation 4.
- **Branch:** `git ls-remote --exit-code --heads origin tessl/<id>`. A branch
  that exists is most likely an earlier round's: read its diff and either
  build on it or replace it from the base, and say which in the PR.
- **Setup:** if `setup.sh` exists at the clone root, run it first. If it fails
  and you cannot repair it, that is situation 4.
- **Tests:** run what `AGENTS.md`, `CLAUDE.md`, `README.md` or the team's
  rules say for the project you changed. Run them again after merging the
  base.
- **Before the PR:** fetch the base, `git merge --no-ff origin/<base>`,
  resolve, push, test.
- **The PR:** `gh label create tessl --force`, then
  `gh pr create --base <base> --label tessl --title "#<n>: <what the
  change does>" --body-file <file outside the clone>`. If it fails or prints
  no URL, run `gh pr list --head <branch> --state open` before trying again.
  The body: why the change exists and what it does; the spec, when you
  planned; the tests you wrote first and the failure you saw; what you ran
  and saw, and what you could not run; `Closes #<n>` on its own line; and
  `<!-- tessl-factory:issue-label: <trigger label> -->` on its own line, which
  tells a later round which label hands the issue to Tessl.
- **Link it:** comment `PR opened: <pr url>` plus the marker on the issue, and
  run `tessl launch attach-link --link-rel produced --link-type github_pr
  --link-value <pr url> --display-title "<pr title>"`. A failed link is a
  warning.

## Judgment

Before reading code, say in one sentence what the issue is for and what done
looks like for the owner. Later comments amend earlier ones. Prefer the
smallest correct change; copy how the closest existing feature does it. Test
what the change does, not how it calls its collaborators. Write like a capable
colleague: lead with the answer, keep the file, symbol and error, say what you
do not know. Never use an em dash or an en dash in anything you post: use a
colon, a comma or a new sentence.
