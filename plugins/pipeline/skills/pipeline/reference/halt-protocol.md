# HALT Protocol

HALT is a fail-closed terminal state. The agent stops all work and waits for human intervention.

## Triggers

| Trigger | Phase | Why |
|---|---|---|
| Global backtrack cap exhausted | Any | Task is pivoting too much — human needs to re-scope |
| CCG unavailable + HIGH risk | Phase 2, 3 | Cannot verify HIGH-risk work without multi-model review |
| Phase hard cap exhausted + unresolved blocker | Any | Loop isn't converging — fundamental issue |
| Unresolvable blocker requiring credentials/access | Any | Agent cannot proceed without human action |

## On HALT

1. **Record state:**
   ```
   state_write(mode: "pipeline", state: {
     status: "halted",
     halt_reason: "<specific reason>",
     current_phase: <N>,
     backtrack_count: <N>,
     // ... all other fields preserved
   })
   ```

2. **Checkpoint:** save full task record snapshot

3. **Report to user:**
   ```
   PIPELINE HALTED
   
   Reason: <specific reason>
   Phase: <current phase>
   Backtracks used: <N>/<max>
   
   Current state:
   - <what's been completed>
   - <what's in progress>
   - <what's blocked>
   
   Recovery options:
   (a) Reset backtrack cap and resume from current phase
   (b) Reduce scope (suggest: <specific scope reduction>)
   (c) Abandon task
   ```

## Recovery

**Human-only.** Agent cannot self-unhalt.

| Option | What Happens |
|---|---|
| **(a) Reset + resume** | Reset backtrack counter to 0. Resume from current phase with existing state. |
| **(b) Reduce scope** | Reset backtrack counter to 0. User specifies what to cut. Agent updates AC (version bump + Change Log), then resumes from Phase 2. |
| **(c) Abandon** | Agent runs cleanup: `state_write(mode: "pipeline", state: { status: "abandoned" })`. No further action. |

## Anti-patterns

- Agent attempts to "work around" a HALT trigger → **forbidden**
- Agent downgrades HIGH to MEDIUM to avoid HALT → **forbidden**
- Agent retries CCG indefinitely hoping it comes back → **forbidden** (1 retry max, then HALT)
