# Upgrade Impact Matrix

Use one row per SAP-documented change, grouped by workstream.

| Workstream | SAP change or step | Official source | Project evidence | Applicability | Complexity | Reasoning | Suggested action |
|---|---|---|---|---|---|---|---|
| Backend | | | | required | straightforward | | |
| Storefront | | | | needs evidence | complex | | |
| Integration | | | | not needed | straightforward | | |

## Classification Guide

| Label | Use when |
|---|---|
| `required` | SAP documents the step and project evidence shows the affected module or path is in use. |
| `recommended` | The change reduces upgrade risk or aligns the project with supported practice but is not proven mandatory. |
| `already addressed` | The project already contains the required behavior or versioned change. |
| `not needed` | Evidence shows the affected module, feature, or code path is not used. |
| `needs evidence` | The docs or codebase do not yet prove applicability. |

| Complexity | Use when |
|---|---|
| `straightforward` | Version/config updates or mechanical changes with a narrow blast radius. |
| `moderate` | Several modules, tests, or deployment data are touched but the solution path is clear. |
| `complex` | Custom override points, migrations, security/auth, external contracts, runtime jumps, or frontend major updates require design decisions. |

## Required Summary Tables

1. Source and target baseline.
2. Required changes by workstream.
3. Complex changes requiring a decision.
4. Changes marked not needed and the evidence.
5. Test and rollback considerations.
