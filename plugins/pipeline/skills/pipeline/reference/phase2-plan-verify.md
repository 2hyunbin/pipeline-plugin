# Phase 2: Plan + Verify

Goal: Research domain standards, generate plan with alternatives, verify empirically then adversarially.

## Step 1: Domain Research (inline sub-step, not separate phase)

1. Identify task domain(s) from Phase 1 AC + codebase exploration
2. Query Context7 or WebSearch for domain best practices:
   - auth → OWASP Top 10, session management guidelines
   - migration → migration safety checklist, rollback patterns
   - API → API design guidelines, versioning strategies
   - concurrency → race condition patterns, idempotency checklists
3. Generate **domain checklist** — separate from AC:
   - AC = "what to achieve" (user-defined outcomes)
   - Checklist = "what standards to meet" (industry-sourced criteria)
4. Brief User Gate: confirm checklist relevance → freeze

## Step 2: Plan

1. **Explore** codebase for implementation surface (explore agent)
2. Generate **2-3 alternatives** with pros/cons for each
3. **Select** best alternative with rationale + risk notes
4. **HIGH only**: 3 failure scenarios + rollback validation plan
   - Each scenario: trigger condition + impact + specific mitigation
   - Not FMEA matrices — concrete "if X happens, do Y"
5. **Assumption register** — structured tracking:
   ```json
   {
     "id": "A1",
     "content": "Existing auth middleware supports JWT refresh",
     "status": "unverified",
     "blocker": true,
     "verified_at": null,
     "invalidated_reason": null
   }
   ```
6. **Verify blocker assumptions first** — before proceeding to Step 3
   - Read code, run tests, check API docs
   - Update status: `unverified` → `verified` or `invalidated`
   - If blocker invalidated → BACKTRACK or re-plan

## Step 3: Verify

### Primary: Empirical (ALL risk levels)

Validate technical preconditions:
- Dependencies exist and are accessible
- APIs/interfaces referenced in plan actually exist in codebase
- Test infrastructure supports planned changes
- No conflicting in-progress work (check git status)

### Secondary: LLM (risk-differentiated)

| Risk | Verification Method |
|---|---|
| HIGH | CCG 3-model (Codex + Gemini + Claude synthesis) |
| MEDIUM | Single LLM judge (Architect agent) |
| LOW | Skip — already routed to Phase 3 |

**CCG availability:** 30s timeout + 1 retry failure = unavailable.
**CCG unavailable + HIGH → HALT.** Do not proceed. Do not fall back to single judge for HIGH.

### CCG Synthesis Attenuation Guard

Applied to every CCG synthesis in this phase. See [ccg-attenuation-guard.md](ccg-attenuation-guard.md) for full protocol.

Summary: after synthesis, diff pre-review plan vs post-review plan. Classify missing items as `intentional_removal`, `transform_lost`, or `unmentioned`. Review `transform_lost` + `unmentioned` before finalizing.

**Max 3 rounds.** → **User Gate (HIGH only).**

## Handoff → Phase 3

≤2K tokens:
- Selected plan + rationale
- Domain checklist file path
- Active assumptions (with status — no unverified blockers)
- AC current version
- Rejected alternatives (brief, for audit trail)
