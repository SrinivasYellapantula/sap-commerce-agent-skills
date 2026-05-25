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
| C-02 | CCv2 deployment | | needs evidence | moderate | | |
| C-03 | Local/on-prem operations | | needs evidence | moderate | | |
| C-04 | Data/system update | | needs evidence | complex | | |

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
  "code_changes": [],
  "non_code_actions": [],
  "verification": [],
  "rollback_notes": [],
  "deferred_items": [],
  "residual_risks": []
}
```
