# SAP Commerce Agent Skills

Reusable AI agent skills for SAP Commerce Cloud and Composable Storefront project analysis, upgrade planning, and gated implementation work.

This repository is intended to behave like a small specialist agent team. The skills are not generic prompts; they encode repeatable SAP Commerce upgrade workflows, evidence rules, human approval gates, and project-specific lessons learned from real 2211 / JDK21 upgrade work. They are written to be portable across AI tools wherever possible.

## Included Skills

| Skill | Scope | Primary Output |
|---|---|---|
| `sap-commerce-project-analyser` | Read-only SAP Commerce project discovery across backend and optional storefront roots. | Architecture/project baseline, integration map, critical-flow analysis, risk register, JSON handoff. |
| `sap-commerce-upgrade-analyser` | Read-only backend upgrade impact analysis from current to target SAP Commerce version. | Upgrade source ledger, applicability matrix, complexity classification, JSON handoff. |
| `sap-commerce-upgrade-implementer` | Approved backend upgrade implementation planning and patching. | Decision pack, approved code/config/data changes, verification summary, residual ledger. |
| `spartacus-upgrade-analyser` | Read-only Composable Storefront / Spartacus upgrade analysis. | Frontend upgrade matrix covering Angular, Spartacus/CX, auth, OCC, SSR, CMS, build and runtime risks. |
| `spartacus-upgrade-implementer` | Approved frontend upgrade implementation planning and patching. | Frontend decision pack, approved package/code/config changes, build/runtime verification notes. |

## What These Skills Are Good At

- Building an evidence-backed SAP Commerce project map before making upgrade decisions.
- Combining repository evidence with public SAP Help, SAP API/JApiCmp compatibility evidence, and user-provided authorized SAP PDFs/KBAs.
- Separating analyser work from implementer work so code changes happen only after explicit human approval.
- Treating SAP upgrades as more than version bumps: OpenRewrite, manifest changes, OAuth/security, Solr, Integration Extension Pack, impex, CCv2 deployment, system update, reindexing, smoke testing, and rollback all matter.
- Capturing runtime feedback and turning it into future scan rules, so the agents improve when new build/startup/browser issues are found.
- Coordinating backend and storefront upgrade work, especially around OAuth authorization code with PKCE, custom login pages, OCC contracts, SSR, and bearer JWT behavior.

## Repository Layout

The skill folders currently live under `.agents/skills/`:

```text
.agents/
  skills/
    sap-commerce-project-analyser/
      SKILL.md
    sap-commerce-upgrade-analyser/
      SKILL.md
      references/
    sap-commerce-upgrade-implementer/
      SKILL.md
      references/
    spartacus-upgrade-analyser/
      SKILL.md
    spartacus-upgrade-implementer/
      SKILL.md
      references/
```

`.agents` is a repository convention for keeping agent instructions grouped in one place. The important content is inside each `SKILL.md`; AI tools that do not understand this folder layout can still use the same instructions by importing, copying, or referencing those files in their own agent/prompt configuration.

`.agents` is hidden in Finder by default because directories that start with `.` are hidden by Unix/macOS convention. It is still tracked by Git. In Finder, press `Cmd + Shift + .` to show hidden files; in terminal, use:

```bash
ls -la
```

## Recommended Workflow

Use the skills in this order for a full upgrade:

1. `sap-commerce-project-analyser`
   - Build the backend/project baseline.
   - Identify active custom extensions, integrations, OCC endpoints, auth/SSO, data setup, CCv2 posture, and upgrade hotspots.

2. `sap-commerce-upgrade-analyser`
   - Compare the baseline with the target SAP Commerce release.
   - Produce the source ledger, change matrix, and JSON handoff.

3. `sap-commerce-upgrade-implementer`
   - Convert the matrix into a decision pack.
   - Run SAP tools dry-run first where applicable.
   - Apply only explicitly approved backend changes.

4. `spartacus-upgrade-analyser`
   - Analyse the storefront against the target Composable Storefront / SAP Commerce patch.
   - Check Angular, Spartacus/CX packages, OAuth, OCC, CMS, SSR, build tooling, and runtime compatibility.

5. `spartacus-upgrade-implementer`
   - Convert the frontend matrix into a decision pack.
   - Apply only explicitly approved frontend changes.
   - Verify with build, local runtime, and backend integration smoke checks.

## Approval Model

The implementer skills are deliberately gated:

- Analysis and decision-pack work is read-only.
- Migration tools default to dry-run first.
- Code, config, impex, manifest, lockfile, package, or generated changes require explicit approval.
- Backend commands such as `ant clean all`, `ant updatesystem`, Solr indexing, and smoke tests require explicit local/dev approval.
- CCv2 deployment actions are documented as runbooks unless the user explicitly provides a safe authenticated execution context and approval.

This keeps the agents useful in enterprise repositories where many files may be dirty, generated, environment-specific, or owned by another team.

## Portability

The workflows are designed to be AI-tool agnostic:

- The core instructions live in plain Markdown.
- Each skill has a clear name, scope, trigger intent, workflow, evidence rules, and output contract.
- Tool-specific behavior is kept to a minimum and should be adapted by the consuming AI runtime.
- Human approval gates, read-only analysis, source-ledger discipline, and verification expectations apply regardless of the AI tool used.

Different tools may package these instructions differently. For example, one tool may discover `.agents/skills/*/SKILL.md` automatically, while another may require pasting the Markdown into a custom agent, system prompt, project instruction, assistant profile, or workspace rule. That should not change the underlying workflow.

## Current Upgrade Knowledge Areas

The skills currently include guardrails for common SAP Commerce 2211 JDK21 and Spring 6 upgrade issues, including:

- SAP OpenRewrite dry-run/apply separation.
- `manifest.json` and CCv2 aspect/webapp review.
- Replacement of legacy `oauth2` webapps with `authorizationserver` and `resourceserver`.
- OAuth client model changes, public PKCE clients, `authenticationMethods(code)`, and update impex strategy.
- Custom authorizationserver login pages and `LoginPageUriPlaceholderProvider` checks.
- Custom login form and `AuthenticationProvider` compatibility with Spring Security 6.
- OCC/resource-server bearer JWT principal handling.
- Spring Security XML request matcher changes.
- Spring 6 `PathPatternParser` duplicate path-variable failures.
- Removed/deprecated SAP or third-party APIs such as Commons Lang 2, SAP product configuration populators, and outboundservices send signatures.
- Solr version/config alignment and reindex planning.
- XSS/CSP review.
- SAML/SSO inventory and keystore/metadata risk tracking.
- Spartacus/Composable Storefront package, Angular, SSR, OCC, auth, custom login, and backend integration checks.

## Evidence Rules

The skills are designed to prefer evidence over guesswork:

- Repository evidence wins over generic Commerce assumptions.
- SAP Help and official compatibility guidance must be consulted for upgrade analysis.
- User-provided SAP PDFs/KBAs are accepted as authorized inputs, but should not be treated as the only source.
- SAP credentials, cookies, secrets, OAuth client secrets, certificates, private keys, and sensitive environment values must not be copied into reports or reusable skill files.
- A text-search miss is not proof that a feature is absent.
- Runtime failures should become new scan rules when they reveal a reusable upgrade blind spot.

## Installing Or Adapting Locally

Use the `SKILL.md` files in the way your AI tool expects custom agents or reusable instructions to be provided.

For tools that support a skills folder, copy or sync the desired folders from `.agents/skills/` into that tool's local skills directory.

Example:

```bash
cp -R .agents/skills/sap-commerce-upgrade-implementer ~/.codex/skills/
```

For tools that do not support skill folders, copy the relevant `SKILL.md` content into the tool's custom agent, assistant instructions, project rules, or prompt-library mechanism.

Depending on the runtime, you may need to restart the tool or refresh agent discovery after changing installed skills.

## Maintenance Notes

When a project exposes a missed upgrade issue:

1. Fix or document the project issue separately.
2. Decide whether the miss is reusable across SAP Commerce upgrades.
3. Add a concise scan rule, guardrail, or feedback loop to the relevant skill.
4. Keep customer-specific names, URLs, secrets, and private details out of the reusable skill text.
5. Commit the skill update with a documentation-style message, for example:

```text
docs: add OCC JWT principal upgrade guardrails
docs: add custom login placeholder upgrade checks
```

## GitHub Repo

Remote:

```text
https://github.com/SrinivasYellapantula/sap-commerce-agent-skills
```

Preferred local clone used by this workspace:

```text
/Users/Srinivas.Yellapantula/Personal Projects/sap-commerce-agent-skills
```
