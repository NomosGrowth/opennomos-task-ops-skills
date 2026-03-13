---
name: opennomos-task-ops
description: Coordinate OpenNomos project-task workflows end to end. Use when the user wants to discover live OpenNomos tasks, classify them into full-auto or human-in-the-loop buckets, generate content packages for social or proof tasks, execute browser flows, and validate completion through points and ledger data.
---

# OpenNomos Task Ops

Use this skill to run OpenNomos tasks with a repeatable workflow instead of handling each project ad hoc.

Always pair this skill with:
- `opennomos-agent-mcp` for live project, task, points, and ledger data (https://github.com/NomosGrowth/opennomos-mcp-skills)
- `actionbook` for browser automation on project sites

## Required Inputs

Collect the smallest set that can unblock the current task:
- `nk_` API key for live OpenNomos data
- OpenNomos account credentials if browser login is required
- Human checkpoint support for captcha, 2FA, email verification, or wallet signing
- For content tasks, platform requirements and any required links, tags, screenshots, or mentions
- For ownership tasks, access to the relevant site, social account, or admin surface

Do not store secrets inside the skill files. Resolve credentials in this order:
- `OPENNOMOS_MCP_KEY`
- `OPENNOMOS_ACCOUNT_EMAIL`
- `OPENNOMOS_ACCOUNT_PASSWORD`
- if any required value is missing, ask the user to provide it in chat

Only use these environment variables for this skill. Do not invent alternate names.

If the user already has a logged-in browser session, prefer using that over logging in again, but still use the environment variables first when checking what credentials are available.

## Workflow

### 1. Discover

Use `opennomos-agent-mcp` first. Default to `https://api.opennomos.com`.

Credential resolution for this phase:
- use `OPENNOMOS_MCP_KEY` first
- if it is missing, ask the user for the `nk_` key in chat

Pull only what is needed:
- `GET /api/v1/mcp/me/points`
- `GET /api/v1/mcp/projects`
- `GET /api/v1/mcp/explore/projects`
- `GET /api/v1/mcp/projects/:project_id/tasks`
- `GET /api/v1/mcp/me/projects/:project_id/points`
- `GET /api/v1/mcp/me/projects/:project_id/ledger`

Treat `404` on `/tasks` as "project rule not found", not as a transport failure.

### 2. Classify

Put each task into one bucket:

- `A: full-auto`
  - Browserable product actions after login
  - Examples: open tool pages, daily-use flows, basic in-product interaction
- `B: half-auto`
  - Automation can reach the checkpoint, but the user must complete one step
  - Examples: captcha, email code, wallet signature, 2FA
- `C: user publishes, agent prepares`
  - The user performs the real post or external submission
  - Examples: X, Xiaohongshu, Discord, Telegram, form-based proof
- `D: user-owned surface`
  - The user controls the asset; the agent provides process support
  - Examples: SEO edits, sitemap submission, internal links, broken-link fixes, real invitations

Rank tasks by:
- points or expected upside
- time to execute
- automation fit
- dependence on external human action
- likelihood the task actually records points

### 3. Execute

For `A` tasks:
- Use `actionbook`
- Follow the manual-first rule for each page type
- If the manual is missing or broken, fall back to a page snapshot
- Prefer short deterministic runs over broad exploratory browsing

For `B` tasks:
- use `OPENNOMOS_ACCOUNT_EMAIL` and `OPENNOMOS_ACCOUNT_PASSWORD` first when login is required
- if either value is missing, ask the user in chat before starting the login flow
- Advance the flow until the user must take over
- State the checkpoint clearly and wait
- Resume immediately after the user confirms completion

For `C` tasks:
- Do not post from the user's social account unless the user explicitly asks
- Produce a content package using the format in [templates.md](references/templates.md)
- Optimize for real, credible content instead of task spam

For `D` tasks:
- Produce an SOP, proof checklist, and validation plan
- Separate what the user must really change from what can be templated
- Do not claim completion before the owned surface is actually updated

### 4. Validate

Do not treat clicks or submissions as success. Check the data.

After each batch, re-check:
- `me/points`
- project-specific points when relevant
- project ledger when relevant

Mark outcomes as:
- `confirmed`: points or ledger changed
- `pending`: action completed but no credit yet
- `unclear`: evidence exists but no ledger confirmation
- `failed`: no evidence or clear rejection

Keep "confirmed by API" separate from any inference.

### 5. Maintain a Project Task Map

For each project, keep a lightweight operating view:
- which tasks exist
- which tasks are worth attempting
- which tasks require human checkpoints
- which tasks appear untracked or delayed
- what proof is required

Use the template in [templates.md](references/templates.md) when the user wants a reusable tracker.

## Output Rules

When planning:
- lead with the recommended task order
- show the class for each task
- show the human checkpoints
- show the evidence needed

When preparing content:
- provide one main version and two alternates
- include tags, hooks, and proof text if needed
- tune tone to the platform instead of reusing one generic draft

When validating:
- show exact dates or timestamps when relevant
- separate confirmed outcomes from assumptions

## Guardrails

- Do not fabricate task completion, engagement, invites, or SEO work
- Do not assume a task counted unless points or ledger back it up
- Do not use memory for current platform state when live API data is available
- Stop and ask for help only at true human checkpoints
