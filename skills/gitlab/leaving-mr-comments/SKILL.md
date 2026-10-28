---
name: leaving-mr-comments
description: Use whenever leaving comments on GitLab merge requests or issues.
---

# Leaving MR Comments

Prefix every agent-authored comment left on the user's behalf with this GitLab
note alert:

```md
> [!note]
> 🤖 **<model> responding on behalf of <user>**

Actual comment starts here.
```

Use the verified public name of the model that wrote the comment, and the
user's name or GitLab username.

If the harness does not expose a model name, use the harness name. Do not guess.

Older self-managed GitLab versions render `[!note]` as plain text. If the
preview shows it literally, drop that line and keep the rest of the quote:
`> 🤖 **<model> responding on behalf of <user>**`.

Before posting, check the entire comment, including quoted text and
attachments, for secrets and private details.

Keep the attribution inside the alert and the actual reply outside it. The
context line must start with the robot emoji. Do not add a horizontal rule,
Setext underline, heading, signature footer, or a second attribution.

Apply this format to top-level MR comments, diff comments, thread replies to
bots or reviewers, and written explanations used before resolving or
dismissing feedback. Do not add it to MR descriptions, commit messages, or text
the user explicitly asked to post verbatim without attribution.

Write the comment itself in a direct style:

- Lead with the conclusion.
- Cite the relevant code, behavior, pipeline job, or reproduction.
- Keep it concise and avoid filler, canned thanks, or status narration.
- Distinguish verified facts from judgment.

Post with `glab`, passing the body from a file or stdin so quoting survives:

- Top-level comment: `glab mr note create <iid> -m "$(cat body.md)"`
- Thread reply: `glab mr note create <iid> --reply <discussion_id> -m ...`
- Diff comment: `glab mr note create <iid> --file <path> --line <n> -m ...`
- Resolve: `glab mr note resolve <discussion_id> <iid>`

Preview the final Markdown before posting so quoting and blank lines render as
shown. After posting, verify that GitLab returned the note (its id or URL).
Resolve a thread only after its response was posted successfully.
