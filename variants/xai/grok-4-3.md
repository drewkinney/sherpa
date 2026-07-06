keywords: grok, grok-4-3, xai-grok

provider: xai

transforms: add-section, leverage-context

## Why these transforms

Sourced from xAI's official developer docs (docs.x.ai/developers/models, docs.x.ai/developers/model-capabilities/text/reasoning, checked 2026-07-06). Confirmed current flagship: grok-4.3 (1M token context, Chat API), described on the models page as "the most intelligent and fastest model we've built" — alongside grok-4.20 reasoning/non-reasoning/multi-agent variants and the separate grok-build-0.1 coding model. The reasoning docs describe an effort-level parameter (none/low/medium/high) mapped to task complexity, but that is an API parameter, not prompt-writing guidance, and doesn't map onto the 8 transform tags.

No xAI documentation was found (official docs pages checked, plus the xai-org/grok-prompts GitHub repo, which is a system-prompt archive with no authoring guidance) that specifies how to structure prompts — no stated position on examples, context density, or delimiter style, the way Google's guidance did for Gemini 3. Per SKILL.md's Discovery step 2 fallback, this pick is reasoned from general capability-tier pattern, not sourced guidance: grok-4.3 is xAI's flagship, general-purpose model, so it leans toward the flagship pattern (add-section, leverage-context) rather than the small/fast-model pattern (explicit-directives, simplify-language, hardened-constraints).

This is an inferred judgment call, not documented guidance — unlike the gemini-3.md file, which cites specific stated Google recommendations.

## How to apply

- **add-section**: Add a RATIONALE/CONTEXT block ahead of the Task slot explaining the underlying goal and any relevant background, rather than only listing bare constraints.
- **leverage-context**: Where the source prompt has more background available than the target format's slots strictly require, include it rather than compressing it away for brevity.

## Note

Re-verify against current xAI documentation before treating this as durable — model naming and doc structure in this space move fast, and no official prompt-structuring guidance existed to confirm this pick against at the time of writing.
