keywords: gemini, gemini-3, google-gemini
provider: google
transforms: add-examples, leverage-context, explicit-directives

## Why these transforms

Sourced from Google's own Gemini API prompt design guidance (ai.google.dev/gemini-api/docs/prompting-strategies, checked 2026-07-06), which documents current recommendations for Gemini 3 models and Gemini 2.5 Flash. The page states plainly: "We recommend to always include few-shot examples in your prompts" — examples are treated as a default, not an edge-case aid. It also instructs authors to "add context" rather than assume the model already has it, and for Gemini 3 specifically to be "precise and direct," use one consistent structural delimiter style (XML tags or Markdown headings, not mixed), and put critical instructions first. The page also recommends breaking complex prompts into smaller chained/parallel steps and leaving temperature/topK/topP at default values for Gemini 3.x — those two points don't map cleanly onto this file's three transform picks and are noted here rather than forced into a tag.

This is a documented-guidance file, not an inferred judgment call — unlike a fallback reasoned purely from capability tier.

## How to apply

- **add-examples**: Include at least one concrete input→output example pair illustrating the desired response shape, even when the task doesn't strictly require one — Google's guidance treats few-shot examples as a default inclusion, not something to add only when the model seems to be struggling.
- **leverage-context**: State background and reader intent explicitly in the Context slot rather than assuming the model will infer it — who's reading this, why it matters, what happens next. Do not rely on implicit understanding for anything load-bearing.
- **explicit-directives**: Use precise, direct phrasing with no persuasive or hedging language. Pick one structural delimiter convention (Markdown headings or XML tags) and hold it consistently through the whole prompt — don't mix. Order the Constraints/instructions so the single most critical requirement comes first, not buried among lower-stakes ones.

## Note

Model naming in this space moves fast and this file was written from a single lookup — re-verify against current Google documentation before treating this as durable guidance, the same caution Fable 5's file applies to its own inferred edges.
