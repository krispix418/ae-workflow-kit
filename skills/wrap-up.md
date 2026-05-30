---
name: wrap-up
description: End-of-session ritual. Updates context files, handles uncommitted changes, pushes the branch, and switches to main — leaving a clean slate for next session.
---

# Wrap Up

Persist your state so the next session can pick up cleanly.

## Steps

### 1. Identify Active Ticket

Check current branches across repos for a ticket ID.

### 2. Update Context File

Read the context file and update:
- **Status**: reflect current state
- **Todo**: check off completed items, add new ones discovered
- **Last Updated**: today's date
- **Technical Notes**: add any gotchas or decisions made this session

### 3. Handle Uncommitted Changes

For each repo with changes:
- Show `git status` and `git diff --stat`
- Ask: commit, stash, or leave?
- If committing, draft a concise commit message

### 4. Push Branch

```bash
git push origin <branch-name>
```

### 5. Switch to Main

```bash
git checkout main && git pull origin main
```

### 6. Confirm

```
Wrapped up <ticket-id>.

Context:     updated
Branch:      pushed
Repos:       back on main

Clean slate for next time.
```
