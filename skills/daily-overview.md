---
name: daily-overview
description: Start-of-day overview. Pulls sprint/cycle tickets, scans work artifacts, reads your TODO, checks open PRs, and surfaces meeting carry-forward items.
---

# Daily Overview

Sprint-focused birds-eye view of all your active work so you can hit
the ground running.

## Steps

### 0. Determine Ticket Provider

Read `~/.claude/skills/_shared/ticket-provider.json` to get the active
`provider` (`"jira"` or `"linear"`). Use the operation tables in
`~/.claude/skills/_shared/ticket-provider.md` to select the correct
MCP calls.

### 1. Preflight -- MCP Health Check

Test provider MCPs before relying on them. If any fail, surface the
fix and skip dependent steps gracefully.

**Jira (if provider is `"jira"`):** Call `getVisibleJiraProjects`.
**Linear (if provider is `"linear"`):** Call `list_teams` with limit=1.
**Meeting notes (optional):** Test your meeting MCP (Granola, etc.).

### 2. Pull Current Sprint/Cycle Tickets

#### If provider is `"jira"`:
```
assignee = currentUser()
AND sprint in openSprints()
AND status != "Done"
AND project = <from config>
ORDER BY status ASC, updated DESC
```
Run a second query with `status = "Done"` for the progress count.

#### If provider is `"linear"`:
```
assignee: "me", cycle: "current", team: <from config>, state: "started"
```
Run again with `state: "unstarted"` (merge results).
Run with `state: "completed"` for the progress count.

### 3. Scan Work Artifacts

Find all context files under your artifacts directory (e.g.,
`~/Desktop/pr_artifacts/`). Extract: ticket ID, category, status,
open todo count, last updated. Skip anything marked done unless
updated in the last 3 days.

### 4. Read TODO

Read your persistent TODO file. Count open vs done items.

### 5. Meeting Carry-Forward (optional)

If you have a meeting notes MCP, search for yesterday's meetings.
Surface action items or follow-ups. Offer to add them to TODO.

### 6. Check Open PRs & Review Queue

**Your PRs:** Check PRs referenced in context files via `gh pr view`.
**Review queue:** `gh search prs "user-review-requested:<you> repo:<org>/<repo> is:open"`

### 7. Synthesize

Cross-reference context files with sprint tickets. Present:

```
Good morning!

-- SPRINT PROGRESS -----
X of Y sprint tickets done

-- IN PROGRESS ---------
<ticket-id> -- <summary>
  Status:  <from context file if exists, else tracker status>
  Todos:   <X open>
  PR:      <link if open, or "no PR yet">
  Updated: <date>

-- TODO LIST -----------
X open items across Y sections

-- REVIEW QUEUE --------
X PRs waiting for your review

-- SUGGESTED FIRST MOVE --
<one concrete recommendation>

What do you want to start with?
```
