# Fable Five Discipline Layer Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a second, default-on rewrite pass (Fable Five discipline) that runs after Sherpa's existing format rebuild, with a `format-only` keyword to skip it and a one-line delta note shown after every rebuild.

**Architecture:** Pure markdown, no code. A new spec file (`discipline/fable-five.md`) follows the exact same two-line-header-plus-prose pattern as `formats/*.md`. `SKILL.md` gains one reserved keyword, one new default pipeline step, and one new output line. There is no runtime to build — the "implementation" is the spec files themselves, since Claude reads and applies them directly at trigger time, same as it already does for `formats/`.

**Tech Stack:** Markdown only. No JS, no build step, no test runner — this repo has never had one. "Tests" in this plan are manual dry-run invocations of the skill, verified by reading Claude's actual output against the spec's expected behavior.

## Global Constraints

- No JS, no executable code of any kind — markdown spec files only (explicit user direction, spec section "Out of Scope")
- Discipline pass runs by default on every rebuild; `format-only` is the sole opt-out keyword
- Delta line shown by default after every rebuild, one line, no essay
- `format-only` must be scanned as a reserved keyword the same way `all` already is — never treated as raw prompt text
- Fable Five is model-agnostic — do not add any model-specific branching to this layer (that's piece 2, out of scope here)
- Existing Sherpa 1.0 format-rebuild behavior must not regress when `format-only` is used

---

### Task 1: Create the Fable Five spec file

**Files:**
- Create: `discipline/fable-five.md`

**Interfaces:**
- Produces: a spec file that Task 3 (`SKILL.md` default-behavior step) references by path `discipline/fable-five.md` and reads in full at trigger time, same as `formats/*.md` files are read today.

- [ ] **Step 1: Write the spec file**

Create `discipline/fable-five.md` with this exact content:

```markdown
applies_to: every rebuild, every format, always — unless "format-only" keyword present
skip_keyword: format-only

## Fable Five discipline pass

Apply this pass to the already-formatted rebuild output, after the format spec has been fully applied. Do not run this pass on the raw input prompt — it operates on the formatted result.

Rewrite engine steps, in order:
1. Parse the formatted output for: hidden directives (instructions buried inside descriptive sentences), soft constraints (hedged language like "try to," "be mindful of," "ideally"), and implicit context (assumptions the reader must infer rather than being told).
2. Restructure using these 5 directives:
   - **Flatten instruction hierarchy** — one command per sentence minimum. No embedded clauses hiding directives.
   - **Name constraints explicitly** — "Do X. Do not Y. Only accept Z." Never "try to be mindful of Z."
   - **Kill context drift** — every sentence either commands, constrains, or defines. No rhetorical scaffolding.
   - **Variable clarity** — anything referenced gets named in caps or code: `{FORMAT}`, `{REJECTION_CASE}`.
   - **Outcome specification first** — state what success looks like before method.
3. Surface the delta: what got compressed, what got surfaced (a hidden directive made explicit), what hierarchy changed (a buried clause promoted to its own sentence).

This pass is model-agnostic — apply it identically regardless of which model the prompt is destined for. Do not add model-specific tuning here.
```

- [ ] **Step 2: Verify the file reads correctly**

Read the file back and confirm it has no `{PLACEHOLDER}`-style unfilled slots and that all 5 directives from the design spec are present verbatim. This is a plain-text file — "verify" here means re-reading it, not running a parser.

- [ ] **Step 3: Commit**

```bash
git add discipline/fable-five.md
git commit -m "$(cat <<'EOF'
Add Fable Five discipline layer spec

Model-agnostic rewrite pass: flattens instruction hierarchy, names
constraints explicitly, kills context drift, adds variable clarity,
states outcome before method.
EOF
)"
```

---

### Task 2: Add `format-only` to SKILL.md's reserved keyword set

**Files:**
- Modify: `SKILL.md` (Invocation parsing section, currently lines 10-21)

**Interfaces:**
- Consumes: nothing new from Task 1
- Produces: a reserved keyword (`format-only`) that Task 3's default-behavior step checks for. Any later task or spec that needs to know "is discipline being skipped" checks for this exact keyword string.

- [ ] **Step 1: Read the current Invocation parsing section**

Confirm current numbered list in `SKILL.md` still matches:
```
1. Before matching anything, list the formats/ directory...
2. Scan the entire user message...
3. Everything in the message that isn't...
4. If two distinct format keywords appear...
5. If no format keyword is found...
6. If the message contains "sherpa" with no prompt content...
```

- [ ] **Step 2: Add `format-only` as a reserved keyword**

In the same section, immediately after the existing sentence about the literal keyword `all` (in step 1 of that list), add:

```markdown
1. Before matching anything, list the formats/ directory and read the `keywords` line from each file to build the current keyword set — never rely on a memorized or hardcoded list, since new formats get added as files, not as edits to this section. Also read the literal keyword `all`, which always means every format currently in formats/, however many that is. Also read the literal keyword `format-only`, which always means: run the format rebuild but skip the Fable Five discipline pass (see discipline/fable-five.md).
```

(This replaces the existing step-1 sentence — same sentence, one clause appended for `format-only`.)

- [ ] **Step 3: Confirm no collision with existing scan logic**

Re-read step 2 and step 3 of the Invocation parsing list (message scanning, raw-prompt extraction). Confirm they already say "scan for skill name and a format keyword" generically enough that adding `format-only` and `all` to "the keyword set" requires no further wording change — both are matched the same way a format keyword is. If the wording only mentions "format keyword" narrowly, broaden step 2's phrasing to say "a format keyword, or the literal keywords `all` / `format-only`" so the scan logic explicitly covers all three.

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "$(cat <<'EOF'
Reserve format-only keyword in Sherpa invocation parsing

Lets a caller skip the upcoming Fable Five discipline pass and get
Sherpa 1.0-equivalent output.
EOF
)"
```

---

### Task 3: Wire the discipline pass and delta line into SKILL.md's rebuild and output flow

**Files:**
- Modify: `SKILL.md` ("The rebuild itself" section, currently lines 23-32; "Output" section, currently lines 68-71)

**Interfaces:**
- Consumes: `discipline/fable-five.md` (Task 1), `format-only` reserved keyword (Task 2)
- Produces: the complete default pipeline behavior that Task 4's manual verification checks against.

- [ ] **Step 1: Add the discipline pass as a new step after the rebuild section**

Immediately after "The rebuild itself" section (after the existing bullet list ending in "...never soften or genericize specifics in the name of 'cleaning up.'"), add a new subsection:

```markdown
## Discipline pass (Fable Five)

After the rebuild above produces formatted output, apply a second pass by default: load `discipline/fable-five.md` and apply its 5 directives to the formatted output. Skip this entire pass only if the `format-only` keyword was present in the invocation — in that case the rebuild output from the section above is returned as-is, unchanged from Sherpa 1.0 behavior.

This pass runs once per format. If `all` was requested, each format's rebuild gets its own discipline pass before moving to the next format.
```

- [ ] **Step 2: Add the delta line to the Output section**

In the "Output" section, after the existing sentence "Directly under the label, one short line stating where that format pastes best — pull this from the best_for field in that format's subfile, don't reinvent it," add:

```markdown
Directly under that pastes-best line, add one more short line: the Fable Five delta for that format's rebuild — what got compressed, what got surfaced, what hierarchy changed (per `discipline/fable-five.md` step 3). Omit this line entirely when `format-only` was used, since no discipline pass ran.
```

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "$(cat <<'EOF'
Wire Fable Five discipline pass into default rebuild pipeline

Runs after format rebuild, once per format under "all", skippable via
format-only. Adds one delta line per format under the pastes-best note.
EOF
)"
```

---

### Task 4: Manual end-to-end verification

**Files:**
- None modified — this task drives the skill as a user would and reads the transcript.

**Interfaces:**
- Consumes: the fully wired `SKILL.md` + `discipline/fable-five.md` from Tasks 1-3.

- [ ] **Step 1: Dry-run a normal rebuild (discipline on by default)**

In a fresh Claude Code turn (or a subagent with no prior context), send:
```
/sherpa markdown "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: markdown-format output with headers per slot, PLUS a delta line under the pastes-best note describing what Fable Five compressed/surfaced/restructured (e.g., "try to" / "kind of" / "professional-ish" hedges named as explicit constraints).

- [ ] **Step 2: Dry-run the same prompt with `format-only`**

Send:
```
/sherpa markdown format-only "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: same markdown-format output as Step 1's base rebuild, but NO delta line, and the hedged phrasing is only as tightened as Sherpa 1.0's existing rebuild rules already do (no Fable Five restructuring on top).

- [ ] **Step 3: Confirm no regression on `all` mode**

Send:
```
/sherpa all "hey can you try to write me something that kind of explains our refund policy, keep it professional-ish"
```
Expected: existing per-format token-cost estimate and confirmation prompt appear unchanged; after confirming, each format in the output carries its own delta line.

- [ ] **Step 4: Record verification result**

If all three steps match expectations, mark this task done. If any step diverges, fix the relevant `SKILL.md` wording from Task 2 or 3 (not the discipline file's substance, unless the divergence is in the 5 directives themselves) and re-run the failing step before proceeding.

---

## Self-Review Notes

- **Spec coverage:** Architecture/flow (Task 3), new file (Task 1), SKILL.md keyword + default behavior + output changes (Tasks 2-3), testing/validation (Task 4 mirrors the design spec's 4 bullet checks exactly). Cost-estimate interaction was explicitly out of scope in the design spec — no task needed.
- **Placeholder scan:** No TBD/TODO. All file content is complete, copy-pasteable text, not descriptions of content.
- **Consistency:** `format-only` keyword string is identical across Tasks 2, 3, and 4. `discipline/fable-five.md` path is identical across Tasks 1, 2 (implicitly), and 3.
