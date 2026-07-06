---
name: sherpa
description: Rebuilds a user's raw prompt into a tightened, gap-filled version in a specified structured format, or all formats at once with a token-cost warning first. Triggers when the user's message contains the skill name "sherpa" (or "/sherpa") anywhere alongside a format keyword — the live list of formats and their trigger keywords is defined by the files in formats/, not hardcoded here — followed by or containing the prompt to rebuild. The format keyword can appear before, after, or in the middle of the raw prompt; scan the whole message for it rather than assuming position. If "sherpa" appears with no recognizable format keyword, treat the remainder as the raw prompt and present the format menu before doing any rebuild work.
---

# Sherpa

Takes a user's raw, often loosely-worded prompt and rebuilds it as a tightened, gap-filled version of itself in a specified structured format. This is prompt engineering, not just reshaping — ambiguity gets resolved, missing pieces get asked for, weak phrasing gets sharpened, regardless of which format is chosen.

## Invocation parsing

1. Before matching anything, list the formats/ directory and read the `keywords` line from each file to build the current keyword set — never rely on a memorized or hardcoded list, since new formats get added as files, not as edits to this section. Also read the literal keyword `all`, which always means every format currently in formats/, however many that is. Also read the literal keyword `format-only`, which always means: run the format rebuild but skip the Fable Five discipline pass (see discipline/fable-five.md). Also list the variants/ directory (recursively, across all provider subfolders) and read the `keywords` line from each file to build a model-keyword set, the same way the format keyword set is built — never a hardcoded model list, since new provider/model files get added over time (some by hand, some by the Variant discovery step below).
2. Scan the entire user message for the skill name (`sherpa` or `/sherpa`), a format keyword or the literal keywords `all` / `format-only`, and optionally a model keyword from the variant set just built. Keyword position is not fixed for any of these — each can precede, follow, or sit inside the raw prompt text. A model keyword is optional per invocation (see "Variant pass" below for what happens when none is given); the others are required as already specified.
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

## Discipline pass (Fable Five)

After the rebuild above produces formatted output, apply a second pass by default: load `discipline/fable-five.md` and apply its 5 directives to the formatted output. Skip this entire pass only if the `format-only` keyword was present in the invocation — in that case the rebuild output from the section above is returned as-is, unchanged from Sherpa 1.0 behavior.

This pass runs once per format. If `all` was requested, each format's rebuild gets its own discipline pass before moving to the next format.

## Variant pass

After the discipline pass above, apply a third pass, tuned to a specific target model. Unlike the discipline pass, this one has no skip keyword — it always runs, because the only real question is which model to tune for, not whether to tune at all.

**Resolve the target model, in order:**
1. If a model keyword was found during invocation parsing, that's the target.
2. Otherwise, the target is whatever model is currently running this skill (self-detect your own model identity — do not default to a hardcoded model name).

**Apply the variant:**
- If `variants/<provider>/<model>.md` exists for the resolved target, load it and apply its transforms to the discipline pass's output exactly as that file's "How to apply" section describes.
- If no matching file exists, run Variant discovery (below) before proceeding.

This pass runs once per format, after that format's discipline pass, same as the discipline pass runs once per format under `all`.

## Variant discovery

Triggered when the resolved target model has no matching file in `variants/<provider>/<model>.md`.

The 8 transform tags:

| Tag | What it does |
|---|---|
| `add-section` | Add a RATIONALE/CONTEXT block before the ask — for models that reason well from background rather than needing everything spelled out |
| `tighten-steps` | Compress multi-step METHOD instructions into fewer, denser steps |
| `shorten-sentences` | Enforce sentence/word-length limits |
| `add-examples` | Include concrete input→output example pairs |
| `hardened-constraints` | Explicit "Do X. Do not Y. Only accept Z." phrasing, no soft hedges |
| `leverage-context` | Deliberately use more background/detail — for models that use long context well rather than getting diluted by it |
| `simplify-language` | Plain vocabulary, no nested clauses |
| `explicit-directives` | Numbered imperative steps, no suggestion-toned language |

1. Use the WebSearch or WebFetch tool to find that provider's current model name and any published prompting guidance for it.
2. From what's found, pick from the 8 transform tags defined above based on the provider's documented guidance. If no specific guidance exists, reason from general model-capability-tier patterns (smaller/faster models lean toward `explicit-directives`/`simplify-language`/`hardened-constraints`; flagship models lean toward `add-section`/`leverage-context`) and state plainly in the new file that this is an inferred judgment call, not sourced guidance.
3. Write the result to `variants/<provider>/<model>.md`, following the same keywords/provider/transforms/"why"/"how to apply" structure as the Anthropic files.
4. Apply the newly-written file's transforms to the current call's output.
5. If the web lookup fails or returns nothing usable, do not write a file. Apply no variant transforms for this call, and say so plainly in the output (see Output section below).

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

If the format keyword is `all`, do not run every format silently. First, estimate token cost per format, not as one flat multiplier — a flat "8x" is useless because formats aren't equal weight: fewshot needs 2-3 full example pairs, anthropic-system needs persona/context framing, while narrative, json, yaml, xml, and markdown stay close to the size of the rebuilt slots alone. A single number hides that spread and gives the user nothing to decide with.

Instead, for the specific raw prompt just given:
1. Estimate a rough token count for each format currently in formats/, based on that prompt's actual content and what that format's spec requires to render it (this varies per call — don't reuse a cached table).
2. List them compact, one line per format, numbers only — flag the outliers with a short reason (e.g., "fewshot ~450 (needs 2-3 examples)").
3. Sum to a total and state it against what one format alone would cost, so the delta is concrete instead of an abstract multiplier.
4. Ask directly: run all of them, or pick one? Wait for their answer before generating anything, unless they've already said "all" after seeing this estimate once before in the session, or explicitly override with something like "yes all, skip the warning."

## Output

Always return inline in chat. Never write to a file for this skill — the whole point is a fast, pasteable, disposable result. Every format goes in its own separate code block, labeled with the format name as a plain line above the block, not a heavy header. Directly under the label, one short line stating where that format pastes best — pull this from the best_for field in that format's subfile, don't reinvent it. Directly under that pastes-best line, add one more short line: the Fable Five delta for that format's rebuild — what got compressed, what got surfaced, what hierarchy changed (per `discipline/fable-five.md` step 3). Omit this line entirely when `format-only` was used, since no discipline pass ran. If Variant discovery ran for this call (no existing profile for the target model), add one more short line stating the outcome plainly: either "no variant profile existed for {MODEL}; researched and created one" or "no variant profile found for {MODEL}; proceeded without variant tuning" — whichever actually happened. Omit this line when an existing variant file was used, since nothing exceptional occurred. This applies whether one format or all of them are returned. Keep commentary between blocks minimal.
