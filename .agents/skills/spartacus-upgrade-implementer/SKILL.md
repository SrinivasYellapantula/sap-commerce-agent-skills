---
name: spartacus-upgrade-implementer
description: Prepare and implement SAP Commerce composable storefront / Spartacus upgrade work with explicit human approval gates. Use to turn a frontend upgrade analysis into a decision pack, compare migration approaches, run package-manager or Angular/Spartacus migration commands in dry-run/read-only mode first, patch approved Angular/Spartacus/OCC/auth/SSR/build changes, and verify with focused frontend builds, tests, and backend integration smoke checks.
---

# Spartacus Upgrade Implementer

## Overview

Convert an approved `$spartacus-upgrade-analyser` handoff into a frontend implementation decision pack first, and make code/config/package changes only after explicit approval. Treat a Spartacus upgrade as dependency, Angular, SAP library, custom code, backend contract, auth, build, SSR, and deployment work.

## Approval State Machine

1. `analysis received`
   - Read the analyser Markdown/JSON handoff, frontend root, target version, backend status, and any authorized SAP Help PDFs/KBAs.
   - Do not edit code, package files, lockfiles, configs, generated files, or run mutating commands.

2. `decision pack proposed`
   - Present package/runtime decisions, migration command plan, expected file families, backend integration assumptions, verification plan, rollback, and unresolved risks.
   - Ask for approval before dependency changes, Angular migrations, package manager installs, or code edits.

3. `migration dry-run approved`
   - Run only approved read-only or dry-run commands where available.
   - Capture outputs and turn them into a residual patch list. Do not apply generated migrations until approved.

4. `final implementation approval`
   - Edit only approved scope.
   - Preserve user changes and avoid unrelated formatting churn.
   - Verify with the narrowest meaningful commands first, then broaden to build/test/runtime smoke checks when approved.

## Command Gates

- Treat package installs, `ng update`, schematics, lockfile writes, build output generation, and formatter runs as mutating unless clearly dry-run.
- Prefer official SAP/Angular migration commands when documented, but review generated diffs; do not assume tool output is complete.
- Do not change backend code from this skill. Record backend dependencies and hand them back to the backend upgrade workflow.
- Do not commit secrets, environment credentials, private endpoint tokens, or copied SAP KBA/PDF contents.
- For network dependency resolution, ask for approval when package managers need registry access.

## Decision Workflow

1. Re-check the frontend evidence cited by the analyser.
   - Re-read relevant current SAP Help and Angular official guidance for the exact target.
   - Confirm package manager, lockfile, Node version, Angular version, Spartacus package families, and custom high-risk areas.

2. Split implementation into approval-sized patches.
   - Runtime/package alignment.
   - Angular migration.
   - Spartacus/CX API/config migration.
   - Auth/OAuth/backend integration.
   - OCC/custom adapter/connector updates.
   - CMS/SmartEdit/SSR/build fixes.
   - Test/smoke readiness.

3. For each complex patch, provide:
   - source evidence and project evidence;
   - candidate approaches;
   - recommendation;
   - touched files;
   - verification and rollback;
   - exact human decision needed.

4. After each approved patch, update the implementation status.
   - Record changed files, commands run, skipped verification, residual risks, and next approval item.

## JDK21 / 2211 Backend Integration Guardrails

- OAuth/Auth: align browser OAuth with the upgraded backend authorizationserver/resourceserver model. Do not preserve implicit/password assumptions for browser clients. Treat PKCE/public-client changes as frontend/backend coupled decisions.
- OCC base URLs: verify the storefront points to the correct API host and webapp paths for the target backend, including `/occ`, `/authorizationserver`, and any custom OCC routes.
- Token handling: verify interceptors, refresh handling, anonymous/user token behavior, logout, SSO redirects, and storage assumptions against the target backend.
- Custom OCC contracts: scan custom adapters/connectors and generated DTO assumptions for backend route or response changes. Runtime HTTP failures must become project-wide connector/adapter scans.
- CMS/SmartEdit: verify component mappings, page slots, navigation, preview config, and SmartEdit integration after package migration.
- SSR/build: verify server bundle, prerender, browser-only API guards, asset paths, and deployment build commands separately from local dev-server success.

## Known Frontend Hotspot Scans

- `package.json`, lockfiles, Angular workspace config, `tsconfig*`, `.browserslistrc`, `.nvmrc`, CI files, Dockerfiles, and deployment scripts.
- Imports from `@spartacus/*`, `@cx-spartacus/*`, Angular core/router/forms/common/http/platform-browser, RxJS, NgRx, and feature libraries.
- `ConfigModule.withConfig`, `provideConfig`, `provideDefaultConfig`, CMS component mappings, route config, feature toggles, i18n, layout config, and environment files.
- Custom services extending Spartacus services, adapters, connectors, converters, normalizers, guards, interceptors, facades, and ngrx effects/selectors.
- Browser APIs in SSR paths, direct `window`/`document`/`localStorage` usage, and third-party scripts.
- Styles/theme imports, Sass API changes, asset references, and storefrontlib styles.

## Verification Ladder

Use the smallest approved useful step first:

1. Static scans and package/version checks.
2. TypeScript compile or Angular build for the app.
3. Lint/unit tests if present and in scope.
4. SSR/server build if enabled.
5. Local dev startup.
6. Browser smoke tests against upgraded backend: home, login, product search/PDP, cart, checkout/order, account/order history, B2B org flows, CMS navigation/footer/header, SmartEdit preview if in scope.
7. CCv2/build pipeline readiness.

## Output Contract

If approval is pending, end with the decision pack and the smallest clear decisions needed. If implementation was approved, end with changed files, verification, skipped checks, residual risks, and next steps. Include a valid JSON status block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
