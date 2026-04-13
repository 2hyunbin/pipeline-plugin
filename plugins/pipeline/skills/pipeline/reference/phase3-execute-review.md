# Phase 3: Execute + Review

**This phase is now a standalone skill: `pipeline-execute`.** This reference doc is kept for architectural context.

Invoke via `Skill("pipeline-execute")` for the authoritative execution instructions.

Goal: Implement plan, verify against AC + domain checklist, detect drift and escalation.

## Execution

1. **Decompose** plan into stories/gaps (ralph-style tracking)
   - Each story maps to 1+ AC items
   - Story = independently implementable + verifiable unit
2. **Per story:**
   - Implement the change
   - Verify the mapped AC criterion with fresh evidence (test output, build log, manual check)
   - Mark complete only with evidence — "should work" is not evidence
3. **Parallel execution** for independent stories:
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

### Risk Escalation

If implementation touches a HIGH-trigger domain not identified in Phase 0:
- auth/authz code, payment logic, PII handling, migration scripts, public API schema, infra config
- Trigger: consume 1 backtrack → re-enter Phase 0 → reclassify
- See Phase 0 escalation rule for full protocol

## BACKTRACK Protocol

Triggered when: spec gap discovered, or assumption invalidated during implementation.

1. Mark assumption `invalidated` with reason in task record
2. Update AC: bump version (v1 → v1.1) with mandatory Change Log entry
3. Re-run **only affected sections** of Phase 2:
   - If assumption about a specific interface → re-verify that interface section
   - If new requirement discovered → add to AC + re-plan that scope only
   - Do NOT re-run entire Phase 2
4. Partial work from before backtrack: **keep + mark stale**
   - Stale work may still be usable after re-plan
   - Do not auto-delete — let the re-planned execution decide
5. Consume 1 global backtrack
6. If backtrack cap exhausted → HALT (see halt-protocol.md)

## Review

### Primary: Empirical (ALL risk levels)

- Tests pass (project test runner)
- Build succeeds
- Lint clean (project linter)

Empirical review is non-negotiable. Every risk level gets this.

### Secondary: LLM (risk-differentiated)

| Risk | Review Method | Criteria |
|---|---|---|
| HIGH | CCG 3-model | AC + domain checklist + failure scenarios |
| MEDIUM | Single LLM judge | AC + domain checklist |
| LOW | Skip | Empirical only |

CCG review applies the attenuation guard (see ccg-attenuation-guard.md).

### Deslop Pass

After LLM review passes:
1. Run AI slop detection on **changed files only** (not entire codebase)
2. Fix any slop found (unnecessary comments, over-abstraction, dead code from AI generation)
3. **Regression re-verify**: re-run empirical checks after deslop changes
4. Only proceed after regression passes

### QA Cycling

- Max 5 rounds per story
- If the same error appears 3 times → **stop and report as fundamental issue**
  - Do not keep trying — escalate to user
  - Include: error pattern, what was tried, why it persists
