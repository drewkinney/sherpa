keywords: haiku, haiku-4-5, claude-haiku
provider: anthropic
transforms: explicit-directives, simplify-language, hardened-constraints, tighten-steps

## Why these transforms

Haiku 4.5 is the fast/smaller-footprint model in the Claude family. Per Anthropic's own prompt-engineering guidance, capability tier correlates with how explicit and scaffolded a prompt needs to be — smaller/faster models benefit from rigid, spelled-out structure and get less value from implicit inference than the flagship model does.

## How to apply

- **explicit-directives**: Convert every instruction into a numbered, imperative step. Replace any suggestion-toned phrasing ("consider," "you might") with a direct command.
- **simplify-language**: Replace nested clauses and multi-part sentences with short, plain-vocabulary sentences. One idea per sentence.
- **hardened-constraints**: State every constraint as "Do X. Do not Y. Only accept Z." Do not leave any constraint implied by context alone.
- **tighten-steps**: Where the rebuild produced a multi-step METHOD or process section, compress it to the fewest steps that still cover every required action — merge adjacent steps that don't need separate numbering.
