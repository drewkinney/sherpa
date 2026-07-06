keywords: opus, opus-4-8, claude-opus
provider: anthropic
transforms: add-section, leverage-context

## Why these transforms

Opus 4.8 is the flagship Claude model — per Anthropic's prompt-engineering guidance, it handles nuance and implicit reasoning well and gets more value from rich context than from having every constraint spelled out and hardened.

## How to apply

- **add-section**: Add a RATIONALE/CONTEXT block ahead of the Task slot explaining the underlying goal and any relevant background — trust the model to draw the right constraints from that context rather than hardening every one explicitly.
- **leverage-context**: Where the source prompt has more background available (earlier conversation, related specifics) than the target format's slots strictly require, include it. Do not compress away context for brevity's own sake.
