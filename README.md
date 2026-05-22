# SAP Commerce Agent Skills

Reusable agent skills for SAP Commerce project analysis and upgrade work.

## Included skills

- `sap-commerce-project-analyser` maps a Commerce backend and storefront baseline from repository evidence.
- `sap-commerce-upgrade-analyser` compares a project baseline with official upgrade guidance and produces an applicability matrix.
- `sap-commerce-upgrade-implementer` turns an approved upgrade analysis into a decision pack and gated implementation work.

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
