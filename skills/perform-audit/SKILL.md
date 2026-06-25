---
name: perform-audit
description: >
  Audits Figma screens and delivers a structured UX audit covering usability, UI pattern appropriateness, and content quality — with every issue prioritised by severity. Use this skill whenever the user wants to audit screens, review flows, check usability, validate UI patterns, inspect content/copy, or get recommendations on Figma frames or artboards. Triggers on phrases like: "audit these screens", "review my flow", "check usability", "does this make sense", "review my UI", "give me UX feedback", "is the content ok", "scan my screens", or any request to evaluate multiple frames/artboards at once. Operates on the screens already in context — the current selection or open Figma file.
---

# Overview

Audits one or more Figma screens — the current selection or open file, accessed live through the host's Figma context — and produces a structured UX audit: usability, UI pattern validity, content quality, and prioritised recommendations.

This skill operates on the live file: it's not waiting for the user to manually export or paste images. If it's unclear which frames/screens the user means (e.g. nothing selected, or multiple candidate flows on the canvas), ask which ones to audit rather than guessing the scope.

# Invocation

## Step 1 — Identify the Screens

Determine which screens to audit from the live context: the current selection if one exists, otherwise the open file's relevant frames. If the scope is ambiguous — nothing selected and the file has multiple unrelated flows, or the user named a flow that doesn't obviously map to what's selected — ask which frames/screens they mean rather than guessing:

> "I see [N] frames in the current selection/file — should I audit all of them, or just [specific flow]?"

Also ask (if not already clear):
- **Platform** — Web, iOS, Android, or other? (affects convention checks)
- **Context** — What is this flow trying to help the user accomplish?
- **Screen type** — Product/app screens (logged-in flows, dashboards, forms) or marketing/landing pages? (determines which Lens 2 rows apply — see [References → Audit Lens 2](#audit-lens-2--ui-pattern--structure-validity))

These questions help calibrate the audit. Beyond this fixed set, **ask whenever something you need for an accurate assessment is genuinely ambiguous** from the canvas alone — e.g. unclear what a control is meant to do, unclear if a flow is the full journey or a fragment, unclear target audience or locale for tone/dialect checks. Don't guess on anything that meaningfully changes a verdict; a clarifying question is cheaper than a wrong finding. Only fall back to stating an assumption if the user has signaled they're in a hurry or unreachable for follow-up.

> **Multi-screen audit**: analyse each screen individually first (Step 2), then look across all screens together for flow coherence (Step 3), then write the combined report (Step 4).

**Check input quality before analysing.** Live access doesn't guarantee a clean view — a frame can still render as effectively low-fidelity for audit purposes: instances with missing overrides, detached/broken components, content cut off by a frame boundary, or layers hidden/clipped in a way that hides real content. Say so up front and treat any findings from an affected screen as lower-confidence rather than hard facts.

**Check for mixed fidelity or mixed platform.** If the batch mixes wireframes with high-fidelity screens, or mobile with desktop, calibrate expectations per screen instead of applying one uniform bar — e.g. don't flag a low-fi wireframe for missing hover states or final copy. Note the mix at the top of the report so the user knows severity was calibrated per screen.

## Step 2 — Analyse Each Screen

Work through every checklist in [References → Audit Lenses](#audit-lens-1--usability) for each screen. **Only surface issues actually present** — skip clean items rather than padding the report. Tag every issue you find with a severity label (see [References → Severity Guide](#severity-guide)) as you identify it.

The checklists are exhaustive on purpose: run down each one row by row so nothing is missed, but report only the rows that actually fail. Don't list passing rows in the final report.

## Step 3 — Cross-Screen Flow Analysis (2+ screens only)

Skip this step if only one screen is provided. Work through the [References → Cross-Screen Checklist](#cross-screen-checklist).

## Step 4 — Deliver the Audit Report

Fill in the [References → Report Format](#report-format) template using the severity labels from the [Severity Guide](#severity-guide). Don't invent extra sections beyond the template; omit sections the template marks as conditional when they don't apply.

## Step 5 — Offer Follow-ups

After delivering the report, offer the options in [References → Follow-up Menu](#follow-up-menu).

## Notes for Good Audits

- **Be specific**: "the blue 'Continue' button at the bottom of the form" not "a button".
- **State assumptions**: if the platform or context wasn't provided, state what you assumed and why.
- **Don't invent issues**: only flag things clearly visible on the canvas. When something is ambiguous, mark it "worth verifying" rather than a hard finding.
- **Calibrate severity honestly**: not every issue is Critical. Reserve 🔴 for genuine blockers.
- **Separate UX from visual design**: spacing tokens, colour variables, and component detachment are visual/DS concerns — flag them only if they directly harm usability or comprehension.
- **Acknowledge what's good**: a one-sided report is less useful and less trusted.

# References

### Audit Lens 1 — Usability

| # | Check | What to look for |
|---|-------|-----------------|
| 1 | Affordance | Interactive elements (buttons, links, inputs) that look identical to static text; clickable things with no visual cue (no underline, button shape, or color); static things styled to look clickable |
| 2 | Primary action clarity | More than one element competing to be "the" primary button; no visually dominant CTA; primary and secondary actions weighted equally |
| 3 | Loading state | No spinner, skeleton, or progress indicator for any data-fetching or async action; user left unsure whether the system is working |
| 4 | Empty state | Lists, tables, dashboards, or search results with no designed zero-data state (no guidance, no "add your first…" prompt) |
| 5 | Error state | No visible way to show form errors, failed actions, or system errors; errors not tied to the field that caused them |
| 6 | Success/confirmation feedback | Destructive or important actions with no confirmation that they succeeded (saved, sent, deleted) |
| 7 | Hover / focus / pressed states | Interactive elements with no apparent state change on interaction (especially relevant for web/keyboard users) |
| 8 | Navigation orientation | Unclear where the user is (no active nav indicator, no breadcrumb, no step indicator in a multi-step flow) |
| 9 | Forward path | Any screen where the next step / primary action is unclear or hard to find |
| 10 | Back / escape path | No way to go back, cancel, or close (modals without a close affordance, flows with no back button) |
| 11 | Error prevention | Destructive or irreversible actions (delete, pay, submit) with no confirmation, no undo, and no disabled/guarded state |
| 12 | Cognitive load | Screen trying to communicate more than ~3 primary ideas at once; too many competing actions; dense walls of controls |
| 13 | Information chunking | Related items not grouped; unrelated items grouped together; no visual hierarchy to scan by |
| 14 | Touch / click targets | Tappable areas below ≈44×44 px (touch) or ≈24×24 px (cursor); targets crowded together with no spacing |
| 15 | Contrast | Text or icons that appear low-contrast against their background (gray-on-gray, light text on photos) |
| 16 | Icon legibility | Icons used alone with no label or tooltip where their meaning is ambiguous |
| 17 | Reading order | Visual order that fights the logical order; eye doesn't land on the most important thing first |
| 18 | Interruptions | Modals, popups, or interstitials that block the task without being genuinely urgent |

### Audit Lens 2 — UI Pattern & Structure Validity

> Rows 1–13 apply to **product/app screens**. Rows 14–17 are **marketing/landing-page** specific (hero clarity, section narrative, redundancy framing, final CTA) — only run them if the screen type from Step 1 is marketing/landing, or skip and note why if it's clearly a product screen. Row 18 (responsive risk) applies to both. If screen type wasn't established and it's not obvious from the image, ask rather than guessing which set applies.

| # | Check | What to look for |
|---|-------|-----------------|
| 1 | Component fit | Wrong control for the job — checkbox where single-select is needed (use radio), dropdown for 2 options (use segmented/toggle), free text where a picker is safer (dates) |
| 2 | Single vs multi select | Radios used for multi-select, or checkboxes used for mutually exclusive choices |
| 3 | Tabs vs steps | Tabs used for sequential steps (should be a stepper/wizard), or a wizard used for peer-level views (should be tabs) |
| 4 | Modal appropriateness | Modal used for non-urgent content, long forms, or content that deserves its own page; nested modals |
| 5 | Platform conventions | Controls/gestures/navigation that don't match the target platform (iOS HIG / Material 3 / Web) — e.g. Android back patterns on iOS, hamburger where a tab bar fits |
| 6 | Pattern consistency | The same task solved with different patterns on different screens (e.g. one screen uses inline edit, another a modal) |
| 7 | Progressive disclosure | Everything shown at once when advanced options could be hidden; or critical info buried behind too many clicks |
| 8 | Form field labels | Placeholder-only fields with no persistent label; labels disappear on input |
| 9 | Form field order | Illogical order (e.g. asking for shipping before product); related fields scattered |
| 10 | Inline validation | No real-time / inline validation; errors only shown after submit; no format hints for constrained fields (password rules, phone format) |
| 11 | Required vs optional | No indication of which fields are required; inconsistent marking |
| 12 | Edge & data states | Missing designs for zero-result, error, first-use, loading, and "too much data" (overflow/pagination) states |
| 13 | Destructive action treatment | Delete/remove not visually differentiated (not red/secondary), placed too close to safe actions |
| 14 | Hero / above-the-fold clarity | (Marketing/landing) Hero doesn't say what it is, who it's for, and what to do next within ~5 seconds |
| 15 | Section narrative | (Marketing/landing) Sections feel like a random pile rather than a sequence (Problem → Solution → Proof → Offer → Objection → CTA) |
| 16 | Redundancy | Same element/section repeated on one screen with no clear reason |
| 17 | Final CTA / dead end | Long page or flow that ends without a clear next action before the footer |
| 18 | Responsive risk | Layouts likely to break at smaller widths; fixed-width elements; paragraphs too long for mobile (>3–4 lines) |

### Audit Lens 3 — Content Quality

| # | Check | What to look for |
|---|-------|-----------------|
| 1 | Lorem ipsum / dummy text | Any "lorem ipsum", "dolor sit amet", or placeholder Latin still present |
| 2 | Placeholder data | Zeroed stats (0, 0%, +0), "Name here", "Title here", "Description", "TBD", "Coming soon", obvious fake avatars/logos left in |
| 3 | Spelling errors | Typos, missing accents, wrong/transposed characters |
| 4 | Grammar errors | Wrong verb conjugation, missing prepositions, subject–verb disagreement, wrong gender/number agreement |
| 5 | Dialect / locale consistency | Mixed variants (US vs UK English — "color"/"colour", "realize"/"realise"; or any inconsistent regional spelling/term across screens) |
| 6 | Tone consistency | Screens that don't sound like the same brand/voice; one screen formal, another casual with no reason |
| 7 | Unintended repetition | Same sentence or phrase reused across screens where it reads as an error, not intent |
| 8 | Weak / generic copy | Vague value props and filler that could apply to any product ("best-in-class", "seamless solution") |
| 9 | Incomplete content | Headings with no body, sections that feel half-finished, truncated sentences |
| 10 | Orphaned text | Floating labels/phrases with no clear parent section or context |
| 11 | Inconsistent terminology | Same concept named differently across screens (e.g. "Workspace" vs "Project" vs "Board" for one thing) |
| 12 | Number consistency | Stats that contradict each other across screens (e.g. "1,400 hours" on one, "1,200 hours" on another) |
| 13 | Clarity / jargon | Copy a first-time user wouldn't understand; internal jargon, acronyms, system terms exposed |
| 14 | Action-oriented CTAs | Vague button labels ("Submit", "OK", "Click here") instead of outcome labels ("Save changes", "Send invoice") |
| 15 | Content hierarchy | No clear headline → sub-headline → body → CTA structure; multiple equally-weighted headings |
| 16 | Truncation risk | Strings likely to overflow at larger text sizes or in longer locales; text already clipped with "…" |
| 17 | Helpful placeholder text | Placeholder that just echoes the label ("Email" in an Email field) instead of showing format/example |
| 18 | Humane microcopy | Error/helper text that exposes raw codes or system strings ("Error 422") instead of plain, actionable guidance |

### Cross-Screen Checklist

> Rows 2 (step count/friction) and 10 (scope creep) assume a task-oriented product flow — if the screens are a marketing/landing sequence (e.g. homepage → pricing → about), judge them against a persuasion narrative instead (does each page build the case logically?) rather than literal step count.

| # | Check | What to look for |
|---|-------|-----------------|
| 1 | Logical progression | Moving A → B → C has gaps or jumps; a step seems missing; order doesn't match the user's mental model |
| 2 | Step count / friction | More steps than the task needs; fields or screens that could be merged or removed |
| 3 | Nav placement consistency | Navigation, headers, or tab bars move position or restyle between screens |
| 4 | Button / control consistency | Same action styled differently across screens (color, label, placement of the primary CTA) |
| 5 | Type & spacing consistency | Headings, body, and spacing treatments drift between screens for the same role |
| 6 | Terminology consistency | The same object/action named differently from screen to screen |
| 7 | Dead ends | Any screen with no clear next action, no exit, or no path back into the main flow |
| 8 | Back navigation & data preservation | Going back loses entered data or loses the user's place in the flow |
| 9 | State carry-over | Selections/filters/cart contents not preserved when moving between screens |
| 10 | Scope creep | A screen tries to do far more (or less) than its peers, breaking the flow's rhythm |
| 11 | Entry/exit completeness | Flow has no clear start (onboarding/empty entry) or no clear completion (success/confirmation) screen |

### Severity Guide

| Label | Meaning |
|-------|---------|
| 🔴 **Critical** | Blocks the user, causes data loss, or creates major confusion. Must fix before shipping. |
| 🟠 **High** | Significantly hurts usability or breaks established conventions. Fix before handoff. |
| 🟡 **Medium** | Inconsistency, unclear copy, or pattern mismatch. Fix in current iteration. |
| 🟢 **Low** | Polish or nice-to-have. Address when time allows. |

### Report Format

```
# UX Audit — [Flow / Screen Name]
[N] screen(s) reviewed

**Platform:** [Web / iOS / Android — note "(assumed)" only if not stated by the user]
**Flow context:** [What this flow helps the user do]
**Fidelity/input notes:** [Only include if input quality was mixed or low — e.g. "Screen 3 is a low-fi wireframe, calibrated accordingly" or "Screen 2 screenshot is cropped, findings there are lower-confidence"]

---

## Screen-by-Screen Findings

### Screen [N]: [Descriptive name, e.g. "Login", "Checkout step 2"]

#### 🔴 Critical
**[Short issue title]**
- **Where:** [Specific area — e.g. "primary CTA button", "email field", "top nav"]
- **Problem:** [Clear, concise description of the issue and why it matters]
- **Recommendation:** [Specific, actionable fix]

#### 🟠 High
**[Issue title]**
- **Where:** ...
- **Problem:** ...
- **Recommendation:** ...

#### 🟡 Medium
... (same structure)

#### 🟢 Low
... (same structure)

*(Omit any severity level that has no issues for this screen)*

---

### Screen [N+1]: [Name]
...

---

## Cross-Screen Issues
*(Only include if ≥ 2 screens were reviewed)*

Same severity structure as above.

---

## ✅ What's Working Well

- [Specific strength — be concrete, not generic]
- [Specific strength]

---

## 📋 Priority Action List

Fix in this order:

1. 🔴 [Issue title] — Screen: [name]
2. 🟠 [Issue title] — Screen: [name]
3. 🟡 [Issue title] — Screen: [name]
4. ...

---

## 💡 Recommendations

[2–4 higher-level suggestions beyond individual issues. Examples: "Define an error state component library to ensure consistency", "Run a 5-second test on the dashboard to validate hierarchy", "Add skeleton loading states across all data-heavy screens."]
```

### Follow-up Menu

| If the user says… | Do this |
|-------------------|---------|
| "Focus on content / copy" | Re-run Audit Lens 3 in depth; produce rewritten versions of flagged strings |
| "Focus on accessibility" | Deep-dive on contrast, touch targets, label coverage, and reading order |
| "Compare to [platform] conventions" | Evaluate specifically against iOS HIG / Material 3 / WCAG 2.2 AA |
| "What should I fix first?" | Return only Critical + High items with a brief rationale for the order |
| "Rewrite the microcopy" | Produce revised copy for all flagged labels, CTAs, and error messages |
| "Show me the full checklist" / "what did you check" | Return every row from the Audit Lens / Cross-Screen checklists per screen marked ✅ pass or ❌ fail, not just the failures — useful for audit-coverage proof |

# Assets

None — this skill produces a chat-delivered report only, with no output files.
