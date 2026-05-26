# Upgrade Implementation Decision Pack

Use this document shape before code changes.

## Baseline

| Field | Value | Evidence |
|---|---|---|
| Current backend version | | |
| Target backend version | | |
| Current integration extension pack | | |
| Target integration extension pack | | |
| Current Java/runtime pin | | |
| Target Java/runtime pin | | |
| Current Solr and Lucene/configset baseline | | |
| Target Solr and Lucene/configset decision | | |
| Storefront baseline | | |
| Project analysis input | | |
| Upgrade analysis input | | |

## Planned Change Set

| ID | Workstream | Change | Applicability | Complexity | Candidate files | Verification |
|---|---|---|---|---|---|---|
| C-01 | Branch and safety | | required | straightforward | Git metadata only | |
| C-02 | Version/runtime | Update Commerce version, compatible extension packs, Java/runtime pins, CI/container settings, and setup docs. | required | moderate | `manifest.json`, `.sdkmanrc`, CI files, README/setup docs | |
| C-03 | Automated tooling dry-run | | needs evidence | moderate | Tool output only | |
| C-04 | Automated tooling apply | | needs evidence | moderate | Java/XML/dependency files reported by tool | |
| C-05 | Post-tool residual pass | Convert tool gaps and SAP Help items into exact residual patches. | required | moderate | Manifest/config/docs/Solr/OAuth/env files not touched by tool | |
| C-06 | OAuth deployables and implementation | Replace legacy OAuth webapps/config/classes with target authorization server/resource server model where applicable. | required/needs evidence | complex | `manifest.json`, `localextensions.xml`, OAuth config, custom token/user matching code, impex | |
| C-07 | CCv2 aspect scheduling | Decide and apply `task.auxiliaryTables.scheduler.enabled` per aspect. | recommended/needs evidence | moderate | `manifest.json`, env properties | |
| C-08 | Solr/search | Align target Solr minor, `luceneMatchVersion`, custom configsets, and reindex plan. | required/needs evidence | moderate | `manifest.json`, Solr configsets, search customizations | |
| C-09 | Third-party/autoloaded extensions | Inspect autoload paths, untracked vendored folders, nested `.git`, and inactive extensions before scope decisions. | required | moderate | `localextensions.xml`, `hybris/bin/custom/**`, vendored extension folders | |
| C-10 | CCv2 deployment | | needs evidence | moderate | Manifest/env/runbook | |
| C-11 | Local/on-prem operations | | needs evidence | moderate | Local scripts/config | |
| C-12 | Data/system update | | needs evidence | complex | Impex/update/system data | |

## Decision Table

| Decision ID | Problem | Approach | Benefits | Costs and risks | Recommendation | Human decision |
|---|---|---|---|---|---|---|
| D-01 | | A | | | | |
| D-01 | | B | | | | |

## Data And Integration Notes

List:

1. migrations, impex/update scripts, reindexing, or cleanup actions;
2. payload/endpoint/auth changes for SAP and external systems;
3. backward compatibility or deployment sequencing constraints;
4. OAuth client/grant/PKCE/token changes and secret-safe handling;
5. Solr reindexing and custom search parity checks;
6. vendored or autoloaded extension ownership decisions.

## Automated Migration Tools

Use this section for SAP-provided tools such as OpenRewrite recipes for JDK 21/Spring framework updates.

| Tool / Recipe | Artifact or Source | Purpose | Dry-Run Command | Apply Command | Expected Files | Manual Review Needed | Rollback |
|---|---|---|---|---|---|---|---|
| OpenRewrite |  | JDK/Spring migration |  |  |  |  |  |

After tool apply, explicitly list what the tool did not cover: CCv2 manifest, runtime pins, setup docs, CI/container settings, OAuth deployable webapps, Solr minor/Lucene config, environment properties, data/impex, and skipped or parser-warning files.

## Operational Upgrade Actions

Separate code work from operational work.

| Area | Action | Environment | Owner / Approval Needed | Verification | Rollback |
|---|---|---|---|---|---|
| System update | | local / CCv2 / on-prem | | | |
| Impex/data migration | | | | | |
| Solr reindexing | | | | | |
| Media migration | | | | | |
| Environment configuration | | | | | |
| OAuth/runtime secret config | | | | | |
| CCv2 aspect scheduling | | CCv2 | explicit approval required | | |
| Smoke tests | | | | | |
| Local build/update command | | local/dev only | explicit approval required | | |
| CCv2 manifest/config edit | | CCv2 | explicit approval required | | |

## Verification Plan

| Scope | Check | Why |
|---|---|---|
| Build | | |
| Tests | | |
| Runtime pins | | |
| Manifest/schema | | |
| OAuth/auth | | |
| Solr/search | | |
| Third-party/autoloaded extensions | | |
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
