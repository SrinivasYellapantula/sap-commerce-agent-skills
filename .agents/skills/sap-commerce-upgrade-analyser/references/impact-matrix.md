# Upgrade Impact Matrix

Use one row per SAP-documented change or required operational activity, grouped by workstream.

| Workstream | SAP change or step | Official source | Project evidence | Applicability | Complexity | Reasoning | Suggested action |
|---|---|---|---|---|---|---|---|
| Backend | | | | required | straightforward | | |
| Storefront | | | | needs evidence | complex | | |
| Integration | | | | not needed | straightforward | | |
| Automated tooling | | | | needs evidence | moderate | | |
| CCv2 deployment | | | | needs evidence | moderate | | |
| Local/on-prem operations | | | | needs evidence | moderate | | |
| Data/system update | | | | needs evidence | complex | | |
| Solr/media/indexing | | | | needs evidence | moderate | | |
| Testing/rollback | | | | recommended | moderate | | |

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
5. Automation/tooling candidates, including SAP-provided OpenRewrite recipes or migration tools, dry-run commands, apply commands, expected changed files, and manual review requirements.
6. Non-code upgrade activities, including system update, impex/data migration, Solr reindexing, media migration, environment configuration, deployment sequencing, smoke tests, and rollback.
7. Test and rollback considerations.

## JSON Handoff

End with a valid JSON block for the implementer:

```json
{
  "schema_version": "sap-commerce-upgrade-analysis-handoff/v1",
  "baseline": {
    "current_backend_version": "",
    "target_backend_version": "",
    "current_storefront_version": "",
    "target_storefront_version": "",
    "deployment_scope": ["ccv2", "local-on-prem"]
  },
  "documentation_sources": [],
  "tool_artifacts": [],
  "automation_candidates": [],
  "required_changes": [],
  "complex_decisions": [],
  "not_needed_changes": [],
  "non_code_activities": {
    "system_update": [],
    "data_migration": [],
    "impex": [],
    "solr_reindexing": [],
    "media_migration": [],
    "deployment": [],
    "smoke_tests": [],
    "rollback": []
  },
  "missing_evidence": [],
  "implementation_inputs": []
}
```
