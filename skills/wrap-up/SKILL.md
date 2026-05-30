---
name: ae-wrap-up
description: End-of-session ritual. Updates context file, handles uncommitted changes, pushes branch, switches to main.
user_invocable: true
allowed-tools: [Read, Write, Edit, Bash, AskUserQuestion]
---

# Wrap Up

Close out the current session cleanly — persist context, push work, leave a clean slate.

## Steps

### Step 1 — Identify Active Ticket

Detect the current branch and extract the ticket ID:
```bash
BRANCH=$(git branch --show-current 2>/dev/null)
echo "$BRANCH"
```

Extract ticket ID from branch name using pattern `[A-Z]+-[0-9]+`.

Find the context file:
```bash
find ~/Desktop/pr_artifacts -path "*/<ticket-id>/context.md" -type f 2>/dev/null
```

If no context file exists and no ticket branch is active, inform the user there's nothing to wrap up.

### Step 2 — Update context.md

Read the existing context file, then update it with:

- **Status:** Summarize where things stand (based on conversation context and git state)
- **TODOs:** Update the checklist — check off completed items, add new ones discovered during the session
- **Technical Notes:** Append any implementation decisions, gotchas, or patterns discovered
- **Last Updated:** Set to current date and time

Use the Edit tool to modify the context file in place. Preserve existing content and add to it — never delete previous notes.

### Step 3 — Handle Uncommitted Changes

Check for uncommitted work:
```bash
git status --short
```

If there are uncommitted changes, show the diff summary and ask the user using AskUserQuestion:

```
You have uncommitted changes:
  M  path/to/file.sql
  M  path/to/other_file.py
  ?? path/to/new_file.sql

What would you like to do?
  1. Commit them (I'll draft a commit message)
  2. Stash them for later
  3. Leave them as-is
```

**If commit:** Stage all changes, draft a concise commit message based on the diff, and commit.
**If stash:** `git stash push -m "WIP: <ticket-id> session wrap-up"`
**If leave:** Do nothing, just note they'll be there next session.

### Step 4 — Push Branch

```bash
git push origin $BRANCH 2>/dev/null
```

If the push fails (e.g., no upstream), set upstream and retry:
```bash
git push -u origin $BRANCH
```

If push still fails, report the error but continue with the rest of wrap-up.

### Step 5 — Switch to Main

```bash
git checkout main 2>/dev/null || git checkout master 2>/dev/null
git pull origin main 2>/dev/null || git pull origin master 2>/dev/null
```

### Step 6 — Confirm Clean Slate

```bash
git status --short
git branch --show-current
```

Present the wrap-up summary:
```
## Session Wrapped Up

Ticket:      <ticket-id> — <title>
Context:     Updated (~/Desktop/pr_artifacts/<category>/<ticket-id>/context.md)
Changes:     <committed / stashed / left uncommitted>
Branch:      Pushed to origin/<branch>
Now on:      main (up to date)

Ready for next session. Use /ae-resume-work <ticket-id> to pick back up.
```
