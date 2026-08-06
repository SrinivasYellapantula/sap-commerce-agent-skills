# Frontend Implementation Gates for Backend 2211-jdk21.11 to .15

Read this reference when the approved frontend handoff integrates with a backend crossing these update levels.

## Patch boundaries

- Keep Composable Storefront package migration separate from SAP Commerce SmartEdit's Angular 21/npm ancillary migration. Require official storefront compatibility evidence before changing `@spartacus/*`, `@cx-spartacus/*`, Angular, Node, TypeScript, RxJS, or lockfiles.
- Treat authorization-code redirect host, CDN cookie rewrite, backend allowed hosts, client Impex, and `LoginPageUriPlaceholderProvider` beans as backend/operations dependencies. Patch only the approved storefront request parameters, redirect URI, language/context propagation, and callback behavior.
- Change wishlist `sort` to `sortId` only in connectors/callers proven to target the affected wishlist operations. Update mocks, contract tests, generated clients, and URL assertions together.
- Do not work around disabled OCC endpoints in UI code. Get an explicit backend manifest rollout decision for wishlist and B2B organization endpoints, then align feature availability and error handling.
- Gate browser smoke tests on a clean 2211-jdk21.15 backend startup because duplicate XML/component-scanned controller registrations can cause ambiguous mappings.

## Verification additions

1. Authorization request: client id, PKCE, redirect URI, `ui_locales`, `ctx`, and any custom context parameters.
2. Callback: public host, HTTPS/port/path, session cookie scope, error redirects, refresh/logout, and multisite behavior.
3. OCC contracts: wishlist `sortId`, wishlist endpoint rollout, B2B unit retrieval/default-selection endpoints, and unauthorized/disabled responses.
4. SmartEdit, when customized: separate Angular 21 compile/lint/unit gate and preview/deeplink smoke tests.
5. Composable Storefront: its own supported Node/Angular/Spartacus build, SSR build, and browser flows; never use SmartEdit's package table as the migration command input.

Record all backend-owned blockers in the status JSON rather than editing backend files from this skill.
