---
name: pipeline-clarify
description: "Pipeline Phase 1: Replace ambiguity with versioned acceptance criteria. Chained from pipeline dispatcher."
---

# Pipeline Phase 1 — Clarify

Replace ambiguity with versioned, user-approved acceptance criteria (AC).

## Prerequisites

Read pipeline state before starting:
```
state_read(mode: "pipeline")
```
Verify `status: "active"` and `current_phase: "0"`. If not, HALT — phase sequencing error.

## Skip Conditions

Skip this phase and invoke `Skill("pipeline-plan")` directly if:
- `--skip-clarify` was passed
- User provided explicit acceptance criteria in original request

## Steps

### 1. Score Ambiguity

Rate 4 dimensions (0.0 = clear, 1.0 = opaque):

| Dimension | Probing Question |
|---|---|
| **Scope** | What's included/excluded? |
| **Acceptance Criteria** | How do we know it's done? |
| **Edge Cases** | What exceptions exist? |
| **Dependencies** | What does this depend on? |

### 2. Targeted Questioning

- Ask **ONE question at a time** (never batch)
- Target the dimension with the HIGHEST ambiguity score
- Gather codebase facts via explore agent BEFORE asking user about them
- Default: 1-2 rounds resolve most ambiguity
- Add rounds only if critical items remain unanswered (max 5 hard cap)

### 3. Perspective Shift (round 4+ only)

If ambiguity persists at round 4, inject contrarian perspective:
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
```
state_write(mode: "pipeline", state: {
  current_phase: "1",
  ac_version: "v1",
  ac_path: ".omc/pipeline/{task_id}/ac-v1.md",
  assumptions: [...]
})
```

Save AC file to `.omc/pipeline/{task_id}/ac-v1.md`.

**Then immediately invoke:** `Skill("pipeline-plan")`

## AC Change Log Format (for future BACKTRACK)

```markdown
## AC v1.1 (BACKTRACK from Phase 3)
- Changed: "JWT auth" → "JWT + OAuth2 auth"
- Reason: OAuth2 requirement discovered during implementation
- Impact: Phase 2 auth section needs re-verification
```

<Escalation_And_Stop_Conditions>
- Max 5 questioning rounds — if still unclear, generate best-effort AC and flag uncertainties
- If user says "just do it" / "skip" → write minimal AC and chain to pipeline-plan
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] Ambiguity scored across 4 dimensions
- [ ] Codebase facts gathered before asking user
- [ ] AC has testable criteria only (no vague items)
- [ ] Assumptions listed with status
- [ ] User approved AC via AskUserQuestion gate
- [ ] AC file saved to disk
- [ ] State updated with ac_version + ac_path
- [ ] Chained to pipeline-plan
</Final_Checklist>
