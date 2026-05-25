---
name: sap-commerce-upgrade-implementer
description: Prepare and carry out SAP Commerce upgrade implementation work with explicit human review gates. Use when Codex must turn a Markdown or JSON upgrade analysis into a documented change plan covering code and non-code upgrade activities, compare approaches for complex customizations, explain reasoning and risks, wait for final approval before touching code or configuration, then implement and verify only the approved SAP Commerce or composable storefront changes.
---

# SAP Commerce Upgrade Implementer

## Overview

Convert an approved upgrade analysis into an implementation decision pack first, and code or configuration changes only after the user explicitly gives final approval to apply them. Treat SAP Commerce upgrades as code plus operational work: CCv2 deployment steps, local/on-prem system update steps, impex/data migration, Solr indexing, media migration, environment configuration, smoke testing, and rollback may all be in scope when approved.

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

## Decision Workflow

1. Read the upgrade matrix and re-check the code hotspots it cites.
2. Split work into mechanical changes, design decisions, data/deployment actions, integration contract actions, and test actions.
   - Keep CCv2 actions separate from local/on-prem actions.
   - Keep code changes separate from system update, impex/data migration, Solr reindex, media migration, environment configuration, and smoke-test actions.
3. For each complex change, provide:
   - why SAP or project evidence makes it relevant;
   - candidate approaches;
   - recommendation;
   - tradeoffs, regression risks, and compatibility effects;
   - touched file families and required tests;
   - the exact decision needed from the human reviewer.
4. For straightforward work, describe the intended patch set and verification without inflating it into a design debate.
5. Keep rejected and deferred approaches visible in the pack so the reasoning survives project switching.

## Implementation Rules

- Do not bypass the final approval gate because a change looks trivial.
- Do not implement changes that the analyser marked `not needed` unless the user explicitly overrides that conclusion.
- Preserve user changes in dirty worktrees and report conflicts or scope drift.
- Prefer project patterns and official migration steps over invented rewrites.
- Keep integration payload, data migration, and storefront/backend compatibility decisions explicit.
- Do not ask for or store SAP credentials, cookies, tokens, or partner-only documents. Use only public SAP docs and authorized user-provided docs for the current task.
- Do not hardcode company-specific, customer-specific, or project-specific details into the skill itself. Use only evidence from the current project being upgraded.
- After implementation, report changed files, verification performed, skipped verification, residual risk, and follow-up decisions.

## Handoff

If final approval has not been given, end with the decision pack and the smallest clear set of decisions the user needs to make next. If final approval has been given, end with the implementation result and verification summary. In both cases, include a valid JSON status block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
