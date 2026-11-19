---
name: pr-review
description: Review a GitHub Pull Request using the gh CLI, list blocking/non-blocking issues, and post them as inline review comments
---

Review the GitHub pull request at the supplied PR URL. Use the `gh` CLI for all reads and writes.

## 1. Fetch the PR
1. If no PR URL was provided, ask for one (e.g. `https://github.com/<owner>/<repo>/pull/<number>`) and wait for the answer before going further.
2. Get the metadata: `gh pr view <url> --json title,body,author,state,isDraft,baseRefName,headRefName,additions,deletions,changedFiles`. If the PR is merged or closed, tell the user and stop.
3. Get the diff and the changed files: `gh pr diff <url>` and `gh pr diff <url> --name-only`.
4. Parse `<owner>`, `<repo>`, and `<number>` out of the URL — the later `gh api` calls need them.
5. Treat the PR body as the author's stated intent and judge the diff against it. When a finding depends on surrounding code, read the file on the `headRefName` branch: locally if the repo is this workspace, otherwise via `gh api "repos/<owner>/<repo>/contents/<path>?ref=<headRefName>"`.

## 2. Analyze the diff
Invoke the `code-review` skill via the Skill tool on the PR URL (e.g. args `high <url>`). Do not pass `--comment`, `--fix` or `--post`: this skill posts the review itself in step 4.

Use its verified findings as the review input, and add checks it does not cover:
- Breaking changes: changed contracts, removed behavior, schema/data migrations without a way back.
- Tests: new or changed behavior without test coverage, tests that cannot fail.
- Fit with the PR body: does the diff do what the author says it does?

Do not invent blocking issues to look thorough — if the change is clean, the review says so. Ignore lockfiles, generated files, and pure formatting churn. Drop any finding you cannot tie to a line in the diff or to code you read in step 1.

## 3. Categorize
- **Blocking**: bugs, security holes, data loss, breaking changes, critical paths without tests. Prefix each comment body with `[blocking]`.
- **Non-blocking**: style, naming, preferences, nice-to-haves. Prefix with `[nit]`.

## 4. Post inline comments
Every inline comment must be anchored to a line that is part of the diff (a changed line or its hunk context). If an issue is on an untouched line, put it in the review body instead of forcing an inline comment.

Write the review summary (verdict plus labelled Blocking/Non-blocking lists) to a temp file, then post everything in one call:

```bash
gh api repos/<owner>/<repo>/pulls/<number>/reviews \
  -f event=REQUEST_CHANGES \
  -f "body=$(cat /tmp/pr-<number>-review.md)" \
  -f 'comments[][path]=src/services/cart.ts' \
  -f 'comments[][side]=RIGHT' \
  -F 'comments[][line]=42' \
  -f 'comments[][body]=[blocking] getTotal() divides by zero when items is empty. Guard with `if (!items.length) return 0`.' \
  -f 'comments[][path]=src/app/layout.tsx' \
  -f 'comments[][side]=RIGHT' \
  -F 'comments[][line]=87' \
  -f 'comments[][body]=[nit] Prefer the existing `cn()` helper over template literals.'
```

- Each path/side/line/body group of `comments[]` fields forms one comment; repeat the group for every finding, in order.
- Use `-F` for line numbers (they must be JSON numbers) and `-f` for text fields.
- `comments[][side]=RIGHT` takes the line number in the new file; use `LEFT` with the old line number for removed lines. For a multi-line range add `comments[][start_line]` and `comments[][start_side]`.
- `event`: `REQUEST_CHANGES` if there is at least one blocking issue, `COMMENT` otherwise.
- One issue per comment: label, what is wrong, why it matters, and a concrete suggested fix. No praise, no restating the diff.
- If there is nothing to say, do not post an empty review — report to the user that it looks good.

## 5. Report
Reply to the user with the verdict: counts of blocking and non-blocking issues, whether the review posted successfully, and the one or two findings that matter most.

