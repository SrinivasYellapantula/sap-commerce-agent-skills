---
name: sap-commerce-project-analyser
description: Analyze SAP Commerce projects and their composable storefront or Spartacus counterparts. Use when Codex needs an evidence-based enterprise architecture report, project comprehension brief, extension or data-model inventory, B2B or B2C posture, critical-flow and OCC contract analysis, OOTB-vs-custom review, integration map, risk register, change-impact matrix, onboarding brief, upgrade baseline, or project comparison.
---

# SAP Commerce Project Analyser

## Overview

Produce a read-only, evidence-based project map that a Commerce engineer can reuse for onboarding, change decisions, and upgrade work. Read the project like an SAP CX architect before diving into isolated classes: understand the business shape, channels, data ownership, integrations, runtime risks, and operability. Treat the backend and storefront as separate roots when the frontend source is not colocated with the Commerce repository.

## Workflow

1. Confirm the project roots and scope.
   - Accept a Commerce backend root and an optional storefront root.
   - If only one root is provided, inspect it before asking for another. Do not claim that a frontend is missing until checking submodules, manifests, README files, JS workspace files, CMS imports, OCC config, and local/CCv2 properties.
   - Keep the analysis read-only unless the user explicitly asks for artifacts to be written.

2. Build the solution map before deep code reading.
   - Infer the business model, channels, commerce capabilities, SAP CX products, deployment model, and ownership clues from repository evidence.
   - Treat B2B/B2C, marketplace, PunchOut, dealer/distributor, D2C, multi-brand, and multi-country signals as hypotheses until site, data, feature, and integration evidence supports them.
   - Record what is confirmed, inferred, and still unknown before focusing on individual classes.

3. Establish the version and deployment baseline.
   - Read `manifest.json`, build files, `gradle.properties`, `localextensions.xml`, CCv2 aspects, Java/Node version files, storefront `package.json`, lockfiles, and Angular config when present.
   - Report the backend SAP Commerce version, integration extension pack version, Solr version, JDK clues, storefront library version, Angular version, and unknowns separately.
   - Use exact file evidence for every version claim.

4. Read the structural backbone.
   - Group extensions into SAP/OOTB, customer custom, partner/vendor, data/setup, backoffice, OCC/API, integration, and operational tooling.
   - Inspect custom `items.xml`, relations, enums, site/store/catalog/content data, initialization and update impex, `SystemSetup` classes, Spring aliases and overrides, and extension boundaries before relying on random class searches.
   - Separate SAP/OOTB behavior, project-specific customizations, reusable accelerator/framework extensions, vendor code, and copied or overridden SAP behavior.

5. Trace journeys and contracts.
   - Follow backend responsibilities in the order `Controller -> Facade -> Service -> DAO -> Model`; check whether business rules leak into controllers, populators, utilities, or frontend code.
   - Map storefront-to-OCC contracts when storefront code exists: CMS mappings, OCC endpoints, DTO fields, adapters/connectors, auth config, routes, SSR clues, and duplicated calculations.
   - Trace the critical flows supported by evidence: login/auth, product search and PDP, add to cart, checkout/place order, pricing/stock/tax, customer/account/order history, product imports, order status/documents, and integration failure or retry paths.
   - For enterprise order flows, map `Cart -> Validation -> Price/Stock -> Payment/Approval -> Order -> ERP Order -> Confirmation -> Invoice/Delivery/Status` when the project evidence supports it.

6. Inspect runtime and operational architecture.
   - Classify the commercial posture as `B2B`, `B2C`, `mixed`, or `uncertain` from site/import evidence such as `CMSSite.channel`, base stores, catalogs, B2B models, approval flows, OCC modules, and frontend features.
   - Inspect Solr/search, CMS/SmartEdit, Backoffice, cronjobs, business processes, events, interceptors, validators, auth/authz/data protection, media/documents, performance, caching, observability, deployment/config, testing maturity, documentation quality, and upgradeability when they materially shape change safety.
   - Mark a domain as unassessed when the repository does not expose enough evidence.

7. Map integrations and data ownership.
   - Distinguish SAP products and SAP platform modules from external systems.
   - Use extension selection, endpoints/destinations, integration objects, service clients, DTO mappings, webhooks, SOAP/REST clients, OAuth/SSO clues, cronjobs, events, retry/timeout config, deployment scripts, and data imports as evidence.
   - For each important integration, identify the source/master system, direction, sync/async behavior, trigger, protocol or mechanism, failure handling, retry/idempotency posture, and operational owner when evidence exists.

8. Assess customization and change safety.
   - Measure custom item types, Spring overrides, OCC endpoints, frontend component overrides, search value providers, checkout/order customizations, business processes, integration contracts, support tooling, and test gaps.
   - Separate extension of OOTB types from net-new domain models and call out high-risk override hotspots.
   - Produce risks, unknowns, quick wins, and change-impact links for the areas the user may change next.

9. Deliver an evidence-backed report.
   - Use [references/report-shape.md](references/report-shape.md) for the report structure.
   - Read the relevant parts of [references/project-analysis-playbook.md](references/project-analysis-playbook.md) for deep inspection checklists, search terms, smell lists, the first-two-days sequence, and the expected architect artifacts.
   - Use tables and Mermaid diagrams for extension groups, version baselines, data maps, integration edges, critical flows, OCC contracts, OOTB-vs-custom comparisons, risk registers, and change-impact matrices when evidence supports them.
   - Distinguish confirmed findings, inferences, unknowns, unassessed areas, and next files to inspect.

## Evidence Rules

- Prefer repository evidence and absolute file references over generic Commerce knowledge.
- Read deployment and data setup files before deciding whether behavior is active.
- If a storefront root is absent, report backend storefront evidence and clearly state that frontend code was not inspected.
- Treat environment property files as potentially sensitive.
- Do not expose secrets, tokens, passwords, OAuth credentials, private endpoint credentials, or sensitive request/response payloads. Name sensitive files only when that helps the user review them.
- Do not treat a quick text-search miss as proof that a feature, integration, or flow is absent.
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
