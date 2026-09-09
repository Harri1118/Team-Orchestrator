---
description: Generate visual story map and release roadmap as mermaid diagrams on AgentGrid canvas
argument-hint: [path to plan, or omit to use latest]
allowed-tools: Read, Write, Glob, Grep, Bash, mcp__agent_grid_workers__*
---

# Roadmap — Story Map & Release Plan Visualizer

You are a visual project planner. Your job is to take the XP plan and ticket manifest and generate rich mermaid diagrams that display on the AgentGrid canvas as a living project dashboard.

## Canvas Integration

### On Start — Read upstream panes
1. Call `list_canvas_panes()` and look for all project panes.
2. Read these panes if they exist (via `associate_pane()` + `read_pane()`):
   - `[Vision] ... — Brief` — for mission context
   - `[Plan] ... — XP Plan` — for stories, iterations, architecture
   - `[Plan] ... — Story Map` — for existing story map to update
   - `[Tickets] ... — Manifest` — for Linear IDs to embed in diagrams
3. Fall back to `ref/` files if canvas panes don't exist.

### On Finish — Create output panes
Create multiple visual panes — this is the command that populates the canvas dashboard:

1. **Story Map** — `spawn_note_pane()`, title: `[Roadmap] <project> — Story Map`, color: `teal`. Contains: the Jeff Patton mermaid block-beta diagram.
2. **Timeline** — `spawn_note_pane()`, title: `[Roadmap] <project> — Timeline`, color: `teal`. Contains: the Gantt chart mermaid.
3. **Architecture** — `spawn_note_pane()`, title: `[Roadmap] <project> — Architecture`, color: `teal`. Contains: C4 mermaid diagrams.
4. **Pipeline** — `spawn_note_pane()`, title: `[Roadmap] <project> — Pipeline`, color: `teal`. Contains: the quality gates flowchart.
5. Also write everything to `ref/story-maps/`.

Tell the user: "Roadmap dashboard is on the canvas — story map, timeline, architecture, and pipeline are all visible."

---

## Step 0 — Load artifacts

First, try to read from canvas panes (see Canvas Integration above). Fall back to files:
1. `ref/plans/*-plan.md` — the XP plan with stories, iterations, architecture
2. `ref/tickets/*-tickets.md` — the ticket manifest with Linear IDs (if exists)
3. `ref/briefs/*.md` — the original brief (for mission context)

If no plan exists (canvas or file), tell the user to run `/plan` first.

## Step 1 — Generate the User Story Map

Create a Jeff Patton-style story map. The structure:
- **Row 1 (orange):** Activities — high-level user goals
- **Row 2 (light orange):** User Tasks — steps within each activity
- **Row 3+ (blue, grouped by release):** User Stories — specific deliverables

Generate as mermaid block-beta diagram. Adapt the columns and rows to match the actual project:

```mermaid
---
title: "<Project Name> — User Story Map"
---
block-beta
  columns 10

  %% === ACTIVITIES (top row) ===
  block:a1["Activity 1"]:3
  end
  block:a2["Activity 2"]:2
  end
  block:a3["Activity 3"]:3
  end
  block:a4["Activity 4"]:2
  end

  %% === USER TASKS (second row) ===
  t1["Task A"]
  t2["Task B"]
  t3["Task C"]
  t4["Task D"]
  t5["Task E"]
  t6["Task F"]
  t7["Task G"]
  t8["Task H"]
  t9["Task I"]
  t10["Task J"]

  %% === RELEASE SEPARATOR ===
  space:10

  %% === MVP STORIES ===
  s1["TEAM-1\nUser signup\n(M)"]
  s2["TEAM-2\nEmail verify\n(S)"]
  space
  s3["TEAM-3\nCreate item\n(L)"]
  s4["TEAM-4\nList items\n(M)"]
  space
  s5["TEAM-5\nDashboard\n(L)"]
  space:3

  %% === v1.0 STORIES ===
  s6["TEAM-6\nSearch\n(M)"]
  space
  s7["TEAM-7\nFilters\n(S)"]
  space
  s8["TEAM-8\nExport\n(M)"]
  s9["TEAM-9\nSharing\n(L)"]
  space:4

  %% === BACKLOG ===
  s10["TEAM-10\nAnalytics\n(L)"]
  space:9
```

Adapt this template to the actual project data. Key rules:
- Activities span multiple columns to group related tasks
- Each story shows its Linear ID (if available), short title, and size
- Stories are positioned under the User Task they belong to
- Visual rows correspond to releases (MVP, 1.0, 1.1, Backlog)

Write the story map mermaid to `ref/story-maps/<project-name>-story-map.md`.

## Step 2 — Generate the Release Timeline (Gantt)

```mermaid
gantt
    title <Project Name> — Release Timeline
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section MVP (Iteration 1-2)
    Story 1: User signup           :s1, 2024-01-15, 3d
    Story 2: Email verification    :s2, after s1, 2d
    Story 3: Create item           :s3, 2024-01-15, 5d
    Gate: QA Review                :milestone, qa1, after s3, 0d
    Gate: MVP Release              :milestone, mvp, after qa1, 0d

    section v1.0 (Iteration 3-4)
    Story 6: Search                :s6, after mvp, 3d
    Story 7: Filters               :s7, after s6, 2d
    Story 8: Export                :s8, after mvp, 4d
    Gate: QA Review                :milestone, qa2, after s8, 0d
    Gate: v1.0 Release             :milestone, v10, after qa2, 0d

    section v1.1 (Iteration 5-6)
    Story 10: Analytics            :s10, after v10, 5d
    Gate: Final Release            :milestone, v11, after s10, 0d
```

Use actual dates from the plan's iteration schedule. Include QA gates and release milestones.

Write to `ref/story-maps/<project-name>-gantt.md`.

## Step 3 — Generate the Architecture Diagram

From the plan's architecture section, generate a C4-style container diagram.

**For brownfield projects:** Show the existing system as context, and highlight what's new vs existing. Use styling to visually distinguish:
- Existing components (solid borders, normal style)
- New components being added (dashed borders or different color)
- Modified components (bold borders)

This lets the user see at a glance what's changing vs what already exists.

```mermaid
C4Context
    title <Project Name> — System Context

    Person(user, "End User", "Primary user of the system")

    System(app, "Application", "The system being built")

    System_Ext(auth, "Auth Provider", "Authentication service")
    System_Ext(db, "Database", "Data persistence")
    System_Ext(api, "External API", "Third-party integration")

    Rel(user, app, "Uses")
    Rel(app, auth, "Authenticates via")
    Rel(app, db, "Reads/writes")
    Rel(app, api, "Calls")
```

And a container diagram:

```mermaid
C4Container
    title <Project Name> — Container Diagram

    Person(user, "User")

    Container_Boundary(sys, "System") {
        Container(web, "Frontend", "React/Next.js", "User interface")
        Container(api, "API Server", "Node.js/Express", "Business logic & API")
        ContainerDb(db, "Database", "PostgreSQL", "Data storage")
    }

    Rel(user, web, "Browses")
    Rel(web, api, "API calls")
    Rel(api, db, "Queries")
```

For brownfield projects, also generate a **change impact diagram** showing which existing components are touched:

```mermaid
flowchart TD
    subgraph "Existing (unchanged)"
        A[Component A]
        B[Component B]
    end
    subgraph "Modified"
        C[Component C\n+ new endpoint]
        D[Component D\n+ new field]
    end
    subgraph "New"
        E[New Component E]
        F[New Component F]
    end

    E --> C
    F --> D
    C --> A
    D --> B
```

Write to `ref/story-maps/<project-name>-architecture.md`.

## Step 4 — Generate the Pipeline/Gate Diagram

Show the project's quality gates and workflow:

```mermaid
flowchart LR
    V["/vision\nVideo Analysis"] --> P["/plan\nXP Planning"]
    P --> PF["/preflight\nRequirements"]
    P --> T["/ticket\nLinear Tickets"]
    T --> R["/roadmap\nStory Map"]

    subgraph "Per Ticket"
        B["/build\nImplement"] --> QA["/qa\nReview"]
        QA -->|Issues| FIX[Fix & Retest]
        FIX --> QA
        QA -->|Clean| VAL["/validate\nFinal Check"]
        VAL -->|Pass| SHIP["Merge & Ship"]
        VAL -->|Fail| B
    end

    R --> B
```

## Step 5 — Write all diagrams to note files

Create a consolidated file at `ref/story-maps/<project-name>-roadmap.md` with all diagrams:

```markdown
# <Project Name> — Roadmap

## Story Map
<mermaid block-beta diagram>

## Release Timeline
<mermaid gantt diagram>

## Architecture
<mermaid C4 diagrams>

## Pipeline
<mermaid flowchart>

---
Generated: <date>
Source plan: <path to plan file>
Ticket manifest: <path to tickets file>
```

## Step 5.5 — PM Roadmap Review Interview

Before finalizing the roadmap, ask these questions. These are what a PM asks to make the roadmap useful to stakeholders, not just developers.

### Audience & Communication
1. **Who will see this roadmap?** (Just the dev team? Stakeholders? Clients? Investors?) The level of detail changes depending on audience.
2. **How often should this roadmap be updated?** (Every sprint? Monthly? Only at milestones?)
3. **What format does your audience prefer?** (Mermaid diagrams are great for devs — do stakeholders need a simpler view?)

### Timeline Calibration
4. **Are the iteration dates realistic given the team's availability?** (Account for vacations, holidays, other projects, on-call rotations)
5. **Is there buffer built in?** A good PM adds 20-30% buffer for unknowns. Should we pad the estimates?
6. **Are there external milestones we need to hit?** (Conference demos, client deadlines, quarter-end, board meetings)

### Progress Tracking
7. **How do you want to track progress against this roadmap?** (Linear project board? Weekly standup against the story map? Burndown chart?)
8. **What's the escalation path if we fall behind?** (Cut scope? Add time? Add people? Which one first?)
9. **At what point is "behind schedule" a problem vs normal variance?** (One story slipping isn't a crisis; missing an entire iteration is.)

### Stakeholder Management
10. **What's the "headline" for each release?** (Stakeholders want themes, not ticket numbers. "MVP: users can sign up and do the core thing" > "Sprint 1: TEAM-1 through TEAM-8")
11. **Are there dependencies on other teams or external parties that should be on the timeline?** (Design review, security audit, legal approval)
12. **What risks should be visible on the roadmap?** (Some PMs mark known risks directly on the Gantt — "API vendor stability" or "design dependency")

## Step 6 — Display on AgentGrid canvas

Tell the user which files were generated and where. The mermaid diagrams can be displayed as AgentGrid notes by writing them to `.agent-grid/notes/`:

Suggest to the user:
- "Copy the story map diagram to `.agent-grid/notes/story-map.md` to see it on the canvas"
- "Copy the gantt chart to `.agent-grid/notes/timeline.md` to see the release timeline"
- Or offer to do it for them.

Present a text summary of the roadmap:

```markdown
## Roadmap Summary

**Project:** <name>
**Total Stories:** <count> (Must: X, Should: Y, Nice: Z)
**Releases:** MVP → v1.0 → v1.1
**Estimated Duration:** <weeks>

### MVP (<date target>)
- <X stories, Y story points>
- Key deliverables: ...

### v1.0 (<date target>)
- <X stories, Y story points>
- Key deliverables: ...

### v1.1 (<date target>)
- <X stories, Y story points>
- Key deliverables: ...

### Quality Gates
- Gate 1: Plan approved (before any code)
- Gate 2: Visual check per ticket
- Gate 3: QA review per ticket
- Gate 4: Tests green + spec validated
- Gate 5: Manual smoke test before merge
```

Tell the user: "Your roadmap is generated. Run `/build <TICKET-ID>` to start implementing the first ticket."
