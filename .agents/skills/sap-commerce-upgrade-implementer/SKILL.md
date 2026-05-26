---
name: sap-commerce-upgrade-implementer
description: Prepare and carry out SAP Commerce upgrade implementation work with explicit human review gates. Use to turn a Markdown or JSON upgrade analysis into a documented change plan covering code, SAP-provided migration tools such as OpenRewrite, and non-code upgrade activities; default tooling to dry-run first; compare approaches for complex customizations; explain reasoning and risks; wait for approval before touching code, configuration, or running local commands; then implement and verify only approved SAP Commerce or composable storefront changes.
---

# SAP Commerce Upgrade Implementer

## Overview

Convert an approved upgrade analysis into an implementation decision pack first, and code, configuration, tool execution, or local command changes only after the user explicitly approves that scope. Treat SAP Commerce upgrades as code plus operational work: SAP-provided migration tooling, CCv2 deployment steps, local/on-prem system update steps, impex/data migration, Solr indexing, media migration, environment configuration, smoke testing, and rollback may all be in scope when approved.

## Approval State Machine

1. `analysis received`
   - Read the project baseline, upgrade matrix, and any JSON handoff from `$sap-commerce-upgrade-analyser`.
   - Do not edit code, config, data, manifests, lockfiles, tests, or generated files.

2. `decision pack proposed`
   - Produce the documentation in [references/decision-pack.md](references/decision-pack.md).
   - Present approach choices for every non-straightforward change.
   - Ask for decisions or approval to refine the pack. This is not permission to edit.

3. `approach chosen`
   - Update the pack with the chosen approaches, expected files, migration/data actions, CCv2 and local/on-prem operational actions, verification plan, rollback notes, and unresolved risks.
   - Remain read-only unless the user gives an explicit final instruction such as `implement now`, `apply the approved changes`, or an equivalent current-turn approval.

4. `final implementation approval`
   - Edit only the approved scope.
   - Keep a change log mapped back to the approved decision pack.
   - Verify with the narrowest meaningful build/tests first and broaden when shared behavior or contracts changed.

## Tooling And Command Gates

- Automated migration tools such as SAP-provided OpenRewrite recipes are allowed only when the relevant artifact and instructions are provided or publicly documented for the current upgrade path.
- Default every migration tool to dry-run first. Present the dry-run command, expected output, changed file families, and rollback approach before asking to apply changes.
- Apply tool-generated changes only after explicit approval for the apply step. After applying, review the diff; do not assume generated changes are correct.
- Treat OpenRewrite as a source transformation aid, not a complete upgrade implementation. It usually will not update CCv2 manifests, runtime pins, README setup docs, CI images, Solr version policy, environment properties, data, or deployment topology.
- Implement remaining manual fixes after tool output has been reviewed and approved.
- Local commands such as `ant clean all`, `ant updatesystem`, Solr indexing, impex imports, or test suites may be run only with explicit approval in a local/dev project context. For shared, staging, or production environments, prepare runbook steps unless the user has provided a safe execution context and explicit approval.
- For CCv2, prepare deployment/update instructions by default. Edit manifest, deployment, or environment configuration files only when approved. Do not trigger cloud deployments unless the user explicitly provides an appropriate authenticated workflow and approval.

## Decision Workflow

1. Read the upgrade matrix and re-check the code hotspots it cites.
2. Split work into mechanical changes, design decisions, data/deployment actions, integration contract actions, and test actions.
   - Keep CCv2 actions separate from local/on-prem actions.
   - Keep code changes separate from system update, impex/data migration, Solr reindex, media migration, environment configuration, and smoke-test actions.
   - Keep automated tool dry-run, automated tool apply, and manual patch work as separate approval items.
3. For each complex change, provide:
   - why SAP or project evidence makes it relevant;
   - candidate approaches;
   - recommendation;
   - tradeoffs, regression risks, and compatibility effects;
   - touched file families and required tests;
   - the exact decision needed from the human reviewer.
4. For straightforward work, describe the intended patch set and verification without inflating it into a design debate.
5. Keep rejected and deferred approaches visible in the pack so the reasoning survives project switching.
6. After any migration-tool dry-run or apply, create a concrete residual patch list. Do not leave SAP Help items as broad workstreams when the required file-level changes can be inferred.

## JDK21 / Spring 6 Upgrade Guardrails

When the target is a JDK21/Spring 6 SAP Commerce line, always make a post-tool configuration pass against current public SAP Help plus the project ledger. Include these checks even when OpenRewrite succeeds:

- Version/runtime pins: `manifest.json` or equivalent Commerce version, compatible extension packs, `.sdkmanrc`, CI/container Java settings, and setup documentation. Use exact runtime pins where the tool expects exact versions; keep placeholders only in human-facing examples.
- OAuth deployment model: replace legacy deployable `oauth2` webapps with the SAP-documented `authorizationserver` and `resourceserver` webapps where applicable, and verify related extensions/config such as `oauth2commons`, OAuth clients, token flows, PKCE, and custom token granters.
- CCv2 aspect scheduling: make an explicit aspect-by-aspect decision for `task.auxiliaryTables.scheduler.enabled`. Prefer scheduler-capable background processing only when that matches the project topology; disable it on user-facing or admin aspects when approved.
- Solr alignment: re-check SAP Help update-release and Solr validity pages for the target. Do not assume an existing Solr minor is still the preferred target. Align the manifest Solr version, configset `luceneMatchVersion`, custom Solr config, and the required reindex plan.
- Integration Extension Pack: verify the target compatible integration pack from SAP Help Update Releases and update it separately from the base Commerce version.
- Third-party and autoloaded extensions: inspect `localextensions.xml` paths, autoloaded folders, inactive custom modules, untracked vendored code, and nested `.git` directories. Decide whether each is in scope, vendored, submodule-managed, or accidental before applying upgrade conclusions.
- Documentation consistency: update project setup docs when runtime or command expectations change, but keep docs separate from executable config in the change log.
- Parse warnings and skipped files: treat parser warnings, inactive extensions, generated Gradle metadata, and untracked files as manual review inputs, not as harmless noise.

## Manifest Patch Checklist

For CCv2 `manifest.json` or an equivalent deployment manifest, inspect and decide each area explicitly before implementation approval, then verify it after patching:

- Top-level metadata: schema compatibility, `commerceSuiteVersion`, compatible `extensionPacks`, `solrVersion`, image processing flags, and any target-specific manifest syntax from SAP Help.
- Runtime and config inputs: `useConfig` property locations, `localextensions.xml` source, Solr config location, persona/aspect-specific config, and whether related env property files need separate approved edits.
- Extension and addon declarations: enabled extensions, storefront addons, integration pack dependencies, OAuth/security extensions, SmartEdit/OCC webservices, and target additions or removals SAP documents for the upgrade.
- Aspect webapps: every aspect's `webapps` list, context paths, duplicate/missing webapps, legacy `oauth2` replacement with `authorizationserver` and `resourceserver`, and whether each webapp belongs on that aspect.
- Aspect properties: scheduler properties such as `task.auxiliaryTables.scheduler.enabled`, auth proxy/header properties such as `authserver.enable.forwarded.header` when CDN/proxy evidence exists, storefront context, XSS/security overrides, node groups, and any target SAP Help property changes.
- Solr manifest/config pair: align the manifest Solr minor with configset changes such as `luceneMatchVersion`, custom Solr config, and the non-code reindex plan.
- Conditional SAP Help items: record each relevant SAP Help suggestion as applied, deferred, or not applicable with project evidence, especially when the manifest does not need a code diff.
- Validation: parse JSON, run any available manifest/schema validation, compare old vs new aspect/webapp exposure, and include rollback notes for manifest/env changes.

## Implementation Rules

- Do not bypass the final approval gate because a change looks trivial.
- Do not implement changes that the analyser marked `not needed` unless the user explicitly overrides that conclusion.
- Preserve user changes in dirty worktrees and report conflicts or scope drift.
- Prefer project patterns and official migration steps over invented rewrites.
- Prefer SAP-provided migration tooling for broad mechanical framework changes, such as JDK 21 or Spring 6 updates, when the tool artifact and instructions are available. Use tooling to accelerate bulk changes, then review and complete manual fixes.
- Keep integration payload, data migration, and storefront/backend compatibility decisions explicit.
- Do not ask for or store SAP credentials, cookies, tokens, or partner-only documents. Use only public SAP docs and authorized user-provided docs for the current task.
- Do not hardcode company-specific, customer-specific, or project-specific details into the skill itself. Use only evidence from the current project being upgraded.
- After implementation, report changed files, verification performed, skipped verification, residual risk, and follow-up decisions.

## Handoff

If final approval has not been given, end with the decision pack and the smallest clear set of decisions the user needs to make next. If final approval has been given, end with the implementation result and verification summary. In both cases, include a valid JSON status block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
