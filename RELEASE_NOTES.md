# Sherpa v1.0.0

A Claude Skill that rebuilds a raw, loosely-worded prompt into a tightened, gap-filled version of itself — in whichever structured format the destination actually rewards.

## What it does

Sherpa is an editing pass that happens to output into a shape. Give it a rough prompt and a format keyword (or none, and it'll ask), and it:

- Resolves ambiguity instead of shipping it — picks the likely reading and flags the assumption, or asks if the stakes are high enough to get wrong
- Fills every structural slot a format expects (role, context, task, constraints, output, examples) — asks rather than stubs a placeholder when something's genuinely unknown
- Cuts hedging and filler, sharpens weak verbs into direct instructions
- Never softens or genericizes the specifics you actually gave it — numbers, names, tools, constraints survive intact

## Formats included

Eight, each mapped to where it pastes best: **Anthropic System**, **OpenAI System**, **XML**, **JSON**, **YAML**, **Markdown**, **Narrative**, and **Few-Shot**. New formats are one file dropped into `formats/` — no code changes, no hardcoded list.

## Highlights in this release

- **Directory-driven format catalog** — `SKILL.md` reads `formats/*.md` fresh every trigger, so adding a format is the entire process of adding one
- **Real cost breakdown for `all`** — requesting every format quotes a per-format token estimate against the actual prompt, not a flat multiplier, and calls out the heavy ones (fewshot needs 2-3 examples, anthropic-system needs persona framing) before you commit to generating all eight
- **Session-scoped voice/brand matching** — asked once, applied silently to every rebuild in the conversation after that
- **Disposable by design** — output is always inline, never written to a file; every format lands in its own labeled code block with a one-line "pastes best here" note

## Install

```bash
git clone https://github.com/drewkinney/sherpa.git
```

Drop `SKILL.md` and `formats/` into your Skills directory (e.g. `~/.claude/skills/sherpa/` for Claude Code), or upload as a Skill wherever your Claude surface supports it.

Full documentation, usage examples, and the research behind why structure matters: see the [README](https://github.com/drewkinney/sherpa#readme).
