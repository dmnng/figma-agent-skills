---
name: markup-layers
description: >
  Renames non-component, non-instance layers in the selected frame(s) so the layer panel mirrors a pseudo-code/markup structure — an auto layout frame becomes div.flex.flex-row or div.flex.flex-col, a plain frame becomes div.relative, text becomes p, vector shapes become svg, image-filled shapes become img. Requires one or more frames to be selected first; if nothing is selected, asks the user to select a frame. Use this skill whenever the user wants to rename layers to reflect structure, make the layer tree read like markup, clean up layer names so they communicate layout instead of "Frame 47"/"Rectangle 12", or asks "can you rename these to match the code structure", "make this look like Tailwind classes", "name these layers by their layout". This is a query-then-mutation skill — it only renames layers after the user confirms which proposed names to apply.
---

# Overview

Walks the node tree of the selected frame(s) and proposes a new name for every non-component, non-instance layer, reflecting its structural role rather than its current arbitrary name — e.g. a horizontal auto layout frame proposes `div.flex.flex-row`, a text node proposes `p`. Lists every proposed rename for the user to confirm before applying any of them. Reads the live selection directly to compute proposals, then writes only the renames the user approves.

The names are deliberately structural, not a property dump: layout-defining signals (display mode, direction, wrap, non-default alignment) are encoded, but gap, padding, exact dimensions, colors, and other cosmetic properties never are. The goal is a layer tree a developer could skim and recognize the markup shape of, not a 1:1 property mirror.

# Invocation

## Step 1 — Verify the Selection

This skill needs a concrete starting point: one or more **frames** selected in the file. If nothing is selected, or the selection is a layer that isn't a frame, ask the user to select the frame(s) to rename rather than guessing scope from the whole page — a page can contain many unrelated frames, and silently renaming layers across all of them risks far more changes than the user asked for.

If multiple frames are selected, compute and report proposals for each one separately.

## Step 2 — Compute Proposed Names

Walk the node tree of each selected frame and, for every layer, determine its proposed name using [References → Element-Type Mapping](#element-type-mapping). This is read-only — nothing is renamed in this step.

- **Components and instances are never renamed**, and nothing nested inside an instance is renamed either — that subtree belongs to the component, not to this skill. Stop descending the moment you hit an instance boundary.
- **Skip a layer from the report if its current name already exactly matches its proposed name** — only real changes belong in the confirmation list, not a wall of no-ops.
- **If a node doesn't match any row in the mapping table** (an unsupported type — a slice, connector, sticky, table, etc.), skip it and note it once as "unsupported, left unchanged" rather than guessing a plausible-looking tag for it.
- **Identical sibling structures get identical proposed names on purpose.** Two parallel list items that are both horizontal auto layout frames both become `div.flex.flex-row` — don't invent index suffixes or other uniqueness hacks to tell them apart; that's not how repeated markup structures read in real code either.

## Step 3 — Report and Confirm

**Always list every proposed rename before changing anything.** Group the report by frame, showing each change as `current name → proposed name`, using a layer path (e.g. `Card > Content > Label`) wherever the current name alone wouldn't make the layer findable. Note the count of layers excluded as components/instances/instance-descendants as a single summary line per frame, not one line per excluded layer. Use the [References → Report Format](#report-format) template.

End the report by asking which renames to apply — all of them, by frame, or a specific numbered subset. Don't apply anything before this confirmation; every rename here is a mutation across the live file, and a name the user disagrees with is just as easy to get wrong at scale as the structural cleanup mistakes this kind of skill is meant to avoid.

## Step 4 — Apply Confirmed Renames

Apply only the renames the user confirmed, via `use_figma` calls. Work frame by frame in the order reported.

After applying, report a summary: how many layers were renamed (per frame), how many were excluded as components/instances, how many were unsupported and left unchanged, and anything that was proposed but not confirmed and so is still outstanding.

## Notes on the naming rules

- **Default alignment is never written.** Figma's auto layout default is start-aligned on both axes — `justify-start`/`items-start` would be true of nearly every frame and would add noise rather than signal, so they're the implicit default and only deviations (`center`, `end`, `space-between`/`justify-between`, `baseline`) get written.
- **No property dump.** Gap, padding, exact widths/heights, corner radius, fills, and effects never appear in a proposed name — they're real properties but not structural ones, and the whole point is a name you can skim, not a serialized node.
- **A frame's own selected-ness doesn't exempt it.** The top-level selected frame gets a proposed name too, same as any descendant — it's a structural layer like any other.

# References

### Element-Type Mapping

| # | Node condition | Proposed tag |
|---|---|---|
| 1 | Frame, auto layout, direction = horizontal | `div.flex.flex-row` (+ `.flex-wrap` if wrapping is on, + `.justify-*`/`.items-*` only when not the start-aligned default) |
| 2 | Frame, auto layout, direction = vertical | `div.flex.flex-col` (+ same modifiers as row 1) |
| 3 | Frame, no auto layout (children positioned manually) | `div.relative` |
| 4 | Group | `div` |
| 5 | Text node | `p` |
| 6 | Vector-type node (vector, boolean operation, star, ellipse, polygon, line) | `svg` |
| 7 | Rectangle or frame with an image fill and no children | `img` |
| 8 | Component, Instance, or any descendant of an Instance | excluded — not renamed, not reported as a change |
| 9 | Anything else unrecognized (slice, connector, sticky, table, etc.) | skip — report once as "unsupported, left unchanged" |

For rows 1–2, the alignment modifiers compare against Figma's auto layout default (primary axis `MIN`/packed, counter axis `MIN`) — only a non-default value contributes a class. A frame with default alignment and no wrap produces just `div.flex.flex-row` or `div.flex.flex-col` with no further modifiers.

### Report Format

```
# Layer Markup — [Frame name(s)]

## [Frame name]

### Proposed Renames ([count])
1. `[current name]` → `[proposed name]` [— path: [layer path], if needed for clarity]
2. ...
*(omit this section if no real changes were found — say "No renames needed in [frame name]" instead)*

### Excluded / Skipped
- [N] layers excluded (component/instance/instance-descendant)
- [N] layers unsupported and left unchanged: `[layer path]` ([node type]), ...
*(omit either line if its count is 0)*

---

**Which renames should I apply?** (all / by frame / specific numbered items)
```

### Follow-up Menu

| If the user says… | Do this |
|-------------------|---------|
| "apply all" / "rename everything" | Apply every proposed rename across all reported frames |
| "just this frame" | Apply only the proposed renames for the named frame, leave the rest unconfirmed |
| "skip the text layers" | Apply every proposed rename except rows where the proposed tag is `p`, and say so |
| "show me the unsupported ones" | List each layer noted as unsupported, with its current name, path, and node type |
| "check another selection too" | Re-run Step 1–3 against a newly specified selection rather than assuming the original selection still applies |

# Assets

None — this skill reads and writes the live Figma file directly within the Figma agent; it has no local output files of its own.
