---
name: update-tickets
description: Sync ticket statuses and estimates with reality. Scans open sprint/cycle tickets, reconciles against branches/PRs/merges, and estimates points for unpointed tickets.
---

# Update Tickets

Keep your tracker in sync with your actual work state.

## Steps

### 0. Determine Ticket Provider

Read `~/.claude/skills/_shared/ticket-provider.json` for provider.

### 1. Fetch All Open Tickets

#### If provider is `"jira"`:
```
assignee = currentUser() AND sprint in openSprints()
AND status != "Done" AND project = <from config>
```
Include `customfield_10016` (story points) in fields.

#### If provider is `"linear"`:
```
assignee: "me", cycle: "current", team: <from config>
```
Run for `state: "started"` and `state: "unstarted"`, merge results.
The `estimate` field is returned directly.

### 2. Check Each Ticket Against Reality

For each ticket, check across repos:
- **Branch:** `git branch -a --list "*<ticket-id>*"`
- **PR:** `gh pr list --head <ticket-id> --json number,state,mergedAt`
- **Context file:** `find ~/Desktop/pr_artifacts -maxdepth 3 -name "<ticket-id>"`

### 3. Determine Target Status

| Signal | Target Status |
|--------|--------------|
| All PRs merged (all repos) | Done |
| PR open + not draft | In Review |
| Branch exists, no PR | In Progress |
| No branch, no PR | leave as-is |

### 4. Estimate Points (unpointed only)

| Signal | Points |
|--------|--------|
| No branch / no context file | 1 |
| < 3 commits, < 5 files | 1 |
| 3-8 commits, 5-15 files | 2 |
| 9-15 commits, 15-30 files | 3 |
| 16+ commits or 30+ files | 5 |

Bump +1 for multi-repo. Cap at 8. Never overwrite existing.

### 5. Present Changes for Approval

### 6. Apply

#### If provider is `"jira"`:
- **Status:** `getTransitionsForJiraIssue` then `transitionJiraIssue`
- **Points:** `editJiraIssue` with `{"customfield_10016": N}`

#### If provider is `"linear"`:
- **Both:** `save_issue` with `id`, `state`, and `estimate` in one call
