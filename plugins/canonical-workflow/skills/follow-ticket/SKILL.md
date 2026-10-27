---
name: follow-ticket
description: >-
  Keep the Canonical Workflow's ship-ticket and land-pr runs in step with what
  people do: cancel runs when a person takes an issue back, close the pull
  request when an issue ends, wake the issues it was blocking, wake an issue
  when a person answers Tessl's question, and cancel runs when a Tessl pull
  request closes. Works with Linear and with GitHub Issues.
schemas:
  inputs:
    $ref: ./schemas/inputs.schema.json
---

# Follow ticket

A person did something to an issue or pull request that Tessl is working on.
You read what happened, cancel runs, close a pull request, post a short note or
wake an issue, and stop. You never change code. Each round is a fresh agent
with no memory of the last; the issue, the pull request and the runs are the
only record.

## Your inputs

- `.cloud-launch/inputs.json`:
  - `TICKET_CHANGE` names what happened: `taken-back`, `issue-ended`,
    `replied` or `pr-closed`.
  - The **subject input**, when given, names what the change concerns:
    `LINEAR_ISSUE_IDENTIFIER` (such as `ABC-123`) or `GITHUB_ISSUE_NUMBER`
    for an issue, with `taken-back`, `issue-ended` or `replied`;
    `GITHUB_PR_NUMBER` for a pull request, with `pr-closed`. A GitHub issue
    or pull request number refers to this repository.
  - For GitHub issues, the label that hands an issue to Tessl, written below
    as `<issue label>`, is `GITHUB_ISSUE_LABEL`, else `ISSUE_LABEL`, else
    `tessl`.
- `.cloud-launch/trigger-events.json`: a JSON array of `{event, payload}`
  webhooks, oldest first, all about one issue or one pull request, or an
  empty array when no event started the run.
  - Linear issue events (`linear:Issue.update`): the issue is `payload.data`,
    with `identifier`, `url`, and `payload.updatedFrom.stateId` for the state
    it left.
  - Linear replies (`linear:AgentSessionEvent.prompted`): the issue is
    `payload.agentSession.issue`, and the reply arrived at the top-level
    `payload.createdAt`.
  - GitHub issue events (`github:issues.*`, `github:issue_comment.created`):
    the issue is `payload.issue`; its URL is
    `https://github.com/<payload.repository.full_name>/issues/<number>`.
  - GitHub pull request events (`github:pull_request.closed`): the pull
    request is `payload.pull_request`; its URL is
    `https://github.com/<payload.repository.full_name>/pull/<number>`.

Stop with no write and say what was wrong when:

- `.cloud-launch/inputs.json` is missing or malformed, or `TICKET_CHANGE` is
  not one of the four;
- more than one subject input is given, or the one given does not fit
  `TICKET_CHANGE`;
- there is no subject input, and the events file is missing or malformed, or
  the events name no issue or pull request, or more than one;
- there is a subject input, and the events file is malformed or an event
  names a different issue or pull request;
- `GITHUB_ISSUE_NUMBER` names a pull request: `gh issue view` returns a `url`
  that is not this repository's `/issues/<n>`;
- there is a subject input and no events, and the subject is not one the
  triggers start on:
  - any change: a Linear issue not delegated to Tessl;
  - `taken-back`: a Linear issue whose state type is not `backlog` or
    `triage`, or a GitHub issue whose latest `labeled` or `unlabeled` event
    for `<issue label>` is not `unlabeled` (read the events with
    `gh api --paginate repos/<owner>/<repo>/issues/<n>/events`);
  - `issue-ended` or `replied`: a GitHub issue without `<issue label>`;
  - `pr-closed`: a pull request that is still open, is not from a `tessl/`
    branch in this repository, or lacks the `tessl` label;
- there are events, and an event does not fit `TICKET_CHANGE`.

With a subject input, an empty or missing events file is fine: the input names
the issue or pull request. Read a Linear issue's URL from the issue itself; a
GitHub issue's or pull request's URL is this repository's `/issues/<n>` or
`/pull/<n>`.

An issue's **branch** is `tessl/<identifier lowercased>` for Linear (for
example `tessl/abc-123`) and `tessl/issue-<number>` for GitHub. Its **Tessl
pull request** is an open pull request from that branch in this repository
with the `tessl` label.

## What done looks like

Read the issue or pull request as it is now, then take the section for
`TICKET_CHANGE`. In each, take the first situation that applies.

### `taken-back`

A person moved a Linear issue to Backlog or Triage, or removed
`<issue label>` from a GitHub issue.

1. **It is Tessl's again.** Linear: delegated to Tessl in a state of type
   `unstarted` or `started`. GitHub: open, with `<issue label>`. Stop with no
   write.
2. **It is taken back.** Cancel the runs on the issue and on its Tessl pull
   request. Then post the note below, unless Tessl's newest comment on the
   issue is already that note:
   - Linear: `Stopped: this issue left Tessl's queue, so Tessl cancelled its
     runs. To hand it back, delegate it to Tessl and move it to Todo.`
   - GitHub: `Stopped: the <issue label> label was removed, so Tessl
     cancelled its runs. To hand it back, add the label again.`

### `issue-ended`

A person moved a Linear issue to a state of type `completed`, `canceled` or
`duplicate`, or closed a GitHub issue with `<issue label>`.

1. **It is not ended.** Linear: its state type is none of those three. GitHub:
   it is open. Stop with no write.
2. **It is ended.** Cancel the runs on the issue. Then:
   - With exactly one Tessl pull request, read the issue and the pull request
     again. If the issue is still ended and the pull request still open,
     comment `Closed: <issue url> is <state>.` on it and close it. `<state>`
     is the Linear state's name, or `closed` for GitHub. With more than one,
     name them in the run output and close none.
   - Then wake every issue it was blocking that is ready (see Unblocking).

### `replied`

A person replied in Tessl's Linear agent session, or commented on an open
GitHub issue with `<issue label>`.

1. **It is not Tessl's.** Linear: not delegated to Tessl, or its state type is
   not `unstarted` or `started`. GitHub: closed, or without `<issue label>`.
   Stop with no write.
2. **No question is waiting.** Tessl's newest comment on the issue does not
   contain `<!-- tessl-factory:question -->`. Stop with no write.
3. **The question is unanswered.** No comment from a person is newer than the
   question, and for Linear the reply in the event, when there is one, is not
   newer either. Stop with no write.
4. **The question is answered.** Read the issue again, apply 1 to 3 to that
   read, and wake the issue.

### `pr-closed`

A pull request on a `tessl/` branch with the `tessl` label closed.

1. **It merged, or Tessl closed it.** Tessl closed it when it has a Tessl
   comment starting `Closed:`. Cancel the runs on the pull request and stop.
2. **A person closed it without merging.** Cancel the runs on the pull
   request. Read it again; if it is still closed and has no Tessl comment
   starting `Closed without merging`, post `Closed without merging. To try
   again, <how>.` `<how>` comes from the body's `Closes` line: `move ABC-123
   to Todo` for Linear, `remove the <issue label> label from #<n> and add it again`
   for GitHub. With no `Closes` line, leave out the second sentence.

If the record fits no situation in its section, say what you found in the run
output and stop.

## Unblocking

A dependent of the ended issue is **ready** when:

- **Linear:** it is delegated to Tessl, its state type is `unstarted`, and
  none of its blockers has a state type other than `completed`, `canceled` or
  `duplicate`. Its dependents are the ended issue's `relations` of type
  `blocks`; a dependent's blockers are its `inverseRelations` of type
  `blocks`.
- **GitHub:** it is open, has `<issue label>`, names the ended issue in a
  `Blocked by #<n>` or `Depends on #<n>` line of its body, and every issue
  those lines name is closed.

Read each dependent and its blockers just before you wake it. Name each one in
the run output as woken, skipped with the reason, or failed.

## Invariants

1. You never change code, branches, or an issue's text, assignee or delegate.
   Your only writes are run cancellations, the comments and pull request
   closes above, and wakes.
2. You cancel only ship-ticket and land-pr runs: a run whose
   `attributes.config.skillRef` names the skill `ship-ticket` or `land-pr`.
   Every other run, your own and other follow-ticket runs included, is left
   alone.
3. Every comment you post ends with `<!-- tessl-factory -->`. You never post
   the same note twice.
4. A decision to write rests on a read taken just before the write.
5. A failed cancellation or comment is a warning in the run output; carry on
   with the rest. A failed wake is reported and not retried, except that when
   adding `<issue label>` back fails, you try once more, because an issue
   without it has left Tessl's queue.
6. Pass ids, URLs and comment bodies as data: write bodies to a file outside
   the clone, never through shell interpolation.

## Environment facts

- Every run started by an event about an issue or pull request is linked to
  its URL. `tessl launch list --link-type
  <linear_issue|github_issue|github_pr> --link-value <url> --status pending
  --limit 100 --json` lists them in `.data[]`, each with an `id` and
  `attributes.config.skillRef`. `tessl launch cancel <id> --json` cancels one.
  A cancelled run stops where it is; what it pushed stays on the branch, and
  ship-ticket reads it if the issue comes back.
- To wake a Linear issue, update its state with `issueUpdate(id, input:
  {stateId})`: for `replied`, to the team's first state of type `unstarted`
  (lowest `position`); for a dependent, to the state it already has. The
  update is what starts ship-ticket. To wake a GitHub issue, remove
  `<issue label>` and add it again.
- Linear: every call goes to `$LINEAR_API_URL/graphql` with
  `Authorization: Bearer $LINEAR_TOKEN`; never call `api.linear.app`
  directly. A response with `errors` is a failed call. Write comments with
  `commentCreate(input: {issueId, body})`. Linear does not always return the
  latest state at once: an issue that ended seconds ago can still read as
  open, including on a dependent's blocker list.
- GitHub: `gh issue view`, `gh issue comment --body-file`,
  `gh issue edit --add-label/--remove-label`,
  `gh pr list --head <branch> --state open`, `gh pr view`,
  `gh pr comment --body-file` and `gh pr close`.
- The issue label is data, never command text. Read it into a variable with
  `label=$(jq -r '.GITHUB_ISSUE_LABEL // .ISSUE_LABEL // "tessl"' .cloud-launch/inputs.json)` and
  pass it only as `"$label"`, for example
  `gh issue edit <n> --add-label "$label"`.
- A GitHub dependent names its blocker as `#<n>`, so a search finds every
  candidate:
  `gh api --paginate -X GET search/issues -f q="repo:<owner>/<repo> is:issue is:open #<n>" -f per_page=100`.
  Keep the results whose `labels` include one named exactly `$label` and whose
  body has a `Blocked by #<n>` or `Depends on #<n>` line.
- Tessl's own comments end with `<!-- tessl-factory -->`; a question also
  carries `<!-- tessl-factory:question -->`.
- Nothing runs this skill again for the same event. A dependent you skip stays
  where it is until a person moves it.

## Judgment

The webhook is a hint; the record as you read it decides. A comment from a bot
or from Tessl is never an answer; a short reply such as "yes, go ahead" is, and
ship-ticket decides whether it is enough. A blocker that reads open but is the
issue that just ended is a stale read: read it again, up to five times, 10 to
30 seconds apart, before you skip the dependent. A branch with the expected
name from a fork is not this repository's.
