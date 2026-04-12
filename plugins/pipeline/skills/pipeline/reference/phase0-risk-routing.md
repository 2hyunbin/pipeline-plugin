# Phase 0: Risk Routing

Classify task risk using domain signals. File count is unreliable — a 1-file auth change is HIGH; a 3-file rename is LOW.

## Classification Criteria

**HIGH** — task touches ANY of:
| Domain | Examples |
|---|---|
| auth/authz | Login, session, JWT, OAuth, permissions, roles, access control |
| Payment | Billing, checkout, refund, subscription, pricing |
| PII/compliance | Personal data, GDPR, encryption, audit logs, data retention |
| Data migration | Schema changes, data backfill, table alterations |
| Public API/schema | External API contracts, protobuf/gRPC, webhook formats |
| Infra/deploy | CI/CD, Docker, k8s, env config, secrets, DNS |
| Irreversible ops | Data deletion, account termination, email/SMS sends |

**MEDIUM** — not HIGH, but:
- 2+ domain areas touched
- New internal interface (service, concern, interaction pattern)
- Concurrency/async changes
- Cross-pack changes

**LOW** — all of:
- Single domain, existing test coverage, low blast radius

## Steps

1. Extract domain keywords from user request
2. Run explore agent: identify blast radius (files, domains, dependencies)
3. If `--risk=` flag → use it (user override, still monitor for escalation)
4. Classify HIGH / MEDIUM / LOW
5. Write canonical task record:
   ```
   state_write(mode: "pipeline", state: {
     task_id: "<uuid>",
     risk_level: "<high|medium|low>",
     current_phase: 0,
     backtrack_count: 0,
     status: "active"
   })
   ```
6. Route to risk path

## Escalation Rule (active in ALL phases)

If any phase discovers the task touches a HIGH domain not identified in Phase 0:
1. Flag with specific evidence ("this change modifies `app/models/user/authentication.rb`")
2. Consume 1 global backtrack
3. Re-enter Phase 0 → reclassify
4. Record reclassification rationale in task record
5. Follow new risk path from current point

The HIGH domain list is NOT exhaustive. If a domain has comparable irreversibility or blast radius, escalate.
