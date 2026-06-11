---
name: spartacus-upgrade-analyser
description: Analyze SAP Commerce composable storefront / Spartacus upgrade impact from a current frontend version to a target SAP Commerce Cloud or Composable Storefront patch. Use for Angular, Node, TypeScript, RxJS, Spartacus/CX package, CMS mapping, OCC contract, OAuth/authorizationserver, SmartEdit, SSR, build, lint, and runtime compatibility analysis using SAP Help, official compatibility guidance, user-provided SAP PDFs/KBAs, and storefront code evidence; produce a Markdown plus JSON upgrade handoff without implementing code.
---

# Spartacus Upgrade Analyser

## Overview

Turn a storefront codebase and target SAP Commerce / Composable Storefront version into an evidence-backed frontend upgrade matrix. This skill is frontend-only: analyse Angular, Spartacus/CX libraries, storefront configuration, OCC/auth integration, build tooling, SSR, and browser/runtime behavior. Do not implement code changes.

## Required Inputs

- Require a storefront root and target version. Accept target labels such as `2211.21.11`, `2211.21`, or the exact Composable Storefront package version when known.
- Derive the current storefront version from `package.json`, lockfiles, Angular CLI config, `ng version` outputs, and installed `@spartacus/*` / `@cx-spartacus/*` packages when available.
- Accept backend target/version evidence, backend upgrade reports, and runtime smoke-test notes as integration context only.
- Use current official SAP Help and official SAP package/compatibility documentation for every run. Use user-provided SAP Help PDFs, SAP KBAs, exported SAP Notes, or local documentation folders as authorized inputs for that run; do not copy proprietary SAP text into reusable public skill files.
- If SAP for Me, SAP Notes, or customer-only KBAs are needed, ask the user to provide authorized exports. Never ask for, store, or expose SAP credentials, cookies, or tokens.

## Workflow

1. Reconfirm the frontend baseline.
   - Record Angular, Node, npm/yarn/pnpm, TypeScript, RxJS, Zone.js, Sass, Cypress/Playwright/Karma/Jest, and browserlist clues.
   - Record Spartacus/CX packages and versions, including feature libraries, schematics, storefront styles, ASM, CDC/Gigya, SmartEdit, organization, product configuration, checkout, order, user/account, B2B, cart, search, and SSR packages.
   - Record project shape: Angular workspace, apps/libs, SSR/server entry points, custom feature modules, CMS component mappings, custom OCC adapters/connectors, guards, interceptors, ngrx state, translations, styles/themes, and environment config.

2. Build an official source ledger.
   - Search current SAP Help for the exact source-to-target path, including Composable Storefront update guides, release notes, compatibility matrices, Angular update requirements, Node/package manager guidance, OAuth/security changes, SmartEdit integration, OCC contract changes, SSR changes, and feature-library migration notes.
   - Include user-provided PDFs/KBAs as `user-provided authorized` evidence, but use them as inputs rather than the only source.
   - Prefer official SAP Help, SAP package docs, Angular official update guides, and package changelogs over blogs or community posts.
   - Record source title, version scope, affected package/module, concrete change, expected migration command or manual action, and whether a dry-run or read-only scan is possible.

3. Filter docs through project evidence.
   - Match SAP/Angular changes to actual imports, configuration providers, modules, routes, CMS mappings, OCC endpoint usage, authentication config, environment files, custom adapters/connectors, and build scripts.
   - Search for removed/deprecated Spartacus APIs and changed package names. Include import scans and call-site scans; do not stop at package versions.
   - Check OAuth/authorizationserver alignment against the upgraded backend: no legacy implicit/password assumptions, browser clients should use authorization code with PKCE when applicable, OCC calls should send bearer JWTs expected by the new backend, and backend base URLs should point at the correct webapps.
   - Check runtime-sensitive areas: SSR hydration/build, SmartEdit preview, CMS component mapping, route guards, interceptors, local storage/session storage auth behavior, CORS assumptions, custom checkout/cart/order flows, B2B org/user flows, language/currency/base-site config, and public asset paths.
   - Mark a change `not needed` only with repository evidence; otherwise use `needs evidence`.

4. Classify work.
   - Applicability: `required`, `recommended`, `not needed`, `needs evidence`, or `already addressed`.
   - Complexity: `straightforward`, `moderate`, or `complex`.
   - Treat Angular major upgrades, Node/runtime jumps, lockfile churn, authentication changes, custom Spartacus overrides, SSR, SmartEdit, custom OCC contracts, and backend integration changes as complexity multipliers.

5. Produce the upgrade brief.
   - Separate package/runtime updates, Angular migrations, Spartacus/CX migrations, custom code migrations, authentication/OAuth, OCC/backend integration, CMS/SmartEdit, SSR/build, styling/assets, tests, deployment/CCv2, and unknowns.
   - Include a recommended sequence: baseline capture, package-manager decision, migration dry-run, dependency update, compile/lint, unit tests, SSR/build, local backend integration, browser smoke tests, and CCv2 readiness.
   - Include a risk register and precise next evidence needed.
   - End with a valid JSON handoff for `$spartacus-upgrade-implementer`.

## Mandatory Scans

- Package scan: `package.json`, lockfiles, Angular config, `tsconfig*`, builder config, `browserslist`, `.nvmrc`, `.node-version`, Docker/CI files, and scripts.
- Spartacus API scan: imports from `@spartacus/*`, `@cx-spartacus/*`, custom extensions of Spartacus classes, custom providers, `ConfigModule.withConfig`, `provideConfig`, routing config, feature toggles, CMS mappings, and converter/normalizer/custom adapter classes.
- Angular migration scan: deprecated Angular APIs, module/provider patterns affected by the target Angular version, RxJS call shapes, TypeScript strictness, Sass/build builder changes, and testing builder changes.
- Auth/OCC scan: OAuth client config, token flow assumptions, interceptors, OCC base URL/webroot, `/authorizationserver` and `/resourceserver` expectations, CORS/origin assumptions, and backend endpoint contract changes.
- Runtime feedback scan: when a build/runtime error appears, convert the failing symbol, package, route, DI token, provider, or endpoint into a project-wide scan pattern and add it to the residual matrix.

## Output Contract

Do not implement. End with:

1. A prioritized frontend upgrade change matrix.
2. Complex decisions for `$spartacus-upgrade-implementer`.
3. Not-needed changes with evidence.
4. Build/runtime feedback items requiring project-wide recheck.
5. Missing SAP/project inputs.
6. A valid JSON handoff block with no comments, no trailing commas, no secrets, and no private endpoint credentials.
