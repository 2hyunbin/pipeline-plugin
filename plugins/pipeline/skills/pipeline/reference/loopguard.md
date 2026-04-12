# LoopGuard — Loop Termination & Progress Detection

Based on agentpatterns.tech's LoopGuard pattern + Anthropic's evaluator-optimizer model.

## 4 Loop Pathologies

| Pathology | What It Looks Like | Detection |
|---|---|---|
| **Hard Loop** | Identical tool call + identical arguments repeated | Deduplicate: if same tool+args seen within run, block |
| **Soft Loop** | Minimal argument variation, illusion of progress | Check semantic similarity of last 3 outputs; if >90% similar, flag |
| **Retry Storm** | Failure retried at multiple layers simultaneously | Unified retry policy: max 2 retries per operation, then escalate |
| **Semantic Loop** | Agent rephrases/re-summarizes without new facts | Track "new facts" count per step; 3 consecutive steps with 0 new facts = stall |

## 5 Metrics

| Metric | Threshold | On Breach |
|---|---|---|
| `steps_per_phase` | Phase 1: 5, Phase 2: 3, Phase 3: 5 | HALT + report |
| `repeated_tool_rate` | >3 identical tool+args in a phase | Block repetition, force alternative |
| `no_progress_steps` | 3 consecutive steps without new facts or file changes | Escalate to user or BACKTRACK |
| `tokens_per_phase` | Handoff output limits (see SKILL.md) | Truncate with priority order |
| `backtrack_count` | HIGH: 3, MEDIUM: 2 | HALT + human review |

## Progressive Escalation

When a metric is breached:
1. **Inject reflection prompt** — force reassessment of approach
2. **Suggest alternative** — different tool, different decomposition
3. **Compress context + restart** — summarize, clear, try fresh
4. **HALT with partial results** — preserve work, report to human

## Escalation vs Backtrack Accounting

- **Escalation** (risk reclassification): consumes 1 backtrack
- **BACKTRACK** (spec gap / assumption invalidated): consumes 1 backtrack
- Both draw from the same global counter
- When counter reaches cap → HALT

## Integration

LoopGuard is a cross-cutting concern. Every phase checks metrics at iteration boundaries. The pipeline orchestrator reads backtrack_count from the task record before allowing any phase transition.
