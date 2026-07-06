keywords: fable, fable-5, claude-fable
provider: anthropic
transforms: hardened-constraints, explicit-directives, tighten-steps

## Why these transforms

Fable 5's native prompt-processing style is sparse and rigid, not rich and flexible — this is the documented behavioral pattern the Fable Five discipline technique (see discipline/fable-five.md) was itself modeled on. Fable 5 treats every sentence as either a command, a constraint, or a definition, and does not benefit from — and can be actively hurt by — added context-setting or rhetorical scaffolding.

## How to apply

- **hardened-constraints**: Every constraint stated as "Do X. Do not Y. Only accept Z." with zero filler words in between. Constraint density matters — every sentence must carry direct load.
- **explicit-directives**: Numbered, gated reasoning steps, not prose-flowing paragraphs. Name variables and exact conditions in caps or code (e.g. `{FORMAT}`, `{REJECTION_CASE}`) rather than describing them in prose.
- **tighten-steps**: Compress any multi-step method to its minimal numbered form.

Do NOT apply `add-section` or `leverage-context` to this model — minimal context-setting is Fable 5's actual preference, not a gap that needs filling. Adding rationale/background sections works against this model rather than for it.
