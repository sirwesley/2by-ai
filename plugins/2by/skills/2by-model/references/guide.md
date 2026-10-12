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
     an engineered beam.

   They are at `https://2by.app/ai/examples/<name>`.
4. **Write the whole file**, following sections 4 to 7. Use the comments to
   say where each number came from.
5. **Hand it back complete**, as one YAML block the person can save as
   `<name>.yaml`. Never hand back fragments or diffs.
6. **Check it yourself, if you can.** 2by's own check is open to you:
   - as tools, from the 2by MCP server at `https://2by.app/mcp`
     (`validate_model`, `get_guide`, `get_example`);
   - over HTTP: `POST https://2by.app/api/ai/validate` with
     `{"yaml": "<the whole file>"}` (described at `/ai/openapi.json`).

   Both answer with `ok`, the ERROR and WARN findings (code, member, line
   for problems in the file itself, message) and a `summary` of what the
   model builds. Read the summary back to the person to confirm it is what
   they meant. Fix and check again until `ok` is true, then hand over the
   file.
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
  Pad sizes if they are known.
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
  the rafter size and spacing.
- **Stairs.** Where they land, and the width.
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
  `hung` and `girder-hung` hang it from another beam. This is a
  construction choice, not an engineering value: if the person has no
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
| `waste_pct` | a number, percent | Waste allowance on area materials, percent. Default: 10. |
| `bearing_ft` | a number, feet | Height of beam bottoms for beams with no height of their own, when there is no deck_top_ft, feet. Default: 8.5. |
| `deck_top_ft` | a number, feet | Top of the floor or deck framing, feet. A beam with no top_ft or bearing_ft tops out here, which is right for flush framing; for drop framing give the beams a top_ft one joist depth lower. |
| `uplift_lb` | a number, lb | Design uplift per connection, lb. |
| `corner_style` | text, e.g. `3-stud`, `california`, `clips` | How wall corners are framed. |
| `projection` | one of `third-angle`, `first-angle`, `first` | Arrangement of the orthographic views. |

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
| `facing` | one of `x`, `y` | Axis the wide face runs along, on a rectangular post. |
| `perimeter` | `true` or `false` | On the perimeter (sizes its footing). |
| `base` | a part name (§7) | Base part for this post, overriding post_bases. |
| `pad_in` | a number, inches | Poured footing pad width, square, inches. Leave out and 2by reports, from each post's load, the footing it needs. |
| `pad_depth_in` | a number, inches | Poured footing pad depth, inches. Default: 12. |
| `standoff_in` | a number, inches | This post's base standoff, inches. |
| `braced` | a number or text (see meaning) | Weak-axis bracing: continuous, or a number of brace points. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `beams`

Beams, girders, rims and headers. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `axis` | one of `x`, `y` | Direction the beam runs. |
| `side` | one of `north`, `south`, `east`, `west`, `centre` | Face of the posts it is flush with, when it is off their centres. Leave out for centred. |
| `supports` | `[ … ]` of post ids | Posts on this beam's line, in any order. |
| `at_ft` | a number, feet | Line the beam runs on, feet across its axis, when it has no supports. |
| `from_ft` | a number, feet | Start of the run along its axis, feet. |
| `to_ft` | a number, feet | End of the run along its axis, feet. |
| `span_ft` | a number, feet | Span, feet, when nothing else fixes it. |
| `plies` | a number | Number of plies. Two or more need a ply_fastening spec for the pack's thickness. Default: 1. |
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
| `bears_on` | `[ … ]` of beam ids | The beams the joists bear on (at least two). |
| `spacing_in` | a number, inches | Joist spacing, inches. Default: 16. |
| `framing` | one of `flush`, `drop` | flush: joists hung between beams. drop: joists on top of them. |
| `hung_at` | `[ … ]` of beam ids | Beams the joists hang from in hangers. |
| `cantilever_ft` | a block of a beam id → a number | Cantilevers, by beam id. |
| `end_at_ft` | a block of a beam id → a number | Joist ends, by beam id. |
| `hanger` | a part name (§7) | Hanger part. |
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
| `type` | one of `notch`, `cap`, `hung`, `girder-hung`, `bears` | How the beam meets the post. |
| `part` | a part name (§7) | Connector part. Leave out and 2by picks one. |
| `into` | a beam id | The beam a hung member hangs from. |
| `on` | a beam id | The beam it bears on. |
| `face` | one of `north`, `south`, `east`, `west` | The face of the post it sits on. |
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
| `top_plates` | a number | Number of top plates. Default: 2. |
| `bottom_plates` | a number | Number of bottom plates. Default: 1. |
| `bearing` | `true` or `false` | Carries the roof. |
| `gable` | a roof id | The roof whose gable this wall closes. |
| `sheathing` | text | Sheathing product, e.g. 7/16" OSB. Without it a stud wall has nothing bracing it, and 2by says so. |
| `housewrap` | `true` or `false` | Housewrapped. |
| `cladding` | text | Cladding, e.g. board and batten. |
| `hold_downs` | text | Hold-down part or note. |
| `stud_ties` | `true` or `false` | Studs tied to plates. |
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
| `ridge_section` | a section name (§6) | Ridge board section. |
| `support_section` | a section name (§6) | Section of the ridge supports. |
| `bears_on` | `[ … ]` of wall ids | Walls the rafters bear on. |
| `tie` | a part name (§7) | Rafter tie part. |
| `collar_ties` | `true` or `false` | Collar ties. |
| `ceiling_joists` | `true` or `false` | Ceiling joists tie the walls. |
| `notch_limit` | a number | Largest birdsmouth as a fraction of rafter depth. Default: 0.333. |
| `blocking` | `true` or `false` | Blocking between rafters. |
| `sheathing` | text | Roof sheathing product. |
| `roofing` | text | Roof covering, e.g. asphalt shingle. |
| `waive` | `[ … ]` of one of `low-util`, `deflection`, `unused`s | Checks that are deliberate here. State why in a comment. |
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

### `stairs`

Stairs. A list. Each item is a block with these keys.

| Key | Written as | Meaning |
|---|---|---|
| `id` **required** | an id | Unique id. Other entries refer to it. |
| `label` | text | Human name shown on drawings and schedules. |
| `from` | a beam id | The beam the flight hangs from. |
| `face` | one of `north`, `south`, `east`, `west` | The side of that beam the flight runs down. Default: south. |
| `at_ft` | a number, feet | Low side of the flight along the beam, feet. |
| `lands_at_ft` | a number, feet | Height the flight lands at, feet. Default: 0. |
| `risers` | a number | Number of risers. |
| `run_in` | a number, inches | Run per tread, inches. Default: 10. |
| `width_in` | a number, inches | Width outside the outer stringers, inches. Default: 36. |
| `stringers` | a number | Number of stringers. Default: 3. |
| `stringer_section` | a section name (§6) | Stringer section. |
| `tread_section` | a section name (§6) | Tread board section. |
| `treads_per_step` | a number | Boards per tread. Default: 2. |
| `open_risers` | `true` or `false` | No riser boards. |
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
| `phase` | a phase id | Build phase id from phases:. Sets build order and which phase a cost lands in. |
| `layer` | a layer id | Layer id from layers:, overriding the layer its kind would land on. |

## 6. Lumber sections and post sizes

Use these names exactly for `section` (beams, joists, studs, rafters, ridge,
stringers, treads and kickers) and in `stock:` and `supply:`. They come from
2by's lumber file, which also holds each one's species, grade and stock
lengths.

| Section | Actual size, in | Species and grade | Stock lengths, ft |
|---|---|---|---|
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

- **Joist hangers**, for `joist_fields.hanger`, and `connections` of type `hung`: `LU24`, `LU26`, `LU28`, `LU210`, `LUS28`, `LUS210`.
- **Girder hangers**, for `connections` of type `girder-hung`: `LUS28-2`, `LUS210-2`, `HGUS28-2`, `HGUS210-2`, `HGUS410`, `HGUS412`, `HGUS414`, `HU210-2`, `HU212-2`, `HUS48`, `HUS410`, `HUS412`, `HHUS48`, `HHUS410`, `HUCQ410-SDS`, `HUCQ412-SDS`, `HUC410`, `HUC412`, `ITS3.56/11.88`, `MIT411.88`, `BA3.56/11.88`, `BA3.12X`, `WP3.12X`, `THAI-2X 3.125`.
- **Post caps**, for `connections` of type `cap`: `AC4` (fits 4x4, 4x6), `AC6` (fits 6x6), `AC4RZ` (fits rough 4x4), `AC6RZ` (fits rough 6x6), `BC4` (fits 4x4), `BC46` (fits 6x6), `BC6` (fits 6x6), `BC8` (fits 8x8), `LSTA12-PAIR` (fits 2x6).
- **Post bases**, for `posts.base` and `post_bases.part_by_size`: `ABA66Z` (fits 6x6), `ABA46Z` (fits 4x6), `ABU44Z` (fits 4x4), `ABU66Z` (fits 6x6), `ABU88Z` (fits 8x8), `ABU46Z` (fits 4x6).
- **Ties**, for `joist_fields.tie` and `roof.tie`: `H2.5A`, `H2.5ASS`, `H2.5T`.

## 8. Reading 2by's findings

2by reports what it found as **ERROR** (wrong or unsafe: fix it), **WARN**
(probably wrong: fix it or confirm it with the person) and **INFO** (for the
record: nothing to fix). Each finding names a code, often a member, and for
problems in the file itself, the line.

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
- `footing`: a pad too small for the load at the stated soil bearing.
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

## 9. A complete example

The smallest useful model: an 8' × 8' freestanding deck on four posts, with
two beams notched into the posts and drop-framed joists. Note the beams'
`top_ft`, one joist depth under the deck (section 4). It derives with no
errors and no warnings. The loads in it are examples only.

```yaml
# A freestanding 8' x 8' deck on four posts. Example only: every load
# below must come from YOUR building department, not from this file.
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
