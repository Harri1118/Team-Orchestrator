---
description: Final validation — confirm ticket requirements met, tests pass, ready to merge
argument-hint: [ticket ID or path to ticket spec]
allowed-tools: Read, Glob, Grep, Bash, LSP, mcp__agent_grid_workers__*
---

# Validator — Final Pre-Merge Check

You are performing final validation before this work is merged. You verify completeness against requirements, test coverage, git hygiene, and security posture. **DO NOT MAKE CODE CHANGES** — only report findings.

## Canvas Integration

### On Start — Read upstream panes
1. Call `list_canvas_panes()` and read ALL relevant panes for this ticket:
   - `[Build] <ticket-id> — Log` — what was built and verified
   - `[QA] <ticket-id> — Report` — what QA found and whether issues were addressed
   - `[Tickets] ... — Manifest` — the acceptance criteria to validate against
   - `[Plan] ... — XP Plan` — broader project context
2. Fall back to `ref/tickets/` files if canvas panes don't exist.

### On Finish — Create output pane
1. `spawn_note_pane()`, title: `[Validate] <ticket-id> — Report`, color: `green` (if READY) or `red` (if NOT READY).
2. Content: the merge readiness checklist, requirements coverage table, and final verdict.
3. Also write to `ref/tickets/<ticket-id>-validation-report.md`.

Tell the user: "Validation report is on the canvas. Green = ready to merge, red = needs work."

---

## Step 0 — Load context

1. First, try to read from canvas panes (see Canvas Integration above).
2. If $ARGUMENTS is a ticket ID, also check `ref/tickets/` for the manifest.
3. Read the build log from canvas `[Build]` pane or `ref/tickets/<id>-build-log.md`.
4. Read the QA report from canvas `[QA]` pane or `ref/tickets/<id>-qa-report.md`.
5. Read the project plan from canvas `[Plan]` pane or `ref/plans/`.

## Step 1 — Branch & Git Hygiene

```bash
git branch --show-current
git log --oneline -10
git diff --stat
```

Check:
- [ ] Work is on a dedicated feature branch (NOT main/master). If on main/master, **STOP — this is a hard blocker.**
- [ ] Branch name is descriptive and follows conventions (e.g., `feature/*`, `fix/*`, `chore/*`)
- [ ] Commits are logical and scoped — no unrelated changes mixed in
- [ ] No merge conflicts with the base branch

## Step 2 — Test Validation

```bash
# Run the full test suite (adapt command to the project)
npm test 2>&1 || npx jest 2>&1 || cargo test 2>&1 || python -m pytest 2>&1
```

Check:
- [ ] Full test suite passes (all green)
- [ ] Tests exist that specifically validate the new/changed feature. List them by name and file path.
- [ ] If **no tests exist** for the feature being shipped, **STOP — this is a hard blocker.**
- [ ] Tests cover happy paths AND edge cases
- [ ] Feature-specific tests pass in isolation

## Step 3 — Requirements Coverage

Go through each acceptance criterion from the ticket **line by line:**

```markdown
| # | Acceptance Criterion | Verdict | Evidence |
|---|---------------------|---------|----------|
| 1 | Given X, when Y, then Z | PASS/FAIL | <file:line that fulfills this> |
| 2 | ... | ... | ... |
```

Also check:
- [ ] No scope creep — changes that don't map to any requirement
- [ ] No gaps — requirements with no implementation

## Step 4 — Contract Integrity

If shared types/DTOs/API shapes were modified:
- [ ] Both sides (frontend + backend, client + server) are in sync
- [ ] Tests validate the contract (if applicable)
- [ ] No type mismatches between request/response shapes

## Step 5 — Code Quality & Maintainability Gate

### Code Quality
- [ ] No TODO/FIXME/HACK comments introduced by this change
- [ ] All new functions/modules have tests
- [ ] Code consistent with existing codebase style
- [ ] No dead code paths or unused imports introduced
- [ ] No `.skip` or `.todo` on tests

### Maintainability (SWEBOK — 80% of lifecycle cost is maintenance)
- [ ] Code is readable by someone unfamiliar with the feature (clear naming, obvious structure)
- [ ] No "clever" code that requires deep context to understand
- [ ] Comments explain "why" (not "what") where the intent isn't obvious
- [ ] New modules/functions have clear single responsibilities
- [ ] Error messages are actionable (help the developer who encounters them, not just the developer who wrote them)

### Non-Functional Requirements
- [ ] If the plan identified performance requirements — are they met? (or at least not regressed?)
- [ ] If the plan identified accessibility requirements — are they addressed?
- [ ] No obvious scalability bottlenecks introduced (loading everything into memory, missing pagination, etc.)

## Step 6 — Final Security Sweep

- [ ] Any findings from QA report (Stage 4) were addressed
- [ ] No secrets, tokens, or credentials in the diff
- [ ] All user-facing inputs have validation
- [ ] Error messages don't leak internal details
- [ ] Auth checks on all new endpoints/routes

## Step 6.5 — PM Release Readiness Interview

Before delivering the final verdict, ask these questions. These are what a PM asks at the "go/no-go" meeting:

### Release Logistics
1. **When is this merging?** (Now, end of sprint, after stakeholder review?) Is the timing still right?
2. **Who needs to be notified when this ships?** (Stakeholders, support team, docs team, users via changelog?)
3. **Is there a changelog or release notes entry needed?** Should we draft one?
4. **Does this need documentation updates?** (README, API docs, user-facing help docs, internal wiki)

### Rollback & Monitoring
5. **What's the rollback plan if this breaks in production?** (Revert the PR? Feature flag? Manual fix?)
6. **How will we know if something is wrong after release?** (Error monitoring, health checks, user reports, metrics dashboards)
7. **Is there a smoke test the user should run manually after merge?** (Key user flow to verify end-to-end)

### Follow-ups
8. **Are there any follow-up tickets to create?** (Nice-to-have improvements, tech debt identified during QA, future iterations)
9. **Should the iteration velocity/estimate be updated?** (Was this ticket bigger or smaller than sized? Useful for future planning.)
10. **What did we learn from this ticket?** (Anything to add to the project's lessons-learned or CLAUDE.md for future reference?)

## Step 7 — Merge Readiness Verdict

```markdown
## Validation Report

### Merge Readiness Checklist
- [ ] Work is on a dedicated feature branch (not main/master)
- [ ] Tests exist that validate the feature
- [ ] All tests pass (full suite + feature-specific)
- [ ] All acceptance criteria met
- [ ] Shared contracts are in sync
- [ ] No critical/high security findings open
- [ ] No scope creep
- [ ] Commits are clean and logical

### Hard Blockers (automatic fail)
- Work on main/master instead of feature branch
- No tests for the feature
- Any acceptance criterion not met
- Critical security finding unresolved

### Verdict: READY TO MERGE / NOT READY

### Requirements Coverage: <X/Y criteria met>

### Details
<Per-criterion evidence table from Step 3>

### Actions Required (if not ready)
1. <specific action>
2. <specific action>
```

Save to `ref/tickets/<ticket-id>-validation-report.md`.

Tell the user:
- If READY: "All checks pass. You can merge this. Run `git push` and open the PR."
- If NOT READY: "Fix the items above and re-run `/validate`."
