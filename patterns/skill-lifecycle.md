# Skill Lifecycle Pattern

## The Idea

Individual skills are useful. Skills that chain together into a
workflow are transformative. This kit organizes skills around the
natural lifecycle of engineering work.

## The Lifecycle

```
┌─────────────────────────────────────────────────────┐
│                    START OF DAY                       │
│                                                       │
│  /daily-overview                                      │
│    Pull sprint tickets, scan artifacts, check PRs     │
│    "Here's your world. What do you want to tackle?"   │
│                                                       │
├───────────────────────┬─────────────────────────────┤
│    NEW TICKET         │    EXISTING TICKET           │
│                       │                              │
│  /start-ticket        │  /resume-work                │
│    Branch, artifacts, │    Load context, git status,  │
│    context file,      │    orient on todos            │
│    implementation     │                              │
│    plan               │                              │
├───────────────────────┴─────────────────────────────┤
│                                                       │
│                    DOING THE WORK                     │
│                                                       │
│  Your actual engineering work happens here.           │
│  Skills support but don't replace the work.           │
│                                                       │
│  /todo          — track tasks                         │
│  /save-validation — capture validation queries        │
│                                                       │
├───────────────────────────────────────────────────────┤
│                    SHIPPING                           │
│                                                       │
│  /create-pr                                           │
│    Push branch, fill PR template, link ticket         │
│                                                       │
│  /update-tickets                                      │
│    Reconcile tracker with reality                     │
│                                                       │
├───────────────────────────────────────────────────────┤
│                    END OF SESSION                     │
│                                                       │
│  /wrap-up                                             │
│    Update context, commit/push, switch to main        │
│                                                       │
│  /session-recap                                       │
│    What you got done, what's dangling                 │
│                                                       │
├───────────────────────────────────────────────────────┤
│                    PERIODIC                           │
│                                                       │
│  /accomplishments update    — weekly/biweekly         │
│    Scan all sources, log what you shipped             │
│                                                       │
│  /accomplishments remind    — quarterly               │
│    Highlight reel for reviews                         │
│                                                       │
└───────────────────────────────────────────────────────┘
```

## Key Design Principles

### 1. Context Flows Forward

Each skill produces artifacts that the next skill consumes:
- `/start-ticket` writes context.md -> `/resume-work` reads it
- `/resume-work` surfaces todos -> you work on them
- `/wrap-up` updates context.md -> next `/resume-work` picks up cleanly
- `/create-pr` reads commits + diff -> fills PR template
- `/accomplishments` reads git + artifacts + tracker -> logs your wins

### 2. Skills Don't Replace Work

Skills handle the overhead — context switching, status tracking, PR
boilerplate, tracker hygiene. The actual engineering work is yours.

### 3. Graceful Degradation

Every external dependency (tracker, GitHub, meeting notes) is optional.
If Jira is down, `/start-ticket` asks you for a description and keeps
going. If GitHub is unreachable, `/daily-overview` skips the PR section.
No skill should block on a flaky integration.

### 4. Provider Agnostic

Tracker operations go through the provider abstraction layer
(`providers/ticket-provider.json`). Switch from Jira to Linear by
changing one config value.

## Customization

These templates are starting points. The skills that work best are
the ones tuned to your specific stack:

- **dbt projects:** Add model validation, schema checks, lineage
  awareness to `/start-ticket`
- **Multi-repo setups:** Add cross-repo dependency tracking
- **On-call rotations:** Add Slack channel checks to `/daily-overview`
- **Team-specific PR templates:** Customize `/create-pr` for your repo

The pattern stays the same. The details are yours.
