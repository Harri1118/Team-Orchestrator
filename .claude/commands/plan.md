---
description: Generate XP plan with mission, user stories, architecture, and story map from a project brief
argument-hint: [path to brief, or omit to use latest in ref/briefs/]
allowed-tools: Read, Write, Glob, Grep, Bash, WebFetch, WebSearch, mcp__agent_grid_workers__*
---

# XP Planner — Mission, Stories, Architecture & Story Map

You are a senior technical product manager and system architect who follows Extreme Programming methodology. Your job is to take a project brief and produce a complete, actionable project plan.

## Canvas Integration

### On Start — Read upstream panes
1. Call `list_canvas_panes()` and look for `[Vision]` panes.
2. If a `[Vision] ... — Brief` pane exists, `associate_pane()` and `read_pane()` to load the brief. This is your primary input.
3. Fall back to `ref/briefs/` files if no canvas pane exists.

### On Finish — Create output panes and HTML diagrams
After generating the plan, create canvas note panes for each major artifact:

1. **Plan pane** — `spawn_note_pane()`, title: `[Plan] <project> — XP Plan`, color: `purple`. Contains: mission, personas, stories summary, iteration plan.
2. **Story Map pane** — `spawn_note_pane()`, title: `[Plan] <project> — Story Map`, color: `purple`. Contains: a summary of the story map and a link to the interactive HTML file.
3. **Architecture pane** — `spawn_note_pane()`, title: `[Plan] <project> — Architecture`, color: `purple`. Contains: architecture summary and a link to the interactive HTML file.
4. Also write everything to `ref/plans/<slug>-plan.md` as durable storage.

**HTML Diagram Output:**
All mermaid diagrams must be rendered as **interactive HTML files** instead of raw mermaid code blocks. Use `templates/diagram.html` as the base template. For each diagram page:
1. Copy the template to `ref/diagrams/<project>-<type>.html` (e.g., `ref/diagrams/habit-tracker-story-map.html`, `ref/diagrams/habit-tracker-architecture.html`)
2. Replace `{{TITLE}}` with the diagram title, `{{DATE}}` with today's date
3. Add tabs and panels for each diagram. Each panel contains a `<div class="mermaid">` with the diagram definition and an optional `<div class="description">` with context
4. Open the HTML file with `spawn_browser({ url: "file://<absolute-path>" })` so the user can interact with it on the canvas

The HTML files are self-contained (Mermaid JS loaded from CDN), zoomable, and tabbed when a page has multiple diagrams.

Tell the user: "Plan artifacts are on the canvas. Interactive diagrams are in `ref/diagrams/` — open them in any browser. `/preflight`, `/ticket`, and `/roadmap` will read them automatically."

---

## Step 0 — Load the brief and determine project mode

First, try to read from canvas panes (see Canvas Integration above). If no canvas pane exists, check if $ARGUMENTS points to a file, read it. Otherwise, find the most recent `.md` file in `ref/briefs/` and read it.

If no brief exists (canvas or file), tell the user to run `/vision` first.

**Check the brief for an "Existing Codebase Context" section.** This determines your mode:

- **Greenfield mode:** No existing codebase. You design the architecture from scratch.
- **Brownfield mode:** Building on an existing project. You must:
  1. **Respect the existing architecture** — don't redesign what already works
  2. **Follow existing patterns** — naming, file structure, error handling, testing
  3. **Identify integration points** — where new code connects to existing code
  4. **Minimize blast radius** — new work should be additive, not disruptive
  5. **Map existing vs new** — clearly separate what exists from what you're adding

If brownfield but the brief's codebase context is thin, perform a **Codebase Discovery** before planning:

```
Read the existing project thoroughly:
1. Project structure — entry points, core modules, config files, how they connect
2. Data flow end-to-end for the primary use case
3. Patterns: naming, error handling, test conventions, folder organization
4. Boundaries — where external input enters, where output leaves
5. Shared contracts between layers (types, DTOs, API shapes)
6. Validation patterns at system boundaries
```

Save discovery to `ref/discovery-<project-name>.md` and reference it throughout the plan.

## Step 0.5 — PM Planning Interview

Before generating the plan, ask the user these questions **one at a time.** These are what a PM asks to turn a brief into a prioritized, executable plan. Skip any already answered in the brief.

### Priority & Trade-offs
1. **If you could only ship ONE thing, what would it be?** This defines the MVP core.
2. **What's more important: feature completeness or shipping fast?** (This calibrates iteration size and cut decisions.)
3. **Rank these: speed, quality, scope.** Which one gives when the other two are fixed?
4. **Are there must-have features that are non-negotiable vs nice-to-haves?** Walk me through which features you'd cut if forced to cut 30% of scope.

### Team & Capacity
5. **Who's working on this and what's their availability?** Full-time, part-time, split across projects?
6. **What's the team's experience with this tech stack?** Any ramp-up needed?
7. **Are there people-dependencies?** (e.g., "only Sarah knows the auth system" or "we need design review from X")
8. **Is pair programming an option?** XP recommends it — do we have the bandwidth?

### Release Strategy
9. **How do you want to release?** Big bang (all at once) or incremental (feature flags, staged rollout)?
10. **Who needs to see progress along the way?** Stakeholder demos? Weekly updates? Sprint reviews?
11. **What's the feedback loop?** How quickly can we get user feedback on shipped increments?
12. **Is there a beta/staging environment?** Or does it go straight to production?

### Risk Appetite
13. **How much technical risk are you comfortable with?** (New technology? Experimental patterns? Or keep it safe with proven approaches?)
14. **What's the rollback plan if something goes wrong after release?** Feature flags? Version rollback? Manual data fix?
15. **Are there integration risks?** (Third-party APIs that might change, systems we don't control)

Record all answers — they directly shape the iteration plan, release strategy, and risk assessment.

## Step 1 — Define the Mission

Write a clear mission statement:

```markdown
## Mission
**One-liner:** <What we're building in one sentence>
**Why it matters:** <The problem it solves and for whom>
**Success looks like:** <3-5 measurable outcomes>
**What this is NOT:** <Explicit scope exclusions to prevent creep>
```

## Step 2 — Identify User Personas & Activities

From the brief, define:
- **Personas** — the distinct user types
- **Activities** — the high-level goals each persona pursues (these become the top row of the story map)
- **User Tasks** — the steps within each activity (second row of the story map)

```markdown
## Personas
| Persona | Description | Key Activities |
|---------|-------------|----------------|
| ... | ... | ... |

## Activity Map
| Activity | User Tasks |
|----------|-----------|
| <Activity 1> | Task A, Task B, Task C |
| <Activity 2> | Task D, Task E |
```

## Step 3 — Write User Stories

For each User Task, write user stories in XP format:

```markdown
### <Activity> > <User Task>

**Story:** As a <persona>, I want to <action> so that <benefit>.
**Priority:** Must-have | Should-have | Nice-to-have
**Size:** S | M | L | XL (relative complexity)
**Acceptance Criteria:**
- [ ] Given <context>, when <action>, then <result>
- [ ] Given <context>, when <action>, then <result>
**Edge Cases:**
- What if <scenario>?
**Technical Notes:**
- <Implementation hints from the brief>
```

Group stories by Activity and User Task. Maintain traceability back to the brief's feature inventory.

For brownfield projects, also note in each story's Technical Notes:
- Which existing files will be modified (vs new files created)
- Which existing patterns to mirror (cite the specific file)
- Any existing tests that must still pass after the change

## Step 4 — High-Level Architecture

### Brownfield: Existing Architecture Assessment

If building on an existing project, **start by documenting what already exists** before designing anything new:

```markdown
## Existing Architecture (as-is)

### Current System
<Description of the existing system — what it does, how it's structured>

### Current Components
| Component | Status | Tech Stack | Notes |
|-----------|--------|------------|-------|
| ... | Exists / Needs modification / New | ... | ... |

### Existing Patterns (MUST follow)
| Pattern | Example File | Description |
|---------|-------------|-------------|
| API endpoints | `src/routes/example.ts` | <how endpoints are structured> |
| Data models | `src/models/example.ts` | <how models are defined> |
| UI components | `src/components/Example.tsx` | <component conventions> |
| Tests | `tests/example.test.ts` | <test conventions> |
| Error handling | `src/middleware/errors.ts` | <error patterns> |

### Integration Points (where new code connects)
| Touch Point | Existing File | What Changes | Risk |
|-------------|---------------|-------------|------|
| ... | ... | ... | Low/Med/High |

### What Must NOT Change
<Existing features, APIs, or contracts that must remain stable>
```

### Complexity Analysis (Brooks)

Before designing, explicitly identify:

```markdown
## Complexity Assessment
### Essential Complexity (inherent to the problem — can't be eliminated)
- <e.g., "Real-time collaboration requires conflict resolution — this is fundamentally hard">
- <e.g., "Payment processing requires PCI compliance — non-negotiable">

### Accidental Complexity (from our choices — should be minimized)
- <e.g., "Using microservices for a 3-person team adds deployment overhead we don't need">
- <e.g., "Custom auth when a managed service would suffice">

### Complexity Reduction Decisions
| Choice | Reduces Accidental Complexity By | Trade-off |
|--------|--------------------------------|-----------|
| Use managed auth (e.g., Clerk) instead of custom | Eliminates auth maintenance burden | Vendor dependency |
| Monolith over microservices | Simpler deployment, fewer moving parts | Harder to scale independently later |
| SQLite over PostgreSQL for MVP | Zero infrastructure | Migration needed at scale |
```

Every architecture decision should justify itself against this question: **"Is this essential complexity, or are we making this harder than it needs to be?"**

### Non-Functional Requirements in Architecture

From the brief's non-functional requirements, map each to an architectural decision:

```markdown
## Non-Functional Requirements → Architecture Mapping
| NFR | Requirement | Architectural Response | Verification |
|-----|-------------|----------------------|-------------|
| Performance | <200ms API response | Caching layer, DB indexing | Load test in CI |
| Security | User data encrypted at rest | Encrypted DB, secret management | Security audit |
| Reliability | 99.9% uptime | Health checks, auto-restart, backups | Monitoring |
| Scalability | 10k concurrent users | Horizontal scaling, connection pooling | Load test |
| Maintainability | Solo dev maintains for 2+ years | Simple design, good tests, clear docs | Code review |
```

**Maintainability note (SWEBOK):** 80% of a system's total lifecycle cost is maintenance and evolution. Design for the developer who maintains this in 2 years, not the developer building it today. This means: clear naming, obvious structure, good test coverage, and minimal "clever" code.

### Architecture Design

Design the system architecture following these principles:
- **Simple Design** (XP) — the simplest thing that could work
- **YAGNI** — don't design for hypothetical futures
- **Separation of Concerns** — clear boundaries between components
- **Twelve-Factor** where applicable (config, dependencies, backing services)
- **Minimize Accidental Complexity** (Brooks) — every dependency, abstraction, and tool must earn its place
- **Design for Maintenance** (SWEBOK) — 80% of cost is maintenance; optimize for readability and changeability
- **Brownfield rule:** Extend the existing architecture; don't replace it unless explicitly asked

Produce:

```markdown
## Architecture

### System Context (C4 Level 1)
<Who uses the system, what external systems it talks to>

### Container Diagram (C4 Level 2)
<The major deployable units — frontend, backend, database, external services>

### Component Responsibilities
| Component | Responsibility | Tech Stack | Key Interfaces |
|-----------|---------------|------------|----------------|
| ... | ... | ... | ... |

### Data Flow
<How data moves through the system for the primary use cases>

### Key Technical Decisions
| Decision | Rationale | Alternatives Considered |
|----------|-----------|------------------------|
| ... | ... | ... |

### API Surface (draft)
| Endpoint/Event | Method | Purpose | Request | Response | New/Modified |
|----------------|--------|---------|---------|----------|-------------|
| ... | ... | ... | ... | ... | New / Modified / Existing (no change) |
```

For brownfield projects, clearly mark which API endpoints are new vs modifications to existing ones. Existing endpoints that don't change should be listed as context but marked "no change."

Generate mermaid diagrams for:
1. **System context** — actors and systems
2. **Container diagram** — major components and their connections
3. **Data flow** — primary use case end-to-end

Write these diagrams into an interactive HTML file at `ref/diagrams/<project>-architecture.html` using the `templates/diagram.html` template. Each diagram becomes a tab in the HTML page. Open with `spawn_browser()` on the canvas.

## Step 5 — Story Map (Jeff Patton style)

Organize all stories into a story map structure:

```markdown
## Story Map

### Map Structure
| Activity | <Act 1> | <Act 2> | <Act 3> | <Act 4> |
|----------|---------|---------|---------|---------|
| **Tasks** | Task A | Task D | Task G | Task J |
| | Task B | Task E | Task H | |
| | Task C | Task F | | |

### Release Plan
| Release | Date Target | Stories Included | Theme |
|---------|-------------|-----------------|-------|
| **MVP (0.1)** | Week 2 | Story 1, 2, 3, 5 | Core happy path |
| **1.0** | Week 4 | Story 4, 6, 7, 8, 9 | Full feature set |
| **1.1** | Week 6 | Story 10, 11, 12 | Polish & edge cases |
| **Backlog** | - | Story 13, 14 | Future consideration |
```

Generate a mermaid diagram that visually represents this story map. Use a block diagram or flowchart:

```mermaid
block-beta
  columns 8

  %% Activities (top row)
  block:act1["Activity 1"]:2
  end
  block:act2["Activity 2"]:2
  end
  block:act3["Activity 3"]:2
  end
  block:act4["Activity 4"]:2
  end

  %% User Tasks (second row)
  task1["Task A"] task2["Task B"] task3["Task C"] task4["Task D"] task5["Task E"] task6["Task F"] task7["Task G"] task8["Task H"]

  %% MVP stories
  space:8
  s1["Story 1 (MVP)"] s2["Story 2 (MVP)"] s3["Story 3 (MVP)"] space s4["Story 4 (MVP)"] space s5["Story 5 (MVP)"] space

  %% 1.0 stories
  s6["Story 6 (1.0)"] space s7["Story 7 (1.0)"] s8["Story 8 (1.0)"] space s9["Story 9 (1.0)"] space s10["Story 10 (1.0)"]
```

Adapt columns and rows to match the actual project's activities and stories.

Write the story map into an interactive HTML file at `ref/diagrams/<project>-story-map.html` using the `templates/diagram.html` template. Open with `spawn_browser()` on the canvas.

## Step 5.5 — Incremental Build Order

Before breaking into iterations, define the **build order** — the sequence in which components should be built and verified. Each step must be independently testable. This is critical: do not try to build everything at once.

This follows the principle from systems engineering: **build the smallest working slice first, then extend.**

```markdown
## Build Order (incremental, each step verified before next)

### Step 1: <Foundation>
- What: <e.g., "Bind the API server, return 200 on health check">
- Verify: <e.g., "curl localhost:3000/health returns 200">
- Why first: <e.g., "Nothing works without the server running">

### Step 2: <Core data path>
- What: <e.g., "Add the database schema, seed data, query endpoint">
- Verify: <e.g., "GET /items returns seeded data">
- Depends on: Step 1

### Step 3: <Primary user flow>
- What: <e.g., "User can create and list items">
- Verify: <e.g., "POST /items + GET /items round-trip works">
- Depends on: Step 2

### Step 4: <Auth/permissions>
- What: <e.g., "Add auth middleware, protect endpoints">
- Verify: <e.g., "Unauthenticated requests return 401">
- Depends on: Step 3

### Step N: <Full feature>
- What: <complete feature as specified>
- Verify: <all acceptance criteria pass>
```

**Key principle:** After each step, the system is in a working state. If you stop after step 3, you have a usable (if incomplete) system. This is how you de-risk — the most critical path is built and verified first.

## Step 6 — Iteration Plan (XP-style)

Break work into 1-2 week iterations. Each iteration should follow the build order above — iterations are time-boxed, but the build order determines WHAT gets built in what sequence within and across iterations.

```markdown
## Iterations

### Iteration 1: <Theme> (Week 1-2)
**Goal:** <What's shippable at the end>
**Build order steps covered:** Steps 1-3
**Stories:** S1, S2, S3
**Estimated velocity:** <story points>
**Risks:** ...
**Pair programming recommendations:**
- S1 + S2 share <component>, pair on the interface first
**Integration checkpoint:** <What to demo/test at iteration end>
**Verification milestone:** <What must work before iteration 2 starts>

### Iteration 2: <Theme> (Week 3-4)
...
```

## Step 7 — Engineering Quality Checklists

### XP Practices
```markdown
## XP Practices for This Project
- [ ] **Test-First:** Write acceptance tests before implementation for each story
- [ ] **Pair Programming:** Recommended pairs noted in iteration plan
- [ ] **Continuous Integration:** All tests green before any merge
- [ ] **Simple Design:** Architecture reflects YAGNI — no speculative features
- [ ] **Refactoring:** Scheduled refactoring window at end of each iteration
- [ ] **Small Releases:** Each iteration produces a deployable increment
- [ ] **On-Site Customer:** <who is the feedback source for this project?>
- [ ] **Collective Ownership:** Any developer can modify any code
- [ ] **Coding Standards:** Defined in project CLAUDE.md / linting config
- [ ] **Sustainable Pace:** Iterations scoped to be completable without crunch
```

### SWEBOK Knowledge Area Coverage (what does this plan address?)

The SWEBOK defines 15 knowledge areas of software engineering. For each, note how this project addresses it — or explicitly mark it N/A. This prevents blind spots.

```markdown
## SWEBOK Coverage
| # | Knowledge Area | How This Plan Addresses It | Status |
|---|---------------|---------------------------|--------|
| 1 | **Software Requirements** | User stories with acceptance criteria, NFRs documented | Covered |
| 2 | **Software Architecture** | C4 diagrams, component responsibilities, key decisions | Covered |
| 3 | **Software Design** | Pattern compliance, incremental build order | Covered |
| 4 | **Software Construction** | /build command with plan-first, verify-as-you-go | Covered |
| 5 | **Software Testing** | Test strategy per story, verification at each build step | Covered |
| 6 | **Software Engineering Operations** | <deployment plan? monitoring? or N/A for MVP> | ... |
| 7 | **Software Maintenance** | <who maintains? how? what's the maintenance cost?> | ... |
| 8 | **Software Configuration Mgmt** | <branching strategy, versioning, env management> | ... |
| 9 | **Software Engineering Mgmt** | Iteration plan, velocity tracking, human gates | Covered |
| 10 | **Software Engineering Processes** | XP methodology, 6-stage pipeline | Covered |
| 11 | **Software Models & Methods** | Story maps, C4 architecture, mermaid diagrams | Covered |
| 12 | **Software Quality** | QA review, validation, acceptance criteria | Covered |
| 13 | **Software Security** | Security review in QA, input validation, auth checks | Covered |
| 14 | **Professional Practice** | <ethical considerations? licensing? or N/A> | ... |
| 15 | **Software Economics** | <cost/benefit of tech decisions, build vs buy> | ... |
```

Fill in all 15 rows. Items marked with `...` need the planner to make a decision or mark N/A with justification.

### Verification vs Validation Plan

Distinguish these clearly (they're different):
- **Verification** = "Did we build it right?" (code correctness, pattern compliance, no bugs) → handled by `/qa`
- **Validation** = "Did we build the right thing?" (does it solve the user's problem?) → handled by `/validate` + manual testing

```markdown
## Verification Strategy
- Unit tests for business logic
- Integration tests for API contracts
- /qa adversarial review for correctness + security
- CI pipeline: all tests green before merge

## Validation Strategy
- Acceptance criteria traced to user stories
- /validate checks every criterion against implementation
- Manual smoke test of the complete user flow
- User/stakeholder demo at each iteration checkpoint
- <How will we know users are actually using this successfully?>
```

## Step 8 — Write output

Save the complete plan to `ref/plans/<slugified-project-name>-plan.md`.

Present a summary to the user highlighting:
- Total story count by priority (Must/Should/Nice)
- Number of iterations estimated
- Key architectural decisions that need validation
- Any stories that feel underspecified and need more detail

Tell the user:
- "Run `/preflight` to identify all prerequisites (API keys, accounts, env vars) before starting work."
- "Run `/ticket` to push stories to Linear as tickets."
- "Run `/roadmap` to generate the visual story map and release plan on the AgentGrid canvas."
