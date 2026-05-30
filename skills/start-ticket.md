---
name: start-ticket
description: Kick off work on a new ticket. Creates a branch, sets up the artifacts directory, writes a context file, and builds an implementation plan from the ticket description and codebase.
---

# Start Ticket

Spin up everything needed to start working on a ticket -- branch,
artifact directory, context file, and implementation plan.

## Usage

```
/start-ticket PROJ-1234
```

## Steps

### 0. Determine Ticket Provider

Read `~/.claude/skills/_shared/ticket-provider.json` for provider.

### 1. Fetch Ticket Details

#### If provider is `"jira"`:
Use `getJiraIssue` to pull summary, description, status, and labels.

#### If provider is `"linear"`:
Use `get_issue` with the ticket ID.
Maps: `title` -> summary, `description` -> description, `state` -> status.

If tracker is unreachable, ask for a short branch description.

### 2. Refresh Repos

Pull latest from main on all relevant repositories.

### 3. Determine Target Repo(s)

Infer from the ticket using labels, components, and description
keywords. If ambiguous, ask.

### 4. Create Branch

Branch name = ticket ID (e.g., `PROJ-1234`). No suffix.

```bash
git checkout main && git pull origin main
git checkout -b <ticket-id>
```

### 5. Set Up Artifacts Directory

Organize by category based on ticket signals:

```bash
mkdir -p ~/Desktop/pr_artifacts/<category>/<ticket-id>
```

Load sibling ticket context files from the same category for
relevant patterns and gotchas.

### 6. Create Context File

```markdown
# <ticket-id>: <summary>

**Branch:** `<ticket-id>`
**Repos:** `<repo-1>`, `<repo-2>`
**Category:** `<category>`
**Ticket:** [<ticket-id>](<ticketUrlBase from config>/<ticket-id>)
**Created:** <YYYY-MM-DD>
**Last Updated:** <YYYY-MM-DD>

---

## Goals

<1-2 sentence summary from ticket description>

## Todo

- [ ]

## Status

Not started.

## Technical Notes

```

### 7. Transition Ticket to In Progress

#### If provider is `"jira"`:
`getTransitionsForJiraIssue` then `transitionJiraIssue`.

#### If provider is `"linear"`:
`save_issue` with `state: "In Progress"`.

Skip silently if already In Progress. Don't block on failure.

### 8. Build Implementation Plan

Explore the codebase using the ticket description as a guide.
Write an implementation plan covering:
- Objective
- Repos and files affected
- Concrete approach steps
- Validation strategy
- Open questions
