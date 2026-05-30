---
name: ae-resume-work
description: Resume work on a ticket. Loads context from artifacts, checks git status, orients on what to do next.
user_invocable: true
argument-hint: "[TICKET-ID]"
allowed-tools: [Read, Bash]
---

# Resume Work

Load context from a previous session and get oriented fast.

## Inputs

Optional ticket ID as argument. If not provided, detect from the current git branch.

## Steps

### Step 1 — Resolve Ticket ID

If a ticket ID was passed as an argument, use it directly.

Otherwise, detect from the current branch:
```bash
git branch --show-current 2>/dev/null
```

Extract the ticket ID from the branch name using pattern `[A-Z]+-[0-9]+` (or the full branch name if it matches a ticket pattern).

If no ticket ID can be determined, list available context files and let the output guide the user:
```bash
find ~/Desktop/pr_artifacts -name "context.md" -type f 2>/dev/null
```

### Step 2 — Load Context

Find the context file:
```bash
find ~/Desktop/pr_artifacts -path "*/<ticket-id>/context.md" -type f 2>/dev/null
```

Read the context.md file. Extract:
- Ticket title and URL
- Current status and open TODOs
- Branch name and repo path
- Technical notes
- Last updated timestamp

If no context file is found, report it and suggest running `/ae-start-ticket <ticket-id>` instead.

### Step 3 — Load Sibling Context

Determine the category directory from the context file path:
```bash
# If context is at ~/Desktop/pr_artifacts/comp/PROJ-1234/context.md
# then category is "comp"
```

Find other context files in the same category:
```bash
find ~/Desktop/pr_artifacts/<category> -name "context.md" -not -path "*/<ticket-id>/*" -type f 2>/dev/null
```

For each sibling, read just the ticket title, status, and any patterns/notes that might be relevant. This helps surface shared context across related tickets.

### Step 4 — Switch to Branch & Pull

```bash
cd <repo-path from context>
git checkout <branch-name>
git pull origin <branch-name> 2>/dev/null || true
```

If the branch doesn't exist locally, check if it exists on remote:
```bash
git fetch origin <branch-name> 2>/dev/null
git checkout <branch-name> 2>/dev/null
```

### Step 5 — Check Git Status

```bash
cd <repo-path>
git status --short
git log --oneline -5
git log --oneline origin/main..<branch-name> 2>/dev/null
```

Capture:
- Uncommitted changes (modified, untracked files)
- Recent commits on this branch
- How many commits ahead of main

### Step 6 — Present Orientation

```
## Resuming: <TICKET-ID> — <ticket title>

**Ticket:** <url>
**Branch:** <branch> (<N> commits ahead of main)
**Last session:** <last updated timestamp>

### Status
<current status from context>

### Open TODOs
- [ ] <todo items from context>

### Repo State
<uncommitted changes summary, or "clean working tree">

### Artifacts
- context.md (last updated <date>)
- implementation_plan.md (if exists)
- validation_query.sql (if exists)
- <any other files in the artifact directory>

### Related Tickets (<category>)
- <sibling ticket ID> — <title> (<status>)
```

List artifacts in the ticket directory:
```bash
ls ~/Desktop/pr_artifacts/<category>/<ticket-id>/
```

Skip the "Related Tickets" section if there are no siblings.
Do not prompt the user for next steps — just present the context and let them direct.
