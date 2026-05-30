# Ticket Provider Abstraction Layer

All skills that interact with a project management tool (Jira or Linear)
MUST use this reference to determine which MCP calls to make. Never
hardcode Jira or Linear calls directly.

## How to Use

1. Read `~/.claude/skills/_shared/ticket-provider.json` to get the
   active `provider` value (`"jira"` or `"linear"`)
2. Use the operation tables below to pick the correct MCP call
3. Use the config values (`project`, `cloudId`, `team`, `teamId`) as
   parameters

---

## Concept Mapping

| Concept | Jira | Linear |
|---------|------|--------|
| Project/workspace unit | Project (`YOUR-PROJECT`) | Team (TBD — set in config once BIDE migrates) |
| Sprint | Sprint (`openSprints()`) | Cycle (`current`) |
| Issue identifier | `YOUR-PROJECT-1234` | `TEAM-123` (team key prefix) |
| Issue type | `issueTypeName` (Task, Bug, Story) | Label (Linear has no built-in issue types) |
| Status change | Transition (2-step: get transitions → apply) | State (direct set — just pass state name) |
| Story points | `customfield_10016` (custom field) | `estimate` (first-class field) |
| Assignee lookup | `lookupJiraAccountId` by email | `get_user` by email (returns id directly) |
| Auth context | `cloudId` required on every call | Not needed |
| Query language | JQL string | Structured filter parameters |
| Ticket URL | `https://your-org.atlassian.net/browse/YOUR-PROJECT-1234` | `https://linear.app/your-org/issue/TEAM-123` |

---

## Operation Translation Table

### 1. Search Current Sprint/Cycle Tickets

**Use case:** `/good-morning`, `/update-jira`, `/bragbook`, `/todo update`

#### Jira
```
mcp__atlassian__searchJiraIssuesUsingJql
  cloudId: <from config>
  jql: "assignee = currentUser() AND sprint in openSprints() AND status != 'Done' AND project = YOUR-PROJECT ORDER BY status ASC, updated DESC"
  fields: ["summary", "status", "priority", "customfield_10016"]
```

#### Linear
```
mcp__linear__list_issues
  assignee: "me"
  cycle: "current"           # equivalent of openSprints()
  team: <from config>
  state: "started"           # or omit and filter client-side; no != operator
```

**Notes:**
- Linear `list_issues` cannot do `status != Done` directly. Either:
  - Query with `state: "started"` (gets In Progress, In Review, Blocked, Validating)
  - Query with `state: "unstarted"` (gets Ready, Planned, Up Next)
  - Run both and merge results
  - Or query without state filter and exclude Done/Canceled client-side
- For the "done count" query in `/good-morning`, use `state: "completed"`

---

### 2. Get Ticket Details

**Use case:** `/start-ticket`, `/resume-work`

#### Jira
```
mcp__atlassian__getJiraIssue
  cloudId: <from config>
  issueIdOrKey: "YOUR-PROJECT-1234"
  fields: ["summary", "description", "status", "labels"]
```

#### Linear
```
mcp__linear__get_issue
  id: "TEAM-123"
```

**Notes:**
- Linear returns summary as `title`, description as `description`
- Linear returns `state` object instead of `status`
- Linear returns `labels` as array of label objects

---

### 3. Create Ticket

**Use case:** `/todo update` (create tickets for ticketless items)

#### Jira
```
mcp__atlassian__lookupJiraAccountId   # step 1: get assignee ID
  cloudId: <from config>
  query: "you@org.com"

mcp__atlassian__createJiraIssue       # step 2: create
  cloudId: <from config>
  projectKey: "YOUR-PROJECT"
  issueTypeName: "Task"
  summary: "..."
  assignee_account_id: <from step 1>

mcp__atlassian__editJiraIssue         # step 3: move into sprint
  cloudId: <from config>
  issueIdOrKey: <new ticket key>
  fields: { sprint: { id: <sprint-id> } }
```

#### Linear
```
mcp__linear__save_issue               # single call does it all
  team: <from config>
  title: "..."
  assignee: "me"
  cycle: "current"                    # adds to current cycle directly
```

**Notes:**
- Linear collapses 3 Jira calls into 1
- No need to look up assignee ID — `"me"` works
- No need for separate sprint assignment — `cycle` param handles it
- No `issueTypeName` equivalent; use `labels` if categorization needed

---

### 4. Update Ticket Fields

**Use case:** `/update-jira` (set story points), `/todo update`

#### Jira
```
mcp__atlassian__editJiraIssue
  cloudId: <from config>
  issueIdOrKey: "YOUR-PROJECT-1234"
  fields: { "customfield_10016": 3 }
```

#### Linear
```
mcp__linear__save_issue
  id: "TEAM-123"
  estimate: 3
```

**Notes:**
- Linear uses `save_issue` with `id` for updates (same endpoint as create)
- Story points → `estimate` (first-class, no custom field gymnastics)

---

### 5. Transition / Change Status

**Use case:** `/start-ticket` (→ In Progress), `/update-jira` (→ Done, In Review, etc.)

#### Jira
```
mcp__atlassian__getTransitionsForJiraIssue   # step 1: discover valid transitions
  cloudId: <from config>
  issueIdOrKey: "YOUR-PROJECT-1234"

mcp__atlassian__transitionJiraIssue          # step 2: apply transition
  cloudId: <from config>
  issueIdOrKey: "YOUR-PROJECT-1234"
  transition: { id: <transition-id-from-step-1> }
```

#### Linear
```
mcp__linear__save_issue                      # single call, direct state set
  id: "TEAM-123"
  state: "In Progress"                       # just pass the state name
```

**Notes:**
- Linear has NO transition workflow — set any state directly by name
- No need for a "get transitions" preflight call
- 2 Jira calls → 1 Linear call
- Valid state names (for Analytics team): Triage, Backlog, Ready, Planned, Up Next, In Progress, Blocked, In Review, Validating, Done, Canceled, Duplicate
- State names may vary by team — use `list_issue_statuses` to discover if unsure

---

### 6. Search Resolved Tickets

**Use case:** `/bragbook update` (scan accomplishments)

#### Jira
```
mcp__atlassian__searchJiraIssuesUsingJql
  cloudId: <from config>
  jql: "assignee = 'you@org.com@your-org.com' AND status in (Done, Closed) AND resolved >= '2026-01-01' ORDER BY resolved ASC"
```

#### Linear
```
mcp__linear__list_issues
  assignee: "me"
  team: <from config>
  state: "completed"
  updatedAt: "2026-01-01"        # ISO-8601 date; filters "updated after"
```

**Notes:**
- Linear doesn't have a `resolved` date — `updatedAt` is the closest proxy
- For more precise date filtering, you may need to filter client-side
- `state: "completed"` covers Done; also check `state: "canceled"` if those matter

---

### 7. Look Up User

**Use case:** `/todo update` (get assignee ID for ticket creation)

#### Jira
```
mcp__atlassian__lookupJiraAccountId
  cloudId: <from config>
  query: "you@org.com"
```

#### Linear
```
mcp__linear__get_user
  query: "you@org.com@your-org.com"    # accepts email, name, or ID
```

**Notes:**
- Not needed for self-assignment in Linear — just use `assignee: "me"`
- Only needed if assigning to someone else

---

### 8. List Available Statuses

**Use case:** `/update-jira` (validate target status exists)

#### Jira
```
mcp__atlassian__getTransitionsForJiraIssue
  cloudId: <from config>
  issueIdOrKey: "YOUR-PROJECT-1234"
```

#### Linear
```
mcp__linear__list_issue_statuses
  team: <from config>
```

**Notes:**
- Jira transitions are per-issue (depend on current state); Linear statuses are per-team (always the same set)
- Linear statuses can be cached per session — they don't change per-issue

---

## Skill Migration Checklist

When updating a skill to use this abstraction:

1. Add this line to the skill's Steps (as the first action):
   `Read ~/.claude/skills/_shared/ticket-provider.json to determine provider.`

2. Add this line after:
   `Use the operation tables in ~/.claude/skills/_shared/ticket-provider.md
   to select the correct MCP calls for the active provider.`

3. Replace all hardcoded Jira MCP calls with provider-conditional logic

4. Replace hardcoded `YOUR-PROJECT` references with the config's project/team value

5. Replace `your-org.atlassian.net/browse/` URLs with the config's ticketUrlBase

6. Update ticket ID regex if needed:
   - Jira: `[A-Z]+-[0-9]+` (e.g., YOUR-PROJECT-1234)
   - Linear: `[A-Z]+-[0-9]+` (e.g., BIDE-123) — same pattern, different prefix

### Skills requiring migration (in priority order):

| Skill | Jira Operations Used | Migration Complexity |
|-------|---------------------|---------------------|
| `/good-morning` | Search sprint tickets (2 queries) | Low — swap 2 calls |
| `/start-ticket` | Get ticket, transition status | Low — swap 2-3 calls |
| `/update-jira` | Search tickets, transition, edit fields | Medium — rename to `/update-tickets`, swap 3+ calls |
| `/todo` | Search, create, edit tickets | Medium — swap 3 call patterns |
| `/bragbook` | Search resolved tickets | Low — swap 1 query |
| `/create-pr` | No MCP calls, but refs Jira URL format | Trivial — swap URL template |
| `/resume-work` | No MCP calls, reads context files | Trivial — just URL references |

---

## Call Count Comparison

| Operation | Jira Calls | Linear Calls | Savings |
|-----------|-----------|-------------|---------|
| Search sprint tickets | 1 | 1-2 | ~same |
| Get ticket details | 1 | 1 | same |
| Create + assign + add to sprint | 3 | 1 | 67% fewer |
| Change status | 2 | 1 | 50% fewer |
| Update fields (story points) | 1 | 1 | same |
| **Typical /update-jira run (5 tickets)** | **~15** | **~7** | **53% fewer** |

Linear's simpler API means fewer round-trips per operation, which also means
fewer tokens spent on tool call/response overhead.

---

## Migration Day Checklist

When BIDE gets a Linear team:

1. Update `ticket-provider.json`:
   - Set `provider` to `"linear"`
   - Set `linear.team` to the team name
   - Set `linear.teamId` to the team UUID
2. Run `list_issue_statuses` for the new team and verify state names match
   the ones referenced in skills (In Progress, In Review, Done, etc.)
3. Run `list_cycles` to verify the team uses cycles (sprint equivalent)
4. Test each skill with a dry-run against a test ticket
5. Rename `/update-jira` to `/update-tickets` (or keep as alias)
