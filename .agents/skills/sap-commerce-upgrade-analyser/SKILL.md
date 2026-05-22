---
name: sap-commerce-upgrade-analyser
description: Analyze SAP Commerce upgrade impact from a current backend and storefront version to a target version. Use when Codex needs to read current official SAP Commerce and composable storefront upgrade documentation, compare required changes against a project analysis, classify straightforward and complex upgrade work, mark changes as applicable or not needed, identify missing evidence, and produce an upgrade decision brief without implementing code.
---

# SAP Commerce Upgrade Analyser

## Overview

Turn a project baseline and a target version into an upgrade applicability matrix. Keep the result documentation-first: identify what SAP says changed, what the codebase uses, and what the project likely needs before any implementation starts.

## Required Inputs

- Require a target SAP Commerce version or update-release target. Ask for it when it is absent.
- Derive the current backend version from project evidence when possible.
- Accept the output of `$sap-commerce-project-analyser` as the preferred project baseline.
- Require the storefront root or storefront baseline before making storefront-specific upgrade claims. If it is absent, list storefront work as unassessed.

## Workflow

1. Reconfirm the baseline.
   - Record current backend version, integration extension pack version, storefront/composable storefront version, Java/Node/Angular clues, enabled modules, and custom override hotspots.
   - Reuse project-analyser evidence instead of rediscovering it when the evidence is still current.

2. Build an official documentation ledger.
   - Search current official SAP sources for the exact source-to-target path each run. Prioritize SAP Help upgrade guides, update-release guides, release notes, compatibility matrices, deprecation/removal notes, and official composable storefront update guidance.
   - Do not rely on blogs or community posts for required upgrade steps.
   - Record the doc title, version or update-release scope, affected module, and the concrete change or step.

3. Filter docs through implementation evidence.
   - Match SAP changes to enabled extensions, custom item types, Spring aliases, OCC endpoints, integration objects, data scripts, tests, storefront packages, and runtime configs.
   - Mark a documented change `not needed` only when repository evidence shows the relevant module, feature, or code path is absent or already addressed.
   - Mark uncertain cases as `needs evidence`; do not force a confident answer.

4. Classify the work twice.
   - Applicability: `required`, `recommended`, `not needed`, `needs evidence`, or `already addressed`.
   - Delivery complexity: `straightforward`, `moderate`, or `complex`.
   - Treat data migrations, OOTB override points, security/auth changes, integration payload changes, build/runtime jumps, and frontend major-library jumps as complexity multipliers.

5. Produce the upgrade brief.
   - Use the matrix in [references/impact-matrix.md](references/impact-matrix.md).
   - Separate backend, storefront, SAP integration extension pack, cloud/runtime, data migration, testing, and unknowns.
   - Include a recommended upgrade sequence and a short risk register.

## Documentation Discipline

- Use exact source and target version labels, including `2211.x` versus `2211-jdk21.x` when SAP distinguishes them.
- Prefer module-specific upgrade steps over broad release-note paraphrases.
- Do not say that a change is irrelevant merely because no quick text search found it.
- State when SAP documentation is accessible only partially or when a note requires customer access.
- End with inputs needed by `$sap-commerce-upgrade-implementer`.

## Output Contract

Do not implement. End with:

1. A prioritized change matrix.
2. A list of complex decisions for the implementer.
3. A list of not-needed changes with evidence.
4. A list of missing project inputs or SAP source constraints.
