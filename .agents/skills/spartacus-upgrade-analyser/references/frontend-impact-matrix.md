# Frontend Upgrade Impact Matrix

Use these workstreams when producing the analyser report:

| Workstream | Evidence | Classification | Complexity | Implementer handoff |
|---|---|---|---|---|
| Runtime/packages | Node, npm/yarn/pnpm, Angular, TypeScript, RxJS, Spartacus packages | required/recommended/not needed/needs evidence/already addressed | straightforward/moderate/complex | exact package families and command gates |
| Angular migration | Angular CLI, tsconfig, builders, deprecated APIs |  |  | dry-run/update command and manual fixes |
| Spartacus/CX migration | imports, providers, feature libs, CMS mappings, config |  |  | package/API changes and affected files |
| Auth/OAuth | token flow, interceptors, OCC config, backend webapps |  |  | backend/client decision and smoke tests |
| OCC/backend contracts | adapters, connectors, endpoints, DTO assumptions |  |  | endpoint tests and backend dependencies |
| CMS/SmartEdit | CMS mappings, preview config, page slots/components |  |  | preview/navigation smoke tests |
| SSR/build/deploy | server entry, prerender, builders, CCv2 deploy config |  |  | build and deployment validation |
| Tests/smoke | unit/e2e/lint/build/browser checks |  |  | exact verification commands |
