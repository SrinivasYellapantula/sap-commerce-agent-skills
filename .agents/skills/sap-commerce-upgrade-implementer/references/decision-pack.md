# Upgrade Implementation Decision Pack

Use this document shape before code changes.

## Baseline

| Field | Value | Evidence |
|---|---|---|
| Current backend version | | |
| Target backend version | | |
| Storefront baseline | | |
| Project analysis input | | |
| Upgrade analysis input | | |

## Planned Change Set

| ID | Workstream | Change | Applicability | Complexity | Candidate files | Verification |
|---|---|---|---|---|---|---|
| C-01 | Backend | | required | straightforward | | |
| C-02 | Automated tooling dry-run | | needs evidence | moderate | | |
| C-03 | Automated tooling apply | | needs evidence | moderate | | |
| C-04 | CCv2 deployment | | needs evidence | moderate | | |
| C-05 | Local/on-prem operations | | needs evidence | moderate | | |
| C-06 | Data/system update | | needs evidence | complex | | |

## Decision Table

| Decision ID | Problem | Approach | Benefits | Costs and risks | Recommendation | Human decision |
|---|---|---|---|---|---|---|
| D-01 | | A | | | | |
| D-01 | | B | | | | |

## Data And Integration Notes

List:

1. migrations, impex/update scripts, reindexing, or cleanup actions;
2. payload/endpoint/auth changes for SAP and external systems;
3. backward compatibility or deployment sequencing constraints.

## Automated Migration Tools

Use this section for SAP-provided tools such as OpenRewrite recipes for JDK 21/Spring framework updates.

| Tool / Recipe | Artifact or Source | Purpose | Dry-Run Command | Apply Command | Expected Files | Manual Review Needed | Rollback |
|---|---|---|---|---|---|---|---|
| OpenRewrite |  | JDK/Spring migration |  |  |  |  |  |

## Operational Upgrade Actions

Separate code work from operational work.

| Area | Action | Environment | Owner / Approval Needed | Verification | Rollback |
|---|---|---|---|---|---|
| System update | | local / CCv2 / on-prem | | | |
| Impex/data migration | | | | | |
| Solr reindexing | | | | | |
| Media migration | | | | | |
| Environment configuration | | | | | |
| Smoke tests | | | | | |
| Local build/update command | | local/dev only | explicit approval required | | |
| CCv2 manifest/config edit | | CCv2 | explicit approval required | | |

## Verification Plan

| Scope | Check | Why |
|---|---|---|
| Build | | |
| Tests | | |
| Data | | |
| Integration | | |
| Storefront | | |

## Final Approval Check

Before editing, state:

1. selected decisions;
2. exact scope to apply;
3. deferred items;
4. whether the user has explicitly given final implementation approval in the current thread.

## JSON Status Block

End with a valid JSON block:

```json
{
  "schema_version": "sap-commerce-upgrade-implementation-status/v1",
  "approval_state": "analysis_received|decision_pack_proposed|approach_chosen|final_implementation_approval|implemented",
  "selected_decisions": [],
  "approved_scope": [],
  "tooling": {
    "dry_runs": [],
    "applied_tools": [],
    "manual_review_findings": []
  },
  "code_changes": [],
  "non_code_actions": [],
  "verification": [],
  "rollback_notes": [],
  "deferred_items": [],
  "residual_risks": []
}
```
