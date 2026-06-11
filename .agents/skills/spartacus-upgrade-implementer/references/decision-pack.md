# Spartacus Upgrade Decision Pack Shape

Use this shape before implementation approval:

| ID | Workstream | Recommendation | Files/families | Approval needed | Verification | Rollback |
|---|---|---|---|---|---|---|
| FE-01 | Runtime/package alignment |  | `package.json`, lockfile, Node config | yes/no | version/build command | revert package/lockfile |
| FE-02 | Angular migration |  | Angular config, tsconfig, app code | yes/no | compile/tests | revert patch |
| FE-03 | Spartacus/CX API migration |  | imports, config, feature modules | yes/no | build/runtime smoke | revert patch |
| FE-04 | Auth/OAuth/backend integration |  | environments, auth config, interceptors | yes/no | token/login/OCC smoke | revert config |
| FE-05 | CMS/SmartEdit |  | CMS mappings, layout config | yes/no | page/navigation/preview smoke | revert patch |
| FE-06 | SSR/build/deploy |  | builders, server files, CI | yes/no | browser/server build | revert patch |
