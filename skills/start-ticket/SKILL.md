---
name: ae-start-ticket
description: Kick off work on a new ticket. Creates branch, artifacts directory, context file, and implementation plan.
user_invocable: true
argument-hint: <TICKET-ID>
allowed-tools: [Read, Write, Bash, AskUserQuestion]
---

# Start Ticket

Full ticket kickoff — from tracker fetch to implementation plan in one command.

## Inputs

The user provides a ticket ID as an argument (e.g., `PROJ-1234`). If no argument is given, ask for the ticket ID using AskUserQuestion.

## Steps

### Step 0 — Load Provider Config

Read `~/.claude/skills/_shared/ticket-provider.json` to determine the tracker provider and settings.

Extract:
- `provider` ("jira" or "linear")
- `ticketUrlBase` (for constructing ticket URLs)
- Project/team identifiers

### Step 1 — Fetch Ticket Details

**If Jira:**
Use `getJiraIssue` with the ticket ID. Extract:
- Title / summary
- Description
- Status
- Priority
- Story points
- Assignee
- Sprint

**If Linear:**
Use `get_issue` with the ticket ID. Extract:
- Title
- Description
- State
- Priority
- Estimate
- Assignee
- Cycle

If the ticket cannot be found, report the error and stop.

### Step 2 — Pull Latest

Determine the repo to work in. If the user has multiple repos configured, ask which one.

```bash
cd <repo-path>
git checkout main && git pull origin main
```

Use `main` as default branch — if it doesn't exist, try `master`.

### Step 3 — Create Branch

```bash
cd <repo-path>
git checkout -b <ticket-id>
```

Use the ticket ID as the branch name (e.g., `PROJ-1234`). If a branch with that name already exists, inform the user and ask whether to switch to it or create a new one with a suffix.

### Step 4 — Set Up Artifact Directory

Ask the user for the work category using AskUserQuestion:
```
What category does this ticket fall under?
(This organizes your work artifacts — e.g., comp, infra, data-quality, api, etl)
```

Create the directory:
```bash
mkdir -p ~/Desktop/pr_artifacts/<category>/<ticket-id>
```

### Step 5 — Create context.md

Write `~/Desktop/pr_artifacts/<category>/<ticket-id>/context.md` with:

```markdown
# <TICKET-ID>: <ticket title>

## Ticket
- **URL:** <ticketUrlBase from config>/<ticket-id>
- **Status:** <current status>
- **Priority:** <priority>
- **Points:** <points/estimate or "unpointed">

## Branch
- **Repo:** <repo path>
- **Branch:** <ticket-id>

## Summary
<ticket description, condensed to key points>

## Goals
- <extracted from ticket description>

## TODOs
- [ ] <initial todos based on ticket requirements>

## Technical Notes
(to be filled during implementation)

## Last Updated
<current date and time>
```

### Step 6 — Transition Ticket to In Progress

**If Jira:**
1. Call `getTransitionsForJiraIssue` to get available transitions.
2. Find the transition that moves to "In Progress" (or similar status).
3. Call `transitionJiraIssue` with that transition ID.

**If Linear:**
Call `save_issue` with:
- id: the issue ID
- state: "In Progress" (or the equivalent started state)

If the transition fails, note it but don't block the rest of the workflow.

### Step 7 — Explore Codebase & Write Implementation Plan

Explore the repo to understand relevant code:
- Search for files related to the ticket's domain
- Read key files that will need changes
- Identify patterns and conventions in the codebase

Write `~/Desktop/pr_artifacts/<category>/<ticket-id>/implementation_plan.md`:

```markdown
# Implementation Plan: <TICKET-ID>

## Objective
<what needs to change and why>

## Files to Modify
- `path/to/file.ext` — <what changes>

## Approach
1. <step-by-step plan>

## Testing / Validation
- <how to verify the changes are correct>

## Open Questions
- <anything unclear from the ticket>
```

### Done

Present a summary:
```
Ticket <TICKET-ID> is ready to work on.

Branch:    <ticket-id> (on <repo>)
Artifacts: ~/Desktop/pr_artifacts/<category>/<ticket-id>/
Status:    Moved to In Progress
Plan:      implementation_plan.md written

Next: start implementing, or review the plan.
```
