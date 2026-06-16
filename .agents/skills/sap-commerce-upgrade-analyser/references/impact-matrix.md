# Upgrade Impact Matrix

Use one row per SAP-documented change or required operational activity, grouped by workstream.

For director-ready, steering-committee, demo, or client-facing outputs, this matrix is the evidence backbone for a polished decision pack. The final PDF should summarize the matrix instead of exposing only raw technical rows.

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
2. Executive decision summary: proceed / proceed with conditions / defer, with the top reasons.
3. Required changes by workstream.
4. Complex changes requiring a decision.
5. Changes marked not needed and the evidence.
6. Automation/tooling candidates, including SAP-provided OpenRewrite recipes or migration tools, dry-run commands, apply commands, expected changed files, and manual review requirements.
7. Non-code upgrade activities, including system update, impex/data migration, Solr reindexing, media migration, environment configuration, deployment sequencing, smoke tests, and rollback.
8. Test and rollback considerations.
9. Deliverable register for PDF, Markdown/source report, rendered diagram assets, and JSON handoff.

## Director-Ready Output Shape

Use this structure when a PDF or demo artifact is requested:

1. Title page: client/project, source version, target version, analysis date, evidence scope.
2. Executive summary: recommendation, confidence, top risks, top decisions, missing evidence.
3. Baseline and target: backend, storefront, Java/Node/Angular, deployment model, integration scope.
4. Upgrade scope by workstream: backend, storefront, integrations, deployment, data/system update, Solr/media, testing/rollback.
5. Required changes matrix: summarized for leadership with an appendix for details.
6. Automation/tooling plan: OpenRewrite or SAP tools, dry-run first, manual review, expected file families.
7. API compatibility and code impact: removed/changed APIs, project call sites, required migrations.
8. Non-code activities: system update, impex, indexing, media, config/secrets, operations, smoke tests.
9. Timeline/dependency view: sequencing, parallelizable work, decision gates.
10. Risk register and mitigations.
11. Recommendation and next steps.
12. Appendices: source ledger, detailed matrix, not-needed changes, missing evidence, JSON handoff.

## Required Diagram Assets

Create only evidence-backed diagrams. For PDF/director output, render them to SVG or PNG and embed the images.

| Diagram | Purpose | Recommended Notation | Required When | Output |
|---|---|---|---|---|
| Upgrade scope map | Show affected platform areas and workstreams | flowchart | Any multi-workstream upgrade | SVG/PNG |
| Source-to-target baseline | Show current and target runtime/application versions | flowchart or table image | Always for director-ready output | SVG/PNG or styled table |
| Workstream dependency timeline | Show sequencing and decision gates | flowchart or timeline | Moderate/complex upgrades | SVG/PNG |
| API compatibility impact map | Show SAP API changes to project call sites | flowchart/table | API/JApiCmp findings exist | SVG/PNG or styled table |
| Auth/security migration flow | Show OAuth/Spring Security changes | sequence/flowchart | Auth/security changes apply | SVG/PNG |
| Deployment/update runbook flow | Show dry-run, update, smoke, rollback stages | flowchart | CCv2/local operations are in scope | SVG/PNG |

Raw Mermaid may be included in support artifacts, but the final PDF should contain rendered images, not Mermaid code blocks as the visible diagrams.

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
  "implementation_inputs": [],
  "presentation_outputs": {
    "pdf_report": "",
    "markdown_source": "",
    "diagram_assets": [],
    "json_handoff": ""
  }
}
```
