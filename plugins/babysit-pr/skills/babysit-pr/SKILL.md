---
name: babysit-pr
description: >
  One iteration of PR babysitting from the author's side, designed to be
  repeated by a loop: read the PR's state, address the reviewers' open
  feedback and failing CI, push, and end the loop once every review thread is
  resolved and every CI check has passed.
argument-hint: "[PR number or URL; defaults to the current branch's PR]"
disable-model-invocation: true
license: MIT
metadata:
  tags: "PullRequest, Review, CI, Loop, Workflow"
  category: "workflow"
---

# babysit-pr

Run exactly one iteration, then report the outcome. The loop calls this again;
do not poll or sleep inside an iteration. Keep no state outside GitHub except
what a loop session keeps in conversation: how many fix attempts each piece of
feedback or failing check has had, and which review-body comments you already
answered. You are the PR's author, not its reviewer: never review the diff
yourself or post findings of your own.
The target is the PR number or URL supplied with the invocation, otherwise the
PR of the current branch. Do the work yourself; no subagents.

## 1. Read state

```bash
gh pr view <pr> --json number,state,isDraft,headRefName,headRefOid,mergeable,isCrossRepository,reviews,comments
gh pr checks <pr>
gh api graphql --paginate -f query='query($o:String!,$r:String!,$n:Int!,$endCursor:String){repository(owner:$o,name:$r){pullRequest(number:$n){reviewThreads(first:100,after:$endCursor){pageInfo{hasNextPage endCursor} nodes{id isResolved isOutdated path line comments(first:20){nodes{author{login} body url}}}}}}}' -F o=<owner> -F r=<repo> -F n=<number>
```

`--paginate` follows `pageInfo` until every review thread is fetched; the output
is one JSON object per page, so read all of them before deciding the outcome.

- `state` is `MERGED` or `CLOSED`: outcome **STOP**.
- The checked-out branch is not `headRefName`, or the PR is cross-repository:
  outcome **BLOCKED** (you cannot push fixes safely).
- `mergeable` is `CONFLICTING`: outcome **BLOCKED** unless the user asked you
  to resolve conflicts.

## 2. Address feedback and CI

Handle, in this order:

1. **Open review threads** (`isResolved: false`). For each, decide whether the
   reviewer is right.
   - Right: change the code, then reply with the fixing commit SHA and resolve
     the thread.
   - Wrong or unclear: reply with the concrete reason or question and leave the
     thread open. Never resolve a thread you did not address.
   - Outdated and already satisfied by the current code: resolve it.
2. **Requests in review bodies and PR comments** that are not attached to a
   thread (for example a review's summary comment). They cannot be
   resolved, so fix the code and answer once with a comment giving the commit
   SHA; do not answer the same comment again in later iterations. Ignore
   approvals, thanks, and bot summaries that ask for nothing.
3. **Failed checks.** Read the failing job's log (`gh run view <run-id> --log-failed`),
   find the cause, and fix it. Reproduce locally when a command exists. Rerun a
   failed job without a code change only after the log points to an
   infrastructure cause, and say what in the log showed that.

Commit only files you changed, with a normal commit message, and `git push`
to the PR branch. Never force-push, never amend pushed commits, never touch
unrelated working-tree changes.

If the same thread, request, or check still fails after 2 fix attempts,
stop fixing it: outcome **BLOCKED** with the evidence.

## 3. Decide the outcome

Evaluate after the fixes. A push starts new CI, so a push this iteration
means the checks are not yet complete. A pending re-review request is not a
reason to WAIT.

| Outcome | Condition | Loop action |
| --- | --- | --- |
| **DONE** | No unresolved threads, no unanswered requests, and every check has concluded successfully (skipped and neutral count as passing). Do not wait for reviewers who have not responded yet | End the loop |
| **WAIT** | Nothing left to fix, but a check is still queued or running, or a push just happened | Continue: the loop runs again |
| **BLOCKED** | A rule above says so, or a reply awaits a human answer while nothing else is left to do | End the loop |
| **STOP** | PR merged or closed | End the loop |

Ending the loop means: in a self-paced loop, do not schedule another
iteration; in a fixed-interval loop, cancel the recurring task with the
client's loop facility. For **WAIT**, prefer a next check about 1-2 minutes
out while checks are running.

## 4. Report

Report the outcome, what changed this iteration (commits pushed, threads
resolved), and what is still pending, so the loop's reader can scan it at a
glance. For **BLOCKED**, state exactly what a human must decide. Nothing else.
