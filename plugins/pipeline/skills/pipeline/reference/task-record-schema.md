# Canonical Task Record Schema

Single source of truth for the pipeline. Stored at `.omc/pipeline/{task_id}/record.json`.

```json
{
  "task_id": "uuid",
  "status": "active | halted | completed",
  "risk_level": "high | medium | low",
  "risk_rationale": "Why this risk level was chosen",
  "halt_reason": null,

  "ac_versions": [
    {
      "version": "v1",
      "content_path": ".omc/pipeline/{task_id}/ac-v1.md",
      "approved_at": "ISO timestamp",
      "change_log": null
    },
    {
      "version": "v1.1",
      "content_path": ".omc/pipeline/{task_id}/ac-v1.1.md",
      "approved_at": "ISO timestamp",
      "change_log": "BACKTRACK: discovered OAuth2 requirement in Phase 3"
    }
  ],

  "assumptions": [
    {
      "id": "A1",
      "content": "Existing auth middleware supports JWT",
      "status": "unverified | verified | invalidated",
      "verified_at": null,
      "invalidated_reason": null
    }
  ],

  "domain_checklist_path": ".omc/pipeline/{task_id}/checklist.md",

  "plan_choice": "Description of selected plan alternative",
  "rejected_alternatives": [
    { "name": "Alternative B", "reason": "Higher complexity, marginal benefit" }
  ],

  "backtracks": [
    {
      "from_phase": 3,
      "reason": "Assumption A1 invalidated — auth middleware lacks OAuth2",
      "ac_version_before": "v1",
      "ac_version_after": "v1.1",
      "timestamp": "ISO timestamp"
    }
  ],
  "backtrack_count": 1,

  "changed_files": [],
  "evidence": [
    { "ac_item": "AC-1", "evidence_type": "test_pass", "detail": "test/auth_test.rb:42 passes" }
  ],

  "open_risks": [],
  "approvals": [
    { "gate": "Phase 1 User Gate", "approved_at": "ISO timestamp" },
    { "gate": "Phase 2 User Gate", "approved_at": "ISO timestamp" }
  ]
}
```

## Field Notes

- **ac_versions**: Append-only. Never overwrite prior versions. Change Log is mandatory for any post-v1 version.
- **assumptions**: Status transitions: `unverified` → `verified` (evidence found) or `invalidated` (contradicted). An `invalidated` assumption triggers BACKTRACK review.
- **evidence**: Links each AC item to its verification proof. Enables AC-to-evidence traceability.
- **backtracks**: Full audit trail of why and when backtracks occurred.
- **status + halt_reason**: On HALT, `halt_reason` must specify the trigger (cap exhaustion, CCG unavailable, blocker). Human reads this to decide recovery action.
