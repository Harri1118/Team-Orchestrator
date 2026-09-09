# Canvas Pane Convention

Each pipeline command creates note panes on the AgentGrid canvas. This makes every phase's output visible and feedable into the next phase.

## Pane Naming Convention

Format: `[Phase] Project Name — Artifact Type`

| Command | Pane Title | Color |
|---------|-----------|-------|
| `/vision` | `[Vision] <project> — Brief` | blue |
| `/plan` | `[Plan] <project> — XP Plan` | purple |
| `/plan` | `[Plan] <project> — Story Map` | purple |
| `/plan` | `[Plan] <project> — Architecture` | purple |
| `/preflight` | `[Preflight] <project> — Requirements` | orange |
| `/ticket` | `[Tickets] <project> — Manifest` | green |
| `/roadmap` | `[Roadmap] <project> — Timeline` | teal |
| `/roadmap` | `[Roadmap] <project> — Story Map` | teal |
| `/build` | `[Build] <ticket-id> — Log` | yellow |
| `/qa` | `[QA] <ticket-id> — Report` | red |
| `/validate` | `[Validate] <ticket-id> — Report` | green |

## Context Flow

Each command reads upstream panes before starting work:

```
/vision    → creates: Brief pane
/plan      → reads: Brief pane → creates: Plan, Story Map, Architecture panes
/preflight → reads: Plan pane → creates: Requirements pane
/ticket    → reads: Plan pane → creates: Manifest pane
/roadmap   → reads: Plan, Manifest, Brief panes → creates: Timeline, Story Map panes
/build     → reads: Plan, Manifest panes → creates: Build Log pane
/qa        → reads: Build Log pane → creates: QA Report pane
/validate  → reads: Build Log, QA Report panes → creates: Validation Report pane
```

## How It Works

1. Command starts → scans canvas for upstream panes via `list_canvas_panes()`
2. If found → `associate_pane()` + `read_pane()` to pull context
3. Command does its work
4. Command creates its output pane via `spawn_note_pane()`
5. Also writes to `ref/` for persistent storage (panes are ephemeral; ref/ is durable)
