---
name: ae-accomplishments
description: Update your running accomplishment log. Pulls from git, tickets, and work artifacts to surface what you've shipped.
user_invocable: true
argument-hint: <update|remind>
allowed-tools: [Read, Write, Edit, Bash, Agent]
---

# Accomplishments

Maintain a running log of what you've shipped — sourced from git, tickets, and work artifacts.

## Inputs

The user provides an operation as argument:
- **update** — Scan recent work and append new entries to the accomplishment log.
- **remind** — Summarize recent accomplishments for quick reference.

If no argument is provided, default to `update`.

## Accomplishment File

The accomplishment log lives at `~/Desktop/ACCOMPLISHMENTS.md`. If it doesn't exist, create it with this structure:

```markdown
# Accomplishments

> Running log of shipped work, auto-populated from git, tickets, and artifacts.

**Last Updated:** <date>

---

## Q1 2025 (Jan - Mar)

## Q2 2025 (Apr - Jun)

(sections added as needed)
```

## Operations

### update

#### Step 0 — Load Provider Config

Read `~/.claude/skills/_shared/ticket-provider.json` to determine the tracker provider.

#### Step 1 — Read Current Log

Read `~/Desktop/ACCOMPLISHMENTS.md`. Find the `Last Updated` date to know where to start scanning.

If the file doesn't exist, create it and set the scan window to the last 30 days.

#### Step 2 — Gather from Git

```bash
# Merged PRs since last update
gh pr list --author @me --state merged --json number,title,mergedAt,url --limit 50 2>/dev/null

# Commits on main since last update
git log --oneline --after="<last-updated-date>" --author="$(git config user.email)" main 2>/dev/null
```

#### Step 3 — Gather from Work Artifacts

```bash
# Find all context files
find ~/Desktop/pr_artifacts -name "context.md" -type f 2>/dev/null
```

Read each context file and extract:
- Ticket ID and title
- Status (look for "Done" or completed indicators)
- Summary of what was accomplished

#### Step 4 — Gather from Tickets

**If Jira:**
Use `searchJiraIssuesUsingJql` with JQL:
```
project = <project from config> AND assignee = currentUser() AND status = Done AND updated >= "<last-updated-date>" ORDER BY updated DESC
```

**If Linear:**
Use `list_issues` with:
- team: `<team from config>`
- assignee: `"me"`
- Filter to completed state
- Updated since last-updated date

#### Step 5 — Deduplicate & Merge

Cross-reference entries from git, artifacts, and tickets. Deduplicate by ticket ID — a single ticket should produce one accomplishment entry regardless of how many sources mention it.

#### Step 6 — Append New Entries

Determine the current quarter section (e.g., `## Q2 2025 (Apr - Jun)`). Create the section if it doesn't exist.

Append new entries in this format:
```markdown
- **2025-04-15** | PROJ-1234 | Built the data validation pipeline for customer events. Reduced data quality incidents by catching schema mismatches before they hit production.
```

Each entry should include:
- Date (completion/merge date)
- Ticket ID (if available)
- Description with impact (what was done + why it matters)

Update the `Last Updated` date at the top of the file.

#### Done

Report:
```
Accomplishments updated.
  New entries: <count>
  Quarter: <current quarter>
  Total entries: <total count across all quarters>
```

---

### remind

#### Step 1 — Read Log

Read `~/Desktop/ACCOMPLISHMENTS.md`.

#### Step 2 — Find Recent Quarter

Identify the most recently completed quarter (not the current one — the one before it). If we're in Q2, look at Q1. If we're early in Q1, look at Q4 of the previous year.

If the current quarter has entries but the previous one doesn't, use the current quarter instead.

#### Step 3 — Generate Highlight Summary

Present:

```
## Accomplishment Highlights — <Quarter>

### Top Accomplishments
- <most impactful items, rewritten for impact>

### By the Numbers
- <N> tickets completed
- <N> PRs merged
- <any quantitative metrics from the entries>

### Themes
- <patterns across the work — e.g., "heavy focus on data quality", "built out monitoring">
```

Keep it concise. This is designed for quick reference during standups, 1:1s, or self-reviews.
