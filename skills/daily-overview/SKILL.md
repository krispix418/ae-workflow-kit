---
name: ae-daily-overview
description: Start-of-day overview. Pulls sprint/cycle tickets, scans work artifacts, reads TODO, checks PRs.
user_invocable: true
allowed-tools: [Read, Bash]
---

# Daily Overview

Morning orientation — surface everything you need to pick your first task.

## Steps

### Step 0 — Load Provider Config

Read `~/.claude/skills/_shared/ticket-provider.json` to determine which tracker to use.

Preflight health check:
- **Jira**: call `getVisibleJiraProjects` — if it errors, note Jira is unavailable and skip tracker steps.
- **Linear**: call `list_teams` — if it errors, note Linear is unavailable and skip tracker steps.

If the config file doesn't exist, skip tracker steps entirely and note that `/ae-setup` should be run.

### Step 1 — Sprint / Cycle Tickets

**If Jira:**
Use `searchJiraIssuesUsingJql` with JQL:
```
project = <project from config> AND sprint in openSprints() AND assignee = currentUser() ORDER BY status ASC, priority DESC
```

**If Linear:**
Use `list_issues` with:
- team: `<team from config>`
- cycle: `"current"`
- assignee: `"me"`

Collect: ticket ID, title, status, assignee, story points/estimate.

### Step 2 — Work Artifacts

```bash
find ~/Desktop/pr_artifacts -name "context.md" -type f 2>/dev/null
```

For each context.md found, read it and extract:
- Ticket ID
- Current status
- Open TODOs
- Associated branch
- Last updated timestamp

### Step 3 — TODO.md

```bash
cat ~/Desktop/TODO.md 2>/dev/null
```

If the file exists, include its contents. If not, skip silently.

### Step 4 — Open PRs & Review Queue

Check for PRs the user authored that are still open:
```bash
gh pr list --author @me --state open --json number,title,url,reviewDecision,statusCheckRollup 2>/dev/null
```

Check for PRs requesting user's review:
```bash
gh search prs --review-requested @me --state open --json repository,number,title,url 2>/dev/null
```

If `gh` is not available or not authenticated, skip and note it.

### Step 5 — Synthesize

Present a structured morning briefing:

```
## Sprint Progress
<ticket count> tickets in sprint — <done count> done, <in-progress count> in progress, <todo count> to do

## Open Tickets
| Ticket | Title | Status | TODOs | PR |
|--------|-------|--------|-------|----|
(one row per ticket, merge tracker data with artifact data)

## TODO.md
(summary of top items from TODO.md)

## Review Queue
(list of PRs needing your review, with repo and title)

## Suggested First Move
(pick the highest-priority item based on: blocked PRs > review requests > in-progress tickets > new tickets)
```

Skip any section that has no data. Keep the output scannable.
