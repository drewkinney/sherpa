# Model Variant Layer — Design Spec

Piece 2 of 3 in the Sherpa 2.0 rebuild (discipline → **variants** → routing/matrix). Markdown-only, no JS — same pattern as `formats/` and `discipline/fable-five.md`: spec files Claude reads and applies directly, nothing executable.

## Context

Piece 1 (`discipline/fable-five.md`, already shipped on this branch) added a model-agnostic tightening pass that runs on every rebuild by default. This piece adds a third, model-*specific* pass on top: given a target model, apply the transforms that model actually responds best to.

Sherpa 2.0's handoff doc (`handoff/sherpa-2-handoff.md`) proposed 20+ model configs as JS files with 8 named transform tags, but never defined what those tags do, and its model version names are stale (`opus-4-6`, `gpt-4o`, `gemini-2-pro`, `grok-3` don't match any real current model). This spec fixes both problems: only builds files for models with real, sourced prompting differences, and defines every transform tag concretely.

## Architecture

Variant is a third pass, chained after discipline:

```
raw prompt
   │
   ▼
[Format rebuild]     (unchanged)
   │
   ▼
[Fable Five pass]    (piece 1 — skippable via "format-only")
   │
   ▼
[Variant pass]       (new — this piece, no skip keyword)
   │
   ▼
formatted + disciplined + variant-tuned output
```

Variant pass always runs — it has no `format-only`-style skip, because "which model's quirks to tune for" isn't something a user needs to opt out of; it's just a question of which model.

**Target resolution, in order:**
1. A named model keyword in the message (e.g. `opus`, `sonnet-5`, `haiku`, `fable`, `gemini`, `grok`) — scanned the same way format keywords are, added to the same reserved-keyword set.
2. If none named: default to whatever model is currently running the skill (Claude self-detects its own model ID at runtime — no hardcoded fallback model).

## Transform Tags

Eight tags, each a concrete instruction, not a vague label:

| Tag | What it does |
|---|---|
| `add-section` | Add a RATIONALE/CONTEXT block before the ask — for models that reason well from background rather than needing everything spelled out |
| `tighten-steps` | Compress multi-step METHOD instructions into fewer, denser steps |
| `shorten-sentences` | Enforce sentence/word-length limits |
| `add-examples` | Include concrete input→output example pairs |
| `hardened-constraints` | Explicit "Do X. Do not Y. Only accept Z." phrasing, no soft hedges |
| `leverage-context` | Deliberately use more background/detail — for models that use long context well rather than getting diluted by it |
| `simplify-language` | Plain vocabulary, no nested clauses |
| `explicit-directives` | Numbered imperative steps, no suggestion-toned language |

## Model Files (Anthropic Family — Built Now)

Grounded in Anthropic's own published prompt-engineering guidance: capability tier correlates with how explicit/scaffolded a prompt needs to be. Not guessed.

- `variants/anthropic/haiku-4-5.md` — fast/smaller model: `explicit-directives`, `simplify-language`, `hardened-constraints`, `tighten-steps`. Needs rigid, spelled-out structure; less benefit from implicit inference.
- `variants/anthropic/sonnet-5.md` — balanced model: `add-section`, moderate `hardened-constraints`. Middle ground — some scaffolding helps, some inference can be trusted.
- `variants/anthropic/opus-4-8.md` — flagship model: `add-section`, `leverage-context`. Handles nuance and implicit reasoning well; benefits more from rich context than from constraint-hardening.
- `variants/anthropic/fable-5.md` — sparse/rigid native style (per Fable Five discipline's own source research, which was modeled on this model's behavior): `hardened-constraints`, `explicit-directives`, `tighten-steps`. Explicitly NOT `add-section` / `leverage-context` — minimal context-setting is this model's actual preference, not a gap to fill.

Each file states its transform picks as the concrete rule to apply — not a restatement of the table above, but the actual instruction: e.g. Haiku's file says "compress every multi-step instruction to its shortest imperative form," not "apply tighten-steps."

## Other Providers — Discovery, Not Guessed

No files are built now for OpenAI, Google, xAI, Meta, Mistral, or any other provider — building those today would mean guessing stale-prone version names again, the exact problem being fixed. Instead:

**New `SKILL.md` section: Variant discovery.** When a named or self-detected target model has no matching `variants/<provider>/<model>.md` file:
1. Use the WebSearch/WebFetch tool to find that provider's current model name and any published prompting guidance.
2. Apply the same transform-tag pattern (pick from the 8 tags above based on what's found, or reason from general model-capability-tier patterns if no specific guidance exists — flagging that inference as a judgment call inline, same as the Anthropic files do for Fable 5's uncertain edges).
3. Write the resulting file to `variants/<provider>/<model>.md` so future calls skip the lookup.
4. If the web lookup fails or returns nothing usable, apply no variant transforms for that call, state plainly that the target model has no established profile and none could be found, and proceed with format + discipline output only.

This makes the catalog grow lazily and stay accurate — a file only exists once it's backed by something real (either Anthropic's own docs, a provider's own published guidance, or an explicitly-flagged best-effort inference).

## SKILL.md Changes

1. **Invocation parsing** — add model-name keywords to the reserved keyword set. Unlike `all`/`format-only`, this set isn't fixed: it's read from `variants/*/*.md` filenames the same way format keywords are read from `formats/*.md`, plus common aliases (a file can list `keywords: opus, opus-4-8, claude-opus`).
2. **New "Variant pass" section** — runs after the discipline pass, every time. Resolves target model (named keyword, else self-detected), loads its file if one exists, else runs Discovery (above), then applies the file's transforms to the already-disciplined output.
3. **Output** — no new required line by default (unlike discipline's delta line) — but if Discovery had to run (no existing file), state that plainly in one line ("no variant profile existed for X; researched and created one" or "no profile found for X; proceeded without variant tuning"), so the user knows a new file just got written or a lookup came up empty.

## Interaction With Discipline Layer

Variant transforms apply on top of Fable Five's output, not instead of it. Fable Five stays universal (same 5 directives regardless of target); variant adds model-specific shaping after. The two do not contradict in practice — Fable Five operates on hierarchy/constraint-clarity generally, variant transforms add or remove scaffolding/examples/context density on top.

## Testing / Validation

Manual dry run once implemented:
- `/sherpa markdown opus "..."` — confirm output shows `add-section` (rationale/context block) and no forced constraint-hardening beyond what discipline already did
- `/sherpa markdown haiku "..."` — confirm output is more rigidly structured, simpler vocabulary, explicit numbered steps
- `/sherpa markdown fable "..."` — confirm output is sparse, no added context section, dense constraint-per-sentence
- `/sherpa markdown gemini "..."` (no existing file) — confirm Discovery step is described as running (web lookup attempted), and either a new file gets written or a plain "no profile found" statement appears — do not require actual network access to pass this check in an offline test; confirm the *instruction* is followed procedurally
- Re-run `/sherpa markdown gemini "..."` a second time after a file exists — confirm it loads the existing file rather than re-running Discovery

## Out of Scope (This Spec)

- User-level routing, prompt-type pills, capability matrix (piece 3)
- Building out non-Anthropic provider files upfront — those come from Discovery, on demand
- Any JS runtime, registry, or discovery *mechanism* beyond "Claude uses its own WebSearch/WebFetch tool per SKILL.md instructions" — no scheduled/background discovery, no startup scan
