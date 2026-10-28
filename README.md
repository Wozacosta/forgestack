# forgestack

Agent skills for shipping code changes end to end, in two flavors: one for
GitHub and one for GitLab. They work in Claude Code and Codex.

The GitHub flavor is [tstack](https://github.com/t3dotgg/tstack) by Theo
Browne, copied unchanged from upstream commit `1085050`. The GitLab flavor is
an adaptation of it for `glab`, merge requests, and GitLab pipelines.

```
skills/
  github/   upstream tstack, uses gh
  gitlab/   adapted for glab and merge requests
```

## Skills

| GitHub | GitLab | Say | What it does |
|---|---|---|---|
| `full-send` | `full-send` | "full send" | Takes a change all the way to a merged PR/MR with nobody watching: proves it, reviews it, files it, babysits it, merges it. |
| `full-stop` | `full-stop` | "full stop" | Same as `full-send`, but stops when the PR/MR is ready for you to merge. |
| `council` | `council` | "council" | Gets review from three models from different providers, and loops until they agree. |
| `sol-loop` | `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `file-pr` | `file-mr` | "file a PR/MR" | Files a short PR/MR that leads with the problem, not an inventory of changes. |
| `takeover` | `takeover` | "take over" | Finishes someone else's PR/MR, branch, or agent thread, and keeps their credit. |
| `babysit-pr` | `babysit-mr` | "babysit" | Watches a PR/MR through CI and review bots, and fixes the real findings. |
| `modernize` | `modernize` | "modernize" | Rebases an old PR/MR and decides if it should still exist. |
| `leaving-pr-comments` | `leaving-mr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |

Some skills call others, so install a whole flavor:

- `full-send` uses `sol-loop`, `file-*`, and `babysit-*`.
- `full-stop` uses `full-send`.
- `takeover` uses `modernize`, `file-*`, `babysit-*`, and `leaving-*-comments`.
- `file-*`, `babysit-*`, and `modernize` use `leaving-*-comments`.

## Install

Six skills share a name across flavors (`full-send`, `full-stop`, `council`,
`sol-loop`, `takeover`, `modernize`), so install **one flavor per skills
directory**. Put your main forge globally and the other one per project.

```bash
git clone https://github.com/Wozacosta/forgestack ~/Code/forgestack

# Global, GitHub flavor
cp -R ~/Code/forgestack/skills/github/* ~/.claude/skills/   # Claude Code
cp -R ~/Code/forgestack/skills/github/* ~/.codex/skills/    # Codex

# One GitLab project (project skills override global ones in Claude Code)
mkdir -p .claude/skills && cp -R ~/Code/forgestack/skills/gitlab/* .claude/skills/
```

## Requirements

- GitHub flavor: an authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- GitLab flavor: an authenticated [GitLab CLI](https://gitlab.com/gitlab-org/cli)
  (`glab auth login`, add `--hostname` for self-managed GitLab).
- `python3` for the watcher scripts (`pr-wait`, `mr-wait`).
- `council` and `sol-loop` need subagents on other providers' models. T3 Code
  does this with `delegate_task`. Otherwise you need a harness that can run
  those models.

## What changed for GitLab

- `gh` → `glab`, PR → MR, base branch → target branch, checks → the head
  pipeline. GitLab has no per-job required checks, so the pipeline status is
  the gate, and `allow_failure` jobs never block.
- `babysit-mr/scripts/mr-wait` replaces `pr-wait`. Same interface and exit
  codes (`--until ready|green|any-change`, 0/1/2), built on the REST API. It
  also accepts MR URLs from self-managed hosts, reports approvals and
  requested changes, a required rebase (fast-forward projects), blocking manual
  jobs, and stale merged-results pipelines.
- Review bots: Greptile and GitLab Duo Code Review gate `ready`; CodeRabbit
  stays advisory. Macroscope is GitHub-only, so it is gone.
- `full-send` merges with `glab mr merge --sha <head> --auto-merge=false`, so
  only the reviewed head lands. `full-stop` never calls `glab mr merge`, because
  it sets auto-merge by default while a pipeline runs, and cancels existing
  auto-merge and merge-train entries.
- Comments use GitLab's `> [!note]` alert, with a plain-quote fallback for
  older self-managed instances.
- T3 Code's `watch_pull_request` is GitHub-only, so the GitLab flavor always
  waits with `mr-wait`.

## Make them yours

These are starting points. Edit them. Put personal settings, like which models
sit on your council, in your `CLAUDE.md` or `AGENTS.md`, so you can update the
skills without losing them.

## License

[MIT](LICENSE). Original skills © 2026 Theo Browne.
