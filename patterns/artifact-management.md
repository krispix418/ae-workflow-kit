# Artifact Management Pattern

## The Problem

Work context lives in too many places — git branches, ticket descriptions,
Slack threads, meeting notes, local scratch files. When you resume work
days later, you spend 20 minutes reconstructing where you were.

## The Solution

A local artifact directory organized by domain category and ticket ID.
Everything related to a ticket lives together.

```
~/Desktop/pr_artifacts/
├── comp/
│   ├── PROJ-1234/
│   │   ├── context.md              # Living status doc
│   │   ├── implementation_plan.md   # Generated at ticket start
│   │   ├── validation_query.sql     # How to verify correctness
│   │   └── impact_analysis.md       # Scope and downstream effects
│   └── PROJ-1235/
│       └── context.md
├── data-quality/
│   └── PROJ-1300/
│       ├── context.md
│       └── validation_query.sql
└── infra/
    └── PROJ-1400/
        └── context.md
```

## The Context File

Every ticket gets a `context.md` — the single source of truth for
that unit of work. It's created by `/start-ticket` and updated by
`/wrap-up`.

```markdown
# PROJ-1234: Short summary

**Branch:** `PROJ-1234`
**Repos:** `repo-a`, `repo-b`
**Category:** `comp`
**Ticket:** [PROJ-1234](https://tracker.example.com/PROJ-1234)
**Created:** 2026-05-01
**Last Updated:** 2026-05-15

---

## Goals

What this ticket is trying to accomplish.

## Todo

- [x] Step that's done
- [ ] Step that's next
- [ ] Step after that

## Status

Current state in plain language. Updated each session.

## Technical Notes

Gotchas, decisions, things the next person (or future you) needs to know.
```

## Why Categories

Tickets in the same domain share patterns. When you start a new comp
ticket, the `/start-ticket` skill reads sibling context files and
surfaces relevant gotchas from Technical Notes. Knowledge compounds
across tickets.

## Why Not in the Repo

- No repo clutter (artifacts are gitignored)
- Cross-repo tickets have one home (a ticket touching 3 repos has
  one context file, not three)
- Survives branch deletion
- Easy to search: "How did we handle PROJ-1234?" → look in pr_artifacts

## Retention

Keep everything. Disk is cheap, context is expensive. Six months from
now when a similar ticket comes up, you'll have the validation queries,
the impact analysis, and the Technical Notes that explain why you made
the decisions you did.
