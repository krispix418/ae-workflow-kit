---
name: accomplishments
description: Update your running accomplishment log. Pulls from git, tickets, work artifacts, and meeting notes to surface what you've shipped.
---

# Accomplishments

Maintain a running log of what you've shipped, for performance reviews,
promotion packets, and personal reference.

## Usage

```
/accomplishments update    # scan sources, add new entries
/accomplishments remind    # highlight reel of last quarter
```

## Operation: Update

### 1. Read Current State

Read your accomplishment file. Find the last-updated date.
Only look for work AFTER this date.

### 2. Gather (parallel agents)

#### Agent 1: Git History
```bash
git log --all --author="<you>" --since="<last-updated>" --format="%ad | %s" --date=short
```
Group by ticket/project.

#### Agent 2: Work Artifacts
Scan `~/Desktop/pr_artifacts/` for directories modified after
last-updated. Read context files, summaries, impact analyses.

#### Agent 3: Tickets

Read `~/.claude/skills/_shared/ticket-provider.json` for provider.

**Jira:** Search resolved tickets since last-updated.
**Linear:** List completed issues since last-updated.

### 3. Deduplicate & Merge

Cross-reference all sources. Same ticket from git + tracker +
artifacts = one entry. Don't duplicate existing entries.

### 4. Update the File

Add entries to the appropriate quarter section. Update summary stats.

### Entry Format

```
- **YYYY-MM-DD** | TICKET-ID | Description of what was done + impact
```

Include quantitative impact whenever possible.

## Operation: Remind

Pull entries from the most recently completed quarter and generate
a highlight summary: top accomplishments, key metrics, themes.
