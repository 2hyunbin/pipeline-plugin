# CCG Synthesis Attenuation Guard

Prevents information loss during multi-model adversarial review. Addresses the structural deletion bias where "merge X into Y" recommendations become "delete X" during synthesis.

## Problem

Multi-model review has a consensus bias toward deletion:
- "Delete X" is easy to agree on (3 models say "remove")
- "Transform X into Y" produces 3 different Y proposals → no consensus → synthesizer defaults to deletion
- Each review round compounds the loss — Round 2 reviewers see the post-synthesis version, not the original
- By Round 3, lost concepts are permanently gone

## Protocol

Apply this after EVERY CCG synthesis step (Phase 2 verification, Phase 3 review):

### 1. Snapshot Before Review

Before sending to CCG reviewers, save the current artifact state as `pre_review_snapshot`.

### 2. Run CCG Review

Standard CCG: decompose → invoke (Codex + Gemini) → collect → synthesize.

### 3. Generate Original↔Result Diff

After synthesis, compare `pre_review_snapshot` against `post_review_result`:

List every concept/section/decision present in pre-review but absent in post-review.

### 4. Classify Each Diff Item

| Classification | Definition | Action |
|---|---|---|
| `intentional_removal` | Reviewer explicitly cited reason for removal | Accept — log reason in decision log |
| `transform_lost` | Reviewer said "change/merge/simplify" but synthesis deleted it | **Restore and transform** per reviewer's actual recommendation |
| `unmentioned` | No reviewer mentioned it; silently dropped during synthesis | **Flag for explicit review** — ask: intentional or accidental? |

### 5. Resolve Before Finalizing

- `intentional_removal` → proceed
- `transform_lost` → apply the transformation the reviewer actually recommended (re-read their output)
- `unmentioned` → include in final output with a note: "Not reviewed by CCG — carried forward from pre-review"

## Integration Points

- **Phase 2 Step 3 (Verify):** diff pre-review plan vs post-review plan
- **Phase 3 Review:** diff pre-review implementation assessment vs post-review assessment
- **Any future CCG usage:** this guard is a cross-cutting concern on all CCG synthesis

## Limits

- Max 2 review rounds per CCG invocation (3rd round = diminishing returns + compounding attenuation)
- If Round 2 produces >3 `transform_lost` items, stop and present the conflict to the user rather than auto-resolving
