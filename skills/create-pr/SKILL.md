---
name: ae-create-pr
description: Create a GitHub Pull Request. Pushes branch, auto-generates title, fills PR template, links ticket.
user_invocable: true
argument-hint: "[optional title override]"
allowed-tools: [Read, Bash]
---

# Create PR

Push the current branch and open a pull request with a filled-out template.

## Inputs

Optional title override as argument. If not provided, the title is auto-generated from the ticket ID and commit messages.

## Steps

### Step 1 — Gather Context

```bash
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"

# Commit history on this branch
git log --oneline origin/main..$BRANCH 2>/dev/null || git log --oneline origin/master..$BRANCH 2>/dev/null

# Diff stats
git diff --stat origin/main..$BRANCH 2>/dev/null || git diff --stat origin/master..$BRANCH 2>/dev/null

# Full diff for understanding changes
git diff origin/main..$BRANCH 2>/dev/null || git diff origin/master..$BRANCH 2>/dev/null
```

### Step 2 — Extract Ticket ID

Extract the ticket ID from the branch name:
```bash
echo "$BRANCH" | grep -oE '[A-Z]+-[0-9]+'
```

If no ticket ID is found in the branch name, proceed without ticket linking.

### Step 3 — Load Config for Ticket URL

Read `~/.claude/skills/_shared/ticket-provider.json` to get `ticketUrlBase`.

Construct the ticket URL: `<ticketUrlBase>/<ticket-id>`

Also attempt to read the context file for additional context:
```bash
find ~/Desktop/pr_artifacts -path "*/<ticket-id>/context.md" -type f 2>/dev/null
```

### Step 4 — Derive PR Title

If a title override was provided, use it (prefixed with ticket ID if available).

Otherwise, generate from:
- Ticket ID as prefix (e.g., `PROJ-1234:`)
- Descriptive suffix derived from commit messages and diff
- Keep under 70 characters total

Format: `PROJ-1234: Short descriptive title`

### Step 5 — Push to Remote

```bash
git push -u origin $BRANCH
```

If the push fails, report the error and stop.

### Step 6 — Build PR Body

Check if the repo has a PR template:
```bash
cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null || cat .github/pull_request_template.md 2>/dev/null
```

If a template exists, fill it in. Otherwise, use this default structure:

```markdown
## Description
<concise summary of what this PR does and why>

## Ticket
<ticket-url> (or "N/A" if no ticket)

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Enhancement / refactor
- [ ] Breaking change
- [ ] Documentation
- [ ] Other: ___

## Changes
- <bullet per logical change>

## Testing
- <how changes were validated>

## Checklist
- [ ] Code follows project conventions
- [ ] Self-reviewed
- [ ] Tests added/updated (if applicable)
- [ ] Documentation updated (if applicable)
```

Fill each section based on the diff, commit messages, and context file.

### Step 7 — Create PR

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
<filled PR body>
EOF
)"
```

If `gh` is not authenticated or not available, print the PR body and title so the user can create it manually.

### Done

Always return the PR URL:
```
PR created: <url>

Title: <title>
Ticket: <ticket-url>
Changes: <N> files changed, <insertions> insertions, <deletions> deletions
```
