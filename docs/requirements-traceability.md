# Requirements Traceability

Use this document to connect each approved business rule to its implementation owner and verification evidence. Keep it aligned with `docs/business-rules.md`. Do not claim coverage until code and tests have been inspected.

## Traceability Table

| Rule IDs | Primary module | Expected verification |
|---|---|---|
| BR-001–BR-004 | Product scope / localization | Scope review; Arabic RTL and English LTR checks |
| BR-010–BR-018 | Accounts and authentication | Registration/login tests, role isolation, phone verification, recovery, worker verification and resubmission tests |
| BR-020–BR-024 | Worker profiles and administration | Profile validation, approved specialty enforcement, area mapping, identity-document access tests |
| BR-030–BR-035 | Service requests | Create/edit/close tests, required time period, optional images, request-state transition tests |
| BR-040–BR-051 | Offers and bookings | Offer sorting and matching, price/time approval, active-job cap, concurrency, schedule-conflict, cancellation tests |
| BR-060–BR-063 | No-show and delays | Grace-period, contact attempt, delay, reassignment, and customer non-response flow tests; clarify BR-063 before implementation |
| BR-070–BR-074 | Reviews and complaints | Review eligibility, complaint category/description, evidence access, outcome notifications, privacy tests |

## Maintenance Rules

- Keep rule identifiers stable once referenced by code or tests.
- When a rule changes, update the business-rules document, relevant module documentation, tests, and this table.
- Link to concrete test files or test names when they exist in the repository.
- Mark coverage as pending until verified; do not infer implementation from documentation alone.
