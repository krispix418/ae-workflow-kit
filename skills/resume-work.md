---
name: resume-work
description: Resume work on a ticket at the start of a new session. Loads context from artifacts, checks git status, and orients you on what to do next.
---

# Resume Work

Load context and orient for the current (or specified) ticket so you
can pick up exactly where you left off.

## Usage

```
/resume-work              # infers ticket from current branch
/resume-work PROJ-1234    # jump to a specific ticket
```

## Steps

### 1. Resolve Ticket ID

If argument provided, use it. Otherwise check branches across repos
for a non-main branch matching `[A-Z]+-[0-9]+`.

If no ticket branch found, list available context files and ask.

### 2. Load Context File

Find the ticket's directory under `~/Desktop/pr_artifacts/`:
```bash
find ~/Desktop/pr_artifacts -maxdepth 3 -type d -name "<ticket-id>"
```

Read `context.md`. Parse Repos, Category, Status, and Todo fields.

Load sibling ticket context from the same category for relevant
patterns (Technical Notes only).

### 3. Switch to Branch

For each repo listed in context:
```bash
git checkout <branch-name>
git pull origin <branch-name>
```

Flag uncommitted changes or merge conflicts before switching.

### 4. Check Git Status

For each repo: uncommitted changes, commits ahead of main,
ahead/behind remote.

### 5. Orient

```
Resuming <ticket-id> -- <summary>

Status:     <from context>
Open todos: <count>

Repos:
  repo-a  -> <branch> (<X commits>, clean)
  repo-b  -> <branch> (<Y commits>, 2 uncommitted)

Artifacts: ~/Desktop/pr_artifacts/<category>/<ticket-id>/

What do you want to tackle first?
```
