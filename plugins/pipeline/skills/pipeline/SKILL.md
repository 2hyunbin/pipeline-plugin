---
name: pipeline
description: Structured multi-phase agent pipeline for complex tasks. Clarifies requirements → researches domain → plans → executes with artifact gates and CCG verification. Triggers on "pipeline", "full pipeline", "systematic execution".
argument-hint: "[--skip-clarify] <task description>"
---

# Pipeline — Entry Point

This skill initializes pipeline state and routes to the appropriate phase skill.

Use this when a task needs structured planning before execution. For simple tasks, just do them directly.

## On Invoke

1. Initialize state:
   ```
   Write(".pipeline/state.json", {
     "task_id": "<generate>",
     "current_phase": "0",
     "backtrack_count": 0,
     "status": "active",
     "task_summary": "<1-2 sentence task description>",
     "ac_version": null,
     "ac_path": null,
     "checklist_path": null,
     "plan_path": null,
     "assumptions": []
   })
   ```
2. Route:
   - Default → `Skill("pipeline-clarify")`
   - `--skip-clarify` → `Skill("pipeline-plan")`

## Flow

```
/pipeline → pipeline-clarify → pipeline-plan → pipeline-execute
```

## Principles

1. **Empirical-first** — tests/build/lint before LLM judge, always
2. **Disk-as-memory** — state on disk; each phase skill reads it fresh
3. **Fail-closed** — cap exhaustion or CCG unavailable → HALT
4. **Phase isolation** — each phase = separate skill invocation with fresh context
5. **Artifact gates** — each phase produces disk artifacts; next phase checks they exist

## Shared State

```json
{
  "task_id": "string",
  "current_phase": "0 | 1 | 2-research | 2-plan | 2-verify | 3",
  "backtrack_count": 0,
  "status": "active | halted | completed",
  "task_summary": "string",
  "ac_version": null,
  "ac_path": null,
  "checklist_path": null,
  "plan_path": null,
  "assumptions": []
}
```

## Infrastructure

- **LoopGuard**: See [reference/loopguard.md](reference/loopguard.md)
- **HALT**: See [reference/halt-protocol.md](reference/halt-protocol.md)
- **CCG Guard**: See [reference/ccg-attenuation-guard.md](reference/ccg-attenuation-guard.md)
- **Task Record**: See [reference/task-record-schema.md](reference/task-record-schema.md)
