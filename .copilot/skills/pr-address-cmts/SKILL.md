---
name: pr-address-cmts
description: Address existing GitHub pull request's review comments using the gh CLI
---

Address existing GitHub pull request's review comments by following these steps. Use the `gh` CLI to fetch the PR, read comments, and post replies.

1. Ask the user for the PR URL if it was not provided (e.g. `https://github.com/<owner>/<repo>/pull/<number>`).
2. Fetch the PR state with `gh pr view <url> --json state,mergedAt,headRefName`. If the PR is merged or closed, tell the user and stop.
3. Fetch the review comments with `gh api repos/<owner>/<repo>/pulls/<number>/comments` and group them by file and line.
4. Evaluate each comment for validity (correctness, conventions, security). Ask the user before acting on any comment you are unsure about.
5. Implement fixes for valid comments on the PR's branch (`headRefName` from step 2).
6. Before committing, run the project's lint check, type check, and Prettier format command, and fix any issues they surface.
7. Commit and push the changes with a message that summarizes the addressed feedback.
8. Reply to each review comment via `gh api repos/<owner>/<repo>/pulls/<number>/comments/<comment_id>/replies -f body="..."`:
   - For addressed comments, include the commit hash and a short explanation of the change.
   - For invalid comments, politely explain why no change was made.
   - Do not use `gh pr comment`, as it posts a new top-level comment instead of a thread reply.
