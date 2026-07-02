---
name: sherpa
description: Rebuilds a user's raw prompt into a tightened, gap-filled version in a specified structured format, or all formats at once with a token-cost warning first. Triggers when the user's message contains the skill name "sherpa" (or "/sherpa") anywhere alongside a format keyword — the live list of formats and their trigger keywords is defined by the files in formats/, not hardcoded here — followed by or containing the prompt to rebuild. The format keyword can appear before, after, or in the middle of the raw prompt; scan the whole message for it rather than assuming position. If "sherpa" appears with no recognizable format keyword, treat the remainder as the raw prompt and present the format menu before doing any rebuild work.
---

# Sherpa

Takes a user's raw, often loosely-worded prompt and rebuilds it as a tightened, gap-filled version of itself in a specified structured format. This is prompt engineering, not just reshaping — ambiguity gets resolved, missing pieces get asked for, weak phrasing gets sharpened, regardless of which format is chosen.

## Invocation parsing

1. Before matching anything, list the formats/ directory and read the `keywords` line from each file to build the current keyword set — never rely on a memorized or hardcoded list, since new formats get added as files, not as edits to this section. Also read the literal keyword `all`, which always means every format currently in formats/, however many that is.
2. Scan the entire user message for the skill name (`sherpa` or `/sherpa`) and a format keyword from the set just built. Keyword position is not fixed — it can precede, follow, or sit inside the raw prompt text.
3. Everything in the message that isn't the skill name or the matched keyword is the raw prompt to rebuild.
4. If two distinct format keywords appear in the same message, stop and ask the user which one they meant — do not guess.
5. If no format keyword is found, do not guess a default. Show the format menu (see below) and ask which one, treating the rest of the message as the raw prompt to hold onto.
6. If the message contains "sherpa" with no prompt content at all, ask what they want rebuilt.

### Format menu (shown when no keyword is given)

List the formats currently in formats/ by name (derived from the directory listing, not memorized), plus "all." Ask which one.

## The rebuild itself (applies to every format)

This is not a reformatting pass — it's an editing pass that happens to output into a shape.

- Resolve ambiguity. If a phrase could mean two things, pick the more likely reading and note the assumption inline, or ask if the stakes seem high enough to get it wrong.
- Fill or flag every structural slot the target format expects: role, context, task, constraints, output format, examples (where relevant). If the source prompt doesn't cover a slot:
  - If it's reasonably inferable from context already in the prompt or the conversation, fill it and say so isn't required — just fill it well.
  - If it's genuinely unknown (most commonly: desired output format/length/audience), do not fill it with a placeholder like "insert output here." Ask the user what they want, get the answer, and fill it in properly. Do not proceed to the final rebuild with a stub in an output slot.
- Tighten language: cut hedging, vague qualifiers, and filler. Sharpen weak verbs into direct instructions.
- Preserve the user's actual intent and any concrete specifics they gave (numbers, names, tools, constraints) — never soften or genericize specifics in the name of "cleaning up."

## Asking questions

Never restate the parsed prompt, the detected format, or your interpretation back to the user before responding. They gave you the prompt — they know what's in it. Go straight to questions if something's missing, or straight to the formatted output if nothing is. No "here's what I'm parsing," no "target: X, deciding this myself," no summary of context you're pulling in.

Any time this skill needs something from the user — missing slot, format collision, cost confirmation, voice/brand check — keep it short. One statement, no throat-clearing, no explaining why you're asking. If there are choices, bullet them, don't prose them out. Get the answer, move on.

Bad: "I need to figure out what output format you're looking for before I can properly fill in this slot — could you let me know whether you want a short version or a longer one?"
Good: "Short or long?"

## Voice/brand enrichment

Once per session, before the first rebuild, check whether a voice, brand, or style pattern is already established — either from active memory or from earlier in the current conversation. 

- If one is established, apply it silently to the rebuild without announcing that you're doing so.
- If none is established, ask once: "Want this tailored to your voice/brand? If so, give me the context." 
  - If yes, take their answer and hold it as active working context for the rest of the session — apply it to this rebuild and any subsequent ones in the same conversation. Do not save it to a file; it's session-scoped only.
  - If no or ignored, proceed generic and don't ask again this session.

## Format specs

Each format's spec lives in its own subfile under formats/ — keywords it triggers on, where it pastes best, and how to build it. List the directory and load the relevant subfile(s) at trigger time; don't hold specs in memory across sessions since the set can change.

To add a new format: create a new file in formats/ with the three-field template (keywords / best_for / spec). Nothing else needs to change — this file reads the directory fresh every time.

## "All" handling

If the format keyword is `all`, do not run every format silently. First, estimate token cost: state roughly what one format costs vs. all of them (a simple multiplier off the single-format estimate, scaled to however many formats currently exist in formats/ — this is a heads-up, not an invoice). Ask directly: run all of them, or pick one? Wait for their answer before generating anything, unless they've already said "all" after seeing this warning once before in the session, or explicitly override with something like "yes all, skip the warning."

## Output

Always return inline in chat. Never write to a file for this skill — the whole point is a fast, pasteable, disposable result. Every format goes in its own separate code block, labeled with the format name as a plain line above the block, not a heavy header. Directly under the label, one short line stating where that format pastes best — pull this from the best_for field in that format's subfile, don't reinvent it. This applies whether one format or all of them are returned. Keep commentary between blocks minimal.
