---
name: pr-create
description: Create a GitHub Pull Request using the gh CLI and the repository's PR template.
---

Create a pull request for the current branch by following these steps:

1. Check `git status`, `git diff`, and recent commits against the base branch to understand all changes.
2. Check if the current branch is pushed to the remote; if not, push it with `git push`.
3. Look for a PR template in `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `pull_request_template.md`, or `.github/PULL_REQUEST_TEMPLATE/`.
4. Draft a clear PR title with the repo's convention (e.g. feat(TASK-1234): UI add-member dialog with user search) and fill out the repository's PR template accurately based on the branch changes.
5. Check for upstream repo name first (i.e `<org>/<repo-name>`)
5. Run `gh pr create --repo "<org>/<repo-name>" --title "<title>" --body "<body>" --label "nitty-review"` (or pass `--template` if appropriate) to open the pull request, and output the resulting PR URL. If user specifies base branch, pass `--base <base branch>`
