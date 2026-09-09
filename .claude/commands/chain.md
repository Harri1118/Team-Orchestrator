---
description: Run a specific pipeline phase as a worker with automatic context from canvas panes
argument-hint: <phase> [args] — e.g., "chain plan" or "chain build TEAM-42" or "chain qa"
allowed-tools: Read, Write, Glob, Grep, Bash, mcp__agent_grid_workers__*
---

# Chain — Spawn a Single Pipeline Phase as a Worker

You are a lightweight orchestrator. You spawn ONE worker for a specific pipeline phase, automatically feeding it context from existing canvas panes. Use this instead of `/orchestrate` when you want to run a single phase, not the full pipeline.

## Usage

`/chain <phase> [args]`

| Phase | What it spawns | Reads from canvas | Creates on canvas |
|-------|---------------|-------------------|-------------------|
| `vision <url>` | Vision analyst | (nothing upstream) | `[Vision] — Brief` |
| `plan` | XP Planner | `[Vision] — Brief` | `[Plan] — XP Plan`, `Story Map`, `Architecture` |
| `preflight` | Requirements agent | `[Plan] — XP Plan`, `Architecture` | `[Preflight] — Requirements` |
| `ticket` | Ticket maker | `[Plan] — XP Plan` | `[Tickets] — Manifest` |
| `roadmap` | Release planner | `[Vision]`, `[Plan]`, `[Tickets]` | `[Roadmap] — Timeline`, `Story Map`, etc. |
| `build <id>` | Builder | `[Plan]`, `[Tickets]`, `[Preflight]` | `[Build] <id> — Log` |
| `qa [id]` | QA reviewer | `[Build] — Log`, `[Plan]`, `[Tickets]` | `[QA] <id> — Report` |
| `validate <id>` | Validator | `[Build]`, `[QA]`, `[Tickets]`, `[Plan]` | `[Validate] <id> — Report` |

## Execution

1. Parse `$ARGUMENTS` to determine the phase and any extra args.

2. **Gather upstream context:**
   - Call `list_canvas_panes()` to find existing panes
   - `associate_pane()` all relevant upstream panes (see table above)
   - Also check `ref/` for durable files as backup

3. **Check for existing workers to reuse:**
   - Call `list_workers()` to see if a worker for this phase already exists
   - If an idle worker with the same role exists, `send_to_worker()` instead of spawning new
   - Only spawn new if no reusable worker exists

4. **Spawn the worker:**

   ```
   spawn_worker({
     role: "<phase>-agent",
     cwd: <cwd>,
     prompt: `
       You are running the /<phase> pipeline phase.

       CONTEXT AVAILABLE:
       - Canvas panes: <list the associated panes by title>
       - ref/ files: <list relevant ref/ files found>
       Read these for context before starting work.

       TASK: <phase-specific instructions>
       Follow the /<phase> command protocol in .claude/commands/<phase>.md

       CRITICAL OUTPUT REQUIREMENTS:
       1. Create your canvas note pane(s) with full content (see Canvas Integration in the command)
       2. Write to ref/ for durable storage
       3. Report completion with a summary of what was produced

       <extra args from user>
     `
   })
   ```

5. **Wait and report:**
   - Use a background Agent to `wait_for_worker()`
   - When done, read the worker's output and new canvas panes
   - Associate any new panes (so future `/chain` calls inherit them)
   - Report to the user: what was created, what panes are on the canvas

## Multi-Phase Chaining

You can chain multiple phases in one call:

`/chain plan+preflight+ticket`

This runs them in dependency order:
1. Spawn plan worker, wait
2. Associate plan panes
3. Spawn preflight and ticket workers in parallel (both read plan)
4. Wait for both

## Fan-Out QA

`/chain qa --fan-out`

Spawns QA workers across multiple model families for diverse review:
1. `spawn_role("qa", harness="claude", model="claude-sonnet-5")`
2. `spawn_role("qa", harness="codex", model="gpt-5.6-sol")`
3. Wait for all, merge findings, deduplicate, present consolidated report

## Context Inspection

`/chain status`

Don't spawn anything — just report what's on the canvas:
1. `list_canvas_panes()` — show all project panes with titles and colors
2. `list_workers()` — show all active/idle workers
3. Check `ref/` for durable files
4. Report: "Here's where the pipeline stands. Next recommended phase: X"
