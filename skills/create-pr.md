---
name: create-pr
description: Create a GitHub Pull Request. Pushes the branch, auto-generates the title, fills out the PR template using commits and diff context, and links the ticket.
---

# Create PR

Push the current branch and open a pull request against main.

## Steps

### 1. Gather Context

```bash
git branch --show-current
git log main..HEAD --oneline
git diff main..HEAD --name-only
git diff main..HEAD --stat
```

### 2. Extract Ticket ID

Parse the branch name for `[A-Z]+-[0-9]+`.

Read `~/.claude/skills/_shared/ticket-provider.json` to get
`ticketUrlBase` for constructing ticket links.

### 3. Derive PR Title

Start with ticket ID, then a descriptive suffix from the branch
name or most descriptive commit. Keep under 70 characters.

### 4. Push to Remote

```bash
git push -u origin $(git branch --show-current)
```

### 5. Fill Out PR Template

Use your repo's PR template. Fill each section concisely:
- **Description**: 2-4 sentences on what changed and why
- **Ticket link**: `<ticketUrlBase>/<ticket-id>`
- **Type of change**: check applicable boxes from the diff
- **Testing**: be specific and honest
- **Checklist**: only check items that truthfully apply

### 6. Create the PR

```bash
gh pr create --title "<title>" --body "<filled template>" --base main
```

### 7. Report

Always return the PR URL. Non-negotiable.
