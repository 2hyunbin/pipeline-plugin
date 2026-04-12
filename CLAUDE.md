# Pipeline Plugin

Risk-routed multi-phase agent pipeline for Claude Code.

## Skills

- `pipeline` — Main orchestrator. Invoke with `/pipeline <task>` or trigger on "pipeline", "full pipeline", "risk-aware", "systematic execution".

## Agents

| Agent | Role | Model |
|-------|------|-------|
| `pipeline-risk-analyzer` | Phase 0: codebase impact → risk classification | Haiku |
| `pipeline-domain-researcher` | Phase 2: domain best practices → checklist | Sonnet |
| `pipeline-ccg-reviewer` | Phase 2/3: multi-model adversarial review + attenuation guard | Sonnet |
| `pipeline-empirical-verifier` | Phase 2/3: tests/build/lint + failure taxonomy | Sonnet |
| `pipeline-assumption-tracker` | All phases: assumption lifecycle tracking | Sonnet |
