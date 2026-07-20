```markdown
# Sherpa 2.0: Prompt Engineering Platform

Sherpa 2.0 = Sherpa 1.0 (format optimization) + Fable Five (constraint discipline) + Model Variants (auto-generated per model) + User-aware routing.

**One skill. Three layers. 20+ models. User-level filtering.**

## Architecture

```
sherpa-2.0/
├── core/
│   ├── user-detection.js       [User level mgmt, persistence]
│   ├── capability-matrix.js    [Model capabilities per task]
│   ├── prompt-router.js        [Filter models by type + level]
│   ├── format-engine.js        [Sherpa 1.0: applies formats]
│   ├── prompt-analyzer.js      [Fable Five: constraint restructuring]
│   └── orchestrator.js         [Coordinates all three layers]
├── variants/
│   ├── index.js                [Registry & loader]
│   ├── discover.js             [Runtime detection]
│   └── models/
│       ├── anthropic/ [opus-4-6, sonnet-4-6, haiku-4-5]
│       ├── openai/ [gpt-4o, gpt-4-turbo]
│       ├── google/ [gemini-2-pro, gemini-1-5-pro]
│       ├── xai/ [grok-3, grok-2]
│       ├── moonshot/ [kimi-k2-6]
│       ├── nvidia/ [build]
│       ├── meta/ [llama-3-2, llama-3-1]
│       ├── mistral/ [mistral-large, mixtral-8x22b]
│       ├── microsoft/ [phi-3]
│       ├── deepseek/ [deepseek-v3]
│       └── local/ [qwen2]
├── formats/ [Sherpa 1.0 format files]
├── index.js [Main entry point]
├── SKILL.md
├── README.md
├── USER-LEVEL-ROUTING.md
├── CHANGELOG.md
└── package.json
```

## User Level Behavior

**Beginner**
- Sees prompt type pills (Chat, Research, Coding, Images, Videos)
- Must pick a type
- Models shown: Paid mainstream (Claude, GPT-4, Gemini)
- Limit: 5 models

**Intermediate**
- Sees prompt type pills (same as Beginner)
- Must pick a type
- Models shown: Paid + freemium (Kimi, Nvidia Build)
- Limit: 10 models

**Power User**
- No pills — skips straight to all models
- Full 20+ model list, no task-type filtering
- Models shown: Everything (paid, freemium, open, local)
- Limit: None

Set level: `/sherpa --set-level [beginner|intermediate|power-user]`

## Prompt Types (Beginner/Intermediate Pills)

- **Chat** — General conversation, brainstorming
- **Research** — Deep analysis, document writing
- **Coding** — Write and debug code
- **Images** — Generate and analyze images
- **Videos** — Generate and edit videos

## Models (20 Total)

**Paid Frontier** (5)
- anthropic/opus-4-6, sonnet-4-6, haiku-4-5
- openai/gpt-4o, gpt-4-turbo
- google/gemini-2-pro, gemini-1-5-pro
- xai/grok-3, grok-2
- mistral/mistral-large

**Freemium** (2)
- moonshot/kimi-k2-6
- nvidia/build

**Open Source** (8)
- meta/llama-3-2, llama-3-1
- mistral/mixtral-8x22b
- microsoft/phi-3
- deepseek/deepseek-v3
- local/qwen2

## Three Processing Layers

1. **Format** (Sherpa 1.0) — Apply system, chain, structured, user, tool format
2. **Discipline** (Fable Five) — Restructure constraints, flatten hierarchy, increase clarity
3. **Variant** (Model-specific) — Apply transforms optimized for that model

All three run in one call. Pick which layers you need.

## Output Modes

- `full` — All layers + analysis + metrics
- `sparse` — Final prompt only (ready to use)
- `format-only` — Format layer only (Sherpa 1.0 behavior)
- `discipline-only` — Format + discipline, no variant
- `delta-only` — Changes + metrics only

## Usage

Beginners/Intermediates:
```
/sherpa "my prompt"
# → Shows 5 prompt type pills

/sherpa "my prompt" coding
# → Shows 5 best models for coding

/sherpa "my prompt" coding system anthropic/opus-4-6
# → Uses Opus, applies format + discipline + variant
```

Power Users:
```
/sherpa "my prompt"
# → Shows all 20+ models, no pills

/sherpa "my prompt" research system local/qwen2 sparse
# → Uses Qwen2 locally, outputs final prompt only
```

## Variant Transforms (Per Model)

Each model config specifies transforms:
- `add-section` — Add RATIONALE, CONTEXT, EXAMPLES
- `tighten-steps` — Compress METHOD instructions
- `shorten-sentences` — Enforce word limits
- `add-examples` — Provide concrete examples
- `hardened-constraints` — Strict requirement/rejection separation
- `leverage-context` — Use available context window
- `simplify-language` — Replace complex terms
- `explicit-directives` — Numbered steps, hard language

## Capability Matrix (Task Support)

| Model | Chat | Coding | Research | Images | Videos |
|-------|------|--------|----------|--------|--------|
| Opus 4.6 | ✓ | ✓ | ✓ | ✗ | ✗ |
| Sonnet 4.6 | ✓ | ✓ | ✓ | ✗ | ✗ |
| GPT-4o | ✓ | ✓ | ✓ | ✓ | ✗ |
| Gemini 2 Pro | ✓ | ✓ | ✓ | ✓ | ✓ |
| Qwen2 (local) | ✓ | ✓ | ✓ | ✗ | ✗ |
| Kimi K2-6 | ✓ | ✓ | ✓ | ✗ | ✗ |

## Runtime Discovery

At startup, Sherpa detects available models and generates variants for new ones. New models are auto-available next startup.

## Backward Compatibility

All Sherpa 1.0 formats work unchanged. Discipline applied by default. Use `format-only` mode to skip discipline.

## Key Files

- `index.js` — Entry point, pill display logic, user routing
- `core/user-detection.js` — Level persistence, visibility rules
- `core/capability-matrix.js` — Task support per model
- `core/prompt-router.js` — Filtering, recommendation ranking
- `core/orchestrator.js` — Layer coordination
- `variants/index.js` — Registry loader
- `variants/discover.js` — Runtime detection
- `variants/models/{provider}/{model}.js` — 20 model configs

## Next: Git & Release

Push to `github.com/drewkinney/sherpa` as v2.0. Tag as `v2.0.0`.
```

Copy that. Paste into Claude Code, review, then push.