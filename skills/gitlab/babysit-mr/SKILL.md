---
name: babysit-mr
description: Monitor a GitLab merge request through review and CI. Use when the user asks to monitor, watch, or babysit an MR (or a PR on GitLab).
---

# Babysit MR

Many repos have AI review bots (Greptile, CodeRabbit, GitLab Duo Code Review).
They're helpful, even if they are not always right.

## Waiting

Never write your own watcher, polling loop, or `sleep` loop. Use one watcher
per MR: `mr-wait`. Never run two on the same MR. Every watcher on a GitLab
account shares one rate limit.

Wait with `scripts/mr-wait` from this skill's directory (use its absolute
path).

```bash
mr-wait <mr> [--repo group/project] [--hostname host] [--until ready|green|any-change]
```

`<mr>` is an MR number (`!123` works too) or an MR URL. A URL also sets the
project and the host, which matters for self-managed GitLab.

It prints one line per real event. It exits 0 when the `--until` condition is
met or the MR merges, 1 when it needs you (the pipeline failed, conflicts,
unresolved threads, a rebase is required, MR closed), and 2 on bad input.

- `ready` (default): head pipeline green, Greptile or GitLab Duo reviewed the
  head, and no unresolved threads. Use this to babysit.
- `green`: head pipeline only. Use it when the repo has no Greptile or Duo
  review.
- `any-change`: exit on the first event.

In Claude Code, run it with the Monitor tool (`timeout_ms: 3600000`), so each
line wakes you. Without a monitor, run it as one long command. It ends on its
own.

Start it again after you push, after you handle an exit 1, and after a resume.
It remembers what it already reported, so nothing that arrived in between is
lost. If it says another pid already watches the MR, do not start a second one.
Kill the old one only if its events do not reach you.

When an event needs no action, end your turn without a message. Never post or
say "still waiting".

## GitLab rate limits

`glab` uses the REST API, which GitLab rate-limits per user (and self-managed
admins can set their own limits). A `429` means back off: wait for the time in
the `Retry-After` or `RateLimit-Reset` header (`glab api -i ...` shows
headers) with one background command, then continue. Never retry in a tight
loop.

Useful reads:

- MR: `glab mr view <iid> -F json`, or
  `glab api projects/:id/merge_requests/<iid>`
- Pipeline jobs: `glab ci status`, or
  `glab api projects/:id/pipelines/<pipeline_id>/jobs`
- Job log: `glab ci trace <job_id>`
- Threads: `glab mr note list <iid> --state unresolved`

## Reviews

Only act on pipelines and comments newer than the latest push. Verify every bot
finding against the source before changing code. Fix real findings and pipeline
failures, distinguish repository failures from infrastructure flakes (runner
lost, `stuck_or_timeout_failure`, `runner_system_failure`), and reply with a
written reason when dismissing false positives. Retry a flaky job with
`glab ci retry <job_id>` once; do not retry a real failure.

Jobs with `allow_failure: true` do not block the pipeline. Read their failures,
but do not hold the MR for them unless they point at a real bug in this change.

CodeRabbit is advisory. Treat its findings like any other finding, but never
wait for it and never hold a merge for it.

If a review bot leaves feedback you believe is not worth addressing, reply with
a written reason and resolve the thread
(`glab mr note create <iid> --reply <discussion_id> -m ...`, then
`glab mr note resolve <discussion_id> <iid>`). Use the `leaving-mr-comments`
skill for every comment posted on the user's behalf.

Screenshots and videos help as well: `glab mr note create <iid> --attach <file>`
or `glab mr update <iid> --attach <file>` uploads them.

Keep the MR at the size the user's goal needs. Review loops tend to grow an
MR one cheap-looking fix at a time. Fix real bugs in the intended case.
Decline the rest with a written reason, and list them as follow-ups:
unneeded tests, guards for edge cases that do not matter, extra integration
or compatibility, and small performance costs. Prefer the fix that deletes or
narrows code.

## Target branch and merging

Keep an eye on changes to the target branch (usually `main`) and rebase when
needed. `mr-wait` reports when new target commits touch this MR's files, when
the MR conflicts, and when GitLab requires a rebase (fast-forward and
semi-linear projects). `glab mr rebase <iid>` rebases on the server when there
are no conflicts; otherwise rebase locally and push with `--force-with-lease`.

If an overlapping MR makes this one obsolete, stop monitoring, report it to the
user, and ask before closing the MR unless closure was explicitly authorized.

If nothing has changed, stay quiet rather than posting filler comments. Stop
when `mr-wait` exits 0. Merge only when the user explicitly requested it;
otherwise report that the MR is ready. Right before a merge, run `git fetch` and
check the MR again (`detailed_merge_status` should be `mergeable`). Never merge
when it conflicts or needs a rebase, or when target commits that landed after
the last green pipeline touch this MR's files. Rebase, push, and wait for green
again.
