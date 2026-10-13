---
name: 2by-model
description: Write, fix or check a 2by model, the YAML file 2by (2by.app) turns into framing drawings, span checks, footing sizes, a cut list and a build log for a deck, shed, loft or similar timber structure. Use when someone wants to design or plan such a structure with 2by, has a 2by model or YAML file to fix, or pastes findings from 2by (its COPY FOR AI button).
---

# Writing a 2by model

2by reads a strict subset of YAML and ignores any key it does not know, so a
model written from general knowledge looks right and silently means
something else. Work from the guide, and check the result with 2by itself.

## 1. Read the guide first

[references/guide.md](references/guide.md) is the complete guide. It covers:

- what 2by is not, and the rules on engineering values;
- how to work, and what to ask;
- the exact YAML 2by reads;
- every key;
- lumber sections and hardware parts;
- how to read findings;
- a complete example.

It is generated from 2by's own vocabulary. Read it before writing anything,
and follow it exactly. When it and anything else disagree, the guide wins.
If the 2by tools are connected, `get_guide` returns the live copy.

## 2. Work the way the guide says

1. **Interview.** Ask one question at a time (guide §3), and accept photos of
   sketches. Confirm the footprint and the post layout in words before
   writing YAML.
2. **Start from an example.** Use the closest one in
   [references/examples/](references/examples/), or `get_example`, and change
   it rather than writing from nothing.
3. **Engineering values.** Never invent loads, soil bearing or code limits.
   Never change what the person asked for (sizes, heights, loads, the longest
   board they can buy) without asking.
4. **Name 2by's libraries.** Every model says which lumber and hardware
   libraries it is written for, with the `materials:` line the guide gives,
   so its design values and connector ratings are 2by's wherever it is
   checked. [references/libraries.json](references/libraries.json) (or
   `list_libraries`, or `2by-check libraries`) lists the sections, species
   and parts each holds, and 2by's default yard: use sections and parts
   from it, and stock lengths a yard sells.

## 3. Check it with 2by before handing it over

Use whichever of these is available, in this order:

- **The 2by tools** (this plugin connects `https://2by.app/mcp`): call
  `validate_model` with the whole file.
- **The `2by-check` command**: `npx @2by/check model.yaml --json` (it needs
  Node 18 or later).
- **The web check**: `POST https://2by.app/api/ai/validate` with
  `{"yaml": "<the whole file>"}`.

Each answers with:

- `ok`;
- the ERROR and WARN findings, each with a code, the member and, for problems
  in the file itself, the line;
- a `summary` of what the model builds;
- `materials`: the libraries it was checked against, with a note when the
  model names none.

Fix every ERROR, and fix each WARN or ask the person about it. Check again
until `ok` is true. Read the summary back to the person in plain words so
they can confirm it is what they meant.

If none of these is available, hand over the file and ask the person to open
it in 2by and paste back what **COPY FOR AI** in FINDINGS copies.

## 4. Hand it over

Give the complete file, never a fragment. Save it as `<name>.yaml` when you
can write files. The person opens it in 2by:

- signed in: **My designs → New design → Choose a .yaml file**;
- or in the demo at `https://2by.app/demo`: **IMPORT**.

Say plainly that 2by is a planning and takeoff tool, not an engineer: a model
that passes its checks is not an engineered or approved design.
