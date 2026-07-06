keywords: sonnet, sonnet-5, claude-sonnet
provider: anthropic
transforms: add-section, moderate hardened-constraints

## Why these transforms

Sonnet 5 is the balanced mid-tier Claude model — per Anthropic's prompt-engineering guidance, it sits between Haiku's need for rigid scaffolding and Opus's ability to run on implicit reasoning. Some structure helps; over-hardening every constraint wastes effort the model doesn't need spent on it.

## How to apply

- **add-section**: If the rebuild's Context or Task slot is thin, add a short RATIONALE note explaining why the request matters — enough to anchor the model's reasoning, not a full brief.
- **moderate hardened-constraints**: State the 1-2 constraints that most affect correctness explicitly ("Do X. Do not Y."). Leave lower-stakes preferences (tone, minor formatting nuance) as normal prose rather than converting every line to hardened Do/Do-not phrasing.
