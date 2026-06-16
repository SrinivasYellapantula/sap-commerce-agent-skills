---
name: sap-commerce-project-analyser
description: Analyze SAP Commerce projects and their composable storefront or Spartacus counterparts. Use for staged large-repository scans, evidence-based enterprise architecture reports, client codebase understanding documents, required diagram portfolios, rendered architecture diagrams, project comprehension briefs, extension or data-model inventories, B2B or B2C posture, critical-flow and OCC contract analysis, OOTB-vs-custom reviews, integration maps, risk registers, Markdown/PDF-ready deliverables plus JSON handoffs, change-impact matrices, onboarding briefs, upgrade baselines, or project comparisons.
---

# SAP Commerce Project Analyser

## Overview

Produce a read-only, evidence-based project map and codebase understanding document that a Commerce engineer, SAP CX architect, support lead, or delivery manager can reuse for onboarding, change decisions, and upgrade work. Read the project like an SAP CX architect before diving into isolated classes: understand the business shape, channels, data ownership, integrations, runtime risks, operability, and change safety. Treat the backend and storefront as separate roots when the frontend source is not colocated with the Commerce repository.

## Large Repo Strategy

SAP Commerce repositories are often too large for a direct full read. Start with a compact quick scan before deep analysis:

1. Index high-signal files and folders: `localextensions.xml`, manifests, build files, custom extension names, `items.xml`, Spring XML, impex/data folders, OCC controllers, cronjobs, process definitions, integration config, storefront `package.json`, Angular config, CMS mappings, and deployment files.
2. Produce a small evidence ledger with confirmed versions, roots, custom extensions, likely business flows, integration clues, diagram candidates, deliverable gaps, and missing inputs.
3. Deep-dive only into high-signal or high-risk areas: checkout, pricing, stock, order submission, ERP/PIM/payment integrations, B2B authorization, Solr, CMS/SmartEdit, Backoffice operations, and upgrade hotspots.
4. Build a diagram decision ledger while scanning: which diagrams are required, which are optional, which are blocked by missing evidence, and which repository artifacts support each diagram.
5. Keep a codebase understanding outline as you inspect: business purpose, solution shape, extension map, data ownership, critical flows, integrations, operational posture, risks, and next inspection steps.
6. Keep the final report evidence-driven; mark broad areas as `unassessed` instead of reading the whole repo indiscriminately.

## Workflow

1. Confirm the project roots and scope.
   - Accept a Commerce backend root and an optional storefront root.
   - If only one root is provided, inspect it before asking for another. Do not claim that a frontend is missing until checking submodules, manifests, README files, JS workspace files, CMS imports, OCC config, and local/CCv2 properties.
   - Keep the analysis read-only unless the user explicitly asks for artifacts to be written.

2. Build the solution map before deep code reading.
   - Infer the business model, channels, commerce capabilities, SAP CX products, deployment model, and ownership clues from repository evidence.
   - Treat B2B/B2C, marketplace, PunchOut, dealer/distributor, D2C, multi-brand, and multi-country signals as hypotheses until site, data, feature, and integration evidence supports them.
   - Record what is confirmed, inferred, and still unknown before focusing on individual classes.
   - Frame the client assessment around business capabilities, data ownership, user journeys, external systems, operational support, and upgrade/change risk, not only Java package structure.

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
   - For custom OCC or web controllers on Spring 6 / SAP Commerce JDK21 baselines, combine class-level `@RequestMapping` paths with method-level `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`, and `@RequestMapping` paths when inventorying endpoints. Flag duplicate captured path-variable names in the effective path, especially repeated `{userId}`, `{baseSiteId}`, `{cartId}`, `{code}`, or regex path variables. Treat this as a runtime startup risk because Spring 6 `PathPatternParser` rejects duplicate captures that older path matching may have tolerated.
   - For custom OCC, web, and security filters on Spring 6 / SAP Commerce JDK21 baselines, inventory `OncePerRequestFilter`, user-matching filters, bearer-token filters, and custom authentication helpers in the login/auth flow. Flag legacy assumptions that `Authentication.getPrincipal()` is always a `String`, that OAuth details are always a `Map`, or that Spring Security OAuth classes are still present. Treat principal-shape changes such as `org.springframework.security.oauth2.jwt.Jwt` as runtime risks for OCC calls after authorization-code or bearer-token login.
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

8. Infer the required diagram portfolio.
   - Decide diagrams from evidence, not from a fixed template. Always consider these baseline diagrams: system context, solution/container architecture, extension dependency map, integration landscape, data ownership, site-store-catalog-content model, critical journey sequence diagrams, deployment/runtime, OOTB-vs-custom impact, and change-impact map.
   - Promote a diagram to `required` when the codebase exposes real complexity or risk in that area. Common triggers: multiple channels, custom OCC controllers, custom itemtypes/relations, multi-site or multi-catalog impex, B2B approvals/permissions, real-time pricing/stock/tax/payment, ERP/PIM/OMS/CDC/CPI integrations, custom Solr value providers, custom CMS components, business processes, cronjobs, security filters, Backoffice tooling, CCv2/aspect config, or copied/overridden OOTB behavior.
   - Mark a diagram `optional` when it would help but evidence shows low complexity. Mark it `blocked` when the diagram is important but key evidence is absent, such as missing storefront source, deployment manifests, impex, integration config, or environment-neutral API contract details.
   - For each required or blocked diagram, record: purpose, audience, evidence files/classes/configs, confidence, missing evidence, recommended notation (`flowchart`, `sequenceDiagram`, `classDiagram`, or table), and whether to include Mermaid in the report.
   - Generate Mermaid only when the diagram can be made concrete with project labels. Use tables instead of vague diagrams when evidence is too thin. Never invent systems, flows, or ownership that repository evidence does not support.
   - Prefer these notation choices: C4-style flowcharts for context/container/component views, sequence diagrams for login/search/cart/checkout/order/integration flows, class or ER-style Mermaid for custom data model hotspots, flowcharts for Solr/CMS/deployment/operations, and matrices for data ownership or change impact.
   - When the user asks for a client-ready, director-ready, PDF, deck, or demo artifact, render diagrams to image files such as SVG or PNG and embed those images in the final document. Keep Mermaid source as a support artifact or appendix, not as the only diagram representation.

9. Prepare the codebase understanding document.
   - Structure the document for multiple audiences: executive summary, SAP CX architecture view, developer code map, integration/data ownership view, operations/support view, test/change-safety view, and appendix with evidence.
   - Explain how the code was analyzed: roots inspected, evidence types reviewed, areas sampled deeply, areas not assessed, and confidence levels.
   - Include a compact extension-to-responsibility map, custom data model map, OCC/API map, integration inventory, critical journey map, runtime/deployment view, risk register, and change-impact matrix when evidence supports them.
   - Separate `facts`, `inferences`, `risks`, `recommendations`, and `unknowns`. Do not hide evidence gaps; make them actionable next-inspection items.
   - For client-facing deliverables, include an artifact register listing the Markdown/source report, rendered diagram images, optional PDF, and JSON handoff. Use the PDF skill or available document/PDF tooling when a polished PDF is requested.

10. Assess customization and change safety.
   - Measure custom item types, Spring overrides, OCC endpoints, frontend component overrides, search value providers, checkout/order customizations, business processes, integration contracts, support tooling, and test gaps.
   - Separate extension of OOTB types from net-new domain models and call out high-risk override hotspots.
   - Produce risks, unknowns, quick wins, and change-impact links for the areas the user may change next.

11. Deliver an evidence-backed report.
   - Use [references/report-shape.md](references/report-shape.md) for the report structure.
   - Read the relevant parts of [references/project-analysis-playbook.md](references/project-analysis-playbook.md) for deep inspection checklists, search terms, smell lists, the first-two-days sequence, and the expected architect artifacts.
   - Include a diagram portfolio section that lists required, optional, and blocked diagrams with evidence and confidence before rendering detailed diagrams.
   - Use tables and Mermaid diagrams for extension groups, version baselines, data maps, integration edges, critical flows, OCC contracts, OOTB-vs-custom comparisons, risk registers, change-impact matrices, and the required diagram portfolio when evidence supports them.
   - For client-ready outputs, render Mermaid diagrams to images, verify the generated images are non-empty/readable, embed them in a PDF-ready report, and keep source Mermaid separately for traceability.
   - Distinguish confirmed findings, inferences, unknowns, unassessed areas, and next files to inspect.
   - End the human-readable Markdown report with a valid JSON handoff for `$sap-commerce-upgrade-analyser`.

## Evidence Rules

- Prefer repository evidence and absolute file references over generic Commerce knowledge.
- Read deployment and data setup files before deciding whether behavior is active.
- If a storefront root is absent, report backend storefront evidence and clearly state that frontend code was not inspected.
- Treat environment property files as potentially sensitive.
- Do not expose secrets, tokens, passwords, OAuth credentials, private endpoint credentials, or sensitive request/response payloads. Name sensitive files only when that helps the user review them.
- Do not treat a quick text-search miss as proof that a feature, integration, or flow is absent.
- Do not hardcode company-specific, customer-specific, or project-specific details into the skill itself. Use only evidence from the current project being analyzed.
- Keep a compact evidence ledger while working so the upgrade analyser can reuse it.

## Output Handoff

End with a compact upgrade baseline in Markdown and a machine-readable JSON block. The JSON must be valid JSON with no comments, no trailing commas, no secrets, and no private endpoint credentials.

| Field | Value | Evidence |
|---|---|---|
| Backend version | | |
| Storefront version | | |
| B2B/B2C posture | | |
| High-risk custom areas | | |
| Integration families | | |
| Missing inputs for upgrade analysis | | |

Use this JSON shape as the final block when enough evidence exists:

```json
{
  "schema_version": "sap-commerce-project-analysis-handoff/v1",
  "project_roots": {
    "backend": "",
    "storefront": "",
    "evidence_status": "confirmed|partial|unknown"
  },
  "baseline": {
    "sap_commerce_version": "",
    "storefront_version": "",
    "java_version": "",
    "node_version": "",
    "angular_version": "",
    "deployment_model": "ccv2|local|on-prem|private-cloud|hybrid|unknown"
  },
  "posture": {
    "business_model": "b2b|b2c|b2b2c|marketplace|dealer-portal|punchout|mixed|unknown",
    "customization_intensity": "low|medium|high|very-high|unknown",
    "integration_intensity": "low|medium|high|very-high|unknown",
    "change_safety": "safe|moderate|risky|fragile|unknown"
  },
  "extensions": [],
  "integrations": [],
  "critical_flows": [],
  "custom_hotspots": [],
  "risks": [],
  "diagram_portfolio": [],
  "deliverables": {
    "markdown_report": "",
    "pdf_report": "",
    "diagram_assets": [],
    "json_handoff": ""
  },
  "upgrade_baseline": {
    "high_risk_custom_areas": [],
    "integration_families": [],
    "missing_inputs": []
  },
  "next_inspection_steps": []
}
```
