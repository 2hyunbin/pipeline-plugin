# Phase 2: Plan + Verify

**This phase is now a standalone skill: `pipeline-plan`.** This reference doc is kept for architectural context.

Invoke via `Skill("pipeline-plan")` for the authoritative execution instructions.

## Key Design: Strict Step Sequencing with Artifact Gates

```
Step 1: Domain Research  → checklist artifact  (MUST complete before Step 2)
Step 2: Plan + Critique  → plan artifact       (MUST complete before Step 3)
Step 3: Verify           → verification result (MUST complete before Phase 3)
```

Each step produces an artifact on disk. The next step checks the artifact exists before starting.
If the artifact is missing → HALT. This prevents step-skipping.

### Step sequencing (adopted from ralplan pattern)
- "MUST spawn domain-researcher BEFORE Plan agent"
- "Do NOT proceed without checklist artifact"
- "WAIT for Architect completion before spawning Critic"
- "Do NOT run Architect and Critic in parallel"

### Artifact gates
- `checklist_path` in state MUST be non-null before Step 2
- `plan_path` in state MUST be non-null before Step 3

### Plan Quality Loop (max 3 rounds)
- Architect reviews → WAIT → Critic evaluates
- Critic REJECT → revise → loop back (max 3)
- Adopted from ralplan's Planner/Architect/Critic consensus pattern

## Handoff → Phase 3

Update state and invoke `Skill("pipeline-execute")`. The state carries:
- Plan file path
- Domain checklist file path
- Active assumptions (with status — no unverified blockers)
- AC current version
