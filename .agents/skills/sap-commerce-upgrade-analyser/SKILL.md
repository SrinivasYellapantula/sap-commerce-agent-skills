---
name: sap-commerce-upgrade-analyser
description: Analyze SAP Commerce upgrade impact from a current backend and storefront version to a target version. Use for upgrade analysis that combines public SAP docs, SAP API compatibility/JApiCmp evidence, plus user-provided authorized SAP notes, PDFs, links, and tool artifacts; compares required code, automated-tool, and non-code upgrade activities against a project baseline; covers CCv2 and local/on-prem workflows; classifies applicability and complexity; identifies missing evidence; and produces director-ready PDF/Markdown upgrade decision packs with rendered diagram images plus JSON handoff without implementing code.
---

# SAP Commerce Upgrade Analyser

## Overview

Turn a project baseline and a target version into an upgrade applicability matrix and decision pack. Keep the result documentation-first: identify what SAP says changed, what the codebase uses, what the project likely needs, and what a director or delivery sponsor must decide before any implementation starts.

## Required Inputs

- Require a target SAP Commerce version or update-release target. Ask for it when it is absent.
- Derive the current backend version from project evidence when possible.
- Accept the Markdown report or JSON handoff from `$sap-commerce-project-analyser` as the preferred project baseline.
- Require the storefront root or storefront baseline before making storefront-specific upgrade claims. If it is absent, list storefront work as unassessed.
- Use public SAP Help documentation by default. If SAP Partner Portal, SAP for Me, SAP Notes, or customer-only material is needed, ask the user to provide authorized PDFs, exported pages, local documentation folders, or links. Never ask for, store, or expose SAP credentials, cookies, tokens, or partner-only documents in this public skill repo.
- Accept optional authorized tool artifacts for the current run, such as SAP-provided OpenRewrite recipe JARs, migration scripts, or tool instructions. Record only file paths, checksums when useful, and source documents; do not commit the artifacts into this public skill repo.

## Workflow

1. Reconfirm the baseline.
   - Record current backend version, integration extension pack version, storefront/composable storefront version, Java/Node/Angular clues, enabled modules, and custom override hotspots.
   - Treat custom subclasses of SAP classes, custom beans with SAP parents, direct calls to SAP facade/service methods, custom integration objects, and overridden SAP integration services as high-risk upgrade evidence.
   - Reuse project-analyser evidence instead of rediscovering it when the evidence is still current.

2. Build an official documentation ledger.
   - Search current official SAP sources for the exact source-to-target path each run. Prioritize SAP Help upgrade guides, update-release guides, release notes, compatibility matrices, SAP API compatibility/JApiCmp reports, deprecation/removal notes, and official composable storefront update guidance.
   - Include user-provided authorized SAP PDFs, exported SAP Notes, local doc folders, links, and tool artifacts in the ledger when provided. Clearly label each source as `public`, `user-provided authorized`, `tool artifact`, `partial access`, or `requires customer access`.
   - Do not rely on blogs or community posts for required upgrade steps.
   - Record the doc title, version or update-release scope, affected module, concrete change or step, API compatibility status when relevant, related tool or recipe when documented, and whether the tool is expected to run in dry-run mode first.

3. Filter docs through implementation evidence.
   - Match SAP changes to enabled extensions, custom item types, Spring aliases, OCC endpoints, integration objects, data scripts, tests, storefront packages, and runtime configs.
   - Map API compatibility changes to custom imports, subclass hierarchies, overridden methods, constructor calls, direct method calls, Spring XML bean parents, aliases, and extension dependencies. Do not stop at import scans; removed overloads and changed method signatures require call-site scans.
   - For JDK 21, Spring 6, OAuth, Jakarta namespace, dependency, or framework-update changes, explicitly identify whether SAP-provided automation such as OpenRewrite recipes is applicable and what manual review remains after tooling.
   - For Spring Security 6 custom webapps, scan every custom web security XML for `<http>` or `<security:http>` URL-pattern definitions. If a pattern-based `http` block has no explicit `request-matcher`, flag it as a runtime startup risk: Spring Security may create `MvcRequestMatcher` and require `mvcHandlerMappingIntrospector` in the same web context. For SAP Commerce custom webapps, recommend `request-matcher="ant"` unless project evidence proves MVC matching is required and the MVC introspector bean is available in that context.
   - For the new OAuth implementation, inventory `OAuthClientDetails` data in init, initialdata, and update-deployment impex paths by client actor. Browser/storefront clients that previously used implicit or password grants must be flagged as public PKCE authorization-code clients (`public=true`, `requireProofKey=true`, `authenticationMethods=none`, no client secret, no `client_credentials` grant) even when storefront code is assessed later. Service clients should remain confidential, while mobile/custom-grant clients need an explicit decision.
   - For the new authorizationserver login flow, inventory custom authentication providers and login-form integrations, not only OAuth client data. Search custom code for `CoreAuthenticationProvider`, `AuthenticationProvider`, `UsernamePasswordAuthenticationToken`, `WebAuthenticationDetails`, `authentication.getDetails()`, casts of authentication details to `Map`, and project-specific data-isolation parameters such as site, store, tenant, or channel. Flag any provider that assumes password/custom-grant request details are still available during authorization-code login.
   - For legacy OAuth token item types, inspect the target `oauth2commons` and `authorizationserver` type system and then scan custom Backoffice config, labels, permissions, impex, and code for stale `OAuthAccessToken` and `OAuthRefreshToken` references. In 2211 JDK21 targets where these token item types are removed and token storage is handled by the new authorizationserver model, flag stale type nodes, editor areas, list views, advanced-search/simple-search config, labels, and permissions for removal instead of migration.
   - For changed `OAuthClientDetails` impex, validate every added or modified attribute against the target type system, especially `oauth2commons/resources/oauth2commons-items.xml`. Do not assume all collection attributes are `StringSet`: when the element type is an enum or item, require the correct impex lookup qualifier. For 2211 JDK21 OAuth data, `authenticationMethods` is an `AuthenticationMethodSet` of `AuthenticationMethodEnum`, so rows using `none`, `client_secret_post`, or `client_secret_basic` must use `authenticationMethods(code)`. Apply this check to init, initialdata, and update-deployment scripts.
   - Mark a documented change `not needed` only when repository evidence shows the relevant module, feature, or code path is absent or already addressed.
   - Mark uncertain cases as `needs evidence`; do not force a confident answer.

## Mandatory API Compatibility Sweeps

Before producing the final handoff for backend upgrades, run a targeted source scan from SAP API compatibility/JApiCmp findings to project evidence:

- Search custom Java and Spring XML for removed classes, deprecated-for-removal classes, changed method signatures, removed overloads, and replacement classes named in SAP API reports.
- Search for direct calls to SAP facades/services whose signatures changed, not only imports. For 2211 JDK21 paths, explicitly scan for old `OutboundServiceFacade.send(payload, integrationObjectCode, destinationId)` calls and flag migration to `SyncParameters`.
- Search custom integration modules for SAP outbound/inbound service overrides, custom integration objects, consumed destinations, and SAP CPI/SCPI quote/order/customer service subclasses.
- When a compiler/build error from an implementation run reveals a missed SAP API migration, convert the failing symbol or method signature into a project-wide scan pattern and update the analysis/handoff. Do not treat the first failing file as the full scope.
- When an impex import error reveals a missed data migration, convert the failing value, attribute, and line into a project-wide impex header/type-system scan. Example: `pk has wrong format` for OAuth values such as `none` or `client_secret_post` usually means an enum/item collection was imported without a qualifier such as `(code)`.
- When a runtime startup error mentions `MvcRequestMatcher`, `HandlerMappingIntrospectorFactoryBean`, or missing `mvcHandlerMappingIntrospector`, convert it into a project-wide Spring Security XML scan for pattern-based `<http>` / `<security:http>` blocks without explicit `request-matcher`.
- When a runtime authentication error mentions `WebAuthenticationDetails`, `UsernamePasswordAuthenticationFilter`, `AuthenticationProvider`, or a `ClassCastException` from authentication details to `Map`, convert it into a project-wide authorizationserver login-flow scan. Check every custom `CoreAuthenticationProvider` or `AuthenticationProvider`, custom login form, and storefront auth-code integration for assumptions that legacy password/custom-grant request details still contain site or tenant values.
- When a Backoffice/startup/import error mentions missing OAuth token types such as `OAuthAccessToken` or `OAuthRefreshToken`, convert it into a project-wide scan for stale legacy token item configuration. Check custom Backoffice explorer tree nodes, editor/list/search contexts, labels, permissions, impex, and code, then classify each stale reference as `remove` unless target type-system evidence shows a supported replacement type.
- Include a concrete residual work item for every custom call site found, even when OpenRewrite is expected to handle broad framework changes.

4. Classify the work twice.
   - Applicability: `required`, `recommended`, `not needed`, `needs evidence`, or `already addressed`.
   - Delivery complexity: `straightforward`, `moderate`, or `complex`.
   - Treat data migrations, OOTB override points, security/auth changes, integration payload changes, build/runtime jumps, cloud deployment changes, system update steps, Solr/media/indexing changes, and frontend major-library jumps as complexity multipliers.

5. Produce the upgrade brief.
   - Use the matrix in [references/impact-matrix.md](references/impact-matrix.md).
   - Separate backend code, storefront code, SAP integration extension pack, CCv2 deployment, local/on-prem operations, system update, data migration, impex, Solr reindexing, media migration, configuration/secrets, smoke testing, rollback, and unknowns.
   - Include an automation/tooling section listing tool artifact path, source document, intended command family, dry-run command, apply command, expected changed file families, and manual review requirements.
   - Include an API compatibility section listing removed APIs, changed signatures, project call sites, replacement API, and whether the implementer must patch, verify locally, or seek a human decision.
   - Include a recommended upgrade sequence, dependency timeline, decision log, and short risk register.
   - Include evidence-backed diagrams for upgrade scope, workstream dependencies, target-state/runtime impact, and critical migration flows when the evidence supports them.
   - For director-ready, demo, client-ready, or PDF requests, render diagrams to SVG/PNG images and export a polished PDF decision pack. The Markdown and JSON remain traceable support artifacts, not the only deliverable.
   - End with a valid JSON handoff for `$sap-commerce-upgrade-implementer`.

## Director-Ready PDF and Diagram Output

When the user needs a demo, director review, steering-committee pack, or client-facing artifact:

- Produce a polished upgrade decision pack, not only a technical Markdown/JSON handoff.
- Use a clear document flow: title page, executive decision summary, baseline and target, upgrade scope, required workstreams, major risks, timeline/dependency view, automation candidates, non-code activities, open decisions, recommendation, appendix.
- Render Mermaid or other diagram sources into real image assets such as SVG or PNG before embedding them. Do not leave visible diagrams as raw Mermaid code in the final PDF.
- Include a diagram asset register with diagram name, source file, rendered image path, evidence basis, and QA status.
- Use the PDF skill or available document/PDF tooling to create the PDF. Render and inspect representative pages or images before delivery; check that diagrams are readable, not clipped, and not blank.
- Keep a machine-readable JSON handoff for implementers, but place it in an appendix or separate support artifact. The primary deliverable for directors should be the PDF decision pack.
- If PDF or diagram rendering is blocked by missing tools, say so explicitly and deliver the Markdown source plus diagram source/assets that can be rendered later.

## Documentation Discipline

- Use exact source and target version labels, including `2211.x` versus `2211-jdk21.x` when SAP distinguishes them.
- Prefer module-specific upgrade steps over broad release-note paraphrases.
- Do not say that a change is irrelevant merely because no quick text search found it.
- State when SAP documentation is accessible only partially or when a note requires customer access.
- Never include SAP credentials, cookies, tokens, or proprietary SAP document contents in reusable public skill files. Use authorized user-provided documents only for the current analysis.
- Do not hardcode company-specific, customer-specific, or project-specific details into the skill itself. Use only evidence from the current project being analyzed.
- End with inputs needed by `$sap-commerce-upgrade-implementer`.

## Output Contract

Do not implement. End with:

1. A director-readable executive decision summary.
2. A prioritized change matrix.
3. A rendered diagram set when diagrams are included.
4. A list of complex decisions for the implementer.
5. A list of not-needed changes with evidence.
6. A list of build-error feedback items that must be rechecked project-wide, if any.
7. A list of missing project inputs or SAP source constraints.
8. A deliverable register showing PDF report, Markdown/source report, diagram image assets, and JSON handoff status.
9. A valid JSON handoff block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
