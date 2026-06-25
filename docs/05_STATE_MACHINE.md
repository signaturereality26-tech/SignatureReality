# 05 - State Machine Definition

## Lead State Machine

### States Definition

```
NEW
 ├─ Lead created, not verified
 ├─ Transitions: → VERIFIED, → INVALID
 └─ Default Duration: 24 hours

VERIFIED
 ├─ Lead verified by agent
 ├─ Transitions: → ASSIGNED, → INVALID
 └─ Default Duration: 48 hours

ASSIGNED
 ├─ Lead assigned to agent
 ├─ Transitions: → QUALIFIED, → INVALID
 └─ Default Duration: 7 days

QUALIFIED
 ├─ Lead meets scoring criteria
 ├─ Transitions: → SHORTLISTED, → INACTIVE
 └─ Default Duration: 14 days

SHORTLISTED
 ├─ Properties shortlisted for lead
 ├─ Transitions: → SITE_VISIT, → NO_MATCH
 └─ Default Duration: 7 days

SITE_VISIT
 ├─ Site visit scheduled/completed
 ├─ Transitions: → NEGOTIATING, → NOT_INTERESTED
 └─ Default Duration: 14 days

NEGOTIATING
 ├─ Offer/counter-offer in progress
 ├─ Transitions: → TOKEN_RECEIVED, → DEAL_FAILED
 └─ Default Duration: 30 days

TOKEN_RECEIVED
 ├─ Token received, agreement ready
 ├─ Transitions: → CLOSED, → DEAL_FAILED
 └─ Default Duration: 45 days

CLOSED
 ├─ Deal completed and closed
 ├─ Transitions: None (terminal state)
 └─ Duration: Final state

INVALID
 ├─ Lead is invalid/duplicate
 ├─ Transitions: None (terminal state)
 └─ Duration: Final state

NO_MATCH
 ├─ No matching properties found
 ├─ Transitions: → ASSIGNED (re-assign)
 └─ Duration: Final state for current assignment

NOT_INTERESTED
 ├─ Lead not interested in properties
 ├─ Transitions: → ASSIGNED (re-assign)
 └─ Duration: Hold state, can be reactivated

DEAL_FAILED
 ├─ Deal negotiation failed
 ├─ Transitions: → ASSIGNED (re-assign)
 └─ Duration: Final state for deal, lead can continue

INACTIVE
 ├─ Lead inactive (no action for 30+ days)
 ├─ Transitions: → ASSIGNED (re-activate)
 └─ Duration: Hold state

ARCHIVED
 ├─ Lead archived after 1 year
 ├─ Transitions: None (final archive)
 └─ Duration: Historical storage
```

### State Transition Diagram

```
          ┌─────────────────────────────────────────────────┐
          │                                                 │
          ▼                                                 │
    ┌──────────┐                                            │
    │   NEW    │ ─────────────────────────────────┐         │
    └────┬─────┘                                  │         │
         │                                        ▼         │
         │                                   ┌─────────┐    │
         └──────────────────────────────────>│ INVALID │    │
                                              └─────────┘    │
                                                             │
    ┌──────────────┐                                        │
    │  VERIFIED    │ ─────────────────────────────┐         │
    └────┬─────────┘                              │         │
         │                                        ▼         │
         │                                   ┌─────────┐    │
         └──────────────────────────────────>│ INVALID │    │
                                              └─────────┘    │
                                                             │
    ┌──────────────┐                                        │
    │  ASSIGNED    │ ─────────────────────────────┐         │
    └────┬─────────┘                              │         │
         │                                        ▼         │
         │                                   ┌─────────┐    │
         └──────────────────────────────────>│ INVALID │    │
                                              └─────────┘    │
                                                             │
    ┌──────────────┐                                        │
    │  QUALIFIED   │ ────────────────┐                      │
    └────┬─────────┘                  │                     │
         │                            ▼                     │
         │                      ┌──────────┐                │
         │                      │ INACTIVE │                │
         │                      └──────────┘                │
         │                                                  │
    ┌────────────┐      ┌──────────────┐                    │
    │SHORTLISTED│─────>│   NO_MATCH   │                    │
    └────┬───────┘      └──────────────┘                    │
         │                                                  │
    ┌────────────┐                                          │
    │ SITE_VISIT │ ─────────────────┐                      │
    └────┬───────┘                   │                     │
         │                           ▼                     │
    ┌────────────────┐          ┌──────────────┐           │
    │  NEGOTIATING   │          │ NOT_INTERESTED          │
    └────┬───────────┘          └──────────────┘           │
         │                                                  │
         ├─────────────────┐                               │
         │                 ▼                               │
         │          ┌──────────────┐                       │
         │          │ DEAL_FAILED  │                       │
         │          └──────────────┘                       │
         │                                                  │
    ┌────────────────┐                                     │
    │ TOKEN_RECEIVED │                                     │
    └────┬───────────┘                                     │
         │                                                  │
    ┌────────────┐                                         │
    │   CLOSED   │ ────────────────────────────────────────┘
    └────────────┘
```

### Transition Rules

| From | To | Condition | Action |
|------|----|-----------|---------|
| NEW | VERIFIED | Agent verifies | Send notification |
| NEW | INVALID | Duplicate found | Archive lead |
| VERIFIED | ASSIGNED | Manager assigns | Send assignment email |
| VERIFIED | INVALID | Agent rejects | Archive lead |
| ASSIGNED | QUALIFIED | Score > 50 | Activate matching |
| ASSIGNED | INVALID | Lead rejects | Archive lead |
| QUALIFIED | SHORTLISTED | Matches found | Send shortlist |
| QUALIFIED | INACTIVE | No action 30+ days | Send reminder |
| SHORTLISTED | SITE_VISIT | Visit scheduled | Send visit details |
| SHORTLISTED | NO_MATCH | No properties match | Archive current |
| SITE_VISIT | NEGOTIATING | Interest expressed | Initialize negotiation |
| SITE_VISIT | NOT_INTERESTED | Lead not interested | Hold lead |
| NEGOTIATING | TOKEN_RECEIVED | Agreement reached | Create token record |
| NEGOTIATING | DEAL_FAILED | Deal collapsed | Mark failed |
| TOKEN_RECEIVED | CLOSED | Registry complete | Generate report |
| TOKEN_RECEIVED | DEAL_FAILED | Deal canceled | Mark failed |

---

Next: See [06_DATABASE_SCHEMA.md](06_DATABASE_SCHEMA.md) for database structure.
