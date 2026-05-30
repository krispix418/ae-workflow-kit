---
description: Interactive setup wizard for ae-workflow-kit. Detects your environment, configures your tracker, and scaffolds your artifact directories.
allowed-tools: [Read, Write, Bash, AskUserQuestion]
---

# AE Workflow Kit — Setup

Interactive wizard that configures the full workflow in under 5 minutes.

## Steps

### 1. Welcome

```
Welcome to ae-workflow-kit!

This will set up your analytics engineering workflow:
  - Ticket provider (Jira or Linear)
  - Artifact directories for your work
  - All skills configured and ready to use

Let's go.
```

### 2. Detect Environment

Check what's already available:

```bash
# Check for Jira MCP
# Try calling getVisibleJiraProjects or similar — if it works, Jira is available

# Check for Linear MCP
# Try calling list_teams — if it works, Linear is available

# Check for GitHub CLI
gh auth status 2>&1

# Check for existing artifact directories
ls ~/Desktop/pr_artifacts 2>/dev/null

# Check for existing config
cat ~/.claude/skills/_shared/ticket-provider.json 2>/dev/null
```

Report what was found:
```
Environment detected:
  GitHub CLI:    ✓ logged in as <user>
  Jira MCP:     ✓ connected / ✗ not found
  Linear MCP:   ✓ connected / ✗ not found
  Artifacts:    ✓ ~/Desktop/pr_artifacts exists / ✗ not found
  Config:       ✓ already configured / ✗ needs setup
```

### 3. Choose Ticket Provider

If both Jira and Linear are available, ask which to use as primary.
If only one is available, confirm it.
If neither is available, set up config with placeholders and note
which MCPs need to be installed.

Use AskUserQuestion:
- "Which ticket provider does your team use?"
- Options: Jira (if detected), Linear (if detected), Neither yet

### 4. Configure Provider

#### If Jira:
Ask for:
- **Project key** (e.g., `PROJ`, `DATA`) — "What's your Jira project key?"
- **Cloud ID** — attempt to detect from `getVisibleJiraProjects` response,
  or ask: "What's your Atlassian site? (e.g., your-org.atlassian.net)"

#### If Linear:
Ask for:
- **Team name** — show teams from `list_teams` and let them pick
- **Team ID** — auto-filled from selection

### 5. Configure Artifact Categories

Ask: "What domains/categories do you work across?"

Show examples:
```
Categories organize your work artifacts by domain. Examples:
  - comp, infra, data-quality, subscriptions
  - backend, frontend, api, database
  - etl, models, dashboards, alerts
```

Accept a comma-separated list or let them type categories.
If `~/Desktop/pr_artifacts` already exists with subdirectories,
show them and ask if they want to keep the existing structure.

### 6. Configure Repos

Ask: "Where do your git repos live?"

Options:
- Single repo (provide path)
- Multi-repo setup (provide base directory, e.g., `~/Desktop/kbi/repos/`)

If multi-repo, detect repos in the directory:
```bash
ls <base-dir>
```

### 7. Write Config

Write `~/.claude/skills/_shared/ticket-provider.json` with the
collected values.

```bash
mkdir -p ~/.claude/skills/_shared
```

Write the config file with provider, project/team, URLs.

Also copy the provider reference doc:
```bash
# Copy ticket-provider.md to _shared if it doesn't exist
```

### 8. Scaffold Artifact Directories

```bash
mkdir -p ~/Desktop/pr_artifacts
# Create category subdirectories from step 5
```

### 9. Verify

Run a quick smoke test:
- Read the config back
- Test a simple query against the configured provider
  (e.g., search for 1 ticket assigned to me)
- Confirm artifact directory structure

### 10. Done

```
ae-workflow-kit is ready!

Provider:    <jira/linear> (<project/team>)
Artifacts:   ~/Desktop/pr_artifacts/ (<N> categories)
Repos:       <path> (<N> repos detected)

Available commands:
  /ae-daily-overview    Start your day
  /ae-start-ticket      Kick off a new ticket
  /ae-resume-work       Resume where you left off
  /ae-create-pr         Open a pull request
  /ae-update-tickets    Sync tracker with reality
  /ae-wrap-up           End your session cleanly
  /ae-session-recap     Quick session summary
  /ae-accomplishments   Track what you've shipped

Run /ae-daily-overview to start your first morning.
```
