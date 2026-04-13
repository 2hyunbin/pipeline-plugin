---
name: pipeline-plan
description: "Pipeline Phase 2: Domain research → plan → verify in a single outer loop. Enforces strict step sequencing with artifact gates. Chained from pipeline-clarify."
---

# Pipeline Phase 2 — Plan + Verify

Research domain standards, generate plan with alternatives, verify empirically then adversarially.

**This phase runs as a single outer loop (max 3 iterations). Each iteration executes 3 steps in strict sequence. Verification feedback determines where the next iteration re-enters.**

## Prerequisites

Read pipeline state before starting:
```
state_read(mode: "pipeline")
```
Verify `status: "active"`. Read `task_summary`, `ac_path`, `assumptions`.

## Outer Loop (max 3 iterations)

```
Iteration N:
  Step 1: Domain Research → checklist artifact
  Step 2: Plan + Critic → plan artifact
  Step 3: Verify (empirical + CCG)
  
  PASS → chain to pipeline-execute
  FAIL → feedback determines re-entry:
    "checklist gaps"  → next iteration from Step 1
    "plan defects"    → next iteration from Step 2
    "verify-only fix" → next iteration from Step 3
```

If 3 iterations without PASS → HALT with best version + failure summary.

---

## Step 1: Domain Research — MANDATORY on first iteration

**MUST complete before Step 2. Do NOT spawn a Plan agent without a checklist artifact.**

On **re-entry from failed verification**: only re-run if feedback indicates checklist gaps. Otherwise skip to Step 2 with existing checklist.

1. Update state: `state_write(mode: "pipeline", state: { current_phase: "2-research" })`
2. Identify task domain(s) from AC + codebase exploration
3. **Spawn `pipeline-domain-researcher` agent** with task summary + AC as input:
   - Agent queries Context7 or WebSearch for domain best practices
   - Agent generates a **domain checklist** (separate from AC):
     - AC = "what to achieve" (user-defined outcomes)
     - Checklist = "what standards to meet" (industry-sourced criteria)
4. **WAIT for agent completion.** Read the checklist output.
5. Save checklist to `.omc/pipeline/{task_id}/checklist.md`
6. Update state: `state_write(mode: "pipeline", state: { checklist_path: ".omc/pipeline/{task_id}/checklist.md" })`
7. **User Gate:** present checklist via `AskUserQuestion`:
   - **Approve** → proceed to Step 2
   - **Request changes** → revise and re-present (max 3 sub-rounds)

**Artifact gate:** `checklist_path` MUST be non-null and file MUST exist before Step 2.

## Step 2: Plan — BLOCKED until checklist artifact exists

**MUST complete before Step 3. Do NOT run verification without a plan artifact.**

On **re-entry from failed verification**: incorporate CCG/Critic feedback into revised plan. Do NOT start from scratch — iterate on previous version.

1. Verify: `checklist_path` is set and file exists. If not → HALT.
2. Update state: `state_write(mode: "pipeline", state: { current_phase: "2-plan" })`
3. **Spawn Explore agent** to map implementation surface in codebase
4. Generate **2-3 plan alternatives** with pros/cons, informed by domain checklist
5. **Select** best alternative with rationale
6. Identify **failure scenarios** + rollback plan for complex changes
7. **Assumption register** — for each assumption:
   ```json
   { "id": "A1", "content": "...", "status": "unverified", "blocker": true }
   ```
8. **Verify blocker assumptions BEFORE proceeding** — read code, run tests, check API docs
   - If blocker invalidated → BACKTRACK (consume global backtrack, re-plan)

### Plan Quality Sub-loop (max 3 rounds)

1. **Spawn Architect agent** — reviews for architectural soundness. MUST provide strongest counterargument and at least one tradeoff tension. **WAIT for completion before Critic.**
2. **Spawn Critic agent** — evaluates against: principle-option consistency, fair alternatives, testable criteria, concrete verification steps. **Do NOT run Architect and Critic in parallel.**
3. If Critic REJECT → revise plan → return to sub-step 1 (max 3 rounds)
4. If 3 rounds without approval → proceed with best version + flag concerns

9. Save plan to `.omc/pipeline/{task_id}/plan.md`
10. Update state: `state_write(mode: "pipeline", state: { plan_path: ".omc/pipeline/{task_id}/plan.md" })`

**Artifact gate:** `plan_path` MUST be non-null and file MUST exist before Step 3.

## Step 3: Verify — BLOCKED until plan artifact exists

1. Verify: `plan_path` is set and file exists. If not → HALT.
2. Update state: `state_write(mode: "pipeline", state: { current_phase: "2-verify" })`

### Empirical Verification

- Dependencies exist and are accessible
- APIs/interfaces referenced in plan actually exist in codebase
- Test infrastructure supports planned changes
- No conflicting in-progress work (check git status)

### CCG Verification

CCG 3-model review (Codex + Gemini + Claude synthesis) with attenuation guard.

**CCG availability:** 30s timeout + 1 retry failure = unavailable → HALT.

CCG synthesis applies the attenuation guard. See pipeline/reference/ccg-attenuation-guard.md.

### Verdict

After verification, classify result:

- **PASS** → exit loop, chain to pipeline-execute
- **FAIL with checklist gaps** (e.g. "missing security considerations") → next iteration re-enters at **Step 1**
- **FAIL with plan defects** (e.g. "architecture won't scale", "wrong approach") → next iteration re-enters at **Step 2**
- **FAIL with verify-only issues** (e.g. "dependency doesn't exist") → next iteration re-enters at **Step 3**

Log feedback in state for next iteration to read.

## Chain to Next Phase

After verification passes:
```
state_write(mode: "pipeline", state: { current_phase: "2" })
```
**Invoke immediately:** `Skill("pipeline-execute")`

<Escalation_And_Stop_Conditions>
- 3 outer loop iterations without PASS → HALT with best version
- Missing artifact when attempting next step → HALT
- Blocker assumption invalidated → BACKTRACK (consume global backtrack)
- CCG unavailable → HALT
- 3 Plan Quality sub-loop rounds without Critic APPROVE → proceed with best version
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Domain-researcher agent spawned and completed
- [ ] Checklist saved to disk AND confirmed by user
- [ ] Plan has 2-3 alternatives with pros/cons
- [ ] Architect reviewed (waited for completion before Critic)
- [ ] Critic approved (or max rounds reached)
- [ ] Blocker assumptions verified (no unverified blockers remain)
- [ ] Plan saved to disk AND plan_path set in state
- [ ] Empirical preconditions validated
- [ ] CCG verification passed
- [ ] Chained to pipeline-execute
</Final_Checklist>
