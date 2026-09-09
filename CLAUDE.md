# Team-Orchestrator

An agentic project manager that uses XP methodology to plan, scope, and ship software projects.

## What This Is

A set of Claude Code slash commands that act as a project management pipeline. There are three ways to use them:

### 1. Manual (run commands yourself)
Run each slash command individually in your Claude Code session. You control the pace and flow.

### 2. Chain (single-phase delegation)
`/chain <phase>` spawns a worker for one phase, automatically feeds it context from canvas panes.

### 3. Orchestrate (full pipeline)
`/orchestrate <url>` runs the entire pipeline end-to-end, spawning workers for each phase and chaining context between them. Human gates at every transition.

### Commands

| Command | Role | Purpose |
|---------|------|---------|
| `/orchestrate` | **Pipeline Controller** | Run the full pipeline — spawns workers, chains context, manages gates |
| `/chain` | **Phase Runner** | Spawn a single phase as a worker with automatic context from canvas |
| `/vision` | Video Analyst | Ingest video/transcript, extract structured project brief |
| `/plan` | XP Planner | Generate mission, user stories, architecture, story map |
| `/preflight` | Requirements Agent | Identify API keys, accounts, env vars needed before work starts |
| `/ticket` | Ticket Maker | Create Linear tickets from user stories with acceptance criteria |
| `/roadmap` | Release Planner | Generate story map + release plan as mermaid on AgentGrid canvas |
| `/build` | Builder | Implement a ticket following the approved plan |
| `/qa` | QA Reviewer | Adversarial code review — find bugs, pattern violations, security issues |
| `/validate` | Validator | Final validation against ticket requirements before merge |

## Pipeline Flow

```
/vision -> /plan -> /preflight -> /ticket -> /roadmap
                        |              |
                     (parallel)    (parallel)
                                     |
                              /build -> /qa -> fix loop -> /validate -> ship
```

## How Agents Share Context

There are three layers. Each serves a different purpose:

### Layer 1: Canvas Panes (visual + agent-to-agent)
Every command creates note panes on the AgentGrid canvas, named and color-coded by phase. **This is the primary context-passing mechanism between agents.**

How it works:
1. Worker A creates a pane: `spawn_note_pane()` with title `[Phase] Project — Artifact`
2. The orchestrator `associate_pane()` that pane to itself
3. Worker B inherits the association — calls `read_pane()` to pull Worker A's output
4. The user sees all panes on the canvas as a visual dashboard

| Phase | Pane Title Pattern | Color | Created By |
|-------|-------------------|-------|------------|
| Vision | `[Vision] <project> — Brief` | blue | `/vision` |
| Plan | `[Plan] <project> — XP Plan` | purple | `/plan` |
| Plan | `[Plan] <project> — Story Map` | purple | `/plan` |
| Plan | `[Plan] <project> — Architecture` | purple | `/plan` |
| Preflight | `[Preflight] <project> — Requirements` | orange | `/preflight` |
| Tickets | `[Tickets] <project> — Manifest` | green | `/ticket` |
| Roadmap | `[Roadmap] <project> — Timeline` | teal | `/roadmap` |
| Roadmap | `[Roadmap] <project> — Story Map` | teal | `/roadmap` |
| Build | `[Build] <ticket-id> — Log` | yellow | `/build` |
| QA | `[QA] <ticket-id> — Report` | red | `/qa` |
| Validate | `[Validate] <ticket-id> — Report` | green/red | `/validate` |

### Layer 2: Shared Filesystem (`ref/`)
All commands write to `ref/` for durable storage that persists across sessions:
- `ref/briefs/` — vision output (project briefs)
- `ref/plans/` — XP plans with user stories and architecture
- `ref/tickets/` — ticket specs, build logs, QA reports, validation reports
- `ref/story-maps/` — generated story map mermaid files

Workers share the filesystem, so any worker can `Read` files another worker wrote. This is the fallback when canvas panes aren't available (e.g., after session resume).

### Layer 3: Prompt Injection (explicit)
When spawning workers, the orchestrator can include key context in the prompt. Used for small critical pieces (project name, ticket ID, mode). Don't paste entire plans — point workers to panes/files.

### Context Flow Diagram
```
/vision worker
    ├── writes ref/briefs/brief.md (durable)
    └── creates [Vision] Brief pane (visual)
              ↓ (orchestrator associates pane)
/plan worker (reads Brief pane + ref/briefs/)
    ├── writes ref/plans/plan.md
    └── creates [Plan] panes ×3
              ↓
/preflight worker ←──────── reads [Plan] panes + ref/plans/
/ticket worker    ←──┘ (parallel)
              ↓
/roadmap worker (reads ALL upstream panes)
              ↓
/build worker (reads [Plan] + [Tickets] + ref/)
    └── creates [Build] Log pane
              ↓
/qa worker (reads [Build] Log + [Plan])
    └── creates [QA] Report pane
              ↓
/validate worker (reads [Build] + [QA] + [Tickets])
    └── creates [Validate] Report pane
```

## Prerequisites

- **yt-dlp**: `brew install yt-dlp` — video download + transcript extraction
- **whisper**: `pip install openai-whisper` — audio transcription for uncaptioned videos
- **Linear API key**: Set `LINEAR_API_KEY` in `.mcp.json` for ticket creation

## Greenfield vs Brownfield

Every command supports both modes:

- **Greenfield** — new project from scratch. Architecture designed fresh.
- **Brownfield** — building on an existing codebase. The pipeline will:
  - Discover and document existing architecture, patterns, and conventions
  - Design new work to extend (not replace) the existing system
  - Enforce pattern compliance with the existing codebase
  - Verify existing tests still pass before and after changes
  - Show existing vs new components in architecture diagrams
  - Only list net-new prerequisites (not things already configured)

The `/vision` command asks which mode at the start. All downstream commands adapt automatically based on whether the brief includes an "Existing Codebase Context" section.

## XP Principles Enforced

- Small iterations (1-2 weeks)
- User stories as the unit of work
- Test-first development
- Continuous integration gates
- Pair programming recommendations
- Simple design — no speculative architecture
- Human gates at every phase transition

## Development Pipeline (6 Stages)

1. **Discovery** — deep codebase understanding before any code
2. **Planning** — verifiable implementation plan with pattern references
3. **Build** — execute plan, test-first, every change traces to plan
4. **QA Review** — adversarial review by separate agent (fan across model families)
5. **Fix & Revalidate** — address QA findings, loop until clean
6. **Final Validation** — spec compliance, test coverage, merge readiness

Nothing advances without proof-of-work. Every stage produces a written artifact.
