# Phase 1: Clarify

Goal: Replace ambiguity with versioned, user-approved acceptance criteria.

Adapted from deep-interview's dimension-targeted questioning. Iterates until ambiguity is resolved, not a fixed round count.

## Skip Conditions

- `--skip-clarify` flag
- User provides explicit acceptance criteria in request
- Risk is LOW (route directly to Phase 3)

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

- Ask ONE question at a time (never batch)
- Target the dimension with the HIGHEST ambiguity score
- Gather codebase facts via explore agent BEFORE asking user about them
- After each answer, re-score ALL 5 dimensions and show updated scores
- Primary exit condition: all dimensions ≤ 0.3
- Safety cap: 10 rounds — generate best-effort AC, flag remaining ambiguity
- User override: "just do it" / "skip" → exit immediately

### 3. Perspective Shift (round 5+ only)

If ambiguity persists at round 5, inject contrarian perspective:
- "What if the opposite assumption is true?"
- "What's the simplest version that would still be useful?"
- Not a separate agent — a prompt-level perspective shift

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

On approval:
- AC becomes baseline (v1)
- Future changes require Change Log + User Gate
- Change Log format:
  ```
  ## AC v1.1 (BACKTRACK from Phase 3)
  - Changed: "JWT auth" → "JWT + OAuth2 auth"
  - Reason: OAuth2 requirement discovered during implementation
  - Impact: Phase 2 auth section needs re-verification
  ```

## Handoff → Phase 2

Update state and invoke `Skill("pipeline-plan")`. The state carries:
- AC version + file path
- Active assumptions (with status)
- Decisions made during clarification
