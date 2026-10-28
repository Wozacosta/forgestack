---
name: file-mr
description: File a concise GitLab merge request. Use when the user asks to file, open, or create an MR (or a PR on GitLab).
---

# File MR

Before filing, check whether an MR for this branch already exists
(`glab mr list --source-branch <branch>`). Review the diff locally against the
target branch (usually `origin/main`) to make sure its contents match the goal.

MR titles often become merge or squash commit messages, so follow the
repository's title conventions. Look at recently merged MRs
(`glab mr list --merged`) and Git history for examples. Prefer a concise,
human-readable title that explains why the change matters:

BAD

> ❌ perf(server): negotiate permessage-deflate on the websocket

GOOD

> ✅ perf(server): cut websocket frame size by 70%+ with gzipping

Open the description with a simple explanation of the problem based on the
user's original prompt, then briefly explain the solution. Do not lead with an
implementation inventory:

BAD

> ❌ Removed implicit workspace carry-over from every "new thread" entry point (cmd+n / cmd+shift+o, sidebar v1/v2 buttons, command palette). New threads inherit only the project from context; branch, worktree, and env mode always come from the configured defaults. Deleted buildContextualThreadOptions, startNewThreadInProjectFromContext, and the v1 sidebar's seed-context machinery.

GOOD

> ✅ My "new worktree" default was ignored when starting new threads on existing worktrees. Super unintuitive. Now your preferences always apply.

If the repo has `.gitlab/merge_request_templates/`, use the default template
(or the one the project's docs name) and fill it in the same direct style. Link
the issue it fixes with `Closes #<n>` when there is one.

End the description with the public model name and harness that made the
changes, for example `Created with <model> in <harness>.` Include subagent
models that wrote part of the change. Use the model name the harness reports.
If model identity is unavailable, name the harness without guessing a model.

Before submission, check the title, description, quotes, and attachments for
secrets and private details.

Create it with a description file so Markdown survives quoting:

```bash
glab mr create --title "<title>" --description-file <file> \
  --target-branch <target> --yes
```

Leave squash and source-branch deletion to the project defaults unless the
repo's conventions say otherwise. Open a real MR rather than a draft (no
`--draft`, no `Draft:` title prefix) so review bots and merge request pipelines
run. Never pass `--auto-merge` here. If the user also asked to babysit it,
continue with the `babysit-mr` skill.

Do not post a follow-up MR comment unless it serves the user's request. When a
comment is needed, use the `leaving-mr-comments` skill.
