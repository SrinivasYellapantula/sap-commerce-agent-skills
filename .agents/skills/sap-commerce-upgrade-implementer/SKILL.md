---
name: sap-commerce-upgrade-implementer
description: Prepare and carry out SAP Commerce upgrade implementation work with explicit human review gates. Use to turn a Markdown or JSON upgrade analysis into a documented change plan covering code, SAP Help and API compatibility evidence, SAP-provided migration tools such as OpenRewrite, and non-code upgrade activities; default tooling to dry-run first; compare approaches for complex customizations; explain reasoning and risks; wait for approval before touching code, configuration, or running local commands; then implement and verify only approved SAP Commerce or composable storefront changes.
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
- After framework/OpenRewrite/version changes, stop before starting the next migration category and request approval for the narrowest useful compile gate, normally `ant clean all` for the backend. If the user has already run it, triage all compiler errors and update the residual patch list before continuing.
- When a compiler error reveals a missed SAP API migration, translate the error into a project-wide scan for the same symbol, method signature, import, bean alias, or superclass before preparing the patch. Fix all in-scope matches together or explicitly mark each deferred match with evidence.
- Local commands such as `ant clean all`, `ant updatesystem`, Solr indexing, impex imports, or test suites may be run only with explicit approval in a local/dev project context. For shared, staging, or production environments, prepare runbook steps unless the user has provided a safe execution context and explicit approval.
- For CCv2, prepare deployment/update instructions by default. Edit manifest, deployment, or environment configuration files only when approved. Do not trigger cloud deployments unless the user explicitly provides an appropriate authenticated workflow and approval.

## Decision Workflow

1. Read the upgrade matrix and re-check the code hotspots it cites.
   - Do not rely only on the analyser ledger or PDFs. Re-read the relevant current SAP Help update-release pages and SAP API compatibility/JApiCmp reports for the current-to-target path, then record any newly discovered mandatory change or absence of evidence.
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

## SAP Help And API Compatibility Sweep

Before declaring an implementation plan complete for an SAP Commerce backend upgrade, perform this sweep even when the analyser output, source ledger, and OpenRewrite output look complete:

- SAP Help coverage: consult the public SAP Help update-release pages for every version segment crossed, including patch/update pages, framework update pages, library migration pages, OAuth/security pages, Solr validity pages, and module-specific pages relevant to enabled extensions. If the user provides authorized SAP PDFs, use them as input, not as the only source.
- API compatibility coverage: check SAP JApiCmp/API compatibility reports for the current-to-target path and search for `REMOVED`, `Deprecated`, `forRemoval`, package moves, constructor signature changes, bean ID changes, and newly introduced replacement classes. Map each relevant change to custom imports, subclasses, Spring XML bean parents, aliases, and extension dependencies.
- Custom call-site scan: search custom backend code for removed or deprecated-for-removal SAP packages, third-party packages, changed method signatures, and removed overloads. At minimum for JDK21/Spring 6 paths, scan `hybris/bin/custom` for `org.apache.commons.lang.`, `javax.`, `org.springframework.security.oauth2`, `de.hybris.platform.sap.productconfig`, `OutboundServiceFacade.send(`, and any packages, methods, or constructors named in the JApiCmp report.
- Build-error feedback scan: for every new compiler error, scan all enabled and custom extensions for the same failing symbol or call shape, then update the decision pack/status so the analyser gap is visible to future runs.
- Bean wiring scan: search custom Spring XML and properties for SAP bean aliases, bean parents, extension names, and webapp names that changed in SAP Help or API reports. Treat custom subclasses of SAP classes and custom beans with SAP parents as high-risk compile/runtime hotspots.
- Runtime web-context feedback scan: for every startup error mentioning `MvcRequestMatcher`, `HandlerMappingIntrospectorFactoryBean`, or missing `mvcHandlerMappingIntrospector`, scan all custom web security XML files for pattern-based `<http>` / `<security:http>` blocks without explicit `request-matcher`, then prepare one in-scope patch for all matching webapps.
- Runtime controller-mapping feedback scan: for every startup error mentioning `requestMappingHandlerMapping`, `PathPatternParser`, `PatternParseException`, or duplicate captured variables such as `Not allowed to capture 'userId' twice`, scan all custom OCC/web controllers by combining class-level and method-level mappings. Fix the shared prefix or method path so the effective endpoint remains intended but no path variable name is captured twice.
- Runtime authorizationserver feedback scan: for every login/runtime error mentioning `WebAuthenticationDetails`, `UsernamePasswordAuthenticationFilter`, `AuthenticationProvider`, or a `ClassCastException` from authentication details to `Map`, scan all custom `CoreAuthenticationProvider` / `AuthenticationProvider` subclasses and custom login form integrations. Fix legacy assumptions that `authentication.getDetails()` is always a project map; handle Spring Security `WebAuthenticationDetails` safely and preserve project data-isolation decisions through an approved mechanism.
- Deprecated SAP security type feedback scan: for every static-analysis, Sonar, deprecation, or API compatibility finding mentioning `CoreUserDetails` or similar deprecated SAP security internals, scan all custom `CoreAuthenticationProvider` subclasses for direct casts, constructor calls, method parameters, and OTP token access. Prefer Spring Security `UserDetails` in custom method signatures. If the deprecated type is used only for OTP verification token access, migrate to `UserVerificationTokenService.lookupTokenWithUser(...)` and keep OTP behavior intact.
- Runtime custom-login-page feedback scan: for every authorizationserver error mentioning `loginPageUri`, `LoginPageUriPlaceholderProvider`, `PlaceholderResolutionException`, unknown placeholders, missing placeholder parameters, invalid custom login redirect, or unexpected default SAP login page usage, inspect `OAuthClientDetails.loginPageUri`, `authserver.oauthclientdetails.loginpageuri.allowed.hosts`, custom placeholder providers, Spring registration of `loginPageUriPlaceholderProviderMap` or the active placeholder resolver, and frontend authorization-code request parameters. Verify that any custom placeholders such as site, language, storefront domain, or storefront base URL have matching provider implementations, validation, tests, and storefront request parameters.
- Runtime OCC/resource-server feedback scan: for every OCC login/runtime error mentioning `Authentication.getPrincipal()`, `Jwt cannot be cast to class java.lang.String`, `org.springframework.security.oauth2.jwt.Jwt`, `UserMatchingFilter`, or bearer-token principal handling, scan all custom OCC filters, `OncePerRequestFilter` implementations, user-matching filters, authentication helpers, and access-check utilities. Fix legacy assumptions that the authenticated principal is always a `String`; handle `Jwt` via `getSubject()` and fall back to `Authentication.getName()` only when that preserves the intended user identity.
- Runtime/backoffice legacy OAuth token feedback scan: for every Backoffice, startup, system update, import, or compile error mentioning missing legacy token item types such as `OAuthAccessToken` or `OAuthRefreshToken`, inspect the target `oauth2commons` and `authorizationserver` type system and scan custom Backoffice config, labels, permissions, impex, and code. In 2211 JDK21 targets where these token item types are removed and token storage is handled by the new authorizationserver model, remove stale type nodes, editor areas, list views, advanced-search/simple-search config, labels, and permissions instead of migrating token data or recreating the old types.
- Runtime SAP superclass dependency feedback scan: for every runtime `NullPointerException`, `IllegalStateException`, or missing dependency error thrown inside an SAP superclass method even though the custom Spring bean appears to inject that dependency, inspect custom subclasses of the SAP class and their bean parents. Search for private fields and overridden setters/getters that shadow SAP parent dependencies such as `modelService`, `cartService`, `userService`, `sessionService`, `configurationService`, `i18nService`, `baseStoreService`, or commerce services. If a custom subclass overrides a dependency setter inherited from SAP, either remove the shadowing field/override when not needed or call the matching `super.setX(...)` before storing the local field. Then scan all custom subclasses with SAP bean parents for the same pattern.
- Residual ledger: add every relevant finding to the decision pack/status as `apply`, `defer`, `not applicable`, or `needs human decision`, with the SAP Help/API evidence and project evidence.

## Known JDK21 Hotspot Scans

For 2211 JDK21 update lines, explicitly inspect these patterns and do not assume OpenRewrite handled them:

- Commons Lang 2 removal: search for `org.apache.commons.lang.` imports and migrate applicable usages to `org.apache.commons.lang3.` or the SAP-documented manual replacement. `ReflectionToStringBuilder` should use `org.apache.commons.lang3.builder.ReflectionToStringBuilder`.
- SAP Product Configuration / CPQ product-info APIs: search for `ConfigurationOrderEntryProductInfoModelPopulator`, `sapProductConfigOrderEntryInfoModelPopulator`, and `sapProductConfigDefaultOrderEntryInfoModelPopulator`. For targets where SAP removed the facade populator, migrate custom code and Spring wiring to the service-layer `ConfigurationProductInfoModelPopulator` / `sapProductConfigInfoModelPopulator` pattern only after verifying the target API locally or in SAP JApiCmp.
- SAP outboundservices API: search for old `OutboundServiceFacade.send(payload, integrationObjectCode, destinationId)` calls and custom SAP CPI/SCPI outbound service subclasses. For targets where the overload is removed, build `SyncParameters` with payload object, integration object, and consumed destination, then call `send(syncParameters)`.
- Custom SAP subclasses: any custom class extending an SAP class, overriding protected SAP methods, or injecting an SAP bean by alias must be revalidated against the target source/API report before claiming the upgrade patch is complete.
- Generated and inactive modules: compile failures can hide in inactive custom extensions, autoloaded vendor folders, or generated metadata skipped by OpenRewrite. Reconcile `localextensions.xml`, autoload folders, and build output with the actual implementation scope.

## JDK21 / Spring 6 Upgrade Guardrails

When the target is a JDK21/Spring 6 SAP Commerce line, always make a post-tool configuration pass against current public SAP Help plus the project ledger. Include these checks even when OpenRewrite succeeds:

- Version/runtime pins: `manifest.json` or equivalent Commerce version, compatible extension packs, `.sdkmanrc`, CI/container Java settings, and setup documentation. Use exact runtime pins where the tool expects exact versions; keep placeholders only in human-facing examples.
- OAuth deployment model: replace legacy deployable `oauth2` webapps with the SAP-documented `authorizationserver` and `resourceserver` webapps where applicable, and verify related extensions/config such as `oauth2commons`, OAuth clients, token flows, PKCE, and custom token granters.
- Authorizationserver login compatibility: inspect custom storefront login forms and custom backend authentication providers together. In Spring Authorization Server / Spring Security 6 username-password login, authentication details may be `WebAuthenticationDetails`, not the legacy request-parameter map used by password or custom grants. Do not cast details to `Map` without an `instanceof` guard. If custom access checks need site, store, tenant, or channel context, choose and document an approved propagation/fallback strategy such as username suffix parsing, request parameter/details source, current site context, client-to-site mapping, or an existing "any assigned site" access check. Also scan custom providers for deprecated-but-still-compiling SAP security types such as `CoreUserDetails`; use Spring Security `UserDetails` for custom signatures, and use `UserVerificationTokenService.lookupTokenWithUser(...)` when OTP token data is the only reason for the deprecated type.
- Authorizationserver custom login-page URI compatibility: when a browser client uses `OAuthClientDetails.loginPageUri`, inspect whether the URI uses SAP built-in placeholders (`redirectUriHost`, `lang`, `ctx`) or project placeholders. For project placeholders, implement `LoginPageUriPlaceholderProvider`, register it in the active placeholder provider map/resolver, validate and normalize user-controlled request parameters, include unit tests, and confirm the storefront sends the required authorization request parameters. Remember that SAP's built-in `redirectUriHost` resolves only the host from `redirect_uri`; it does not include scheme or port, so local HTTPS ports and multi-site storefront paths may require a project provider such as storefront base URL, site, or language.
- OCC resource-server principal compatibility: inspect custom OCC filters and user-matching code together with the new bearer JWT flow. In Spring Security 6 resource-server requests, `Authentication.getPrincipal()` may be an `org.springframework.security.oauth2.jwt.Jwt` instead of the legacy username `String`. Do not cast the principal directly to `String`; resolve the user id through a small helper that accepts `String`, `Jwt.getSubject()`, and a documented `Authentication.getName()` fallback.
- Spring Security XML request matchers: inspect every custom `*-spring-security*.xml`, OCC security XML, and webapp security XML containing `<http>` or `<security:http>`. For Spring Security 6, do not leave pattern-based `http` blocks to the default matcher strategy in SAP Commerce root web contexts. Add `request-matcher="ant"` unless the webapp deliberately needs MVC matching and provides `mvcHandlerMappingIntrospector` in the same context. Verify this for active and inactive custom webapps that are compiled or may be enabled later.
- OCC/controller path patterns: inspect custom controller mappings under OCC and web extensions. Combine class-level `@RequestMapping` prefixes with method-level mappings and flag duplicate path-variable captures in the effective path. For Spring 6 `PathPatternParser`, repeated variables such as `/{baseSiteId}/users/{userId}` plus `/users/{userId}/...` fail at runtime even though the Java compiles. Preserve the intended endpoint shape by moving the shared prefix to the correct level or removing the duplicated segment.
- OAuth client data model: when removing unsupported implicit or password grants, classify each `OAuthClientDetails` row by actor before editing. Browser/storefront clients should become public PKCE authorization-code clients (`public=true`, `requireProofKey=true`, `authenticationMethods=none`, no client secret, no `client_credentials` grant). Do not collapse legacy browser clients into confidential service clients unless repository evidence proves server-to-server usage. Keep service clients confidential, preserve custom/mobile grants only with explicit approval, and apply equivalent changes to init, initialdata, and update-deployment impex paths.
- OAuth impex type-system validation: before applying or claiming OAuth client data complete, inspect the target `oauth2commons` item type definitions for every changed `OAuthClientDetails` attribute. Do not treat enum/item collections like string collections. In 2211 JDK21, `authenticationMethods` is an `AuthenticationMethodSet` of `AuthenticationMethodEnum`; impex headers must use `authenticationMethods(code)` for values such as `none`, `client_secret_post`, and `client_secret_basic`. Validate this in init, initialdata, and update-deployment scripts.
- Legacy OAuth token type cleanup: scan for `OAuthAccessToken`, `OAuthRefreshToken`, `oauthaccesstoken`, and `oauthrefreshtoken` across custom Backoffice XML, labels, permissions, impex, code, and generated/imported configuration. If the target type system no longer defines these token item types, remove stale custom Backoffice/admin configuration and document that existing token rows are not migrated; users and clients must obtain new tokens through the authorizationserver after upgrade.
- SAP superclass dependency setter compatibility: inspect custom classes extending SAP facade/service/strategy/filter/controller classes, especially when the custom bean uses an SAP bean parent. Flag overridden dependency setters/getters and duplicate private dependency fields that shadow the SAP superclass. Spring injection into the custom override does not populate the SAP superclass's private field unless the override delegates to `super.setX(...)`, so add the delegation or remove the override before claiming runtime smoke tests are complete.
- CCv2 aspect scheduling: make an explicit aspect-by-aspect decision for `task.auxiliaryTables.scheduler.enabled`. Prefer scheduler-capable background processing only when that matches the project topology; disable it on user-facing or admin aspects when approved.
- Solr alignment: re-check SAP Help update-release and Solr validity pages for the target. Do not assume an existing Solr minor is still the preferred target. Align the manifest Solr version, configset `luceneMatchVersion`, custom Solr config, and the required reindex plan.
- Integration Extension Pack: verify the target compatible integration pack from SAP Help Update Releases and update it separately from the base Commerce version.
- Third-party and autoloaded extensions: inspect `localextensions.xml` paths, autoloaded folders, inactive custom modules, untracked vendored code, and nested `.git` directories. Decide whether each is in scope, vendored, submodule-managed, or accidental before applying upgrade conclusions.
- Documentation consistency: update project setup docs when runtime or command expectations change, but keep docs separate from executable config in the change log.
- Parse warnings and skipped files: treat parser warnings, inactive extensions, generated Gradle metadata, and untracked files as manual review inputs, not as harmless noise.
- Removed/deprecated API sweep: incorporate the SAP Help and API compatibility sweep above into the post-tool pass. A successful OpenRewrite run is not enough if custom imports, subclasses, or bean parents still reference removed SAP or third-party APIs.

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
