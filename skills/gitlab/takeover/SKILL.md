---
name: takeover
description: Take over existing work. Use when the user asks to take over an MR (or PR), branch, or agent thread, or requests a takeover.
---

# Takeover

Take responsibility for finishing the original goal while preserving the
original contributors' credit. Do not defend an implementation just because
another agent already built it.

Read the source MR, branch, or thread, including review threads and the user's
later corrections (`glab mr view <iid> --comments`, `glab mr diff <iid>`).
Resolve the handoff to the actual code and current head, and account for
uncommitted work. Treat the handoff as context, not proof. If the MR comes from
a fork, `glab mr checkout <iid>` fetches its branch.

For a replacement MR, import the original commits onto a new branch and add
your fixes separately. Honor an explicitly requested in-place or stacked
continuation instead. Pushing to someone else's fork needs "Allow commits from
members who can merge" on their MR; do not assume it. Do not overwrite
unrelated work or change the original MR without authorization.

Preserve Git authorship and existing co-author trailers. Credit the original
contributors in the MR description and link the original MR (`!<iid>`) when
there is one. If the project squashes, preserve human author/co-author credit
in the squash commit message too, using verified identities rather than guessed
emails. Model attribution is not a substitute for contributor credit.

Use `modernize` to update the continuation against its intended target branch
and current architecture. Rework what is necessary to finish the same goal, not
unrelated features. Simplify the implementation without dropping intended
behavior or surfaces. Recheck the source MR too: it may have improved while
you were working.

Independently verify the original failure and realistic workflows, following
repository guidance and permissions. Green mocked tests do not prove the user
experience works. State what remains unverified.

When the requested finish includes an MR, use `file-mr` to file or update it,
then `babysit-mr` rather than stopping at creation. Use `leaving-mr-comments`
for replies. Merge or close the original MR only when authorized.

Report what you inherited, what you changed, how you verified it, how credit
was preserved, and any remaining decision.
