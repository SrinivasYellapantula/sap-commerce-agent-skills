---
name: sap-commerce-upgrade-analyser
description: Analyze SAP Commerce upgrade impact from a current backend and storefront version to a target version. Use for upgrade analysis that combines public SAP docs plus user-provided authorized SAP notes or PDFs, compares required code and non-code upgrade activities against a project baseline, covers CCv2 and local/on-prem workflows, classifies applicability and complexity, identifies missing evidence, and produces a Markdown plus JSON upgrade decision brief without implementing code.
---

# SAP Commerce Upgrade Analyser

## Overview

Turn a project baseline and a target version into an upgrade applicability matrix. Keep the result documentation-first: identify what SAP says changed, what the codebase uses, and what the project likely needs before any implementation starts.

## Required Inputs

- Require a target SAP Commerce version or update-release target. Ask for it when it is absent.
- Derive the current backend version from project evidence when possible.
- Accept the Markdown report or JSON handoff from `$sap-commerce-project-analyser` as the preferred project baseline.
- Require the storefront root or storefront baseline before making storefront-specific upgrade claims. If it is absent, list storefront work as unassessed.
- Use public SAP Help documentation by default. If SAP Partner Portal, SAP for Me, SAP Notes, or customer-only material is needed, ask the user to provide authorized PDFs, exported pages, local documentation folders, or links. Never ask for, store, or expose SAP credentials, cookies, tokens, or partner-only documents in this public skill repo.

## Workflow

1. Reconfirm the baseline.
   - Record current backend version, integration extension pack version, storefront/composable storefront version, Java/Node/Angular clues, enabled modules, and custom override hotspots.
   - Reuse project-analyser evidence instead of rediscovering it when the evidence is still current.

2. Build an official documentation ledger.
   - Search current official SAP sources for the exact source-to-target path each run. Prioritize SAP Help upgrade guides, update-release guides, release notes, compatibility matrices, deprecation/removal notes, and official composable storefront update guidance.
   - Include user-provided authorized SAP PDFs, exported SAP Notes, local doc folders, or links in the ledger when provided. Clearly label each source as `public`, `user-provided authorized`, `partial access`, or `requires customer access`.
   - Do not rely on blogs or community posts for required upgrade steps.
   - Record the doc title, version or update-release scope, affected module, and the concrete change or step.

3. Filter docs through implementation evidence.
   - Match SAP changes to enabled extensions, custom item types, Spring aliases, OCC endpoints, integration objects, data scripts, tests, storefront packages, and runtime configs.
   - Mark a documented change `not needed` only when repository evidence shows the relevant module, feature, or code path is absent or already addressed.
   - Mark uncertain cases as `needs evidence`; do not force a confident answer.

4. Classify the work twice.
   - Applicability: `required`, `recommended`, `not needed`, `needs evidence`, or `already addressed`.
   - Delivery complexity: `straightforward`, `moderate`, or `complex`.
   - Treat data migrations, OOTB override points, security/auth changes, integration payload changes, build/runtime jumps, cloud deployment changes, system update steps, Solr/media/indexing changes, and frontend major-library jumps as complexity multipliers.

5. Produce the upgrade brief.
   - Use the matrix in [references/impact-matrix.md](references/impact-matrix.md).
   - Separate backend code, storefront code, SAP integration extension pack, CCv2 deployment, local/on-prem operations, system update, data migration, impex, Solr reindexing, media migration, configuration/secrets, smoke testing, rollback, and unknowns.
   - Include a recommended upgrade sequence and a short risk register.
   - End with a valid JSON handoff for `$sap-commerce-upgrade-implementer`.

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

1. A prioritized change matrix.
2. A list of complex decisions for the implementer.
3. A list of not-needed changes with evidence.
4. A list of missing project inputs or SAP source constraints.
5. A valid JSON handoff block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
