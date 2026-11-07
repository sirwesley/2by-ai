# Writing a 2by model: a guide for AI assistants

You are helping someone describe a small timber structure (a deck, a shed, a
loft, a platform) as a **2by model**. A 2by model is a YAML file. 2by reads it
and derives everything else from it: framing drawings, span checks, footing
sizes, a cut list, a hardware list and a build log.

Your job is to turn what the person tells you into a valid model file and hand
it back to them complete. This guide is everything you need. Follow it
exactly. 2by's reader is stricter than general YAML, and it ignores any key it
does not know.

Model format: **1**. Guide generated from 2by's own vocabulary
(`src/model/schema.js`). If the person's 2by reports a newer format, ask them
for the newer guide.

## 1. What 2by is, and what it is not

- 2by is a **takeoff and planning tool**. It is not an engineer, and a model
  that passes its checks is not an approved design. Say so to the person when
  you hand over the file. Never describe 2by's findings as approval, code
  compliance or a stamp.
- **Never invent an engineering value.** Loads (`live_psf`, `dead_psf`,
  `roof_live_psf`, `roof_dead_psf`), soil bearing (`allowable_psf`) and stair
  code limits come from the person's local building code or building
  department. Ask for them. If the person does not know a value yet, write
  the value they choose with a comment saying it is a placeholder to confirm
  (`live_psf: 40   # placeholder: confirm with the building department`). Never
  present a placeholder as fact.
- **Never change what the person asked for without asking them first**: the
  size, the heights, the loads, the longest board they can buy. When 2by's
  findings need one of those changed, explain why and ask; change the
  framing (posts, beams, member sizes) to fit what they asked for instead
  where you can.
- Never write `verified:` anywhere. That marks a value as checked against a
  printed source, and only a person who has read the page can say that.
- Lumber species, grades and design values live in 2by's lumber file, not in
  the model. Connectors live in its hardware catalogue. The model only names
  them (see sections 6 and 7).

## 2. How to work

1. **Interview first** (section 3). Ask about one thing at a time. Accept a
   photo of a sketch or a plan, and read the dimensions off it, but confirm
   anything you are unsure of before writing.
2. **Confirm the footprint and the post grid** in words before writing any
   YAML: "Posts at the four corners of an 8' × 8' square, beams running
   east-west on the north and south lines, joists spanning north-south." Most
   wrong models are wrong here.
3. **Start from the example closest to the job**, and change it rather than
   writing from nothing:
   - `small-deck.yaml` (below): the minimal complete model;
   - `garden-shed.yaml`: a small building on blocks, with walls and a gable
     roof;
   - `tiered-deck.yaml`: a deck on several levels;
   - `shop-loft.yaml`: post-and-beam with a loft, a stair, notched posts and
     an engineered beam;
   - `screened-porch.yaml`: a raised floor on posts and beams, with roof
     posts standing on the floor posts, a gable roof on a ridge beam, screen
     walls with guards, a door and a stair.

   They are at `https://2by.app/ai/examples/<name>`.
4. **Write the whole file**, following sections 4 to 7. Use the comments to
   say where each number came from.
5. **Hand it back complete**, as one YAML block the person can save as
   `<name>.yaml`. Never hand back fragments or diffs. Connected to the
   person's 2by account (`https://2by.app/mcp/account`), save it there with
   `save_design` once they have seen the preview and said to, and give them
   the link instead. To change a design already there, read it with
   `get_design`, change only what was asked, and save the whole file with
   `update_design` and the version `get_design` gave; if 2by says it changed
   since, read it again rather than save over it.
6. **Check it yourself, if you can.** 2by's own check is open to you:
   - as tools, from the 2by MCP server at `https://2by.app/mcp`
     (`validate_model`, `preview_model`, `get_guide`, `get_example`,
     `list_libraries`);
   - over HTTP: `POST https://2by.app/api/ai/validate` with
     `{"yaml": "<the whole file>"}` (described at `/ai/openapi.json`);
   - from a command line: `npx @2by/check model.yaml --json`.

   Both answer with `ok`, the ERROR and WARN findings (code, member, line
   for problems in the file itself, message) and a `summary` of what the
   model builds. Read the summary back to the person to confirm it is what
   they meant. With the MCP server, `preview_model` also returns a picture
   of the plan and the 3D view as 2by draws them: show it, so the person
   can see what they are getting before they open 2by. Fix and check again
   until `ok` is true, then hand over the file.
7. **Otherwise, ask for the findings.** The person opens the file in 2by:
   - signed in: **My designs → New design → Choose a .yaml file**;
   - or in the demo at `https://2by.app/demo`: **IMPORT**.

   2by lists its findings in **FINDINGS**. **COPY FOR AI** there copies them
   as text for the person to paste back to you. Fix every ERROR and WARN (see
   section 8), and hand back the whole corrected file again. Repeat until
   there are none, or until the only ones left are things the person has
   decided to accept.

## 3. The interview

Ask about each of these. Skip what does not apply.

- **What it is and where.** Deck, shed, platform or loft; freestanding or
  against a building; the town or county, so the person knows whose code
  applies.
- **Footprint.** Overall length and width; the shape; any levels or tiers.
- **Foundation.** Concrete piers, precast deck blocks, or an existing slab.
  Pad sizes if they are known. A post on a precast pier block takes
  `pier_block` (the block's base width, and its height if they know it), not
  `pad_in`, which is a poured pad.
- **Posts.** Where they are, the size (4x4, 6x6…), and how tall the deck or
  floor is above grade.
- **Beams.** Which way they run, which posts carry them, and whether they sit
  in notches cut in the posts, on post caps, or hang in hangers.
- **Joists.** Which way they span, their size and spacing (12", 16" or 24"),
  whether they sit on top of the beams (drop) or hang between them (flush),
  and any overhang (cantilever).
- **Decking or subfloor.**
- **Walls.** Where they run, the stud size and spacing, wall height, doors
  and windows.
- **Roof.** Gable or single slope (mono), pitch (rise in 12), overhangs, and
  the rafter size and spacing. What it bears on: stud walls, or beams on
  posts (a porch roof: `bears_on` names the beams, and the posts run up to
  them through any deck beam with a `cap` connection). Whether the ridge is
  a board or a beam (`ridge_beam`), and if a beam, what holds it up: posts,
  or king posts standing on king-post frames (`frames`, and the king post's
  `stands_on`). A post that does not reach the ground stands on the member
  under it (`stands_on`).
- **Stairs.** Where they land, and the width.
- **What is beside it.** The house wall it stands against, a patio, a pool,
  a fence: where, and how high. They go under `extras:` and are only drawn.
- **Loads and soil.** These come from the building department. Ask; never
  guess (section 1).
- **Buying.** The longest board the person can get delivered (`supply:
  max_stock_ft`), and whether anything is bought from a different supplier.
- **Build order.** Whether it is built in phases, for example floor first and
  walls later.

## 4. The file format

### The YAML 2by reads

2by reads a strict subset of YAML. Stay inside it:

- **Indent with spaces only**, two per level. A tab is an error.
- **Lists** are written with `- ` items. **Blocks** are written as `key:` with
  their keys indented beneath it.
- **Inline lists `[a, b]` and inline blocks `{ a: 1, b: 2 }` hold plain
  values only.** Never nest a list or a block inside them, and never put a
  comma inside a value there. Write anything nested in block form.
- **Lengths are written unquoted**, either as feet-and-inches the way a tape
  reads (`8'-6"`, `8' 6"`, `6"`, `6-1/4"`, `1/2"`) or as decimal feet
  (`8.5`). A quoted length is read as text, not a number.
- **Numbers** are plain decimals with a leading digit: `0.5`, not `.5`; no
  `1e3`; no units after them (`16`, not `16 in`). The key says the unit:
  `_in` keys are inches, `_ft` keys are feet, `_psf` keys are pounds per
  square foot, `_lb` keys are pounds.
- **Flags** are `true` or `false`, never `yes` or `no`.
- **Quote text** only when it contains `: ` or starts with `#`.
- **Comments** start with `#` after a space, and may follow a value. Use them
  to say where each number came from.
- **Long text** may use `>` (folded) or `|` (literal) blocks.
- **Not supported:** anchors and aliases (`&`, `*`), several documents in one
  file (`---`), tags, and a key written twice in one block (an error).

### Where things are

- **Plan coordinates** are `[x, y]` in feet. **x runs east and y runs south**,
  measured from the north-west corner of the work, or from the inside faces of
  the west and north base rails when there is a `site:`. Points name centres:
  a post's `at` is its centre.
- **Heights** are in feet above the top of the slab or grade: `deck_top_ft`,
  a beam's `top_ft` or `bearing_ft` (its bottom), a wall's `bottom_ft` and
  `top_ft`.
- **Ranges** (`bounds_x`, `bounds_y`) are `[low, high]` in feet.
- **Ids** are short names you choose (`P1`, `B-N`, `J1`, `W-N`). Each kind of
  entry has its own list, and an id must be unique within it. Entries refer
  to each other by id: a beam lists the posts it sits on, a joist field
  lists the beams it bears on, a door names its wall.

### Doing it right

- Say how each beam meets the posts it sits on with `connections:` entries.
  `notch` lets the beam into the side of the post (6x6 and larger); `cap`
  sets it on top of the post in a post cap; `bears` sets it on another beam;
  `hung` and `girder-hung` hang it from another beam. A beam beside its
  posts rather than on them is `face` (it runs past the post, lag-screwed or
  bolted against the `face:` it names, and that face sets its line) or
  `face-hung` (its end sits in a face-mount hanger on the post's `face:`).
  The post stays in the beam's `supports`. 2by does not count or rate the
  lag screws or bolts of a `face` joint: it says so, to be sized by hand.
  This is a construction choice, not an engineering value: if the person has no
  preference, choose one, say which, and let 2by check it.
- A joist field bears on at least two beams. Its `bounds_x` and `bounds_y`
  must lie within the run of those beams.
- `framing: drop` puts the joists on top of the beams. `flush` hangs them
  between the beams in hangers, and is what you get if you leave it out.
- **Heights.** A beam with no `top_ft` or `bearing_ft` of its own tops out
  at `deck_top_ft`. That is right for flush framing. **For drop framing, give
  each beam a `top_ft` one joist depth below the deck** (2'-6" less a 2x8's
  7-1/4" is 1'-10-3/4"), or the joists end up above the deck height you were
  given. Actual sizes are in section 6.
- **`trib_ft`** is the width of floor a beam carries: half the joist span on
  each side where it carries joists. A beam with joists on one side only
  carries half of that one span.
- **Doubled beams** (`plies: 2` or more) need a `ply_fastening` block whose
  `specs` cover the pack's thickness (two 1-1/2" plies make 3"). Copy the
  block from `shop-loft.yaml`, which cites the published schedules it
  comes from, and keep the specs that match the packs in this model. Never
  write a fastening schedule from memory.
- **Spans.** Start with posts no more than about 8' apart under 2x10 and
  2x12 beams. 2by tells you when a member is overstressed and what would fix
  it, so add posts or deepen members as its findings say, rather than
  guessing a big span up front.
- **Stock lengths.** A joist is cut longer than its span: it runs past the
  beam centres to their far faces, about one beam's width longer. A 12' span
  does not come out of a 12' board. Leave room under `supply: max_stock_ft`, or 2by reports a cut with
  no stock length that holds it.
- **Walls** need `sheathing` (or 2by reports studs with nothing bracing
  them). Ask what the person will use.
- **What the structure stands beside** (the house wall, a patio, a pool, a
  fence) goes under `extras:`, one entry per thing, each with an `id` and a
  `kind`. They are drawn ghosted on their own Context layer and never
  counted, checked or ordered, so they change no quantity. They are not the
  stud `walls:` 2by frames: a house wall is `kind: wall` under `extras:`.
  A door or window in that wall is one of its `openings:`, measured along
  the wall from its `from` end, with `sill_ft` the height of its bottom
  above grade (not above the wall's bottom). Write an `outline` one corner per line (`- [x, y]`), never as a list of
  lists on one line.
- **Name the libraries** the model is written for, so it checks the same
  everywhere:

  ```yaml
  materials: { lumber: southern-pine@l8, hardware: simpson@c23 }
  ```

  These are 2by's current lumber and hardware libraries: the lumber design
  values and the connector ratings come from them, so the model checks the
  same wherever it is opened. The person's own yard decides the stock
  lengths (signed in, they set it on Materials). What each library holds,
  and 2by's default yard, is listed by `list_libraries` on the 2by tools,
  by `2by-check libraries`, and at https://2by.app/ai/libraries.json.
- **Railings.** Ask how the person builds a railing, then write that detail
  once under `railing_styles:` and name it in `project.railing`, so every
  guard and every stair's `guards:` is built to it. A style `extends` one of
  2by's presets (`dca6-fig24`: AWC DCA 6 Figure 24, rails on the posts'
  inside face, 2x2 balusters, a 2x6 cap; `inline-rods`: a 2x4 cap flat over
  the posts, 2x4 rails on edge between them, 3/4" rods in connectors) and
  changes only what differs, e.g. `rail_position: inside` for inline rails
  flush with the posts' deck-side face. A guard or stair guard may still
  name another style (`railing:`) or write any detail field itself, which
  wins. Height, `decking_in` and the `code:` limits stay on each guard: a
  deck guard and a stair guard differ (DCA 6 Figure 24 and Figure 30).

  ```yaml
  project:
    railing: MY-RAIL
  railing_styles:
    - id: MY-RAIL
      extends: inline-rods
      rail_position: inside
  ```
- **A stair hung off a joist.** When the flight hangs from the deck's
  outside joist rather than a beam or rim, write the joist field's id in
  the stair's `from:` and the side in `face:`. The flight then hangs from
  that field's outside joist on that side, so put a joist on that bound
  (`layout_origin`). 2by checks the joist with the stringer tops on it,
  across its spans between the beams it bears on, and reports an ERROR if
  the joist is too small. Do not make up a header beam that the person did
  not build.
- **Stair treads and risers.** The treads are `treads_per_step` boards of
  `tread_section` on each step, laid with the deck's `gap_in` between them.
  For closed risers, name the riser board in the stair's `riser_section`
  (the deck board, e.g. `5/4x6`). Add `riser_boards` when one board does not
  close the rise. 2by checks what the boards leave open against
  `code.max_riser_gap_in`. For open risers, write `open_risers: true`.
  ```yaml
      tread_section: 5/4x6
      treads_per_step: 2
      riser_section: 5/4x6    # one deck board on each riser
  ```
- Give every entry a `phase` from `phases:`, and declare the phases in build
  order.
- Members are sized by `section` (beams, joists, studs, rafters) and posts by
  `size`. Both are names from section 6.
- Leave out a connector `part` unless the person has chosen one. 2by then
  picks the lightest adequate part from its catalogue and says which.

## 5. Every key 2by reads

Anything not listed here is ignored and reported as an unknown key. Required
keys are marked. Where a key has a default, leaving it out gives that value.

### `project`

Job-wide settings and loads. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `slug` | text | Short machine name, e.g. garden-shed. |
| `title` | text | Title shown on every sheet. |
| `revision` | text | Revision label, e.g. r5. |
| `date` | text | Revision date, YYYY-MM-DD. |
| `format` | a number | The model format this file is written in. Leave it out or write 1. Default: 1. |
| `live_psf` | a number, psf | Floor live load, psf. From your code or building department, never guessed. |
| `dead_psf` | a number, psf | Floor dead load, psf. |
| `roof_live_psf` | a number, psf | Roof live (snow) load, psf. Falls back to live_psf, with a finding. |
| `roof_dead_psf` | a number, psf | Roof dead load, psf. Falls back to dead_psf, with a finding. |
| `allowable_psf` | a number, psf | Allowable soil bearing, psf. Default: 1500. |
| `railing` | text | The railing style every guard and every stair's guards is built to unless it names its own (railing:): an id from railing_styles:, or a 2by preset (dca6-fig24, inline-rods). |
| `waste_pct` | a number, percent | Waste allowance on area materials, percent. Default: 10. |
| `bearing_ft` | a number, feet | Height of beam bottoms for beams with no height of their own, when there is no deck_top_ft, feet. Default: 8.5. |
| `deck_top_ft` | a number, feet | Top of the floor or deck framing, feet. A beam with no top_ft or bearing_ft tops out here, which is right for flush framing; for drop framing give the beams a top_ft one joist depth lower. |
| `uplift_lb` | a number, lb | Design uplift per connection, lb. |
| `guard_above_in` | a number, inches | A deck edge higher than this above grade, inches, needs a guard (AWC DCA 6 p.19: 30). Given, every open edge over it without a guard, stair or ledger is an error; left out, edges are not checked. |
| `corner_style` | text, e.g. `3-stud`, `california`, `clips` | How wall corners are framed. |
| `projection` | one of `third-angle`, `first-angle`, `first` | Arrangement of the orthographic views. |

### `materials`

Which of 2by's libraries the model is written for, so it checks the same everywhere. Leave it out and it is checked against the lumber and hardware files it is given: in 2by, its current libraries. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `lumber` | text | The lumber library the model is written for, as id@revision (southern-pine@l8); leave out the revision for 2by's current one. Its species, design values and sections are used, with the stock lengths from the person's yard. |
| `hardware` | text | The hardware library, as id@revision (simpson@c23). Its connectors are used. |

### `decking`

Deck boards over every joist field, in place of sheathing. The rows, the pieces, their joints, the fasteners and the drawing are derived from it. Joist fields that meet end to end along the boards at the same deck top are decked as one surface: each row runs on across the meeting line in stock lengths, its joints on joists and staggered between rows, and breaks only where a field stops short across the boards. Leave it out and the frame is sheathed (a floor). A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `board` | a section name (§6) | The deck board, a section in the lumber library (5/4x6 radius-edge, 2x6). Its thickness and width are the library's. |
| `gap_in` | a number, inches | Gap between boards, inches (AWC DCA 6 p.3: about 1/8). |
| `end_gap_in` | a number, inches | Gap where two boards butt over a joist, inches. Default: 0. |
| `trim_in` | a number, inches | Cut off each end of a stock board before it is used, inches (Trex p.19: 3/16). Leave out for none. Default: 0. |
| `per_joist` | a number | Fasteners at each joist a board crosses (IRC R507.7, DCA 6 p.3: 2). |
| `fastener` | text | The fastener, as it is bought, e.g. #8 x 2-1/2" deck screw. |
| `joint_fastener` | text | What goes at each butt joint, one per joint, as it is bought (Trex p.27: a Hideaway butt joint clip). Leave out when the per-joist fasteners do. |
| `code` | a block (keys below) | The limits to check against, from a printed source. |
| `code.max_spacing_in` | a number, inches | Widest joist spacing the board allows laid perpendicular, inches (IRC Table R507.7, multiple span: 16 for 5/4, 24 for 2x). |
| `code.min_supports` | a number | Fewest joists a board may bear on (DCA 6 p.3: 3). |
| `code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |

### `braces`

Diagonal knee braces at each beam's end posts, parallel to the beam (AWC DCA 6 p.10, Figure 10); none on the posts between, which the figure prohibits. Which posts get one, and each brace's length, are derived. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `section` | a section name (§6) | The brace (AWC DCA 6, Figure 10: 2x4). |
| `leg_in` | a number, inches | How far down the post and out along the beam the brace meets them, inches (Figure 10: 24). Shortened to fit on a short post, still at 45 degrees. Default: 24. |
| `above_in` | a number, inches | A corner post taller than this, grade to the beam's underside, is braced, inches (DCA 6 p.10: 24). Default: 24. |
| `lag` | text | The fastener at each end, as it is bought (DCA 6 p.10: a 1/2" lag screw with washers). |

### `slab`

An existing concrete slab the posts stand on. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `field_in` | a number, inches | Slab thickness in the field, inches. |
| `perimeter_in` | a number, inches | Thickened-edge depth, inches. |
| `perimeter_width_in` | a number, inches | Thickened-edge width, inches. |
| `perimeter_confidence` | text, e.g. `measured`, `assumed` | How the edge figures are known. |

### `post_bases`

Post bases under every post. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `standoff_in` | a number, inches | Height a standoff base lifts the post off the concrete, inches. |
| `confidence` | text, e.g. `catalog`, `measured` | How the standoff is known. |
| `part_by_size` | a block of a post size, e.g. 6x6 → a part name (§7) | Base part by post size, unless a post names its own base. |

### `ply_fastening`

How multi-ply beams are fastened together. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `rows` | a number | Rows of fasteners along a multi-ply beam. |
| `spacing_in` | a number, inches | Spacing along each row, inches. |
| `end_extra` | a number | Extra fasteners per row near each end. |
| `confidence` | text, e.g. `table` | Where the schedule comes from. |
| `source` | text | The printed source of the schedule (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `specs` | a list of blocks (keys below) | Fasteners by ply-pack thickness. |
| `specs[].for_thickness_in` | a number, inches | Total ply-pack thickness this fastener suits, inches. |
| `specs[].kind` | text, e.g. `engineered`, `sawn` | What it fastens. |
| `specs[].part` | text | Fastener part name. |
| `specs[].desc` | text | Description printed on the schedule. |
| `specs[].per_box` | a number | Fasteners per box. |
| `specs[].vendor` | a vendor id | Vendor id from supply.vendors. |
| `specs[].note` | text | Why this fastener. (Recorded for people; 2by does not use it.) |

### `site`

The slab and rails the structure sits inside. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `slab_ft` | `[x, y]`, feet | Slab size outside to outside, [east-west, north-south] feet. |
| `rail_in` | a block (keys below) | Base-rail widths around the slab. |
| `rail_in.west` | a number | Rail width on the west side, inches. |
| `rail_in.east` | a number | Rail width on the east side, inches. |
| `rail_in.north` | a number | Rail width on the north side, inches. |
| `rail_in.south` | a number | Rail width on the south side, inches. |
| `origin` | text, e.g. `rail-inside` | Where x = 0, y = 0 is. 2by reads only rail-inside: the inside faces of the west and north rails. (Recorded for people; 2by does not use it.) |

### `supply`

What can be bought and delivered. A block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `max_stock_ft` | a number, feet | Longest stock length that can be delivered, feet. |
| `cap_except` | `[ … ]` of section name (§6)s | Sections ordered to length. |
| `vendors` | a list of blocks (keys below) | Where material is bought. |
| `vendors[].id` **required** | an id | Unique id. Other entries refer to it. |
| `vendors[].label` | text | Human name shown on drawings and schedules. |
| `vendors[].default` | `true` or `false` | Buy from this vendor unless a section names another. |
| `vendors[].sections` | `[ … ]` of section name (§6)s | Sections bought from this vendor. |
| `vendors[].takes` | `[ … ]` of texts | Kinds of material bought from this vendor. |

### `stock`

Stock lengths for this job only, overriding the lumber file. Written as a block of a section name → `[ … ]` of numbers.

### `layers`

Drawing layers. Leave out for the defaults. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `hint` | text | Tooltip on the layer chip. |
| `hidden` | `true` or `false` | Start switched off. |
| `draws` | `[ … ]` of texts | What this layer draws. |

### `phases`

Build phases, in build order. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `name` | text | Phase name. |

### `posts`

Posts. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `at` | `[x, y]`, feet | Post centre, [x, y] feet. |
| `size` | one of `4x4`, `4x6`, `6x6`, `8x8`, `2x4`, `2x6`, `2x8` | Post size. |
| `facing` | one of `x`, `y` | Axis the wide face runs along, on a rectangular post. On a post turned square to a skew beam, x is along the beam. |
| `perimeter` | `true` or `false` | On the perimeter (sizes its footing). |
| `base` | a part name (§7) | Base part for this post, overriding post_bases. |
| `pad_in` | a number, inches | Poured footing pad width, square, inches. Leave out and 2by reports, from each post's load, the footing it needs. |
| `pad_depth_in` | a number, inches | Poured footing pad depth, inches. Default: 12. |
| `pier_block` | a block (keys below) | A precast concrete pier block the post stands on, bought, not poured (in place of pad_in). 2by checks the soil under its base at the project's allowable, orders the blocks and no concrete, and rates neither the block nor how deep it is set: it says so in a WARN. |
| `pier_block.base_in` | a number, inches | Width of the block's base, square, inches, as bought: the area its load bears on the soil through. |
| `pier_block.height_in` | a number, inches | Height of the block, inches, as bought. 2by draws the block from it, under the post; leave it out and the block is listed and checked but not drawn. |
| `standoff_in` | a number, inches | This post's base standoff, inches. |
| `braced` | a number or text (see meaning) | Weak-axis bracing: continuous, or a number of brace points. |
| `top_ft` | a number, feet | Top of the post, feet, when it runs on past the highest member it carries: a screen-porch post standing up to the roof framing between screen walls. Leave out and the post is cut at its highest member. |
| `stands_on` | a posts_beams_or_frames id | What the post stands on when it does not reach the ground: a beam (a post on the deck over its ledger), a roof frame (a king post on its tie), or another post (a roof post on the deck boards over the deck post below it, the two joined with a tie plate: give splice). It has no footing, pad or base; it is cut from that member's top (for a post, the deck boards over it) up to the beam it carries, and its load lands on that member. |
| `splice` | a block (keys below) | How a post standing on another post is tied to it against uplift. 2by lists the part on the hardware order and states its rated tension; it does not compute uplift, so give the project's uplift_lb to check it against. |
| `splice.part` | a part name (§7) | The connector joining this post to the post it stands on: a strap or tie plate from the hardware library (Simpson HTP37Z). |
| `splice.count` | a number | How many of the part at the joint. Default: 1. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `beams`

Beams, girders, rims and headers. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `axis` | one of `x`, `y`, `skew` | Direction the beam runs: x, y, or skew. A skew beam runs from its first support to its last, at the angle that line makes; every support must lie on that line, within 1/8". Its spans, length and angle are derived from the posts. |
| `side` | one of `north`, `south`, `east`, `west`, `centre` | Face of the posts it is flush with, when it is off their centres. Leave out for centred. |
| `supports` | `[ … ]` of post ids | Posts on this beam's line, in any order. |
| `at_ft` | a number, feet | Line the beam runs on, feet across its axis, when it has no supports. |
| `from_ft` | a number, feet | Start of the run along its axis, feet. |
| `to_ft` | a number, feet | End of the run along its axis, feet. |
| `span_ft` | a number, feet | Span, feet, when nothing else fixes it. |
| `plies` | a number | Number of plies. Two or more need a ply_fastening spec for the pack's thickness. Default: 1. |
| `continuous` | `true` or `false` | One board (each ply) over all its supports, with no butt joints over the posts: an LVL ridge bought as one length. Leave out and the beam is cut one board per span, jointed over each post. Each span is still checked as a simple span, which is the safe side for a continuous board; the stock needs a length as long as the whole run. |
| `section` | a section name (§6) | Lumber section of one ply. |
| `trib_ft` | a number, feet | Width of floor it carries, feet: half the joist span on each side where it carries joists. |
| `extra_plf` | a number | Extra line load on top of the tributary load, plf. |
| `extra_note` | text | What the extra load is. |
| `bearing_ft` | a number, feet | Height of the beam bottom, feet. |
| `top_ft` | a number, feet | Height of the beam top, feet. Wins over bearing_ft; either wins over project.deck_top_ft, then project.bearing_ft. |
| `sister` | a block (keys below) | A short member face-fastened along part of the beam. |
| `sister.section` | a section name (§6) | Section of the sister. |
| `sister.plies` | a number | Plies in the sister. |
| `sister.from` | a post id | Post the sister starts at. |
| `sister.to` | a post id | Post the sister ends at. |
| `sister.bears` | `true` or `false` | Bears on the post tops with the beam, rather than stopping at the post faces. |
| `sister.side` | one of `north`, `south`, `east`, `west` | Which face it is fastened to. |
| `ledger` | a block (keys below) | This beam is a ledger: fastened to the house along its length rather than carried on posts. The house carries it, so it has no bending check; its fasteners are checked against code.schedule and listed with the hardware. Give it from_ft, to_ft and at_ft, no supports. |
| `ledger.to` | one of `band-joist`, `veneer`, `hollow-masonry`, `overhang` | What it is fastened to on the house. band-joist: a 2x band or rim joist on the sill, the one a schedule covers (AWC DCA 6, Figure 14). veneer (brick, stone), hollow-masonry and overhang (a cantilevered floor or bay window) are prohibited (DCA 6 p.15): the deck must stand on its own posts. |
| `ledger.fastener` | one of `lag`, `bolt` | What fastens it, with washers: lag (1/2" lag screws) or bolt (1/2" through-bolts). |
| `ledger.flashing` | `true` or `false` | Flashed against water where it meets the house (DCA 6 Table 5, note 3). |
| `ledger.code` | a block (keys below) | The fastening schedule to check against, from a printed source. Leave it out and the fasteners are not scheduled, and 2by says so. |
| `ledger.code.live_psf` | a number, psf | The live load the schedule is written for, psf (DCA 6 Table 5: 40). |
| `ledger.code.dead_psf` | a number, psf | The dead load it is written for, psf (DCA 6 Table 5: 10). |
| `ledger.code.min_section` | a section name (§6) | The smallest ledger it allows (DCA 6 Table 5, note 5: 2x8). |
| `ledger.code.schedule` | a list of blocks (keys below) | The fastener spacing by the span of the joists hung on it, shortest span first. Past the last row the ledger has to be engineered. |
| `ledger.code.schedule[].up_to_span_ft` | a number, feet | The longest joist span this row covers, feet. |
| `ledger.code.schedule[].spacing_in` | a number, inches | Fasteners on centre, inches, staggered in two rows. |
| `ledger.code.source` | text | The printed source of the schedule (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `rim` | a block (keys below) | This beam is a rim joist nailed across the joists' ends (AWC DCA 6 p.11, Figure 11): the joists carry it. Its fasteners are counted from the joist ends that meet it and listed with the hardware. A rim that runs between two members, butting them rather than passing through them, says ends: hung. |
| `rim.per_joist` | a number | Fasteners into each joist end (AWC DCA 6, Figure 11: 3). |
| `rim.fastener` | text | The fastener, as it is bought (DCA 6 Figure 11: 10d threaded nails or #10 x 3" wood screws). |
| `rim.ends` | one of `hung` | hung: each end of the rim sits in a face-mount joist hanger on the joist or beam it butts. 2by finds that member at each end, picks the hanger from the library as it does for hung joists, and orders one per end. |
| `rim.hanger` | a part name (§7) | The hanger at the rim's hung ends. Leave out and 2by picks the lightest adequate one from the library. |
| `waive` | `[ … ]` of one of `low-util`, `deflection`, `unused`s | Checks that are deliberate here. State why in a comment. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `joist_fields`

Floor and deck joists, one entry per bay. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `section` | a section name (§6) | Joist section. |
| `span_axis` | one of `x`, `y` | Direction the joists span. |
| `bounds_x` | `[low, high]`, feet | East-west extent, [low, high] feet. |
| `bounds_y` | `[low, high]`, feet | North-south extent, [low, high] feet. |
| `bears_on` | `[ … ]` of beam ids | The beams the joists bear on (at least two). When one is a skew beam, each joist ends where it meets that beam, so every joist has its own length; the bounds are the box round the field, and its outline and area are the bounds cut by the beam's line. Deck boards are laid to that angled edge, the rows that cross it ripped to a taper; subfloor panels are ordered by the outline's area. |
| `spacing_in` | a number, inches | Joist spacing, inches. Default: 16. |
| `framing` | one of `flush`, `drop` | flush: joists hung between beams. drop: joists on top of them. |
| `hung_at` | `[ … ]` of beam ids | Beams the joists hang from in hangers. |
| `cantilever_ft` | a block of a beam id → a number | Cantilevers, by beam id. |
| `end_at_ft` | a block of a beam id → a number | Joist ends, by beam id. |
| `hanger` | a part name (§7) | Hanger part. On a field ending on a skew beam, the part at its square end. |
| `skew_hanger` | a part name (§7) | The skewed hanger where the joists meet a skew beam, one per joist. Its page has to list the joists' skew and hand (2by derives both). Leave out and 2by picks one, or says there is none. |
| `skew_hanger_unrated` | `true` or `false` | skew_hanger names a field-skewable hanger whose rating is not in 2by's library yet. It is counted and listed, and 2by warns that its skew range, rating and nails are unchecked until its page is added. |
| `tie` | a part name (§7) | Hurricane tie part, where joists sit on a beam. |
| `blocking` | `true` or `false` | Blocking between joists. |
| `joists_from` | one of `low`, `high` | Which end the joist layout starts from. |
| `layout_origin` | a number | Where the joist rhythm is measured from, feet. |
| `skip_first_joist` | `true` or `false` | No joist on the first line. |
| `shared_first_joist` | `true` or `false` | The first joist is shared with the neighbouring field. |
| `span_ft` | a number, feet | Joist span, feet, overriding the bearing beams. |
| `width_ft` | a number, feet | Field width, feet, overriding the bounds. |
| `waive` | `[ … ]` of one of `low-util`, `deflection`, `unused`s | Checks that are deliberate here. State why in a comment. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `connections`

How beams meet posts. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `at` | a post id | The post. |
| `member` | a beam id | The beam meeting it. |
| `type` | one of `notch`, `cap`, `hung`, `girder-hung`, `bears`, `face`, `face-hung` | How the beam meets the post. face: the beam runs past the post against its face (face:), lag-screwed or bolted to it; its line is that face. face-hung: the beam's end is in a face-mount hanger on the post's face (face:); it bears there, not at the post's centre. Either way the post is in the beam's supports, so its load reaches the post's footing. |
| `part` | a part name (§7) | Connector part. Leave out and 2by picks one. |
| `into` | a beam id | The beam a hung member hangs from. |
| `on` | a beam id | The beam it bears on. |
| `face` | one of `north`, `south`, `east`, `west` | The face of the post it sits on: for face, the face the beam lies against (east or west of a beam along y, north or south of one along x); for face-hung, the face its end is hung on (square to the beam). |
| `fastener` | one of `lag`, `bolt` | For face: what fastens the beam to the post's face. 2by does not count or rate them: size them by hand. Default: lag. |
| `align` | one of `centre`, `north`, `south`, `east`, `west` | The post face a seated beam is flush with. |
| `seat_in` | a number, inches | Seat depth, inches. |
| `notch_depth_in` | a number, inches | Notch depth, inches. |
| `label` | text | Human name shown on drawings and schedules. |

### `walls`

Stud walls. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `from` | `[x, y]`, feet | Start of the wall line, [x, y] feet. |
| `to` | `[x, y]`, feet | End of the wall line, [x, y] feet. |
| `align` | one of `centre`, `outside` | Whether the line is the wall's centre or its outside face. |
| `section` | a section name (§6) | Stud section. |
| `bottom_ft` | a number, feet | Bottom of the bottom plate, feet. |
| `top_ft` | a number, feet | Top of the top plate, feet. |
| `spacing_in` | a number, inches | Stud spacing, inches. Default: 16. |
| `end_studs` | `true` or `false` | A stud at each end of the wall. false for a wall that runs between two posts, face to face (a screen wall between porch posts): the posts are its ends, and its studs are centred a spacing apart from the first post's face, none within half a spacing of the other. Default: true. |
| `top_plates` | a number | Number of top plates. Default: 2. |
| `bottom_plates` | a number | Number of bottom plates. Default: 1. |
| `bearing` | `true` or `false` | Carries the roof. |
| `gable` | a roof id | The roof whose gable this wall closes. |
| `sheathing` | text | Sheathing product, e.g. 7/16" OSB. Without it a stud wall has nothing bracing it, and 2by says so. |
| `housewrap` | `true` or `false` | Housewrapped. |
| `cladding` | text | Cladding, e.g. board and batten. |
| `hold_downs` | text | Hold-down part or note. |
| `stud_ties` | `true` or `false` | Studs tied to plates. |
| `guard` | a block (keys below) | A guard framed into the wall: a rail between each pair of studs with balusters under it, as on a screen porch. The wall is then the guard on the deck edge it stands on; no openings are guarded. 2by does not rate the rail's fixing to the studs. |
| `guard.height_in` | a number, inches | Top of the rail above the floor, inches: the guard's height. |
| `guard.decking_in` | a number, inches | The deck boards' thickness, inches, when the wall's bottom_ft is the joists' top rather than the boards' top. Default: 0. |
| `guard.rail_section` | a section name (§6) | Rail section between each pair of studs: face style, laid flat, a top rail at height_in and a bottom rail on the bottom plate; inline style, on edge, a top rail under the cap and a bottom rail on the bottom plate. |
| `guard.cap_section` | a section name (§6) | inline style: the cap laid flat between the studs, its top at height_in, the top rail on edge under it. |
| `guard.baluster_section` | a section name (§6) | Baluster section, between the rails. Leave out for rods. |
| `guard.rod` | a block (keys below) | Round metal balusters (rods) in place of baluster_section: plumb, rail to rail, spaced under opening_in by their diameter. Counted and listed as bought items, not cut from lumber. Not Figure 24's 2x2 baluster screwed top and bottom: 2by says so in a WARN and does not rate them. |
| `guard.rod.diameter_in` | a number, inches | The rod's diameter, inches (e.g. 0.75). |
| `guard.rod.connectors` | `true` or `false` | Each end sits in a connector screwed to the rail: two a rod, counted. Leave out or false for rods set in drilled holes. Default: false. |
| `guard.style` | one of `face`, `inline` | How the rails and infill sit. face: the rails on edge on the studs' inside face, the balusters on the rails' inside face (AWC DCA 6 Figure 24). inline: the rails on edge between the studs, centred on their line, the balusters or rods centred in the rails' edges, the cap flat over them; nothing stands proud of the rails. inline is not Figure 24's detail: 2by says so in a WARN. Default: face. |
| `guard.opening_in` | a number, inches | Largest gap the balusters leave, inches; 2by spaces them under it. |
| `guard.code` | a block (keys below) | The limits to check against, from a printed source. Leave one out and it is not checked. |
| `guard.code.min_height_in` | a number, inches | Lowest guard, inches (AWC DCA 6 Figure 24: 36). |
| `guard.code.max_opening_in` | a number, inches | The sphere nothing may pass, inches (Figure 24: 4). |
| `guard.code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `openings`

Doors, windows and floor openings. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `in_wall` | a wall id | The wall it is in. |
| `kind` | text, e.g. `door`, `window` | Door or window. |
| `at_ft` | a number, feet | Distance along the wall to the opening, feet. |
| `width_in` | a number, inches | Rough opening width, inches. |
| `height_in` | a number, inches | Rough opening height, inches. |
| `sill_in` | a number, inches | Sill height, inches (windows). |
| `clear_width_in` | a number, inches | Clear width, inches. |
| `min_width_in` | a number, inches | Minimum clear width required, inches. |

### `roof`

Roofs. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `type` | one of `gable`, `mono` | Roof type. |
| `ridge_axis` | one of `x`, `y` | Direction the ridge runs. |
| `bounds_x` | `[low, high]`, feet | East-west extent of the walls it covers, [low, high] feet. |
| `bounds_y` | `[low, high]`, feet | North-south extent, [low, high] feet. |
| `pitch` | a number or text (see meaning) | Rise in 12: 7, 4/12, 4:12 or 4 in 12. |
| `section` | a section name (§6) | Rafter section. |
| `spacing_in` | a number, inches | Rafter spacing, inches. |
| `overhang_in` | a number, inches | Eave overhang, inches. Default: 0. |
| `rake_overhang_in` | a number, inches | Rake overhang, inches. Default: 0. |
| `barge_section` | a section name (§6) | The barge rafters' section, at the rake: their top flush with the rafters' tops. Leave out for the rafter section. |
| `outriggers` | a block (keys below) | Outriggers (lookouts) carrying the barge rafter at the rake: flat in notches in the top edge of the gable-end rafter, from the barge rafter back to the first common rafter. Drawn, cut and the notches listed; not rated. |
| `outriggers.section` | a section name (§6) | Outrigger section, laid flat (e.g. 2x4). |
| `outriggers.spacing_in` | a number, inches | Spacing along the rafter, inches, from the plate line toward the ridge. |
| `outriggers.ends` | `[ … ]` of texts | The gable ends that have them; leave out for both. |
| `tie_in` | a block (keys below) | Ties this roof into a host roof (the house's) past one gable end: the ridge beam runs on to meet it, its end cut to the host's slope to land flat on it (set the beam's from_ft or to_ft at the cut's long point, where its top meets the host), and fill rafters continue the layout over it, each from the ridge down to where it lands on the host roof's slope. That end has no rake, barge rafter or outriggers. The host roof is not checked: it carries the fill rafters. |
| `tie_in.end` | text | The gable end that runs on over the host roof: north, south, east or west. |
| `tie_in.pitch` | a number | The host roof's pitch, inches of rise in 12. |
| `tie_in.eave_ft` | a number, feet | The host roof's eave height, feet above grade. |
| `tie_in.eave_at_ft` | a number, feet | Where the host roof's eave line is, feet along this roof's ridge axis (y for a ridge on y). The host roof rises from there, away from this roof. |
| `tie_in.valley_plate` | a block (keys below) | Valley plates (valley boards): a board laid flat on the host roof along each valley, where this roof meets it, for the fill rafters' feet to land on. Drawn and cut; not rated. |
| `tie_in.valley_plate.section` | a section name (§6) | The plate's section, laid flat (e.g. 2x8). |
| `ridge_section` | a section name (§6) | Ridge board section. |
| `support_section` | a section name (§6) | Section of the ridge supports. |
| `bears_on` | `[ … ]` of wall or beam ids | Walls or beams the rafters bear on. A beam carrier is a plate beam on posts (or a ledger): the rafters seat on its top, and their load lands on it as a line load. |
| `ridge_beam` | a beam id | A ridge BEAM, not a board: a beam running along the ridge that carries the top half of every rafter, so the roof pushes nothing outward. It is an ordinary beams: entry on posts (king posts standing on frames, or posts to the ground), sized and checked like one. |
| `frames` | a list of blocks (keys below) | King-post frames under a ridge beam. A king post (a posts: entry) stands_on each; 2by derives the frame's load from it and the force in the frame's rafters and tie by statics. |
| `frames[].id` **required** | an id | Unique id. Other entries refer to it. |
| `frames[].at_ft` | a number, feet | Where the frame stands, feet along the ridge axis (its y for a ridge on y). |
| `frames[].tie_section` | a section name (§6) | The tie across the frame at plate height: one ply. |
| `frames[].tie_plies` | a number | Tie plies, one on each side of the rafter pair. Default: 2. |
| `frames[].top_ft` | a number, feet | Height of the tie's top, feet. Leave out for a tie sitting on the plate beams; give it when the tie is framed in level with them, between their faces. |
| `frames[].spacer_section` | a section name (§6) | A spacer sandwiched between the tie plies, the king post standing on it. Leave out for none. |
| `frames[].bolts` | a number | Through-bolts at each heel, as built, on the rafter's centre line through every tie ply and the rafter. With bolt_dia_in they are checked (NDS 12.3); leave both out and 2by designs the heel: how many, what size and where. |
| `frames[].bolt_dia_in` | a number, inches | Diameter of the heel bolts, as built, inches (0.5 for 1/2"). |
| `frames[].brace` | a block (keys below) | Diagonal braces from the king post up to the frame rafters, one on each side, cut plumb at both ends. Drawn, cut and listed; not counted in the frame-rafter check, because their lower ends land on the king post, which stands on the middle of the tie, and 2by has no printed source for what such a brace restrains. |
| `frames[].brace.section` | a section name (§6) | One ply of the brace. |
| `frames[].brace.plies` | a number | Brace plies, one on each face of the king post and the frame rafter. Default: 2. |
| `frames[].brace.spacer_section` | a section name (§6) | A spacer between the brace plies, from the king post's face to the rafter's underside. Leave out for none. |
| `frames[].brace.at_king_in` | a number, inches | Where a brace crosses the king post: the height of the brace's middle over the tie's top, inches. |
| `frames[].brace.out_in` | a number, inches | Where a brace ends on the rafter: level distance from the king post's centre out to the brace's end, inches. Its middle lines up with the rafter's middle there. |
| `tie` | a part name (§7) | Rafter tie part. |
| `collar_ties` | `true` or `false` | Collar ties. |
| `tied` | `true` or `false` | The roof is held by its frames' ties, not by its ridge: every rafter's load goes to the plates, the ridge member (ridge_beam, if any) carries none of it and its king posts only its own weight, and each frame's tie holds the rafters' outward push for its share of the roof's length. Needs frames. |
| `ceiling_joists` | `true` or `false` | Ceiling joists tie the walls. |
| `notch_limit` | a number | Largest birdsmouth as a fraction of rafter depth. Default: 0.333. |
| `blocking` | `true` or `false` | Blocking between rafters. |
| `sheathing` | text | Roof sheathing, with its thickness: 7/16" OSB. Drawn on the rafters and ordered sheet by sheet, and weighed in the roof's dead load (ASCE 7 commentary Table C3-1). |
| `roofing` | text | Roof covering, e.g. asphalt shingles. Ordered by the sloped area and weighed in the roof's dead load (ASCE 7 commentary Table C3-1). |
| `ceiling` | text | Boards or panels on the rafters' underside, with their thickness: 3/4" T&G pine. Drawn under the rafters from the plates to the ridge, ordered by area, and weighed in the roof's dead load (ASCE 7 commentary Table C3-1). |
| `waive` | `[ … ]` of one of `low-util`, `deflection`, `unused`s | Checks that are deliberate here. State why in a comment. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `stairs`

Stairs. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `from` | a beams_or_fields id | The beam the flight hangs from, or a joist field: the flight then hangs from that field's outside joist on the side `face` names, one full-length board on the field's bound (put a joist there: layout_origin). 2by checks that joist with the flight's stringer tops on it, over its spans between the carriers it bears on. |
| `face` | one of `north`, `south`, `east`, `west` | The side of that beam the flight runs down. Off a skew beam (axis: skew), the compass point nearest the side it goes down; the flight runs square to the beam. Default: south. |
| `at_ft` | a number, feet | Low side of the flight along the beam, feet. Off a skew beam, feet along it from its first support. |
| `lands_at_ft` | a number, feet | Height the flight lands at, feet: 0 for grade, or the top of the landing's joists. Above grade, a joist field at that height has to be under the flight's foot, as wide as the flight (AWC DCA 6 p.20). A flight off a skew beam lands on grade. Default: 0. |
| `risers` | a number | Number of risers. |
| `run_in` | a number, inches | Run per tread, inches. Default: 10. |
| `width_in` | a number, inches | Width outside the outer stringers, inches. Default: 36. |
| `stringers` | a number | Number of stringers. Default: 3. |
| `stringer_section` | a section name (§6) | Stringer section. |
| `tread_section` | a section name (§6) | Tread board section. |
| `treads_per_step` | a number | Boards per tread. Default: 2. |
| `open_risers` | `true` or `false` | No riser boards: the stringers' riser cuts are left open (checked against code.max_riser_gap_in). Overrides riser_section. |
| `riser_section` | a section name (§6) | A board on each riser (closed risers): e.g. the deck board, 5/4x6. The top one goes on the head member's face; each tread butts the riser board behind it. |
| `riser_boards` | a number | Boards on each riser, stacked from the tread above down. Default: 1. |
| `top` | text, e.g. `hung` | How the top is fixed. 2by reads only hung. (Recorded for people; 2by does not use it.) |
| `top_part` | text | Stringer connector at the top, e.g. LSCZ. Listed as written; not looked up in the catalogue. |
| `bottom` | text, e.g. `kicker` | How the foot is restrained. 2by reads only kicker. (Recorded for people; 2by does not use it.) |
| `kicker_section` | a section name (§6) | Kicker section. |
| `kicker_angle` | text | Angle fastening the kicker, e.g. A35. Listed as written; not looked up in the catalogue. |
| `kicker_anchor` | text | Anchor fixing the kicker to the slab. |
| `kicker_anchors` | a number | Number of kicker anchors. Default: 4. |
| `tread_screw` | text | Tread screw. |
| `code` | a block (keys below) | The code limits to check against, from your jurisdiction. |
| `code.max_rise_in` | a number, inches | Largest riser, inches. |
| `code.min_run_in` | a number, inches | Smallest run, inches. |
| `code.min_width_in` | a number, inches | Narrowest width, inches. |
| `code.min_headroom_in` | a number, inches | Least headroom, inches. |
| `code.max_riser_gap_in` | a number, inches | Largest open-riser gap, inches. |
| `code.max_stringer_span_in` | a number, inches | Longest horizontal span of a cut stringer, inches (AWC DCA 6, Figure 28: 72, i.e. 6'-0"). Past it, a post at mid-span or a landing. Leave out and it is not checked. |
| `code.guard_rise_in` | a number, inches | A flight rising this much or more needs a guard on each open side, inches (AWC DCA 6, Figure 30: 30). Leave out and it is not checked. |
| `code.handrail_risers` | a number | A flight of this many risers or more needs a handrail (AWC DCA 6 p.22: 4). Leave out and it is not checked. |
| `code.max_flight_rise_in` | a number, inches | Most one flight may rise before an intermediate landing is required, inches (AWC DCA 6 p.20: 144, i.e. 12'-0"). Leave out and it is not checked. |
| `code.min_landing_in` | a number, inches | Least depth of the landing a flight stops on, clear past its bottom riser in the direction of travel, inches (AWC DCA 6 p.20, R311.7: 36). Leave out and it is not checked. |
| `mid_posts` | a block (keys below) | 4x4 posts under the stringers at mid-span, to shorten a cut stringer's span instead of a landing (AWC DCA 6 p.20). A landing is required over 12'-0" of rise. |
| `mid_posts.count` | a number | How many lines of posts, evenly along the flight; each stringer then spans the gap between supports. |
| `mid_posts.section` | a section name (§6) | Post section (AWC DCA 6 p.20: 4x4), notched and bolted to each stringer with two 1/2" bolts (Figure 8A). |
| `mid_posts.footing_in` | a number, inches | Square footing under each post, inches (DCA 6 p.20: 10, or 12 round). |
| `mid_posts.footing_thick_in` | a number, inches | Footing thickness, inches (DCA 6 p.20: 6), at least 12" below grade or the frost line (Figure 34). |
| `foot` | a block (keys below) | Where the flight meets grade: each stringer on a bearing block on a post, the posts on footings (AWC DCA 6 p.22, Figure 34). Stringers never bear on a pad or patio not founded that deep. |
| `foot.post_section` | a section name (§6) | Post beside each stringer (AWC DCA 6, Figure 34: 4x4). On an outer stringer with a guard, the guard's foot post is this post, through-bolted. |
| `foot.footing_in` | a number, inches | Square footing under each post, inches (Figure 34: 10, or 12 round). Default: 10. |
| `foot.footing_thick_in` | a number, inches | Footing thickness, inches (Figure 34: 6 at least). Default: 6. |
| `foot.footing_depth_in` | a number, inches | The footing's bottom below grade, inches: at least 12 below undisturbed ground, or below the frost line if that is deeper (DCA 6 p.22, Figure 34). Default: 12. |
| `foot.block_section` | a section name (§6) | Bearing block under each stringer, from the footing up to it, fastened to the post (Figure 34: 2x4). |
| `foot.block_fastener` | text | Fastener for the block (Figure 34: #8 wood screw 3-1/2" or longer, or 16d threaded nail). |
| `foot.block_fasteners` | a number | Fasteners per block (Figure 34: 8). Default: 8. |
| `foot.post_fastener` | text | Fastener from a post with no guard into its stringer (Figure 34: #8 wood screw 3-1/2" or longer, or 16d threaded nail); a guard post takes two 1/2" bolts. |
| `foot.post_fasteners` | a number | Fasteners per post with no guard (Figure 34: 2). Default: 2. |
| `foot.code` | a block (keys below) | The limits to check against, from a printed source. |
| `foot.code.min_depth_in` | a number, inches | Least depth of the footing's bottom below grade, inches (p.22, Figure 34: 12). |
| `foot.code.min_footing_in` | a number, inches | Smallest square footing, inches (Figure 34: 10). |
| `foot.code.min_thick_in` | a number, inches | Thinnest footing, inches (Figure 34: 6). |
| `guards` | a block (keys below) | Guards along the flight's open sides. The posts, rails, cap and balusters are derived from the stair. |
| `guards.sides` | `[ … ]` of one of `north`, `south`, `east`, `west`s | The sides of the flight that get a guard, by compass: the two sides across its run. Off a skew beam, the compass points nearest the beam's two ways. |
| `guards.height_in` | a number, inches | Top of the cap above the nosing line, inches (AWC DCA 6, Figure 30: 34 at least). |
| `guards.post_section` | a section name (§6) | Post section, bolted to the outer stringer (Figure 30: 4x4). |
| `guards.post_spacing_in` | a number, inches | Most between posts, inches, measured level (Figure 30: 72). |
| `guards.rail_section` | a section name (§6) | Top and bottom rail section, on the slope. |
| `guards.cap_section` | a section name (§6) | Cap section, on the slope. |
| `guards.baluster_section` | a section name (§6) | Baluster section. Leave out for rods. |
| `guards.rod` | a block (keys below) | Round metal balusters (rods) in place of baluster_section: plumb, rail to rail, spaced under opening_in by their diameter. Counted and listed as bought items, not cut from lumber. Not Figure 24's 2x2 baluster screwed top and bottom: 2by says so in a WARN and does not rate them. |
| `guards.rod.diameter_in` | a number, inches | The rod's diameter, inches (e.g. 0.75). |
| `guards.rod.connectors` | `true` or `false` | Each end sits in a connector screwed to the rail: two a rod, counted. Leave out or false for rods set in drilled holes. Default: false. |
| `guards.style` | one of `face`, `inline` | How the rails and infill sit. face: the rails on edge on the posts' inside face, the balusters on the rails' inside face (AWC DCA 6 Figure 24). inline: the rails on edge between the posts, centred on their line, the balusters or rods centred in the rails' edges, the cap flat over them; nothing stands proud of the rails. inline is not Figure 24's detail: 2by says so in a WARN. Default: face. |
| `guards.railing` | text | The railing style this is built to: an id from railing_styles:, or a 2by preset (dca6-fig24: AWC DCA 6 Figure 24, rails on the posts' inside face, 2x2 balusters, 2x6 cap; inline-rods: a 2x4 cap flat over the posts, 2x4 rails on edge between them, 3/4" rods in connectors). Leave out for project.railing. Any detail field written here wins over the style. |
| `guards.rail_position` | one of `centred`, `inside`, `outside` | Where inline rails sit across the posts: centred on their line, inside (flush with the posts' and a post-wide cap's deck-side face, screwed into the post faces) or outside (flush with their outside face). Face-style rails are on the posts' inside face (AWC DCA 6 Figure 24). Default: centred. |
| `guards.opening_in` | a number, inches | Largest gap the balusters leave, inches; 2by spaces them under it. |
| `guards.bottom_gap_in` | a number, inches | The bottom rail's underside above the nosing line, inches, plumb. |
| `guards.end_posts` | `[ … ]` of texts | Posts already standing at the flight's head or foot on one side's post line, that side's guard ends on: a post in the model, or a deck guard's post, named by the guard (2by takes its post at that end). That side sets no post of its own there; its rails, cap and infill end on the post's face toward the flight. The post must reach the stair's cap there. A guard that names this stair back in its own end_posts is an ERROR: name the joint's post from one side only. |
| `guards.code` | a block (keys below) | The limits to check against, from a printed source. |
| `guards.code.min_height_in` | a number, inches | Lowest guard above the nosings, inches (Figure 30: 34). |
| `guards.code.max_post_spacing_in` | a number, inches | Widest post spacing, inches (Figure 30: 72). |
| `guards.code.max_opening_in` | a number, inches | The sphere nothing beside the treads may pass, inches (Figure 30: 4-3/8). |
| `guards.code.max_triangle_in` | a number, inches | The sphere the triangle of tread, riser and bottom rail may not pass, inches (Figure 30: 6). |
| `guards.code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `handrail` | a block (keys below) | A handrail from over the top riser to over the lowest, returned at its ends (AWC DCA 6 p.22, Figure 33), mounted to the guard on brackets. |
| `handrail.side` | one of `north`, `south`, `east`, `west` | The side of the flight it runs on. |
| `handrail.height_in` | a number, inches | Its top above the nosing line, inches (AWC DCA 6 p.22: 34 to 38). |
| `handrail.section` | a section name (§6) | Handrail section; it has to be graspable (Figure 32B). |
| `handrail.code` | a block (keys below) | The limits to check against, from a printed source. |
| `handrail.code.min_height_in` | a number, inches | Lowest, inches (p.22: 34). |
| `handrail.code.max_height_in` | a number, inches | Highest, inches (p.22: 38). |
| `handrail.code.min_perimeter_in` | a number, inches | Smallest perimeter of a graspable rail, inches (Figure 32B, Type I: 4). |
| `handrail.code.max_perimeter_in` | a number, inches | Largest perimeter, inches (Figure 32B, Type I: 6-1/4). |
| `handrail.code.max_cross_in` | a number, inches | Largest cross-section dimension, inches (Figure 32B, Type I: 2-1/4). |
| `handrail.code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `box_steps`

Box steps: framed boxes on grade stepping down from a deck or landing rim, each topped with tread boards. For low steps where a cut stringer would be deeper than the rise. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `from` | a beam id | The rim they step down from, north-south or east-west. |
| `face` | one of `north`, `south`, `east`, `west` | The side of that rim they step down on. |
| `at_ft` | a number, feet | Low end of the steps along the rim, feet. |
| `width_in` | a number, inches | Width of the steps along the rim, inches. |
| `risers` | a number | Risers from the rim's deck boards to grade, the step onto the rim's deck the top one; one box for each riser below it. |
| `run_in` | a number, inches | Depth of each box, the run of each step, inches. |
| `section` | a section name (§6) | Box frame section, ripped to each box's height (front, back, ends and cross members). |
| `frame_spacing_in` | a number, inches | Most between the frame's cross members, inches: what the treads span. Default: 16. |
| `tread_section` | a section name (§6) | Tread board section. |
| `treads_per_step` | a number | Tread boards on each box. Leave out for as many as fill the run. |
| `code` | a block (keys below) | The limits to check against, from a printed source. Leave one out and it is not checked. |
| `code.max_riser_in` | a number, inches | Tallest riser, inches. |
| `code.min_run_in` | a number, inches | Shallowest run, inches. |
| `code.max_frame_spacing_in` | a number, inches | Most the tread boards may span, inches. |
| `code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `railing_styles`

Railing details a builder uses, each named once and reused: every guard and stair guard names one (railing:) or takes project.railing, and fills in from it the detail fields it does not write itself. Height, decking and the code block stay on each guard (a deck guard and a stair guard differ: DCA 6 Figure 24 vs Figure 30). A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Its name; guards, stairs' guards and project.railing name it. |
| `label` | text | Human name shown on drawings and schedules. |
| `extends` | text | A 2by preset (dca6-fig24, inline-rods) or another style's id: this style is that one with the fields written here changed. |
| `post_section` | a section name (§6) | Post section. |
| `post_spacing_in` | a number, inches | Most on centre between posts, inches. |
| `rail_section` | a section name (§6) | Top and bottom rail section. |
| `cap_section` | a section name (§6) | Cap section, flat over the posts. |
| `baluster_section` | a section name (§6) | Baluster section. Leave out for rods. |
| `rod` | a block (keys below) | Round metal balusters (rods) in place of baluster_section: plumb, rail to rail, spaced under opening_in by their diameter. Counted and listed as bought items, not cut from lumber. Not Figure 24's 2x2 baluster screwed top and bottom: 2by says so in a WARN and does not rate them. |
| `rod.diameter_in` | a number, inches | The rod's diameter, inches (e.g. 0.75). |
| `rod.connectors` | `true` or `false` | Each end sits in a connector screwed to the rail: two a rod, counted. Leave out or false for rods set in drilled holes. Default: false. |
| `style` | one of `face`, `inline` | How the rails and infill sit. face: the rails on edge on the posts' inside face, the balusters on the rails' inside face (AWC DCA 6 Figure 24). inline: the rails on edge between the posts, centred on their line, the balusters or rods centred in the rails' edges, the cap flat over them; nothing stands proud of the rails. inline is not Figure 24's detail: 2by says so in a WARN. Default: face. |
| `rail_position` | one of `centred`, `inside`, `outside` | Where inline rails sit across the posts: centred on their line, inside (flush with the posts' and a post-wide cap's deck-side face, screwed into the post faces) or outside (flush with their outside face). Face-style rails are on the posts' inside face (AWC DCA 6 Figure 24). Default: centred. |
| `opening_in` | a number, inches | Largest gap the infill leaves, inches. |
| `bottom_gap_in` | a number, inches | Gap under the bottom rail, inches (from the deck surface; on a stair, above the nosing line). |

### `guards`

Guards (railings) along deck edges. The posts, rails, cap, balusters and their hardware are derived from the run. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `from` | `[x, y]`, feet | One end of the deck edge it runs along, [x, y] feet. North-south, east-west, or on a skew beam's line (axis: skew), whose beam is then its rim. |
| `to` | `[x, y]`, feet | The other end, [x, y] feet. |
| `height_in` | a number, inches | Top of the cap above the deck surface, inches (AWC DCA 6 Figure 24: 36 at least). |
| `decking_in` | a number, inches | The deck boards' thickness, inches: the height is measured from their top. Default: 0. |
| `post_section` | a section name (§6) | Post section (DCA 6 Figure 24: 4x4, not notched). |
| `post_spacing_in` | a number, inches | Most on centre between posts, inches (Figure 24: 72). |
| `rail_section` | a section name (§6) | Top and bottom rail section, on edge on the posts' inside face (Figure 24: 2x4). |
| `cap_section` | a section name (§6) | Cap section, flat on the posts (Figure 24: 2x6). |
| `baluster_section` | a section name (§6) | Baluster section (Figure 24: 2x2). Leave out for rods. |
| `rod` | a block (keys below) | Round metal balusters (rods) in place of baluster_section: plumb, rail to rail, spaced under opening_in by their diameter. Counted and listed as bought items, not cut from lumber. Not Figure 24's 2x2 baluster screwed top and bottom: 2by says so in a WARN and does not rate them. |
| `rod.diameter_in` | a number, inches | The rod's diameter, inches (e.g. 0.75). |
| `rod.connectors` | `true` or `false` | Each end sits in a connector screwed to the rail: two a rod, counted. Leave out or false for rods set in drilled holes. Default: false. |
| `style` | one of `face`, `inline` | How the rails and infill sit. face: the rails on edge on the posts' inside face, the balusters on the rails' inside face (AWC DCA 6 Figure 24). inline: the rails on edge between the posts, centred on their line, the balusters or rods centred in the rails' edges, the cap flat over them; nothing stands proud of the rails. inline is not Figure 24's detail: 2by says so in a WARN. Default: face. |
| `railing` | text | The railing style this is built to: an id from railing_styles:, or a 2by preset (dca6-fig24: AWC DCA 6 Figure 24, rails on the posts' inside face, 2x2 balusters, 2x6 cap; inline-rods: a 2x4 cap flat over the posts, 2x4 rails on edge between them, 3/4" rods in connectors). Leave out for project.railing. Any detail field written here wins over the style. |
| `rail_position` | one of `centred`, `inside`, `outside` | Where inline rails sit across the posts: centred on their line, inside (flush with the posts' and a post-wide cap's deck-side face, screwed into the post faces) or outside (flush with their outside face). Face-style rails are on the posts' inside face (AWC DCA 6 Figure 24). Default: centred. |
| `end_posts` | `[ … ]` of texts | Posts already standing at an end of the run, on the guard posts' line: a post in the model (a deck or porch post), or a stair's or another guard's post, named by the stair or the guard (2by takes its post at that end). The run stops at the post's face, no guard post is set there, and the rails end on it. Or a post beside the line at an end, its face on the face of the guard's posts (a corner post inside the corner): the infill stops at its near face, and the rails and cap clear of it run on along its side to its far face, fastened to it (a WARN gives any gap between the rails and its face). 2by counts no guard bolts or hold-down for it here. A stair's post at its head or foot that stops under this guard's cap is cut longer, up to the cap's underside, and the stair's own cap stops at its face; any other post that stops under the cap is an ERROR. |
| `opening_in` | a number, inches | Largest gap the balusters leave, inches; 2by spaces them under it (Figure 24: a 4" sphere may not pass). |
| `bottom_gap_in` | a number, inches | Gap from the deck surface to the bottom rail, inches. |
| `openings` | a list of blocks (keys below) | Where the guard stops: a stair or a gate. |
| `openings[].at_ft` | a number, feet | Where the opening starts, feet along the run from its low end (on a skew beam's line, the end nearer its first support). |
| `openings[].width_in` | a number, inches | Its width, inches. |
| `code` | a block (keys below) | The limits to check against, from a printed source. Leave one out and it is not checked. |
| `code.min_height_in` | a number, inches | Lowest guard, inches (DCA 6 Figure 24: 36). |
| `code.max_post_spacing_in` | a number, inches | Widest post spacing, inches (Figure 24: 72). |
| `code.max_opening_in` | a number, inches | The sphere nothing may pass, inches (Figure 24: 4). |
| `code.min_member` | a section name (§6) | The smallest rim or joist a post bolts to (DCA 6 p.19: 2x8). |
| `code.min_post` | a section name (§6) | The smallest post (DCA 6 p.19: 4x4). |
| `code.holddown_lb` | a number, lb | The hold-down each post needs, allowable tension, lb (DCA 6 p.19: 1,800). |
| `code.holddown_height_in` | a number, inches | The tallest guard that rating is for, inches (DCA 6 p.19: 36). |
| `code.source` | text | The printed source of these limits (rule 4: cite it). (Recorded for people; 2by does not use it.) |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `grade`

The ground, where it is not at the datum (z = 0). A post stands on the ground at its centre (the highest region over it): its base, cut length, pad and knee braces are measured from there. Drawn ghosted with the extras, on the Context layer, as earth from the datum up to the surface; never counted. Outside every region the ground is the datum. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Its name, used in any finding about it. |
| `label` | text | What it is (the ground under tier 1, held by the retaining wall). |
| `outline` | `[ … ]` of `[ … ]` of numberss | Its corners in order round the outline, three or more, one per line: - [x, y, z], the ground's height z at that corner, or - [x, y] with top_ft. The ground over it is the plane through them; a corner more than 1" off that plane is an error (split the region). |
| `top_ft` | a number, feet | The ground's height at any corner given as [x, y], feet above the datum. |

### `extras`

Things beside the structure, drawn for context and never counted, checked or ordered: the house wall, a patio, a pool, a fence. They are drawn ghosted on their own Context layer, in the plan, the isometric, the elevations and the 3D view. A kind 2by does not know, or a shape it cannot draw, is a warning and is left out; it never stops the structure. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `kind` | text, e.g. `wall`, `slab`, `pool`, `fence`, `box`, `label` | What it is: wall (a line with a thickness and a height, the house the deck stands against), slab (a flat outline: a patio, a path), pool (an outline at its coping, the water drawn below it), fence (a line and a height), box (a footprint and a height, for anything else) or label (text at a point). Any other kind is reported and left out. |
| `label` | text | Its name, written beside it on the plan. On a label, the text itself. |
| `color` | text | The colour it is drawn in, a hex colour (#4a7fb5) or a colour name (blue). Leave out for its kind's, or its finish's. |
| `from` | `[x, y]`, feet | Wall or fence: one end of its line, [x, y] feet. The line may run at any angle. |
| `to` | `[x, y]`, feet | Wall or fence: the other end, [x, y] feet. |
| `side` | text, e.g. `north`, `south`, `east`, `west`, `centre` | Wall: which side of the from-to line its thickness lies on, by compass (a side within 30 degrees of the line's own direction is reported and centred), or centre. Default: centre. |
| `thick_in` | a number, inches | Wall: its thickness, inches (default 8). Fence: its boards' thickness, inches (default 1.5). Slab: its thickness, inches (default 4). Drawn only. |
| `finish` | text, e.g. `brick`, `stone`, `stucco`, `siding`, `concrete`, `wood` | Wall: what its face is, drawn as a tone. Leave out for plain grey. |
| `outline` | `[ … ]` of `[ … ]` of numberss | Slab, pool or box: its corners in order round the outline, three or more, written one per line (- [x, y]). A slab's corners may carry their height (- [x, y, z]), for a slab that follows a slope, as a driveway on graded ground does; a corner left [x, y] takes top_ft. |
| `at` | `[ … ]` of numbers | Label: where the text sits, [x, y] or [x, y, z] feet. Box: the centre of its footprint, [x, y] feet, with size. |
| `size` | `[x, y]`, feet | Box: its footprint, [east-west, north-south] feet, centred on at. Or give an outline. |
| `bottom_ft` | a number, feet | Wall, fence or box: its bottom, feet above grade. Default: 0. |
| `top_ft` | a number, feet | Wall or box: its top, feet. Slab: its top surface, feet (default 0, grade). Pool: the top of its coping, feet (default 0). |
| `height_ft` | a number, feet | Wall or box: its height above bottom_ft, feet, in place of top_ft. |
| `height_in` | a number, inches | Fence: its height above bottom_ft, inches. |
| `water_in` | a number, inches | Pool: how far below the coping the water is drawn, inches. The coping is drawn 12" wide outside the outline. Default: 6. |
| `openings` | a list of blocks (keys below) | Wall: its doors and windows, cut out of it in every view and filled with a panel. One that runs past the wall's ends, above its top or below its bottom, or over another, is reported and not cut. |
| `openings[].id` **required** | an id | Its name, used in any warning about it. |
| `openings[].kind` | text, e.g. `door`, `window` | Door or window. A door is drawn as a leaf in its own colour, a window as tinted glass. |
| `openings[].at_ft` | a number, feet | Where it starts, feet along the wall from its from end. |
| `openings[].width_in` | a number, inches | Its width along the wall, inches. |
| `openings[].height_in` | a number, inches | Its height, inches. |
| `openings[].sill_ft` | a number, feet | The height of its bottom above grade, feet (not above the wall's bottom). A door left without one stands on the wall's bottom; a window needs one. |
| `openings[].color` | text | The colour its panel is drawn in, a hex colour or a colour name. Leave out for wood (a door) or glass (a window). |

### `addressed`

Findings that have been dealt with outside the model, each with how. The finding is still reported, marked ADDRESSED with this note, and is no longer counted as an open error or warning. An entry that matches no finding (the model changed under it) is a warning, so a stale note never sits silently in the file. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `code` | text | The finding's code, as 2by reports it (stair-guard, low-util, ...). |
| `member` | text | The id the finding names (S1, B3-N). Leave out to cover every finding with this code. |
| `how` | text | How it is handled, in words anyone reading the file can check: what was done, decided or built instead. |
| `by` | text | Who decided it. |
| `date` | text | When, YYYY-MM-DD. |

## 6. Lumber sections and post sizes

Use these names exactly for `section` (beams, joists, studs, rafters, ridge,
stringers, treads and kickers) and in `stock:` and `supply:`. They come from
2by's lumber file, which also holds each one's species, grade and stock
lengths.

| Section | Actual size, in | Species and grade | Stock lengths, ft |
|---|---|---|---|
| `2x2` | 1-1/2 × 1-1/2 | Southern Pine, 2"-4" wide No.2 | 3, 3.5, 4, 8 |
| `5/4x6` | 1 × 5-1/2 | Southern Pine, 5"-6" wide No.2 | 8, 10, 12, 16 |
| `trex-enhance-1x6` | 2/2 × 5-1/2 | Trex Enhance composite decking | 12, 16, 20 |
| `2x4` | 1-1/2 × 3-1/2 | Southern Pine, 2"-4" wide No.2 | 8, 10, 12, 14, 16 |
| `2x6` | 1-1/2 × 5-1/2 | Southern Pine, 5"-6" wide No.2 | 8, 10, 12, 14, 16 |
| `2x8` | 1-1/2 × 7-1/4 | Southern Pine, 8" wide No.2 | 8, 10, 12, 14, 16 |
| `2x10` | 1-1/2 × 9-1/4 | Southern Pine, 10" wide No.2 | 8, 10, 12, 14, 16 |
| `2x12` | 1-1/2 × 11-1/4 | Southern Pine, 12" wide No.2 | 10, 12, 14, 16 |
| `4x4` | 3-1/2 × 3-1/2 | Southern Pine, 2"-4" wide No.2 | 8, 10, 12 |
| `4x6` | 3-1/2 × 5-1/2 | Southern Pine, 5"-6" wide No.2 | 8, 10, 12, 14, 16, 20 |
| `6x6` | 5-1/2 × 5-1/2 | Southern Pine, timbers 5x5 and larger No.2 | 8, 10, 12, 14, 16 |
| `8x8` | 7-1/4 × 7-1/4 | Southern Pine, timbers 5x5 and larger No.2 | 8, 10, 12, 14, 16 |
| `LVL-9.25` | 1-3/4 × 9-1/4 | Boise Cascade VERSA-LAM 2.1E 3100 LVL (Southern Pine) | 20, 22, 24, 26, 28 |
| `LVL-11.875` | 1-3/4 × 11-7/8 | Boise Cascade VERSA-LAM 2.1E 3100 LVL (Southern Pine) | 24, 26, 28, 30, 32 |

Post `size` must be one of: `4x4`, `4x6`, `6x6`, `8x8`, `2x4`, `2x6`, `2x8`.

## 7. Hardware parts

Name a part only when the person has chosen it. Otherwise leave it out and
2by picks one. The names are from 2by's hardware catalogue, by role.

- **Joist hangers**, for `joist_fields.hanger`, and `connections` of type `hung`: `LU24`, `LU26`, `LU28`, `LU210`, `LUS28`, `LUS210`, `SUR24`, `SUL24`, `SUR26`, `SUL26`, `SUR210`, `SUL210`, `SUR214`, `SUL214`.
- **Girder hangers**, for `connections` of type `girder-hung`: `LUS28-2`, `LUS210-2`, `HGUS28-2`, `HGUS210-2`, `HGUS410`, `HGUS412`, `HGUS414`, `HU210-2`, `HU212-2`, `HUS48`, `HUS410`, `HUS412`, `HHUS48`, `HHUS410`, `HUCQ410-SDS`, `HUCQ412-SDS`, `HUC410`, `HUC412`, `ITS3.56/11.88`, `MIT411.88`, `BA3.56/11.88`, `BA3.12X`, `WP3.12X`, `THAI-2X 3.125`.
- **Post caps**, for `connections` of type `cap`: `AC4` (fits 4x4, 4x6), `AC6` (fits 6x6), `AC4RZ` (fits rough 4x4), `AC6RZ` (fits rough 6x6), `BC4` (fits 4x4), `BC46` (fits 6x6), `BC6` (fits 6x6), `BC8` (fits 8x8), `LSTA12-PAIR` (fits 2x6).
- **Post bases**, for `posts.base` and `post_bases.part_by_size`: `ABA66Z` (fits 6x6), `ABA46Z` (fits 4x6), `ABU44Z` (fits 4x4), `ABU66Z` (fits 6x6), `ABU88Z` (fits 8x8), `ABU46Z` (fits 4x6).
- **Ties**, for `joist_fields.tie` and `roof.tie`: `H2.5A`, `H2.5ASS`, `H2.5T`.

## 8. Reading 2by's findings

2by reports what it found as **ERROR** (wrong or unsafe: fix it), **WARN**
(probably wrong: fix it or confirm it with the person) and **INFO** (for the
record: nothing to fix). Each finding names a code, often a member, and for
problems in the file itself, the line. A fourth level, **ADDRESSED**, is a
finding the model's `addressed:` list answers (below): it is still listed,
with the note, and is not counted as open.

Problems in the file itself:

- `unknown-key`: a key 2by does not read where it is written. It names the
  key it expected or the block the key belongs under. Rename it or move it.
- `schema-shape`: a list written as a block, or a block written as a value.
  Rewrite it in the shape section 5 gives.
- `schema-type`: text where a number belongs (often a quoted length or a
  unit after a number), or a flag that is not `true` or `false`.
- `schema-value`: a value 2by does not read. It says what it is read as.
- `schema-id`, `dup-id`, `ref`: a missing id, two entries with one id, or an
  id that names nothing. Check the spelling against the list it names.
- `phase`: a phase that `phases:` does not declare, or a member built before
  what it stands on.
- `materials`: a library or revision in `materials:` that 2by does not have
  (use the ones in section 4), or a note that 2by has a newer revision than
  the one named. Update it only if the person agrees.
- `wrong-file`: this is 2by's lumber file or hardware catalogue, not a
  model. Those are separate files that belong to the person's account (the
  LUMBER and HARDWARE tabs) and hold cited engineering values: never write
  or edit them. Write the model instead.
- `format`: the file says it is a newer model format than this 2by reads.
  Write `format: 1` and use only the keys in this guide.
- `dimension`: a number 2by could not read. Fix the value; 2by will not
  guess.
- `parse`: a line 2by could not read at all, usually indentation, a tab or a
  missing `: `. Fix that line first: everything after it may be misread.

Problems in the structure. Fix these by changing the design, then ask the
person to confirm the change:

- `bending`, `deflection`, `joist-deflection`: a member is too small for its
  span. Use a deeper section, add a ply, shorten the span with another post
  or beam, or close up the joist spacing.
- `low-util`: a member is far bigger than its load needs. Use a smaller
  section, or add `waive: [low-util]` with a comment saying why it is
  deliberate.
- `coverage`, `orphan`, `not-borne`, `alignment`: joists or beams with
  nothing under part of their length. Check `bounds_x`/`bounds_y` against the
  beams' runs, and the beams' `supports`.
- `zero-span`: two supports of one beam at the same coordinate, often a beam
  whose `axis` does not match the line its posts are on.
- `hw-unknown`, `hw-nofit`: a part not in the catalogue, or one that does not
  fit. Leave the part out, or pick one from section 7.
- `footing`: a pad or a pier block's base too small for the load at the
  stated soil bearing.
- `pier-block`: a post on a precast pier block. 2by rates only the soil
  under its base, not the block or how deep it is set. Tell the person; with
  their say, record how they checked it in `addressed:`.
- `new-stock`: a section the lumber file does not stock. Use a name from
  section 6.
- `stock`: a cut longer than any stock length the person can buy. Shorten
  the span, or raise `supply: max_stock_ft` if they can get longer boards.
- `ply-screw`: a multi-ply beam with no `ply_fastening` spec for its pack
  (section 4).
- `orphan-span`: a beam carrying nothing over part or all of its run. A
  beam running the same way as the joists carries none of them: usually
  remove it (the outermost joist is the edge), or make it a rim at joist
  level as `garden-shed.yaml` does with `RIM-W` and `RIM-E`.
- `bending` or `deflection` that a deeper member does not cure: shorten the
  span. 2by's message says where one more post would bring it within
  limits; that is usually the cheaper fix.
- `stud-bracing`, `wall-bracing`: an unsheathed wall. Add `sheathing`.
- `below-grade`: a beam's, rim's or joist field's underside is below the
  ground of a `grade:` region under it. Lower the ground there (a strip
  region under the member, a few inches below it), or raise the member. The
  message gives the place and the depth.
- `clash`: two framing members in one place: a rim run to a joist's
  centre line instead of its face, a beam through a joist, a field's first
  joist half inside a rim, or two of one field's joists overlapping where
  `layout_origin` puts a spaced joist within a joist's width of the one on
  the bound. The message names both and where. Bring the end
  to the other member's face. A rim that runs between two members rather
  than through them says `rim: { ..., ends: hung }`: each end sits in a
  face-mount joist hanger on the member it butts, and 2by picks the hanger
  and orders it. A field's edge joist under a side rim on its bound is one
  board, not a clash.
- `rim-end`: a rim with `ends: hung` whose end butts no joist or beam at its
  height. Bring the end to a member's face.

### Findings handled outside the model: `addressed:`

Some findings are settled by something the model does not describe: the
side of a stair that runs along a brick wall is not open, a part has been
chosen whose rating is checked by hand, an engineer has designed a
connection. When the person has decided that, record it at the end of the
file, in their words:

```yaml
addressed:
  - code: stair-guard          # the finding's code, as 2by reports it
    member: S1                 # the member it names; leave out for every finding with this code
    how: >-
      S1's north side runs along the house's brick wall, 3" off it, so it
      is not an open side. A handrail is mounted on the brick.
    by: the owner              # who decided
    date: 2026-11-02
```

- The finding stays in the list, marked ADDRESSED with the note, so anyone
  who opens the file sees the finding and the answer together. It no longer
  counts as an open error or warning, and `validate_model` reports it under
  `counts.addressed`.
- An entry that matches no finding (the code or member misspelt, or the
  model changed under it) is a WARN `addressed`, and so is one with no
  `how`. Fix the entry or take it out.
- **Only the person addresses a finding.** Never add an entry to make an
  ERROR or WARN go away that changing the model would fix, and never
  without their say. `how` says what was done, decided or built, so that
  someone else can check it: "the hanger's page is on order" addresses
  nothing; "the hanger chosen, with its skew range and rating read off its
  printed page" does.
- By hand: open the design in 2by, find the finding's code and member in
  **FINDINGS**, add the entry, and check that FINDINGS now shows it as
  ADDRESSED.

## 9. A complete example

The smallest useful model: an 8' × 8' freestanding deck on four posts, with
two beams notched into the posts and drop-framed joists. Note the beams'
`top_ft`, one joist depth under the deck (section 4). It derives with no
errors and no warnings. The loads in it are examples only.

```yaml
# A freestanding 8' x 8' deck on four posts. Example only: every load
# below must come from YOUR building department, not from this file.
materials: { lumber: southern-pine@l8, hardware: simpson@c23 }   # 2by's libraries this is written for

project:
  slug: small-deck
  title: Small Deck 8x8
  revision: r1
  date: 2026-10-12
  format: 1
  live_psf: 40            # example value: ask your building department
  dead_psf: 10            # example value: framing and decking
  allowable_psf: 1500     # example value: soil bearing, from your local code
  deck_top_ft: 2'-6"      # top of the joists above grade

phases:
  - id: p1
    name: Deck

# x runs east, y runs south, both in feet from the north-west corner.
posts:
  - id: P1
    at: [0, 0]
    size: 6x6
    phase: p1
  - id: P2
    at: [8, 0]
    size: 6x6
    phase: p1
  - id: P3
    at: [0, 8]
    size: 6x6
    phase: p1
  - id: P4
    at: [8, 8]
    size: 6x6
    phase: p1

# The joists sit ON the beams (framing: drop), so each beam tops out one
# joist depth below the deck: 2'-6" less a 2x8's 7-1/4" is 1'-10-3/4".
beams:
  - id: B-N
    label: North beam
    axis: x
    supports: [P1, P2]
    section: 2x12
    trib_ft: 4              # half the 8' joist span it carries
    top_ft: 1'-10-3/4"
    phase: p1
  - id: B-S
    label: South beam
    axis: x
    supports: [P3, P4]
    section: 2x12
    trib_ft: 4              # half the 8' joist span it carries
    top_ft: 1'-10-3/4"
    phase: p1

connections:
  - at: P1
    member: B-N
    type: notch
  - at: P2
    member: B-N
    type: notch
  - at: P3
    member: B-S
    type: notch
  - at: P4
    member: B-S
    type: notch

joist_fields:
  - id: J1
    label: Deck joists
    section: 2x8
    span_axis: y
    bounds_x: [0, 8]
    bounds_y: [0, 8]
    bears_on: [B-N, B-S]
    spacing_in: 16
    framing: drop
    blocking: true
    phase: p1
```
