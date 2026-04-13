# Pipeline Plugin for Claude Code

Risk-routed multi-phase agent pipeline with empirical-first verification.

## Install

```bash
# Clone into your Claude Code plugins directory
git clone https://github.com/2hyunbin/pipeline-plugin.git ~/.claude/plugins/pipeline-plugin
```

Or manually copy the `plugins/pipeline` directory into your project's `.claude/plugins/` folder.

## What It Does

Classifies task risk (HIGH/MEDIUM/LOW) by domain analysis, then routes through phases with controls proportional to stakes:

```
HIGH   → Clarify → Plan (full) → Execute (full)     [CCG, User Gates]
MEDIUM → Clarify → Plan (lite) → Execute (lite)     [single judge, 1 Gate]
LOW    → Execute + empirical verification only
```

## Key Features

- **Empirical-first verification** — tests/build/lint before any LLM judgment
- **Domain research** — auto-fetches best practices (OWASP, migration checklists, etc.)
- **Assumption tracking** — structured lifecycle (PENDING → CONFIRMED → INVALIDATED)
- **BACKTRACK** — when assumptions break, rewind to affected phase only
- **HALT protocol** — fail-closed on cap exhaustion or CCG unavailable + HIGH
- **CCG attenuation guard** — prevents information loss during multi-model synthesis
- **LoopGuard** — defends against 4 loop pathologies with 5 tracked metrics

## Architecture

```
SKILL.md (orchestrator, 88 lines)
  ├── Phase 0: Risk Routing          → pipeline-risk-analyzer agent
  ├── Phase 1: Clarify               → dimension-targeted questioning
  ├── Phase 2: Plan + Verify         → pipeline-domain-researcher + pipeline-ccg-reviewer
  ├── Phase 3: Execute + Review      → pipeline-empirical-verifier + pipeline-ccg-reviewer
  └── Cross-cutting                  → pipeline-assumption-tracker
```

## License

MIT
