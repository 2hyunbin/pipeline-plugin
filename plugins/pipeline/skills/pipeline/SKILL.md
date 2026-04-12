---
name: pipeline
description: Risk-routed multi-phase agent pipeline with empirical-first verification, backtrack recovery, and adversarial CCG review. Triggers on "pipeline", "full pipeline", "risk-aware", "systematic execution", or complex tasks needing structured phase control.
argument-hint: "[--risk=high|medium|low] [--skip-clarify] <task description>"
---

# Pipeline — Risk-Routed Agent Orchestration

Classifies task risk, then routes through Clarify → Plan → Execute with controls proportional to stakes.

## Principles

1. **Empirical-first** — tests/build/lint before LLM judge, always
2. **Risk-proportional** — LOW skips ceremony; HIGH gets full pipeline
3. **Disk-as-memory** — canonical task record on disk; fresh-context subagents read it each iteration
4. **Fail-closed** — cap exhaustion or CCG unavailable on HIGH → HALT, never silent proceed
5. **Attenuation guard** — CCG synthesis diffs original↔result to catch "merge→delete" drift

## Risk Paths

```
HIGH   → Phase 0 → 1 → 2 (full) → 3 (full)     [CCG, User Gates]
MEDIUM → Phase 0 → 1 → 2 (lite) → 3 (lite)     [single judge, 1 Gate]
LOW    → Phase 0 → 3 (execute + empirical only)  [no plan, no LLM review]
```

## Execution Flow

### Phase 0: Risk Routing
Classify via domain signals (not file count). Route to risk path.
Escalation rule: any phase discovers HIGH trigger → consume 1 backtrack → re-enter Phase 0.
→ See [reference/phase0-risk-routing.md](reference/phase0-risk-routing.md)

### Phase 1: Clarify
Replace ambiguity with versioned acceptance criteria (AC). 1-2 rounds default, max 5.
Skip if: `--skip-clarify`, explicit AC provided, or LOW risk.
→ See [reference/phase1-clarify.md](reference/phase1-clarify.md)

### Phase 2: Plan + Verify
Domain research → plan alternatives → empirical verification → LLM verification (risk-gated).
→ See [reference/phase2-plan-verify.md](reference/phase2-plan-verify.md)

### Phase 3: Execute + Review
Implement, detect scope creep + risk escalation, BACKTRACK on spec gaps, empirical-first review.
→ See [reference/phase3-execute-review.md](reference/phase3-execute-review.md)

## Infrastructure

- **LoopGuard**: Phase caps (5/3/5) + global backtrack cap (HIGH:3, MED:2). See [reference/loopguard.md](reference/loopguard.md)
- **HALT**: Fail-closed on cap exhaustion or CCG+HIGH unavailable. See [reference/halt-protocol.md](reference/halt-protocol.md)
- **CCG Guard**: Original↔result diff prevents synthesis attenuation. See [reference/ccg-attenuation-guard.md](reference/ccg-attenuation-guard.md)
- **Task Record**: Single source of truth on disk. See [reference/task-record-schema.md](reference/task-record-schema.md)
- **Token Budget**: Handoff output limits — Phase 0→1: 500, 1→2: 1K, 2→3: 2K, result: 1K. Truncation: AC > decisions > diff > prior.
- **Checkpoints**: At phase completion + critical decisions (AC approval, plan selection, BACKTRACK).
- **Reflections**: Never during loops. Only after Phase 3 completion + verified outcome.

## State

```
state_write(mode: "pipeline", state: {
  task_id, risk_level, current_phase, backtrack_count,
  ac_version, status: "active|halted|completed",
  halt_reason, assumptions, checklist_path, plan_path
})
```

<Escalation_And_Stop_Conditions>
- HALT trigger → immediate stop, report to user
- "stop" / "cancel" / "abort" → clean exit
- Same error 3x → fundamental issue, report
- Risk escalation → Phase 0 re-entry (consume backtrack)
- Backtrack cap exhausted → HALT
- CCG unavailable + HIGH → HALT
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Phase 0: risk classified, task record created
- [ ] Phase 1: AC approved (User Gate), assumptions listed
- [ ] Phase 2: domain checklist frozen, plan selected, verification passed
- [ ] Phase 2: blocker assumptions verified
- [ ] Phase 3: all AC items met with fresh evidence
- [ ] Phase 3: domain checklist passed
- [ ] Phase 3: no unverified assumptions
- [ ] Phase 3: empirical review passed (tests + build + lint)
- [ ] Phase 3: LLM review passed (HIGH/MEDIUM)
- [ ] Phase 3: deslop + regression re-verify
- [ ] Task record completed, lessons committed, state cleaned
</Final_Checklist>
