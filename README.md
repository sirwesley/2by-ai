# 2by for AI assistants

[2by](https://2by.app) turns a YAML model of a small timber structure (a
deck, a shed, a loft) into framing drawings, span checks, footing sizes, a
cut list and a build log. This repository lets an AI assistant write that
model for you and check it with 2by itself.

## Claude Code

```
/plugin marketplace add sirwesley/2by-ai
/plugin install 2by@2by
```

The plugin adds:

- the **2by-model** skill: the complete model guide and examples, and how
  to work;
- the **2by** connector (`https://2by.app/mcp`): `validate_model`,
  `get_guide` and `get_example`, so Claude checks every model against 2by's
  own engine before handing it over.

Then describe your project: "Write a 2by model for a 10 × 12 deck, 2 feet up,
on six 6x6 posts…"

## Claude (claude.ai and the desktop app)

- **Skill:** zip the folder `plugins/2by/skills/2by-model` and upload it as
  a skill.
- **Connector:** add a custom connector with the address
  `https://2by.app/mcp`. It needs no sign-in.

## Any assistant that runs commands

```
npx @2by/check model.yaml
```

[@2by/check](https://www.npmjs.com/package/@2by/check) checks a model with
2by's own engine, from the command line.

## ChatGPT and other assistants

- **Guide:** start at [2by.app/ai](https://2by.app/ai). The guide is at
  `https://2by.app/ai/guide.md`.
- **Connector:** assistants that support MCP connectors can add
  `https://2by.app/mcp`.
- **HTTP check:** `POST https://2by.app/api/ai/validate` with
  `{"yaml": "…"}`, described at `https://2by.app/ai/openapi.json`.

## What 2by is not

2by is a planning and takeoff tool. A model that passes its checks is not an
engineered or approved design, and neither is anything an assistant tells
you about it. Loads and soil bearing come from your building department.

## Where this comes from

This repository is published from the 2by source. The skill's
`references/` (the guide and the examples) are generated from 2by's own
vocabulary and catalogues. Do not edit them here: changes are made in 2by
and published again.
