---
name: ae-session-recap
description: End-of-session summary. Recaps accomplishments, tickets touched, flags loose ends.
user_invocable: true
allowed-tools: []
---

# Session Recap

Quick summary of what happened this session — no tools, no API calls, just a recap from conversation context.

## Instructions

This skill uses ONLY the current conversation context. Do not make any MCP calls, do not run any commands, do not read any files. Synthesize entirely from what has happened in this conversation.

## Output Format

### DONE
- <one bullet per accomplishment from this session>
- <include: what was built, fixed, configured, shipped, reviewed>
- <be specific — file names, ticket IDs, PR numbers where known>

### LOOSE ENDS
- <anything started but not finished>
- <blockers discovered during the session>
- <follow-up items that surfaced>

## Rules

- Skip any section that would be empty (e.g., if there are no loose ends, omit that section entirely).
- Keep bullets concise — one line each.
- Do not ask follow-up questions. Do not prompt for next steps.
- Do not editorialize or add commentary beyond the facts.
- This is a quick-reference summary, not a narrative.
