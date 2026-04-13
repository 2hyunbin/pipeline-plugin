---
name: pipeline-clarify
description: "Pipeline Phase 1: Replace ambiguity with versioned acceptance criteria. Chained from pipeline dispatcher."
---

# Pipeline Phase 1 — Clarify

Replace ambiguity with versioned, user-approved acceptance criteria (AC).

## Prerequisites

Read pipeline state:
```
Read(".pipeline/state.json")
```
Verify `status: "active"` and `current_phase: "0"`. If not, HALT — phase sequencing error.

## Skip Conditions

Skip this phase and invoke `Skill("pipeline-plan")` directly if:
- `--skip-clarify` was passed
- User provided explicit acceptance criteria in original request

## Steps

### 1. Score Ambiguity

Rate 5 dimensions (0.0 = clear, 1.0 = opaque):

| Dimension | Probing Question | Origin |
|---|---|---|
| **Goal** | Why are we doing this? What problem does it solve? | Systems Ambiguity: task + expectation |
| **Scope** | What's included/excluded? How far does the change reach? | IEEE 830: complete |
| **Constraints** | What must NOT break? Compatibility, performance, API stability? | Systems Ambiguity: exception; LLM Instruction Ambiguity: critical |
| **Outcome** | How do we verify it's correct? What does "done" look like? | IEEE 830: verifiable/testable |
| **Approach** | Are there preferred methods, patterns, or tools? | Systems Ambiguity: method; LLM Instruction Ambiguity: procedural |

### 2. Iterative Questioning

Loop until all dimensions score ≤ 0.3 (safety cap: 10 rounds):

```
round = 0
while max(ambiguity_scores) > 0.3 and round < 10:
    1. Pick dimension with HIGHEST score
    2. Gather codebase facts via explore agent BEFORE asking user
    3. Ask ONE question targeting that dimension (never batch)
    4. Re-score ALL 5 dimensions based on answer
    5. Show updated scores to user
    round += 1
```

- **Primary exit condition**: all dimensions ≤ 0.3
- **Safety cap**: 10 rounds — if hit, generate best-effort AC and flag remaining ambiguity
- If user says "just do it" / "skip" → exit loop immediately, carry unresolved ambiguity into AC assumptions

### 3. Perspective Shift (round 5+ only)

If ambiguity persists at round 5, inject contrarian perspective:
- "What if the opposite assumption is true?"
- "What's the simplest version that would still be useful?"

### 4. Generate Acceptance Criteria (AC v1)

```markdown
## Acceptance Criteria v1
- [ ] AC-1: <specific, testable criterion>
- [ ] AC-2: <specific, testable criterion>
- [ ] AC-3: ...

## Open Assumptions
- A1: <assumption> [status: unverified]
- A2: <assumption> [status: unverified]
```

Rules:
- Every AC must be testable (can write a test or empirical check for it)
- No generic criteria ("implementation is complete") — replace with specific outcomes
- Assumptions tracked with status: `unverified` → `verified` → `invalidated`

### 5. User Gate

Present AC to user for approval via `AskUserQuestion`:
- Show AC list + assumption list
- Options: Approve / Request changes / Add items

On approval: AC becomes baseline (v1). Future changes require Change Log + User Gate.

### 6. Update State and Chain

**MUST update state before chaining:**
1. `Read(".pipeline/state.json")` to get current state
2. Update fields: `current_phase: "1"`, `ac_version: "v1"`, `ac_path: ".pipeline/{task_id}/ac-v1.md"`, `assumptions: [...]`
3. `Write(".pipeline/state.json", <updated state>)`

Save AC file to `.pipeline/{task_id}/ac-v1.md`.

**Then immediately invoke:** `Skill("pipeline-plan")`

## AC Change Log Format (for future BACKTRACK)

```markdown
## AC v1.1 (BACKTRACK from Phase 3)
- Changed: "JWT auth" → "JWT + OAuth2 auth"
- Reason: OAuth2 requirement discovered during implementation
- Impact: Phase 2 auth section needs re-verification
```

<Escalation_And_Stop_Conditions>
- Primary exit: all ambiguity dimensions ≤ 0.3
- Safety cap: 10 rounds — generate best-effort AC and flag remaining uncertainties as assumptions
- User override: "just do it" / "skip" → write minimal AC with unresolved dimensions noted, chain to pipeline-plan
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Ambiguity scored across 5 dimensions (Goal, Scope, Constraints, Outcome, Approach)
- [ ] Codebase facts gathered before asking user
- [ ] AC has testable criteria only (no vague items)
- [ ] Assumptions listed with status
- [ ] User approved AC via AskUserQuestion gate
- [ ] AC file saved to disk
- [ ] State updated with ac_version + ac_path
- [ ] Chained to pipeline-plan
</Final_Checklist>
