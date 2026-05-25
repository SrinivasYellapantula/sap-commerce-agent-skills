# SAP Commerce Agent Skills

Reusable agent skills for SAP Commerce project analysis and upgrade work.

## Included skills

- `sap-commerce-project-analyser` produces an evidence-based SAP Commerce architecture analysis covering the solution map, backend and storefront structure, critical flows, integrations, risks, change impact, and upgrade baseline.
- `sap-commerce-upgrade-analyser` compares a project baseline with SAP upgrade guidance, user-provided authorized docs, and tool artifacts to produce an applicability matrix and JSON handoff.
- `sap-commerce-upgrade-implementer` turns an approved upgrade analysis into a decision pack and gated implementation work, including dry-run-first migration tooling, code changes, and non-code upgrade actions.

## Layout

The skill folders live under `.agents/skills/` so agent tooling that scans repo skills can discover them when this repository is opened.

```text
.agents/skills/
  sap-commerce-project-analyser/
  sap-commerce-upgrade-analyser/
  sap-commerce-upgrade-implementer/
```

Each skill contains its workflow instructions in `SKILL.md` and may include supporting `references/`, `agents/`, scripts, or assets as the suite grows.

## Portability

The repo keeps the workflows as skill folders instead of baking them into one project. Discovery, triggering, and installation depend on the agent runtime that loads those folders.
