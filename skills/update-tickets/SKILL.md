---
name: ae-update-tickets
description: Sync ticket statuses and estimates with reality. Scans open sprint/cycle tickets, reconciles against branches/PRs/merges.
user_invocable: true
allowed-tools: [Read, Bash, AskUserQuestion]
---

# Update Tickets

Reconcile your tracker with git reality — statuses, points, and all.

## Steps

### Step 0 — Load Provider Config

Read `~/.claude/skills/_shared/ticket-provider.json` to determine the provider and settings.

If the config doesn't exist, inform the user to run `/ae-setup` first and stop.

### Step 1 — Fetch Open Tickets

**If Jira:**
Use `searchJiraIssuesUsingJql` with JQL:
```
project = <project from config> AND sprint in openSprints() AND assignee = currentUser() AND status != Done ORDER BY status ASC
```

Extract: ticket ID, title, status, story points.

**If Linear:**
Use `list_issues` with:
- team: `<team from config>`
- cycle: `"current"`
- assignee: `"me"`
- Filter to started + unstarted states (exclude completed/canceled)

Extract: issue ID, title, state, estimate.

### Step 2 — Check Git Reality

For each ticket, check against actual git state:

```bash
# Does a branch exist for this ticket?
git branch -a 2>/dev/null | grep -i "<ticket-id>"

# Is there an open PR?
gh pr list --head "<ticket-id>" --state all --json number,state,mergedAt,url 2>/dev/null

# Is there a context file?
find ~/Desktop/pr_artifacts -path "*/<ticket-id>/context.md" -type f 2>/dev/null
```

Classify each ticket's git state:
- **No branch, no PR** → Not Started
- **Branch exists, no PR** → In Progress
- **PR open** → In Review
- **PR merged** → Done

### Step 3 — Estimate Unpointed Tickets

For tickets without story points/estimates that have branches, estimate based on scope:

```bash
# Count commits and files changed
git log --oneline origin/main..<ticket-branch> 2>/dev/null | wc -l
git diff --stat origin/main..<ticket-branch> 2>/dev/null
```

Heuristic for estimates:
- 1-2 files, < 50 lines → 1 point
- 3-5 files, 50-150 lines → 2 points
- 5-10 files, 150-300 lines → 3 points
- 10+ files or 300+ lines → 5 points

Only suggest estimates — never auto-assign without approval.

### Step 4 — Determine Proposed Changes

Build a change table comparing current tracker state vs. git reality:

```
## Proposed Changes

| Ticket | Title | Current Status | Target Status | Points |
|--------|-------|---------------|---------------|--------|
| PROJ-1234 | Fix widget | To Do | In Progress | — |
| PROJ-1235 | Add feature | In Progress | In Review | 2 → 3 |
| PROJ-1236 | Refactor | In Review | Done | 1 |

No changes needed:
| PROJ-1237 | Update docs | In Progress | (matches) | 2 |
```

### Step 5 — Get Approval

Present the proposed changes to the user using AskUserQuestion:
```
Here are the proposed tracker updates. Approve all, or tell me which to skip.
```

Wait for explicit approval before making any changes. The user may:
- Approve all
- Approve selectively (e.g., "skip PROJ-1235")
- Override a status or estimate
- Cancel entirely

### Step 6 — Apply Changes

**If Jira:**
For status changes:
1. Call `getTransitionsForJiraIssue` with the ticket ID to get available transitions.
2. Find the transition that moves to the target status.
3. Call `transitionJiraIssue` with that transition ID.

For story point updates:
- Call `editJiraIssue` with the story points field.

**If Linear:**
For status and/or estimate changes:
- Call `save_issue` with the issue ID, setting `state` and/or `estimate` in a single call.

### Done

Report results:
```
## Ticket Update Results

Updated:
  PROJ-1234: To Do → In Progress ✓
  PROJ-1235: In Progress → In Review, 2 → 3 pts ✓
  PROJ-1236: In Review → Done ✓

Skipped:
  PROJ-1237: Already correct

Errors:
  (any failures with details)
```
