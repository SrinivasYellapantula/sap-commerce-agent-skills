---
name: sap-commerce-project-analyser
description: Analyze SAP Commerce projects and their composable storefront or Spartacus counterparts. Use when Codex needs an evidence-based architecture report for a Commerce backend, storefront, extension stack, B2B or B2C posture, OOTB-vs-custom assessment, data-model review, SAP integrations, external integrations, upgrade baseline, onboarding brief, or project comparison.
---

# SAP Commerce Project Analyser

## Overview

Produce a read-only, evidence-based project map that a Commerce engineer can reuse for onboarding and upgrade decisions. Treat the backend and storefront as separate roots when the frontend source is not colocated with the Commerce repository.

## Workflow

1. Confirm the project roots.
   - Accept a Commerce backend root and an optional storefront root.
   - If only one root is provided, inspect it before asking for another. Do not claim that a frontend is missing until checking submodules, manifests, README files, JS workspace files, CMS imports, OCC config, and local/CCv2 properties.
   - Stay read-only unless the user explicitly asks for artifacts to be written.

2. Establish the version and deployment baseline.
   - Read `manifest.json`, build files, `gradle.properties`, `localextensions.xml`, CCv2 aspects, Java/Node version files, storefront `package.json`, lockfiles, and Angular config when present.
   - Report the backend SAP Commerce version, integration extension pack version, Solr version, JDK clues, storefront library version, Angular version, and unknowns separately.
   - Use exact file evidence for every version claim.

3. Inventory the architecture.
   - Group extensions into SAP/OOTB, customer custom, partner/vendor, data/setup, backoffice, OCC/API, integration, and operational tooling.
   - Trace the main request path from storefront to OCC to facades/services/domain/data/integration edges.
   - Identify deployment nodes or aspects, Solr/search, CMS/SmartEdit, cronjobs/processes, and data import/update mechanisms that materially shape runtime behavior.

4. Classify the commercial posture.
   - Prefer site/import evidence such as `CMSSite.channel`, base stores, catalogs, B2B unit/customer models, quote/approval flows, B2B OCC modules, and frontend feature modules over naming guesses.
   - Report `B2B`, `B2C`, `mixed`, or `uncertain`, with the strongest evidence and counter-signals.

5. Measure customization.
   - Inspect `items.xml`, Spring aliases/overrides, custom controllers, populators/converters, interceptors, strategies, process definitions, impex trees, backoffice config, frontend component overrides, OCC endpoint overrides, and custom tests.
   - Separate extension of OOTB types from net-new domain models.
   - Call out override hotspots that increase upgrade risk.

6. Map integrations.
   - Distinguish SAP products and SAP platform modules from external systems.
   - Use extension selection, endpoints/destinations, integration objects, credentials placeholders, service clients, DTO mappings, webhooks, SOAP/REST clients, SSO metadata, and deployment scripts as evidence.
   - Do not expose secrets, tokens, passwords, or private endpoint credentials. Name sensitive files only when that helps the user review them.

7. Deliver an attractive report.
   - Use tables for extension groups, version baselines, integration edges, OOTB-vs-custom comparison, data-model hotspots, and upgrade-risk hotspots.
   - Include a Mermaid architecture diagram when a diagram can be inferred from code evidence.
   - Distinguish confirmed findings, inferences, unknowns, and next files to inspect.
   - Use the report shape in [references/report-shape.md](references/report-shape.md).

## Evidence Rules

- Prefer repository evidence and absolute file references over generic Commerce knowledge.
- Read deployment and data setup files before deciding whether behavior is active.
- If a storefront root is absent, report backend storefront evidence and clearly state that frontend code was not inspected.
- Treat environment property files as potentially sensitive.
- Keep a compact evidence ledger while working so the upgrade analyser can reuse it.

## Output Handoff

End with a compact upgrade baseline:

| Field | Value | Evidence |
|---|---|---|
| Backend version | | |
| Storefront version | | |
| B2B/B2C posture | | |
| High-risk custom areas | | |
| Integration families | | |
| Missing inputs for upgrade analysis | | |
