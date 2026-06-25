---
name: clean-layers
description: >
  Scans the selected frame(s) in a Figma file for structural cleanup opportunities — deletable hidden layers, redundant single-child auto layout wrappers that can be flattened, and fixed-width layers that should be fill-width — and reports every finding before changing anything. Requires one or more frames to be selected first; if nothing is selected, asks the user to select a frame. Use this skill whenever the user wants to tidy up, simplify, clean up, or streamline a frame's layer structure, asks "can this be flattened", "is this autolayout redundant", "are there hidden layers I can delete", "should this be fill width instead of fixed", or wants a structural housekeeping pass on a component or frame before handoff. This is a query-then-mutation skill — it only mutates the file after the user confirms which findings to fix.
---

# Overview

Audits the node tree of the currently selected frame(s) for three structural cleanup patterns — deletable hidden layers, collapsible single-child auto layout chains, and fixed-width children that match their parent's content width — and lists every finding for the user to confirm before applying any fix. Reads the live selection directly to find candidates, then writes only the changes the user approves.

# Invocation

## Step 1 — Verify the Selection

This skill needs a concrete starting point: one or more **frames** selected in the file. If nothing is selected, or the selection is a layer that isn't a frame (e.g. a single text node or vector), ask the user to select the frame(s) to clean up rather than guessing scope from the whole page — a page can contain many unrelated frames, and silently auditing all of them risks reporting findings the user didn't ask about.

If multiple frames are selected, scan each one and report findings grouped by frame.

## Step 2 — Scan for Cleanup Opportunities

Walk the node tree of each selected frame and run all three checks in [References → Cleanup Checks](#cleanup-checks). Do this read-only — nothing is written to the file in this step.

Skip a check entirely for a given subtree rather than guessing when the underlying data isn't available (e.g. prototype interaction/reaction data isn't exposed by the tools you have access to) — say so once in the report rather than silently treating "I couldn't check" the same as "this is clean."

## Step 3 — Report Findings and Confirm

**Always list findings before changing anything.** Group the report by the three categories in References, each item identified by layer name and its path from the selected frame (e.g. `Card > Content > Label`) so the user can locate it without you having made any change yet. Use the [References → Report Format](#report-format) template.

End the report by asking which findings to act on — all of them, by category, or a specific numbered subset. Don't assume "report it" means "fix it"; wait for explicit confirmation, since every finding here is a mutation across the live file.

## Step 4 — Apply Confirmed Fixes

Apply only the findings the user confirmed, via `use_figma` calls, following the resolution rules in [References → Cleanup Checks](#cleanup-checks) — particularly the conflict-handling rule for collapsing auto layout chains (Check 2): if you hit a conflict you can't resolve automatically, stop on that item, ask, and continue with the rest rather than guessing or aborting the whole batch.

After applying, report a summary: what changed (counts per category), what was confirmed but turned out to need a manual call mid-way (and why), and what's still outstanding because it wasn't in the confirmed set.

## Notes for Good Cleanups

- **Bias toward lossless.** Checks 1 and 2 should only ever be flagged when applying them produces no visible or behavioral difference. If you're not sure a specific instance is truly lossless (e.g. a wrapper frame has a property you can't fully account for), don't flag it as cleanly fixable — surface it as "needs manual review" instead of silently skipping or silently applying.
- **Component instance internals are off-limits.** Never flag or touch a layer whose ancestor chain includes a component instance — that subtree is governed by the component, and bulk edits there create overrides that drift from the source component in ways that are easy to create and tedious to find later.
- **A hidden layer might not be dead.** Prototyping interactions can target a hidden layer (e.g. "show on click"). Treat a hidden layer that's a known interaction target as not safe to delete, the same way you'd treat one inside a component instance — and if you can't determine whether a hidden layer is an interaction target with the tools available, say that limitation out loud rather than implying you checked.
- **Don't auto-resolve genuine conflicts.** If two wrapper frames in a chain being collapsed (Check 2) each set a real, different value for the same property, that's a judgment call about which one the user actually wants — flag it for manual review rather than picking one silently.

# References

### Cleanup Checks

| # | Check | What to look for |
|---|-------|-----------------|
| 1 | Deletable hidden layers | A layer with `visible = false`. Exclude any hidden layer whose ancestor chain includes a component instance, and exclude any hidden layer that's a target of a prototype interaction (e.g. a "show/hide" or "change to" reaction) if that's detectable with the tools available — note in the report if it isn't detectable, rather than treating undetected as safe. |
| 2 | Redundant single-child auto layout chain | A chain of two or more auto layout frames where each frame's only child is itself an auto layout frame, down to the innermost frame that actually holds the real (non-wrapper) children. Flag this for collapsing into a single auto layout frame **only if lossless**: the merged frame should take its layout properties (direction, gap, padding, alignment, sizing modes) from the **innermost** frame in the chain, since that's what determines how the real children are actually laid out — and should carry forward any visual property (fills, strokes, effects, corner radius, opacity, rotation, clipping) that any wrapper in the chain set to a non-default value. If two or more wrappers in the same chain set a real, *different* value for the same property, don't auto-resolve it — flag the chain as "needs manual review: conflicting `<property>` values" instead of collapsing it. |
| 3 | Fixed-width child matching parent's fill width | A direct child of an auto layout frame with `layoutSizingHorizontal = FIXED` and an explicit width equal to that parent's current available content width (parent width minus horizontal padding). Flag as "could be Fill width instead of Fixed" — note this is a robustness suggestion, not a visual fix: the rendered result is identical today, but Fixed won't track the parent if it resizes later, where Fill would. |

### Report Format

```
# Layer Cleanup — [Frame name(s)]

## [Frame name]

### 🗑️ Deletable Hidden Layers ([count])
1. `[Layer path]` — [one-line reason if non-obvious, e.g. "hidden, no instance ancestor, no detected interaction target"]
2. ...
*(omit this section if none found)*

### 🧱 Redundant Auto Layout Chains ([count])
1. `[Outer wrapper path]` → collapses to a single frame using [innermost frame]'s layout + [carried-over properties, if any]
2. `[Chain path]` — needs manual review: conflicting `[property]` values between `[wrapper A]` and `[wrapper B]`
*(omit this section if none found; list manual-review items even if no auto-collapsible ones exist)*

### 📏 Fixed-Width → Fill-Width Candidates ([count])
1. `[Layer path]` — currently Fixed at [width]px, matches parent's content width
2. ...
*(omit this section if none found)*

---

[If nothing was found in any category for a frame: "No cleanup opportunities found in [frame name]."]

**What would you like me to fix?** (all / by category / specific numbered items)
```

### Follow-up Menu

| If the user says… | Do this |
|-------------------|---------|
| "fix everything" / "apply all" | Apply every confirmable finding across all three categories, in the order: hidden layers, then auto layout collapses, then fill-width changes |
| "just the hidden layers" | Apply only Check 1 findings |
| "skip the autolayout ones, too risky" | Apply Checks 1 and 3 only, leave Check 2 findings untouched and say so |
| "what about the conflicting ones" | List the manual-review items from Check 2 with their specific conflicting property values, and ask which value to keep for each |
| "check this for other frames too" | Re-run Step 1–3 against a newly specified selection rather than assuming the original selection still applies |

# Assets

None — this skill reads and writes the live Figma file directly within the Figma agent; it has no local output files of its own.
