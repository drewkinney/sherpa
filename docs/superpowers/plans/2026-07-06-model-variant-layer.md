# Model Variant Layer Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a third, model-specific rewrite pass that runs after the Fable Five discipline pass, tuned per target model, with 4 built-in Anthropic-family profiles and a web-lookup discovery step for any other provider.

**Architecture:** Pure markdown, no code. Four new spec files under `variants/anthropic/` follow the same two-field-header-plus-prose pattern as `formats/*.md` and `discipline/fable-five.md`. `SKILL.md` gains: a dynamically-read model-keyword set (mirrors how format keywords are read from `formats/*.md`), a new "Variant pass" pipeline step, and a new "Variant discovery" procedure for uncovered providers.

**Tech Stack:** Markdown only. No JS, no build step, no test runner. "Tests" are manual dry-run invocations verified by reading Claude's actual output against the spec's expected behavior — same testing approach as the discipline-layer plan already executed on this branch.

## Global Constraints

- No JS, no executable code — markdown spec files only
- Variant pass always runs, no skip keyword (unlike `format-only` for discipline)
- Target model resolution order: named keyword in message, else self-detected running model — never a hardcoded fallback model name
- Only 4 files built now (`variants/anthropic/haiku-4-5.md`, `sonnet-5.md`, `opus-4-8.md`, `fable-5.md`) — every other provider is populated lazily via Discovery, never guessed upfront
- Each Anthropic file's transform picks, verbatim from the spec:
  - Haiku 4.5: `explicit-directives`, `simplify-language`, `hardened-constraints`, `tighten-steps`
  - Sonnet 5: `add-section`, moderate `hardened-constraints`
  - Opus 4.8: `add-section`, `leverage-context`
  - Fable 5: `hardened-constraints`, `explicit-directives`, `tighten-steps` — explicitly NOT `add-section` / `leverage-context`
- Discovery must write a new file after a successful lookup, and must state plainly (not silently) when it ran, whether it succeeded or came up empty
- Variant pass applies on top of the discipline pass's output, never replaces or contradicts it

---

### Task 1: Create the 4 Anthropic variant files

**Files:**
- Create: `variants/anthropic/haiku-4-5.md`
- Create: `variants/anthropic/sonnet-5.md`
- Create: `variants/anthropic/opus-4-8.md`
- Create: `variants/anthropic/fable-5.md`

**Interfaces:**
- Produces: 4 files, each with a `keywords:` line (model aliases) that Task 2's keyword-scanning step reads, and a spec body that Task 3's "Variant pass" step applies verbatim.

- [ ] **Step 1: Write `variants/anthropic/haiku-4-5.md`**

```markdown
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
```

- [ ] **Step 2: Write `variants/anthropic/sonnet-5.md`**

```markdown
keywords: sonnet, sonnet-5, claude-sonnet
provider: anthropic
transforms: add-section, moderate hardened-constraints

## Why these transforms

Sonnet 5 is the balanced mid-tier Claude model — per Anthropic's prompt-engineering guidance, it sits between Haiku's need for rigid scaffolding and Opus's ability to run on implicit reasoning. Some structure helps; over-hardening every constraint wastes effort the model doesn't need spent on it.

## How to apply

- **add-section**: If the rebuild's Context or Task slot is thin, add a short RATIONALE note explaining why the request matters — enough to anchor the model's reasoning, not a full brief.
- **moderate hardened-constraints**: State the 1-2 constraints that most affect correctness explicitly ("Do X. Do not Y."). Leave lower-stakes preferences (tone, minor formatting nuance) as normal prose rather than converting every line to hardened Do/Do-not phrasing.
```

- [ ] **Step 3: Write `variants/anthropic/opus-4-8.md`**

```markdown
keywords: opus, opus-4-8, claude-opus
provider: anthropic
transforms: add-section, leverage-context

## Why these transforms

Opus 4.8 is the flagship Claude model — per Anthropic's prompt-engineering guidance, it handles nuance and implicit reasoning well and gets more value from rich context than from having every constraint spelled out and hardened.

## How to apply

- **add-section**: Add a RATIONALE/CONTEXT block ahead of the Task slot explaining the underlying goal and any relevant background — trust the model to draw the right constraints from that context rather than hardening every one explicitly.
- **leverage-context**: Where the source prompt has more background available (earlier conversation, related specifics) than the target format's slots strictly require, include it. Do not compress away context for brevity's own sake.
```

- [ ] **Step 4: Write `variants/anthropic/fable-5.md`**

```markdown
keywords: fable, fable-5, claude-fable
provider: anthropic
transforms: hardened-constraints, explicit-directives, tighten-steps

## Why these transforms

Fable 5's native prompt-processing style is sparse and rigid, not rich and flexible — this is the documented behavioral pattern the Fable Five discipline technique (see discipline/fable-five.md) was itself modeled on. Fable 5 treats every sentence as either a command, a constraint, or a definition, and does not benefit from — and can be actively hurt by — added context-setting or rhetorical scaffolding.

## How to apply

- **hardened-constraints**: Every constraint stated as "Do X. Do not Y. Only accept Z." with zero filler words in between. Constraint density matters — every sentence must carry direct load.
- **explicit-directives**: Numbered, gated reasoning steps, not prose-flowing paragraphs. Name variables and exact conditions in caps or code (e.g. `{FORMAT}`, `{REJECTION_CASE}`) rather than describing them in prose.
- **tighten-steps**: Compress any multi-step method to its minimal numbered form.

Do NOT apply `add-section` or `leverage-context` to this model — minimal context-setting is Fable 5's actual preference, not a gap that needs filling. Adding rationale/background sections works against this model rather than for it.
```

- [ ] **Step 5: Verify all 4 files**

Read each file back and confirm: `keywords:` line present and non-empty, `transforms:` line matches the Global Constraints list above exactly, no `{PLACEHOLDER}`-style unfilled prose (the `{FORMAT}`/`{REJECTION_CASE}` examples in Fable 5's file are intentional illustrations of the technique, not unfilled slots — leave those as-is).

- [ ] **Step 6: Commit**

```bash
git add variants/anthropic/haiku-4-5.md variants/anthropic/sonnet-5.md variants/anthropic/opus-4-8.md variants/anthropic/fable-5.md
git commit -m "$(cat <<'EOF'
Add Anthropic-family model variant profiles

Grounded in Anthropic's published prompt-engineering guidance:
capability tier correlates with how explicit/scaffolded a prompt needs
to be. Fable 5's sparse/rigid profile is the documented pattern the
Fable Five discipline technique was itself modeled on.
EOF
)"
```

---

### Task 2: Add model-keyword scanning to SKILL.md's invocation parsing

**Files:**
- Modify: `SKILL.md` (Invocation parsing section)

**Interfaces:**
- Consumes: the 4 files from Task 1 (their `keywords:` lines)
- Produces: a dynamically-read model-keyword set that Task 3's Variant pass step uses to resolve the target model.

- [ ] **Step 1: Read the current Invocation parsing section**

Confirm it currently reads (after the discipline-layer changes already on this branch):
```
1. Before matching anything, list the formats/ directory and read the `keywords` line from each file to build the current keyword set — never rely on a memorized or hardcoded list, since new formats get added as files, not as edits to this section. Also read the literal keyword `all`, which always means every format currently in formats/, however many that is. Also read the literal keyword `format-only`, which always means: run the format rebuild but skip the Fable Five discipline pass (see discipline/fable-five.md).
2. Scan the entire user message for the skill name (`sherpa` or `/sherpa`) and a format keyword, or the literal keywords `all` / `format-only`, from the set just built. Keyword position is not fixed — it can precede, follow, or sit inside the raw prompt text.
```

- [ ] **Step 2: Add model-keyword reading to step 1**

Append this sentence to the end of step 1:

```markdown
Also list the variants/ directory (recursively, across all provider subfolders) and read the `keywords` line from each file to build a model-keyword set, the same way the format keyword set is built — never a hardcoded model list, since new provider/model files get added over time (some by hand, some by the Variant discovery step below).
```

- [ ] **Step 3: Extend step 2's scan to include model keywords**

Replace step 2 with:

```markdown
2. Scan the entire user message for the skill name (`sherpa` or `/sherpa`), a format keyword or the literal keywords `all` / `format-only`, and optionally a model keyword from the variant set just built. Keyword position is not fixed for any of these — each can precede, follow, or sit inside the raw prompt text. A model keyword is optional per invocation (see "Variant pass" below for what happens when none is given); the others are required as already specified.
```

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "$(cat <<'EOF'
Read model keywords from variants/ during invocation parsing

Mirrors how format keywords are read from formats/ — no hardcoded
model list, set grows as variant files are added by hand or by the
discovery step.
EOF
)"
```

---

### Task 3: Wire the Variant pass and Discovery procedure into SKILL.md

**Files:**
- Modify: `SKILL.md` (new section after "Discipline pass (Fable Five)"; Output section)

**Interfaces:**
- Consumes: model-keyword set (Task 2), the 4 Anthropic files (Task 1)
- Produces: the complete 3-layer pipeline behavior that Task 4's manual verification checks against.

- [ ] **Step 1: Add the "Variant pass" section**

Immediately after the existing "## Discipline pass (Fable Five)" section (which currently ends with "...before moving to the next format."), add:

```markdown
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

1. Use the WebSearch or WebFetch tool to find that provider's current model name and any published prompting guidance for it.
2. From what's found, pick from the 8 transform tags (defined in variants/anthropic/*.md as a working example of the pattern) based on the provider's documented guidance. If no specific guidance exists, reason from general model-capability-tier patterns (smaller/faster models lean toward `explicit-directives`/`simplify-language`/`hardened-constraints`; flagship models lean toward `add-section`/`leverage-context`) and state plainly in the new file that this is an inferred judgment call, not sourced guidance.
3. Write the result to `variants/<provider>/<model>.md`, following the same keywords/provider/transforms/"why"/"how to apply" structure as the Anthropic files.
4. Apply the newly-written file's transforms to the current call's output.
5. If the web lookup fails or returns nothing usable, do not write a file. Apply no variant transforms for this call, and say so plainly in the output (see Output section below).
```

- [ ] **Step 2: Add the Discovery-outcome line to the Output section**

In the "Output" section, after the existing sentence about the Fable Five delta line ("...Omit this line entirely when `format-only` was used, since no discipline pass ran."), add:

```markdown
If Variant discovery ran for this call (no existing profile for the target model), add one more short line stating the outcome plainly: either "no variant profile existed for {MODEL}; researched and created one" or "no variant profile found for {MODEL}; proceeded without variant tuning" — whichever actually happened. Omit this line when an existing variant file was used, since nothing exceptional occurred.
```

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "$(cat <<'EOF'
Wire model variant pass and discovery procedure into SKILL.md

Third pipeline stage after format + discipline. Resolves target model
from a named keyword or self-detection, applies its variant file, and
falls back to a WebSearch-based discovery step for any provider with
no existing profile yet.
EOF
)"
```

---

### Task 4: Manual end-to-end verification

**Files:**
- None modified — this task drives the skill as a user would and reads the transcript.

**Interfaces:**
- Consumes: the fully wired `SKILL.md` + all 4 `variants/anthropic/*.md` files from Tasks 1-3.

- [ ] **Step 1: Dry-run with an explicit Opus target**

In a fresh subagent (no prior context), send:
```
/sherpa markdown opus "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: markdown output with a RATIONALE/CONTEXT addition ahead of the Task slot (per `add-section`), background/detail included rather than compressed away (per `leverage-context`), and no over-hardened constraint-by-constraint Do/Do-not phrasing beyond what the discipline pass already applied. No Discovery-outcome line (Opus file already exists).

- [ ] **Step 2: Dry-run with an explicit Haiku target**

Send:
```
/sherpa markdown haiku "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: numbered imperative steps, plain short sentences, every constraint stated as explicit Do/Do-not, compressed method section. Visibly more rigid than the Opus output from Step 1.

- [ ] **Step 3: Dry-run with an explicit Fable target**

Send:
```
/sherpa markdown fable "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: sparse output, no added rationale/context section, dense constraint-per-sentence phrasing, variables/conditions named in caps or code where referenced.

- [ ] **Step 4: Dry-run with no model keyword (self-detect fallback)**

Send:
```
/sherpa markdown "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: the agent identifies its own running model and applies that model's variant file (or Discovery if its own model has no file yet) — confirm the agent states which model it resolved to and that the resulting output matches that model's expected transform pattern from Steps 1-3 above.

- [ ] **Step 5: Dry-run with an uncovered provider (Discovery path)**

Send:
```
/sherpa markdown gemini "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: the agent describes attempting a WebSearch/WebFetch lookup for Gemini's current model and prompting guidance, then either (a) writes a new `variants/google/*.md` file and applies it, stating the "researched and created one" outcome line, or (b) if the agent has no web tool access in this test context, states plainly that Discovery could not run and proceeds without variant tuning, per Task 3 Step 1's Discovery procedure step 5. Either outcome is acceptable for this test — confirm the *procedure* was followed, not that a live network call necessarily succeeded.

- [ ] **Step 6: Confirm no regression on discipline layer or `all` mode**

Send:
```
/sherpa all "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: existing per-format token-cost estimate and confirmation prompt appear unchanged; after confirming, each format's output carries its discipline delta line (from piece 1) followed by variant-tuned content for the self-detected or default-resolved model.

- [ ] **Step 7: Record verification result**

If all six dry runs match expectations, mark this task done. If any step diverges, fix the relevant `SKILL.md` wording (Task 2 or 3) or the relevant variant file's "How to apply" wording (Task 1) — not the transform-tag assignments themselves unless the divergence reveals those were wrong — and re-run the failing step before proceeding.

---

## Self-Review Notes

- **Spec coverage:** Architecture/flow (Task 3 Step 1), 4 Anthropic files with grounded transforms (Task 1), keyword scanning (Task 2), Discovery procedure (Task 3 Step 1), Output line for Discovery outcome (Task 3 Step 2), testing (Task 4 covers all 4 built models, self-detect fallback, Discovery path, and `all`-mode regression). No spec section without a task.
- **Placeholder scan:** No TBD/TODO. The `{FORMAT}`/`{REJECTION_CASE}` strings in Fable 5's file are intentional illustrations of the technique's own "variable clarity" rule, not unfilled plan placeholders — called out explicitly in Task 1 Step 5 so this isn't mistaken for a plan defect during execution.
- **Consistency:** `variants/anthropic/<model>.md` paths and `keywords:`/`provider:`/`transforms:` field names are identical across Tasks 1, 2, and 3. Transform tag names match the Global Constraints list and the parent spec's table exactly (no renamed tags).
