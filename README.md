# ae-workflow-kit

A Claude Code plugin that automates the overhead of analytics engineering — from morning orientation to PR creation to accomplishment tracking.

## Install

```
/plugin install krispix418/ae-workflow-kit
```

Then run `/ae-setup` to configure your environment.

## What You Get

| Command | Phase | What It Does |
|---------|-------|-------------|
| `/ae-setup` | One-time | Interactive wizard — detects MCPs, configures tracker, scaffolds artifacts |
| `/ae-daily-overview` | Start of day | Sprint status, PR queue, TODO, suggested first move |
| `/ae-start-ticket` | New ticket | Branch, artifact directory, context file, implementation plan |
| `/ae-resume-work` | Resume work | Context loading, git status, sibling ticket patterns |
| `/ae-update-tickets` | During work | Reconcile tracker statuses against branches/PRs/merges |
| `/ae-create-pr` | Ship it | Push branch, fill PR template, link ticket |
| `/ae-wrap-up` | End of session | Update context, commit/push, switch to main |
| `/ae-session-recap` | End of session | Quick summary of what you got done |
| `/ae-accomplishments` | Periodic | Log what you shipped from git + tickets + artifacts |

## How It Works

Skills chain together into a complete workflow. Each skill produces context that the next skill consumes.

```
/ae-daily-overview  →  /ae-start-ticket  →  [your work]  →  /ae-create-pr  →  /ae-wrap-up
                       /ae-resume-work       /ae-update-tickets                /ae-session-recap
```

### Provider Abstraction

Works with **both Jira and Linear** out of the box. A single config file determines which tracker to use.

```json
{ "provider": "jira" }   ← change to "linear" when your org migrates
```

The `/ae-setup` wizard configures this for you. When your org switches trackers, run setup again or edit one JSON file. Every skill switches automatically.

### Artifact Management

Every ticket gets a local artifact directory:

```
~/Desktop/pr_artifacts/
├── data-quality/
│   └── PROJ-1234/
│       ├── context.md              # Living status doc (created by start-ticket)
│       ├── implementation_plan.md   # Codebase-informed plan
│       └── validation_query.sql    # How to verify correctness
└── infra/
    └── PROJ-1400/
        └── context.md
```

Context files flow between skills — `/ae-start-ticket` creates them, `/ae-resume-work` reads them, `/ae-wrap-up` updates them. Knowledge compounds across related tickets in the same category.

### Graceful Degradation

Every external dependency (tracker MCP, GitHub CLI, meeting notes) is optional. If a service is down, skills skip that step and keep going.

## Plugin Structure

```
ae-workflow-kit/
├── .claude-plugin/
│   └── plugin.json                     # Plugin metadata
├── commands/
│   └── ae-setup.md                     # Interactive setup wizard
├── skills/
│   ├── daily-overview/SKILL.md         # /ae-daily-overview
│   ├── start-ticket/SKILL.md           # /ae-start-ticket
│   ├── resume-work/SKILL.md            # /ae-resume-work
│   ├── create-pr/SKILL.md              # /ae-create-pr
│   ├── update-tickets/SKILL.md         # /ae-update-tickets
│   ├── wrap-up/SKILL.md                # /ae-wrap-up
│   ├── session-recap/SKILL.md          # /ae-session-recap
│   ├── accomplishments/SKILL.md        # /ae-accomplishments
│   └── _provider-reference/
│       └── ticket-provider.md          # Jira ↔ Linear translation table
├── LICENSE
└── README.md
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- GitHub CLI (`gh`) — for PR operations
- At least one tracker MCP:
  - Jira: [Atlassian MCP](https://www.npmjs.com/package/@anthropic-ai/atlassian-mcp)
  - Linear: [Linear MCP](https://linear.app/docs/mcp)

## Concept Mapping

| Concept | Jira | Linear |
|---------|------|--------|
| Project unit | Project | Team |
| Sprint | Sprint | Cycle |
| Status change | 2-step transition | Direct state set |
| Story points | Custom field | First-class `estimate` |
| Create + assign + sprint | 3 API calls | 1 API call |

## Design Principles

**Context flows forward.** Each skill produces artifacts the next skill consumes. `/ae-start-ticket` writes context → `/ae-resume-work` reads it → `/ae-wrap-up` updates it → next session picks up cleanly.

**Skills don't replace work.** They handle overhead — context switching, status tracking, PR boilerplate, tracker hygiene. The engineering is yours.

**Provider-agnostic.** Tracker operations go through an abstraction layer. Switch from Jira to Linear by changing one config value.

**Graceful degradation.** If Jira is down, `/ae-start-ticket` asks you for a description and keeps going. No skill blocks on a flaky integration.

## License

MIT
