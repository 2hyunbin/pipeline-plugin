---
name: pipeline-execute
description: "Pipeline Phase 3: Implement plan, verify against AC + domain checklist, detect drift. Chained from pipeline-plan."
---

# Pipeline Phase 3 — Execute + Review

Implement plan, verify against AC + domain checklist, detect drift.

## Prerequisites

Read pipeline state before starting:
```
state_read(mode: "pipeline")
```

- Verify `plan_path` is set and file exists — read the plan
- Verify `checklist_path` is set and file exists — read the domain checklist
- Verify `ac_path` is set and file exists — read the acceptance criteria
- If any artifact is missing → HALT with "Phase 2 incomplete"

Update state: `state_write(mode: "pipeline", state: { current_phase: "3" })`

## Execution

### 1. Decompose Plan into Stories

- Each story maps to 1+ AC items
- Story = independently implementable + verifiable unit

### 2. Per Story

1. Implement the change
2. Verify the mapped AC criterion with **fresh evidence** (test output, build log, manual check)
3. Mark complete **only with evidence** — "should work" is not evidence

### 3. Parallel Execution (for independent stories)

- haiku: trivial tasks (rename, config tweak)
- sonnet: standard tasks (new method, new test)
- opus: complex tasks (architectural change, multi-file refactor)

## Detection (every round)

### Scope Creep

Compare current work against AC at each round boundary:
- Work that addresses an AC item → proceed
- Work not in any AC item → WARN user via `AskUserQuestion`:
  - "This change isn't covered by current AC. Add to AC, or skip?"
  - If add → AC version bump with Change Log
  - If skip → revert and continue

## BACKTRACK Protocol

Triggered when: spec gap discovered, or assumption invalidated during implementation.

1. Mark assumption `invalidated` with reason in state
2. Update AC: bump version (v1 → v1.1) with mandatory Change Log entry
3. Re-invoke `Skill("pipeline-plan")` for **only affected sections** — do NOT re-run entire Phase 2
4. Partial work from before backtrack: **keep + mark stale**
5. Consume 1 global backtrack
6. If backtrack cap (3) exhausted → HALT (see pipeline/reference/halt-protocol.md)

## Review

### Primary: Empirical (non-negotiable)

- Tests pass (project test runner)
- Build succeeds
- Lint clean (project linter)

### Secondary: CCG Review

CCG 3-model review with attenuation guard. Criteria: AC + domain checklist + failure scenarios.

CCG review applies the attenuation guard (see pipeline/reference/ccg-attenuation-guard.md).

### Deslop Pass

After CCG review passes:
1. Run AI slop detection on **changed files only** (not entire codebase)
2. Fix any slop found (unnecessary comments, over-abstraction, dead code from AI generation)
3. **Regression re-verify**: re-run empirical checks after deslop changes
4. Only proceed after regression passes

### QA Cycling

- Max 5 rounds per story
- If the same error appears 3 times → **stop and report as fundamental issue**
  - Do not keep trying — escalate to user

## Completion

After all stories verified and review passed:

```
state_write(mode: "pipeline", state: {
  status: "completed",
  current_phase: "3"
})
```

Report to user:
- Summary of what was implemented
- Evidence for each AC item
- Domain checklist pass/fail status
- Any open risks or follow-ups

Clean up via `/oh-my-claudecode:cancel` or let user decide.

<Escalation_And_Stop_Conditions>
- HALT trigger → immediate stop, report to user
- "stop" / "cancel" / "abort" → clean exit via /oh-my-claudecode:cancel
- Same error 3x → fundamental issue, report
- Backtrack cap exhausted → HALT
- CCG unavailable → HALT
- Scope creep detected → AskUserQuestion before proceeding
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] All stories decomposed from plan
- [ ] Each AC item has fresh verification evidence
- [ ] Domain checklist passed
- [ ] No unverified assumptions remain
- [ ] Empirical review passed (tests + build + lint)
- [ ] CCG review passed
- [ ] Deslop pass completed on changed files
- [ ] Regression re-verified after deslop
- [ ] State updated to completed
- [ ] Results reported to user
</Final_Checklist>
