# 2211-jdk21.11 to 2211-jdk21.15 Implementation Gates

Read this reference when an approved backend handoff crosses these update levels. Re-open the release roots and relevant child pages linked from the analyser's source ledger before proposing or applying patches.

## Approval-sized patch groups

1. Runtime and supplied libraries: Commerce target pin, compatible Integration Extension Pack, current Solr target, Tomcat/Spring/Spring Security, and CCv2/local runtime evidence. Usually verify rather than patch vendored SAP libraries.
2. Authorizationserver/CDN: decide `authserver.enable.forwarded.header`; separately plan proxy/CDN `Set-Cookie` rewriting; verify redirect, session cookie, allowed hosts, login URI placeholders, and frontend parameters.
3. 2211-jdk21.11 accelerator/operations: remove an obsolete `wro_addons.xml` production-filter exclusion when present; decide JGroups JMX exposure.
4. 2211-jdk21.13 JDBC regression: inventory custom `JDBCInterceptorFactory` code and preserve an existing `createJDBCInterceptor(Tenant, String)` override when needed. Do not add one merely because the release mentions the fix.
5. 2211-jdk21.14 Hibernate/Data Hub: remove obsolete explicit dialect configuration and deleted statistics-bean references only where present; verify startup and monitoring.
6. 2211-jdk21.14 API/data/web changes: patch Orbeon overrides, Chinese checkout validation, wishlist `sortId`, multipart configuration, and Integration API replacements only after exact project matches and target-source verification.
7. Charon: make inventory/dry-run review a separate item from apply. Decide WebClient versus JDK client where SAP permits, synchronous versus Reactor contracts, OAuth repository/filter design, retries, concurrency, proxy, SSL, file upload, exception handling, and XML factory/filter order. Keep credentials external and never copy example insecure SSL settings into production.
8. Feature rollout: patch manifest endpoint lists and feature toggles only after an explicit activate/deactivate decision. Compare effective defaults before and after the target.
9. 2211-jdk21.15 duplicate registration: scan component-scanned controllers/classes also declared in XML under different bean names; remove or consolidate duplicates and run a startup/request-mapping smoke gate.

## Required verification additions

- Compile scans must include the full Integration API module families named by the target API report, not only the first failing extension.
- Startup must verify duplicate bean/request mappings, authorizationserver redirect host, cookie scope, custom login placeholders, servlet multipart listeners, and Hibernate/Data Hub configuration when applicable.
- Contract tests must cover wishlist sorting with `sortId`, B2B organization endpoints whose rollout state changed, Charon-replacement HTTP behaviors, OAuth token refresh, retry/concurrency behavior, proxy routing, and multipart/file uploads where used.
- SmartEdit customizations require their own Angular 21/npm ancillary build gate. Do not apply SmartEdit pins to an external Composable Storefront without its official compatibility evidence.
- Record exact live SAP Help URLs and target-source/API evidence in the decision pack. Treat the user-provided PDF as authorized run evidence, not a reusable embedded artifact.
