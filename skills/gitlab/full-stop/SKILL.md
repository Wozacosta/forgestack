---
name: full-stop
description: Trigger this if the user says 'full stop', even as a closing "Full stop." that reads like emphasis.
---

# Full Stop

Do the same work as `full-send`, but stop with the MR ready for the user to merge.
Never merge it.

Read [full-send](../full-send/SKILL.md) and follow its workflow, proof rules,
review, and babysitting steps. An explicit "full stop" request authorizes that
work without a separate "full send" request. That includes full-send's
testing permission. A "full stop" request is the permission for the test
methods Proof needs, even when a project file says to ask first, for
example before computer use. Use those methods directly. Its merge
permission does not apply. The overrides below take precedence.

## No merging

You may commit, push, rebase (locally or with `glab mr rebase`), fix conflicts,
and use `--force-with-lease` on this task's branch. Never run `glab mr merge`
(it sets auto-merge by default while a pipeline runs), set auto-merge, add the
MR to a merge train, push to the target branch, bypass approval rules or
protected branches, or delegate those actions.

Cancel existing auto-merge and remove the MR from any merge train before work
begins; check again before each push. Report these changes. If you cannot
prevent automatic merging, report the blocker before proceeding.

- Cancel auto-merge:
  `glab api -X POST projects/:id/merge_requests/<iid>/cancel_merge_when_pipeline_succeeds`
- Leave a merge train:
  `glab api -X DELETE projects/:id/merge_trains/merge_requests/<iid>`
- Check: `glab mr view <iid> -F json` shows `merge_when_pipeline_succeeds: false`.

## Stop at the gate

Use full-send's merge gate as the readiness check. Replace its merge and
final report steps with a handoff. Proof and review must cover the final diff;
recheck behavior changes and meaningful conflict fixes before handoff.

Fetch the target branch and check the live MR's head, target, pipeline,
threads, approvals, and `detailed_merge_status`. Confirm it is open, not a
draft, and not set to merge without the user. If someone else merged it, report
that instead.

- **Ready to merge:** the merge gate passes. Leave the MR open and unmerged.
- **Waiting for required approval:** only human approval remains
  (`detailed_merge_status` is `not_approved`). Confirm that this is the actual
  blocker.
- **Blocked:** explain what prevents readiness, what you tried, and what is
  needed. Follow full-send's rules for asking the user.

Stop any `mr-wait` you started before the handoff or a question.
Report the MR link, final head, verdict, proof, reviewer outcome, and remaining
risks. State whether the MR is still open and unmerged.
