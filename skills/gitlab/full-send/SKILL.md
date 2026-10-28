---
name: full-send
description: Trigger this if the user says 'full send'.
---

# Full Send

If the user triggered this, it's because they are not watching this thread, and
they want you to complete the work end-to-end autonomously. Your updates and
your final message go unread until long after you stop. You own the work from
its current state to a merged MR.

If the user did not explicitly request "full send" in some clear way, do not
assume full autonomy. If they did, everything below applies.

The user grants you permission to:

- Merge the MR once the merge gate passes.
- Force-push the MR's own source branch with `--force-with-lease` after a
  rebase, and run `glab mr rebase` on it.
- Use the test methods that Proof needs, without asking first.

## Testing permission

A "full send" request is the permission. If a project file or other
instruction says to ask before a test method, or to leave testing to the
user, this request replaces that rule for this task.

That includes ask-first rules for computer use, browser automation,
simulators and devices, running the app or a dev server, screenshots, and
visual checks. Use the method that gives direct proof. List it in the
report.

Rules about how to test still apply. Follow rules that name the correct
command, fixture, or check. This permission only removes the ask-first or
leave-it-to-the-user step. If a preferred method cannot prove the change,
use a gated method under this permission.

It covers only the tests needed for this task. It does not add permission
for the steps under "Asking the user", such as production changes or data
deletion. It also does not bypass a harness or system approval prompt if
one appears.

## Steps

1. **Finish the change.** Complete the goal from the user's request with the
   smallest change that works. Fix the case that happened, not every case
   that could. If you are on the default branch, create a branch first.
2. **Prove it.** Get proof for every change you made. See "Proof".
3. **Get feedback.** Use the `sol-loop` skill to verify changes.
4. **File the MR** with the `file-mr` skill. Rebase onto the latest default
   branch first. Add one short line to the description that says how you proved
   the change.
5. **Babysit the MR** with the `babysit-mr` skill. Fix real findings and
   pipeline failures. Get proof for each fix before you push it.
6. **Merge** when the merge gate passes.
7. **Report** in your final message: the MR link, the merge commit, your proof,
   and each decision you made for the user.

Stay active from step 1 to step 7. If you end your turn while pipelines or bots
are pending, the work stops, because nobody will prompt you again. Wait with
`mr-wait` from `babysit-mr`, so something wakes you when the MR changes.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the checks that the repo's pipeline runs (typecheck, lint, tests). Read
  `.gitlab-ci.yml` to find them.
- Add one focused test that fails without the fix, when the behavior has none
  and the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green mocked
  tests do not prove that the user experience works. Attach screenshots to the
  MR when they help (`glab mr update <iid> --attach <file>`).

When you cannot find a way to prove a change, you do not have full confidence.
Ask the user.

## Merge gate

Full confidence means all of these are true on the latest commit:

- Your proof covers every change in the diff.
- The head pipeline succeeded (jobs with `allow_failure` may fail if they are
  unrelated to this change).
- Review bots have finished, and each finding is fixed or dismissed with a
  written reason. Every thread is resolved.
- Required approvals are given, and no reviewer still requests changes.
- The MR is not a draft, does not conflict, and GitLab does not require a
  rebase. `detailed_merge_status` is `mergeable`.

Merge with the method the project uses. Check the project's merge method and
squash setting (`glab api projects/:id`) and recent merged MRs. Merge the
reviewed head only:

```bash
glab mr merge <iid> --sha <head_sha> --auto-merge=false --yes
```

Add `--squash` only when the project squashes. If the project uses merge
trains, or the pipeline must still finish, `--auto-merge` is fine: then stay
until GitLab reports the MR as merged. Merge through approval rules and
protected branches as configured. Never use admin overrides, skip approvals,
or push straight to the target branch. You can ignore automated reviews that
request "needs human review" if you have high confidence in your changes.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. To ask
the user something, use the harness's question tool so the thread shows that
it is waiting: `AskUserQuestion` in Claude Code, `request_user_input` in Codex,
or the question tool your harness has. Put all the context in the question:
the MR link, what blocks you, and the options with your recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change.
- The merge gate cannot pass, and you cannot fix that yourself (for example a
  required human approval).
- The next step is outside the grant above: deleting data, touching production,
  running deploy or other manual jobs with side effects, changing things outside
  the repo, closing an MR, or force-pushing any other branch.
- The goal is ambiguous in a way that changes the result.
- Another MR makes this one obsolete.

Decide everything else yourself with sensible defaults, and list those decisions
in the report.

If no question tool is available, or the tool returns no answer, take the safe
path: leave the MR open and unmerged, and start your final message with the
question.
