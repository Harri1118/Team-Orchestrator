---
description: Create Linear tickets from user stories with full acceptance criteria
argument-hint: [path to plan, or "interactive" to build a ticket from scratch]
allowed-tools: Read, Write, Glob, Grep, Bash, mcp__linear__*, mcp__agent_grid_workers__*
---

# Ticket Maker — Linear Issue Creator

You are a technical project manager. Your job is to take user stories from an XP plan and create well-structured Linear tickets with full acceptance criteria, implementation context, and traceability.

## Canvas Integration

### On Start — Read upstream panes
1. Call `list_canvas_panes()` and look for `[Plan]` panes.
2. If `[Plan] ... — XP Plan` exists, `associate_pane()` and `read_pane()` to load user stories and metadata.
3. Also check for `[Preflight] ... — Requirements` to note any blockers in ticket descriptions.
4. Fall back to `ref/plans/` and `ref/tickets/` files if no canvas panes exist.

### On Finish — Create output pane
1. `spawn_note_pane()`, title: `[Tickets] <project> — Manifest`, color: `green`.
2. Content: the ticket summary table with Linear IDs, titles, priorities, and URLs.
3. Also write to `ref/tickets/<project>-tickets.md`.

Tell the user: "Ticket manifest is on the canvas. `/build` and `/roadmap` will read it automatically."

---

If $ARGUMENTS is "interactive" or empty with no plan files, skip to **Interactive Mode** below.

## Step 0 — Connect to Linear

Before anything else, establish the Linear context using the **Linear MCP server** (`mcp__linear__*` tools).

The MCP server is `linear-mcp` (configured in `.mcp.json`). It exposes **dedicated tools** for each operation — not raw GraphQL. Use these tools directly.

### Available Tools

#### Discovery (run these first)
| Tool | Purpose | Parameters |
|------|---------|------------|
| `mcp__linear__linear_get_teams` | Get all teams with their states and labels | None |
| `mcp__linear__linear_get_user` | Get current authenticated user info | None |
| `mcp__linear__linear_list_projects` | List all projects (optional filter) | `teamId?` |
| `mcp__linear__linear_get_project` | Get a specific project | `id` |

#### Issue Operations
| Tool | Purpose | Required Parameters |
|------|---------|-------------------|
| `mcp__linear__linear_create_issue` | Create a single issue | `title`, `description`, `teamId` |
| `mcp__linear__linear_create_issues` | Create multiple issues at once (bulk) | `issues[]` (array of {title, description, teamId}) |
| `mcp__linear__linear_search_issues` | Search/filter issues | `query?`, `teamIds?`, `assigneeIds?`, `states?` |
| `mcp__linear__linear_search_issues_by_identifier` | Get issue by ID (e.g., ENG-123) | `identifier` |
| `mcp__linear__linear_bulk_update_issues` | Update multiple issues | `issueIds[]`, update fields |
| `mcp__linear__linear_delete_issue` | Delete a single issue | `id` (identifier like ENG-123) |
| `mcp__linear__linear_delete_issues` | Bulk delete issues | `ids[]` |

#### Project Operations
| Tool | Purpose | Required Parameters |
|------|---------|-------------------|
| `mcp__linear__linear_create_project_with_issues` | Create a project AND its issues together | `project` ({name, teamIds[]}), `issues[]` |

#### Comment Operations
| Tool | Purpose | Required Parameters |
|------|---------|-------------------|
| `mcp__linear__linear_create_comment` | Add comment to an issue | `issueId`, `body` |
| `mcp__linear__linear_update_comment` | Edit a comment | `id`, `body` |
| `mcp__linear__linear_delete_comment` | Delete a comment | `id` |
| `mcp__linear__linear_resolve_comment` | Mark comment as resolved | `id` |

### Bootstrap Sequence

**1. Get teams (includes states and labels):**
```
mcp__linear__linear_get_teams()
```
Returns all teams with their workflow states and labels. If multiple teams exist, ask the user which one. Save the `teamId`.

**2. Get current user:**
```
mcp__linear__linear_get_user()
```

**3. Get existing projects:**
```
mcp__linear__linear_list_projects()
```

### Creating a Single Ticket
```
mcp__linear__linear_create_issue({
  title: "[Activity] Story title",
  description: "## User Story\n...",
  teamId: "<team-id>",
  priority: 2,
  assigneeId: "<user-id>",      // optional
  createAsUser: "Team-Orchestrator"  // optional display name
})
```
Priority values: 0 = No priority, 1 = Urgent, 2 = High, 3 = Medium, 4 = Low.

### Creating a Project with All Tickets at Once (preferred)
```
mcp__linear__linear_create_project_with_issues({
  project: {
    name: "Daily Habit Tracker — MVP",
    description: "Iteration 1: Core habit loop",
    teamIds: ["<team-id>"]
  },
  issues: [
    { title: "[Auth] Project setup", description: "...", teamId: "<team-id>", priority: 2 },
    { title: "[Habits] CRUD operations", description: "...", teamId: "<team-id>", priority: 2 },
    ...
  ]
})
```
This is the most efficient path — creates the project and all tickets in one call.

### Bulk Create (without a project)
```
mcp__linear__linear_create_issues({
  issues: [
    { title: "...", description: "...", teamId: "...", projectId: "...", labelIds: ["..."] },
    ...
  ]
})
```

> **If `mcp__linear__*` tools are not available**, stop and tell the user to:
> 1. Copy `.mcp.json.example` to `.mcp.json`
> 2. Get a Linear API key from Settings > API > Personal API keys
> 3. Set it in `.mcp.json` under `LINEAR_API_KEY`
> 4. Run `/mcp` to reconnect

## Step 1 — Load the plan

If $ARGUMENTS points to a file, read it. Otherwise, find the most recent `*-plan.md` file in `ref/plans/` and read it.

Extract all user stories and their metadata (priority, size, acceptance criteria, edge cases, technical notes).

## Step 2 — Map stories to Linear tickets

For each user story, prepare a Linear ticket:

### Title
`[<Activity>] <Concise action description>`
Example: `[Auth] User can sign up with email and password`

### Description (markdown)
```markdown
## User Story
As a <persona>, I want to <action> so that <benefit>.

## Context
- **Activity:** <from story map>
- **User Task:** <from story map>
- **Iteration:** <which iteration this is planned for>
- **Size:** <S/M/L/XL>
- **Brief reference:** <link to ref/briefs/ file>

## Acceptance Criteria
- [ ] Given <context>, when <action>, then <result>
- [ ] Given <context>, when <action>, then <result>
- [ ] ...

## Edge Cases
- What if <scenario>? Expected: <behavior>

## Technical Notes
- <Implementation hints from the plan>
- **Pattern to follow:** <cite specific reference file from plan>
- **Files likely touched:** <from plan's file inventory>

## Existing Codebase Context (if brownfield)
- **Project path:** <path to existing codebase>
- **Existing files to modify:** <list files that will be changed, not just created>
- **Existing patterns to follow:** <cite specific files as reference implementations>
- **Existing tests that must still pass:** <test files/suites that cover affected areas>
- **Integration points:** <where this ticket's work connects to existing code>

## Definition of Done
- [ ] All acceptance criteria pass
- [ ] Tests written and green (test-first per XP)
- [ ] QA review passed (/qa)
- [ ] No security regressions
- [ ] Code follows existing patterns
```

### Priority mapping
| XP Priority | Linear Priority |
|-------------|-----------------|
| Must-have | Urgent or High |
| Should-have | Medium |
| Nice-to-have | Low |

### Labels
Apply labels for:
- The activity/theme (e.g., `auth`, `onboarding`, `dashboard`)
- The component (e.g., `frontend`, `backend`, `api`)
- The iteration (e.g., `mvp`, `v1.0`, `v1.1`)

## Step 2.5 — PM Ticket Refinement Interview

Before creating tickets, walk through these questions with the user. A PM refines tickets by pressure-testing them against reality.

### Priority & Ordering
1. **Look at the Must-have stories — do you agree with all of them?** Would you demote any to Should-have? Promote any Should-haves?
2. **Within the MVP, what's the build order?** Which story needs to ship first because others depend on it?
3. **Are any stories paired?** (e.g., "backend API" + "frontend UI" that should be built together or sequentially)

### Sizing & Estimation
4. **Do the sizes feel right?** Walk through the L and XL stories — are they actually that complex, or can they be broken down further?
5. **Are there any stories that feel vague?** If you can't clearly picture what "done" looks like, we need to refine it before it becomes a ticket.
6. **Any stories that are actually spikes/research?** (i.e., "we don't know enough to estimate yet" — these should be timeboxed exploration tickets, not implementation tickets)

### Dependencies & Blockers
7. **Are any stories blocked by external factors?** (Waiting on another team, pending API access, design not finalized)
8. **Are there stories that should be grouped into a single Linear project or cycle?**
9. **Who owns each story?** (Assign now, or leave unassigned for the team to pick up?)

### Acceptance Criteria Quality Check
10. **For each Must-have story, read the acceptance criteria aloud.** Can you unambiguously tell if it's done? If not, we need to sharpen it.
11. **Are there acceptance criteria missing?** Think about: error states, empty states, loading states, permissions, mobile/responsive.
12. **Are there any cross-cutting concerns?** (Logging, analytics, feature flags, A/B testing that should be in every ticket)

## Step 3 — Present ticket batch for review

Before creating anything in Linear, show the user a summary table:

```markdown
| # | Title | Priority | Size | Iteration | Labels |
|---|-------|----------|------|-----------|--------|
| 1 | [Auth] User signup | High | M | MVP | auth, backend |
| 2 | [Auth] Email verification | Medium | S | MVP | auth, backend |
| ... | ... | ... | ... | ... | ... |
```

Ask:
- "Want to adjust any priorities, sizes, or groupings?"
- "Should I create all of these, or a subset?"
- "Should I create a Linear project or cycle for these iterations?"

## Step 4 — Create tickets in Linear

Once the user confirms, use the dedicated `mcp__linear__*` tools:

**Preferred approach — project + issues in one call:**
Use `mcp__linear__linear_create_project_with_issues` to create the project and all tickets together. This is the most efficient path.

**Alternative — tickets only (no project):**
1. Use `mcp__linear__linear_create_issues` for bulk creation, or `mcp__linear__linear_create_issue` one at a time
2. Priority mapping: Must-have → 2 (High), Should-have → 3 (Medium), Nice-to-have → 4 (Low)

**Report back** with a table of created tickets:

```markdown
| # | Linear ID | Title | URL |
|---|-----------|-------|-----|
| 1 | TEAM-123 | [Auth] User signup | https://linear.app/... |
| ... | ... | ... | ... |
```

4. Save the ticket manifest to `ref/tickets/<project-name>-tickets.md`

## Step 5 — Generate story map reference

Update `ref/story-maps/<project-name>-map.md` with Linear ticket IDs mapped to the story map positions so `/roadmap` can generate a visual with real ticket references.

---

## Interactive Mode

If no plan exists or the user wants to create a single ticket:

### Interview (one question at a time, conversationally)

1. What needs to be built or fixed? Describe the feature/bug.
2. Is this for a new project or an existing codebase? (If existing, what's the project path?)
3. Who is this for — which user persona benefits?
4. Walk me through the user flow step by step (success + failure states).
5. What's the tech stack / which part of the system does this touch?
6. Are there existing patterns in the codebase this should follow? (cite files if known)
7. What are the edge cases — what could go wrong?
8. Does this depend on or block any other ticket? (provide Linear ID if known)
9. How do we know this is done? What does success look like?
10. Priority: Must-have / Should-have / Nice-to-have?
11. Which Linear team and project should this go to?

If the user specifies an existing codebase, perform a quick scan (package.json, folder structure, existing patterns) to populate the "Existing Codebase Context" section of the ticket automatically.

Then create the ticket following the same format as Step 2.

After creation, suggest: "Run `/build <LINEAR-ID>` to start implementing this ticket."
