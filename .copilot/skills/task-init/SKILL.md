---
name: task-init
description: 'Initialize a Jira ticket into a ready-to-run implementation prompt with the right expert role. Use when the user starts or preps Jira work: "task-init TASK-1234", "/task-init", "init this ticket", "get the Jira ticket", "prepare work from a ticket", "what role should the agent take for <KEY>". Pulls the ticket via the jira CLI, classifies it as backend (NestJS/TypeScript) or frontend (Next.js/TypeScript), and emits a role-primed planning prompt.'
argument-hint: '<JIRA-KEY>'
---

# Task init — Jira ticket → role-primed implementation prompt

Turn a Jira ticket into a copy-ready prompt that assigns one of two expert roles — backend (Nest.js) or frontend (Next.js) — carries the ticket's title, todos and context, and ends with a "confirm details & assumptions" gate before any code is written.

**Inputs:** the ticket key, e.g. `TASK-1234`. Also accepts a bare issue number (uses the jira-cli default project) or a full ticket URL — in that case extract the key after `/browse/`. If no key is provided, ask for one and stop; never guess a key.

## Procedure

### 1. Ensure the jira CLI is installed

Run `command -v jira && jira version`.

- **Installed** → continue.
- **Missing** → tell the user this workflow needs [`ankitpokhrel/jira-cli`](https://github.com/ankitpokhrel/jira-cli) and how to get it, then stop until they confirm it is installed:
  - macOS/Linux (Homebrew): `brew install jira-cli`
  - Go: `go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest`
  - Or download a binary from the project's GitHub releases.
  - After install, they must run `jira init` once (server URL such as `https://<org>.atlassian.net`, auth via API token from https://id.atlassian.com/manage-profile/security/api-tokens, and a default project) — this is interactive, so the user does it themselves.
- Re-verify with `jira version` before proceeding; do not fall back to the Atlassian REST API.

### 2. Ensure the CLI is authenticated

Run `jira me`. On failure: walk the user through `jira init`, wait for them to finish, and re-run `jira me` to confirm.

### 3. Pull the ticket

```sh
jira issue view <KEY> --plain --comments 5
```

- `--plain` gives readable description text; comments often carry the agreed approach, so always include them.
- If the visible output is empty or mangled (tables, ADF markup, attachments), try `jira issue view <KEY> --raw`, or `jira open <KEY>` and ask the user to paste what you still need.
- From the output collect: title/summary, task-type prefix if any, description (purpose, API/UI details, test expectations), todo items, and any referenced designs or docs.

### 4. Determine the role

Check the ticket **title prefix first**, then description keywords:

| Evidence | → Role |
| --- | --- |
| Title starts with `API`, `BE`, `Backend`; mentions endpoint, route, controller, service, DTO, permission, token, paging/envelope, database, migration, queue, cron, webhook, integration, API contract | **Backend (API)** |
| Title starts with `UI`, `FE`, `Frontend`; mentions page, view, tab, panel, dialog, modal, form, input, table, layout, loading/error/not-found state, routing, design, shadcn, mockup | **Frontend (UI)** |

Full-stack tickets or tickets with no clear signal → ask the user which role to assume before building the prompt.

### 5. Build the prompt

Fill the matching template below. Replace `{TITLE}`, `{TODO}` (methodically extracted from the ticket) and `{CONTEXT}` with real ticket content:

- **Purpose**: 1–2 sentences distilled from the description.
- **API/UI block**: substantive detail bullets from description and comments, keeping the ticket's specifics (permissions, routes, envelopes, states) verbatim where possible.
- **Test block**: only if the ticket states test expectations.
- **Reference block**: only if the ticket references designs/docs (e.g. an HTML mock).
- Never invent acceptance criteria. If required details are missing, add an explicit "open questions" list instead.

Output the finished prompt in a **single fenced code block** so the user can copy it into a fresh chat, and state which role was chosen and why in one sentence.

Plan-file naming rule (used inside the generated prompt): output the plan under `../tmp/plan/<task-type>/<task>.md` where `task-type` is `API` or `UI`; `<task>` is the ticket title minus the type prefix, kebab-cased, e.g. `../tmp/plan/API/members-list-with-embedded-roles-paging-search.md`.

## Backend (API) role template

```xml
<role>
  You are a full stack developer who's expert at implementing backend system with Nest.js and Typescript.
</role>

<tasks>
  Create a plan for implementing below JIRA task. Output the plan under `../tmp/plan/API/<task>.md`. `task-type` is API.
  <title>{TITLE}</title>
  <todo>
{TODO}
  </todo>
  Confirm the details & assumption before proceed with execution
</tasks>

<context>
{CONTEXT}
</context>
```

## Frontend (UI) role template

```xml
<role>
  You are a full stack developer who's expert at implementing frontend system with Next.js and Typescript.
</role>

<rules>
  - Refer to frontend/ AGENTS.md for UI rules
  - Do not re-implement atomic UI like input, table or modal. All atomic UI should be reusing the existing ones, or if it does not exist, imported from shadcn.
</rules>

<tasks>
  Create a plan for implementing below JIRA task. Output the plan under `../tmp/plan/UI/<task>.md`. `task-type` is UI.
  <title>{TITLE}</title>
  <todo>
{TODO}
  </todo>
  Confirm the details & assumption before proceed with execution
</tasks>

<context>
{CONTEXT}
</context>
```

In the context block, group detail bullets under `API:` / `UI:` / `Test:` / `Reference:` headings matching whichever template was used.