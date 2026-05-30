# ae-workflow-kit

A Claude Code skill framework that automates the overhead of analytics engineering — from morning orientation to PR creation to accomplishment tracking.

## What This Is

A collection of Claude Code skill templates and patterns that chain together into a complete workflow for analytics engineers (or any engineer working across tickets, PRs, and project trackers).

Instead of building skills one at a time, this kit provides the full lifecycle:

```
/daily-overview  ->  /start-ticket  ->  [do the work]  ->  /create-pr  ->  /wrap-up
                     /resume-work       /todo                              /session-recap
                                        /update-tickets
                                                                           /accomplishments
```

Each skill produces context that the next skill consumes. Your work artifacts persist across sessions, so picking up where you left off is instant.

## The Skill Lifecycle

| Phase | Skill | What It Automates |
|-------|-------|-------------------|
| Start of day | `daily-overview` | Sprint status, PR queue, TODO, meeting carry-forward |
| New ticket | `start-ticket` | Branch, artifact directory, context file, implementation plan |
| Resume work | `resume-work` | Context loading, git status, sibling ticket patterns |
| During work | `update-tickets`, `todo` | Tracker sync, task management |
| Ship it | `create-pr` | PR template filling, ticket linking |
| End of session | `wrap-up`, `session-recap` | Context persistence, branch cleanup |
| Career growth | `accomplishments` | Accomplishment logging from git + tickets + artifacts |

## Key Features

### Provider Abstraction Layer

Skills work with **both Jira and Linear** out of the box. A single config file determines which tracker to use — switch providers by changing one line.

```json
{
  "provider": "jira",
  "jira": { "project": "YOUR-PROJECT", "cloudId": "your-org.atlassian.net" },
  "linear": { "team": "Your Team", "teamId": "uuid" }
}
```

When your org migrates trackers, you change the config. Every skill switches automatically. See [`providers/`](providers/) for the full translation table.

### Artifact Management

Every ticket gets a local artifact directory organized by domain:

```
~/Desktop/pr_artifacts/
├── comp/
│   └── PROJ-1234/
│       ├── context.md              # Living status doc
│       ├── implementation_plan.md
│       └── validation_query.sql
└── data-quality/
    └── PROJ-1300/
        └── context.md
```

Context files flow between skills — `/start-ticket` creates them, `/resume-work` reads them, `/wrap-up` updates them. Knowledge compounds across related tickets in the same category. See [`patterns/artifact-management.md`](patterns/artifact-management.md) for the full pattern.

### Graceful Degradation

Every external dependency (tracker, GitHub, meeting notes MCP) is optional. If a service is down, skills skip that step and keep going. No skill blocks on a flaky integration.

## Setup

### 1. Copy provider config and reference

```bash
mkdir -p ~/.claude/skills/_shared
cp providers/ticket-provider.json ~/.claude/skills/_shared/
cp providers/ticket-provider.md ~/.claude/skills/_shared/
```

### 2. Edit the config for your org

Set your project key, cloud ID (Jira), or team name/ID (Linear) in `ticket-provider.json`.

### 3. Copy and customize skills

```bash
cp skills/*.md ~/.claude/skills/
```

Each skill is a starting point — customize for your stack, repos, and team conventions.

### 4. Set up artifacts directory

```bash
mkdir -p ~/Desktop/pr_artifacts
```

## File Structure

```
ae-workflow-kit/
├── README.md
├── LICENSE
├── providers/
│   ├── ticket-provider.json         # Provider toggle + org settings
│   └── ticket-provider.md          # Jira <-> Linear operation mapping
├── skills/
│   ├── daily-overview.md            # Start of day
│   ├── start-ticket.md              # New ticket kickoff
│   ├── resume-work.md               # Session resumption
│   ├── create-pr.md                 # PR creation
│   ├── update-tickets.md            # Tracker reconciliation
│   ├── wrap-up.md                   # End of session
│   ├── session-recap.md             # Session summary
│   └── accomplishments.md           # Accomplishment tracking
└── patterns/
    ├── artifact-management.md       # The pr_artifacts system
    └── skill-lifecycle.md           # How skills chain together
```

## Concept Mapping (Jira vs Linear)

| Concept | Jira | Linear |
|---------|------|--------|
| Project unit | Project (`PROJ`) | Team |
| Sprint | Sprint | Cycle |
| Status change | 2-step transition | Direct state set |
| Story points | Custom field | First-class `estimate` |
| Create + assign + sprint | 3 API calls | 1 API call |

## Why This Exists

Analytics engineers spend a surprising amount of time on overhead — remembering what they were working on, filling out PR templates, keeping trackers in sync, preparing for reviews. This kit automates that overhead so you can focus on the actual engineering.

The skills are designed around two beliefs:

1. **Context is expensive, disk is cheap.** Save everything. Your future self will thank you when a similar ticket comes up six months from now.

2. **Skills should chain, not stand alone.** A daily overview that feeds into ticket selection that feeds into context loading that feeds into PR creation — that's a workflow. Individual skills are nice; a workflow is transformative.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- GitHub CLI (`gh`) for PR operations
- MCP server for your tracker:
  - Jira: [Atlassian MCP](https://www.npmjs.com/package/@anthropic-ai/atlassian-mcp)
  - Linear: [Linear MCP](https://linear.app/docs/mcp)

## License

MIT
