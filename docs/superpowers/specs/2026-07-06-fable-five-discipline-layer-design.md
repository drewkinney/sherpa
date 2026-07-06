# Fable Five Discipline Layer — Design Spec

Piece 1 of 3 in the Sherpa 2.0 rebuild (discipline → variants → routing/matrix). Markdown-only, no JS — mirrors how `formats/` already works: a spec file Claude reads and applies directly, nothing executable.

## Context

Sherpa 1.0 (this repo, current state) is a single skill (`SKILL.md`) that rebuilds a raw prompt into a chosen structured format via `formats/*.md` spec files. The rebuild step already resolves ambiguity, fills structural slots, and cuts hedging/filler — a soft tightening pass.

Sherpa 2.0's handoff doc (`handoff/sherpa-2-handoff.md`) proposed a 3-layer architecture (Format → Discipline → Variant) plus user-level routing and a capability matrix, all built as JS modules (`core/*.js`, `variants/*.js`). That JS architecture doesn't match reality: no executable code exists anywhere in this repo today. This spec keeps the same all-markdown pattern as `formats/` and scraps the JS scaffold entirely.

This spec covers only the Discipline layer (Fable Five). Variants and routing/matrix are separate specs, built after this one lands.

## What Fable Five Is

A constraint-discipline rewrite technique (name is unrelated to the Fable 5 model — confirmed with user). Core directives:

1. **Flatten instruction hierarchy** — one command per sentence minimum, no embedded clauses hiding directives
2. **Name constraints explicitly** — "Do X. Do not Y. Only accept Z," not "try to be mindful of Z"
3. **Kill context drift** — every sentence either commands, constrains, or defines; no rhetorical scaffolding
4. **Variable clarity** — anything referenced gets named in caps or code: `{FORMAT}`, `{REJECTION_CASE}`
5. **Outcome specification first** — state what success looks like before method

Rewrite engine steps: parse the input for hidden directives, soft constraints, and implicit context → restructure with the 5 directives above → surface a delta (what got compressed, what got surfaced, what hierarchy changed).

Fable Five is model-agnostic — it's a prompt-quality technique, not tuned to any specific model's quirks (that's the Variant layer's job, piece 2).

## Architecture

Discipline is a second pass, applied on top of the existing format rebuild — not a replacement for it.

```
raw prompt
   │
   ▼
[Format rebuild]   (unchanged — SKILL.md today)
   │
   ▼
[Fable Five pass]  (new — discipline/fable-five.md)
   │  skipped if "format-only" keyword present
   ▼
formatted + disciplined output
   │
   ▼
delta line appended (always shown)
```

Runs once per format when `all` is requested — each format gets its own discipline pass and its own delta line, consistent with how `all` already handles per-format costing today.

## New File: `discipline/fable-five.md`

Same three-part pattern as a `formats/` spec file:

- **keywords** — none needed for triggering (see Default Behavior below); this field exists only so `format-only` has something to reference as "the thing being skipped"
- **applies_to** — every rebuild, all formats, always (unless skipped)
- **spec** — the 5 directives verbatim (as listed above) plus the rewrite-engine steps, written as instructions for Claude to apply to the already-formatted output, not as prose description

## SKILL.md Changes

1. **Invocation parsing** — add `format-only` to the reserved keyword set alongside `all`. It is never treated as raw prompt text, same rule as any format keyword.
2. **Default behavior** — Fable Five runs automatically after every rebuild, no opt-in keyword needed. `format-only` is the sole opt-out.
3. **Output** — directly under each format's code block (where the "pastes best here" line already lives), add one short delta line: what got compressed, what got surfaced, what hierarchy changed. Keep it one line, matching the existing "keep commentary between blocks minimal" rule — not a full essay per rebuild.

## Interaction With Existing `all` Cost Estimate

The `all` mode's token-cost warning (SKILL.md today) estimates format-layer cost only. Discipline pass adds a small fixed overhead per format (a rewrite pass + one delta line) — this spec does not change the cost-estimate step itself; that's a candidate follow-up if estimates drift noticeably once this ships, not part of this build.

## Testing / Validation

Manual check once implemented: run `/sherpa "some rambling prompt with buried constraints" markdown`, confirm:
- Output slot-filling behaves exactly as Sherpa 1.0 does today (no regression)
- Formatted output additionally shows flattened hierarchy, explicit named constraints, no soft-scaffolding language
- One delta line appears under the code block
- Re-run with `/sherpa "..." markdown format-only` — confirm discipline pass is skipped, delta line absent, output matches pre-2.0 behavior exactly

## Out of Scope (This Spec)

- Model variant transforms (piece 2)
- User-level routing, prompt-type pills, capability matrix (piece 3)
- Any JS runtime, registry, or discovery mechanism — explicitly rejected per user direction toward markdown-only
