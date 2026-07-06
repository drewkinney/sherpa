applies_to: every rebuild, every format, always — unless "format-only" keyword present
skip_keyword: format-only

## Fable Five discipline pass

Apply this pass to the already-formatted rebuild output, after the format spec has been fully applied. Do not run this pass on the raw input prompt — it operates on the formatted result.

Rewrite engine steps, in order:
1. Parse the formatted output for: hidden directives (instructions buried inside descriptive sentences), soft constraints (hedged language like "try to," "be mindful of," "ideally"), and implicit context (assumptions the reader must infer rather than being told).
2. Restructure using these 5 directives:
   - **Flatten instruction hierarchy** — one command per sentence minimum. No embedded clauses hiding directives.
   - **Name constraints explicitly** — "Do X. Do not Y. Only accept Z." Never "try to be mindful of Z."
   - **Kill context drift** — every sentence either commands, constrains, or defines. No rhetorical scaffolding.
   - **Variable clarity** — anything referenced gets named in caps or code: `{FORMAT}`, `{REJECTION_CASE}`.
   - **Outcome specification first** — state what success looks like before method.
3. Surface the delta: what got compressed, what got surfaced (a hidden directive made explicit), what hierarchy changed (a buried clause promoted to its own sentence).

This pass is model-agnostic — apply it identically regardless of which model the prompt is destined for. Do not add model-specific tuning here.
