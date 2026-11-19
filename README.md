# Neat skills/convenient commands for coding agents

## Install

Clone this repo, then copy the skills into your agent's skills folder:

```bash
mkdir -p ~/.claude/skills ~/.copilot/skills
# Claude Code
cp -r ./.claude/skills/. ~/.claude/skills/
# or Copilot/Codex
cp -r ./.copilot/skills/. ~/.copilot/skills/
```

Restart your agent session so it picks them up.

## Use

Invoke a skill as a slash command (e.g. `/pr-create`) or just ask in plain words ("create a PR for this branch"); the agent loads the matching skill.

| Skill | What it does | Example |
|-------|--------------|---------|
| `pr-create` | Opens a GitHub PR with `gh`, filling in the repo's PR template from the branch changes | `/pr-create` |
| `pr-review` | Reviews a PR, lists blocking/non-blocking issues, posts them as inline review comments | `/pr-review 123` |
| `pr-address-cmts` | Reads a PR's review comments and addresses them | `/pr-address-cmts 123` |
| `task-init` | Pulls a Jira ticket and turns it into a role-primed implementation prompt (backend NestJS or frontend Next.js) | `/task-init TASK-1234` |
| `review-changes` | Reviews a PR or the current diff, optionally checked against a Jira ticket (Claude only) | `/review-changes TASK-1234` |

## Requirements

- [`gh`](https://cli.github.com/), authenticated (`gh auth login`), for the PR skills.
- [`jira-cli`](https://github.com/ankitpokhrel/jira-cli), initialised once with `jira init`, for `task-init` and ticket-aware `review-changes`.

## Layout

- `.claude/skills/` — skills for Claude Code
- `.copilot/skills/` — the same skills adapted for Copilot/Codex (no `review-changes`)

Highly recommended: add ponytail as a plugin.
