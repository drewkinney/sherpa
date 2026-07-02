<div align="center">

# 🏔️ Sherpa

**A Claude Skill that turns a loosely-worded prompt into a tightened, gap-filled, structured one**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat)](./LICENSE)
[![Formats](https://img.shields.io/badge/formats-8-blue?style=flat)](#format-catalog)
[![Type](https://img.shields.io/badge/type-Claude%20Skill-8A2BE2?style=flat)](#what-you-get)

</div>

---

> Most prompts don't fail on ideas. They fail on **missing slots**, **buried intent**, and **the wrong shape for where they're pasted**.
> Sherpa fixes that category of failure — without you having to know prompt engineering to begin with.

---

## Table of Contents

- [Why Sherpa](#why-sherpa)
- [Every Prompt Starts Unstructured](#every-prompt-starts-unstructured)
- [What You Get](#what-you-get)
- [How It Works](#how-it-works)
- [Format Catalog](#format-catalog)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Adding a New Format](#adding-a-new-format)
- [Design Principles](#design-principles)
- [Works Cited](#works-cited)
- [License](#license)

---

## Why Sherpa

### Ask yourself:

- Do you write prompts as a stream of thought and hope the important part survives?
- Do you paste the same rough idea into Claude, ChatGPT, a config file, and a teammate's doc — and reshape it by hand every time?
- Do your prompts skip role, constraints, or output format because you didn't think to specify them?
- Do you know which format a given surface actually wants — XML for Claude, SYSTEM/USER for GPT, YAML for a CI pipeline — or do you guess?

**If any of that sounds familiar, Sherpa is worth the two seconds it costs to invoke.**

Sherpa is a Claude Skill that takes your raw prompt and rebuilds it as a complete, sharpened version of itself, in whichever structured format your destination actually needs. It is an editing pass that happens to output into a shape — not a reformatting pass.

> [!NOTE]
> **Best for:** rebuilding prompts before they go into a system prompt, an API call, a config file, or a teammate's hands.
> **Not for:** generating prompts from nothing — Sherpa needs your raw intent to sharpen. Garbage in, sharpened garbage out.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Every Prompt Starts Unstructured

Nobody types a system prompt into existence fully formed. A prompt starts as whatever's in your head — a run-on sentence, a half-finished thought, a task description missing the constraints you only remembered after hitting send. That's normal. It's also the reason two people asking a model the same underlying question get wildly different answers: the model isn't reading intent, it's reading the words on the page, and unstructured text buries the parts that matter (role, constraints, output shape) inside the parts that don't (tone, throat-clearing, run-on context).

This isn't a matter of opinion. It's measured. Anthropic's own prompt-engineering documentation recommends wrapping distinct prompt components — role, context, task, examples, output format — in explicit tags specifically because it "helps Claude parse your prompt more accurately, leading to higher-quality outputs," and because unlabeled prompts leave the model to guess where one instruction ends and another begins (Anthropic). OpenAI's guidance points the same direction: prompt engineering is described as an iterative discipline where clearer structure and grounding data consistently produce more reliable output than loosely worded requests (OpenAI). Independent, peer-reviewed research backs this up outside vendor documentation too — a 2024 clinical study found that organizing information into a standardized template *before* asking a large language model to reason over it measurably improved diagnostic accuracy compared to feeding the model the same information unstructured ("Structured Clinical Reasoning"), and a 2025 evaluation of GPT-4o found that prompt style alone — independent of the underlying task — changed output accuracy, efficiency, and token cost ("Enhancing Structured Data Generation"). Amazon's applied guidance for Claude 3 on Bedrock reaches the same conclusion from the deployment side: production prompt quality is a function of explicit structure, not prompt length or cleverness (Amazon Web Services).

The pattern across all of it: structure isn't decoration. It's how the model figures out what's actually being asked. Sherpa exists because that structuring step is tedious to do by hand every time, and because the "right" structure isn't one-size-fits-all — a prompt bound for a Claude system parameter wants XML; the same prompt bound for a CI pipeline wants YAML. Sherpa does the editing pass once and outputs into whichever shape the destination actually rewards.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## What You Get

**If you write prompts for Claude, GPT, agents, or teammates, here's what changes when you use Sherpa.**

- **Ambiguity resolved, not shipped** — a phrase that could mean two things gets the more likely reading, flagged inline, or a direct question if the stakes are high enough to get it wrong.
- **Every structural slot filled or flagged** — role, context, task, constraints, output format, examples. Inferable gaps get filled silently and well. Genuinely unknown gaps get asked about — never stubbed with a placeholder.
- **Hedging and filler cut** — vague qualifiers stripped, weak verbs sharpened into direct instructions.
- **Your specifics survive** — numbers, names, tools, constraints never get softened or genericized in the name of "cleaning up."
- **The right shape for the destination** — eight formats on tap, each mapped to where it actually pastes best, chosen by keyword, not by guesswork.
- **A cost warning before the expensive option** — asking for `all` gets you a token-cost estimate first, not a silent multi-format dump.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## How It Works

Sherpa is one `SKILL.md` plus a `formats/` directory of small spec files. There is no hardcoded format list — the skill reads the directory at trigger time, every time, so adding a file is the entire process of adding a format.

```
┌───────────────────────────────────────────────────────────────┐
│  raw prompt + "sherpa" + format keyword                        │
│           │                                                   │
│           ▼                                                   │
│  scan formats/ for keywords  ──►  match format, isolate prompt │
│           │                                                   │
│           ▼                                                   │
│  editing pass: resolve ambiguity, fill/flag slots, tighten      │
│           │                                                   │
│           ▼                                                   │
│  output in matched format, labeled, with a "pastes best" line  │
└───────────────────────────────────────────────────────────────┘
```

**Invocation parsing, in order:**

1. Read `formats/*.md` and build the live keyword set from each file's `keywords` line — plus the literal keyword `all`.
2. Scan the whole message for the skill name (`sherpa`) and a format keyword. Position is not fixed — it can sit before, after, or inside the raw prompt.
3. Everything that isn't the skill name or the matched keyword is the raw prompt to rebuild.
4. Two format keywords in one message → stop and ask which one. No guessing.
5. No format keyword found → show the format menu, hold the rest of the message as the prompt to rebuild.
6. `sherpa` with nothing else → ask what they want rebuilt.

**What Sherpa handles automatically:**
- Building the keyword set from whatever formats currently exist
- Resolving ambiguity with a stated assumption or a direct question
- Filling structural slots that are inferable from context
- Tightening language without losing the source's concrete specifics
- Applying an established voice/brand pattern silently, once detected per session

**What you write:**
- The raw prompt, however rough
- A format keyword, or nothing if you want the menu
- Answers to the (rare, terse) questions Sherpa asks when something's genuinely missing

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Format Catalog

| Format | Keywords | Best For |
|---|---|---|
| **Anthropic System** | `system`, `anthropic` | Claude API system parameter, Claude Projects custom instructions |
| **Few-Shot** | `fewshot`, `few-shot`, `few shot` | Classification/extraction tasks, fine-tuning-style prompts, pattern-matching |
| **JSON** | `json` | API calls, function-calling schemas, config-driven agent tools |
| **Markdown** | `markdown`, `md` | Notion, Google Docs, README-style prompt libraries, human handoff docs |
| **Narrative** | `improve`, `narrative` | Raw chat interfaces — Claude.ai, ChatGPT web, any single-turn chat box |
| **OpenAI System** | `openai` | GPT API system/user split, Cursor/Windsurf/Antigravity system prompts |
| **XML** | `xml` | Claude system prompts, CLAUDE.md, tool/function definitions |
| **YAML** | `yaml` | Config files, CI/CD prompt pipelines, Claude Code frontmatter, n8n/Zapier steps |

Ask for `all` and Sherpa quotes a token-cost estimate before generating every format — confirm once per session and it stops asking.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Getting Started

### Requirements

| Dependency | Minimum | Notes |
|---|---|---|
| Claude | Any surface with Skills support | Claude.ai, Claude Code, API with Skills enabled |
| Nothing else | — | No build step, no dependencies, no install |

### Install

```bash
git clone https://github.com/drewkinney/sherpa.git
```

Drop `SKILL.md` and `formats/` into your Skills directory (e.g. `~/.claude/skills/sherpa/` for Claude Code), or upload as a Skill wherever your Claude surface supports it.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Usage

**Bare, no format specified — Sherpa shows the menu:**

```
sherpa: draft me something that gets a support bot to stop apologizing so much
```

**With a format keyword, anywhere in the message:**

```
sherpa xml: draft me something that gets a support bot to stop apologizing so much
```

```xml
<role>...</role>
<context>...</context>
<task>...</task>
<constraints>...</constraints>
<output>...</output>
```

**Every format at once:**

```
sherpa all: draft me something that gets a support bot to stop apologizing so much
```

Sherpa quotes the token cost across all 8 formats first, then waits for confirmation.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Adding a New Format

Create one file in `formats/` with three fields:

```markdown
keywords: yourkeyword, alt-keyword
best_for: where this format pastes best

Spec text describing how to build the format from the rebuilt prompt's slots.
```

Nothing else changes. `SKILL.md` reads the directory fresh every time it runs, so the new keyword and format are live the next time Sherpa triggers.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Design Principles

- **Editing, not reshaping.** The point is a sharper prompt, not a prettier one wearing the same gaps.
- **Never invent what you can ask.** Genuinely unknown slots (output length, audience, voice) get a short question, never a placeholder.
- **Specifics are sacred.** Numbers, names, tools, and constraints the user gave never get softened for the sake of polish.
- **Terse questions.** "Short or long?" — not a paragraph explaining why the question is being asked.
- **Disposable by design.** Output is always inline, never written to a file. Rebuild it, paste it, done.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## Works Cited

Amazon Web Services. "Prompt Engineering Techniques and Best Practices: Learn by Doing with Anthropic's Claude 3 on Amazon Bedrock." *AWS Machine Learning Blog*, Amazon Web Services, aws.amazon.com/blogs/machine-learning/prompt-engineering-techniques-and-best-practices-learn-by-doing-with-anthropics-claude-3-on-amazon-bedrock/. Accessed 2 July 2026.

Anthropic. "Use XML Tags to Structure Your Prompts." *Claude Docs*, Anthropic, platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags. Accessed 2 July 2026.

"Enhancing Structured Data Generation with GPT-4o: Evaluating Prompt Efficiency Across Prompt Styles." *PMC*, National Center for Biotechnology Information, 2025, pmc.ncbi.nlm.nih.gov/articles/PMC11979239/. Accessed 2 July 2026.

OpenAI. "Prompt Engineering." *OpenAI API Documentation*, OpenAI, developers.openai.com/api/docs/guides/prompt-engineering. Accessed 2 July 2026.

"Structured Clinical Reasoning Prompt Enhances LLM's Diagnostic Capabilities in Diagnosis Please Quiz Cases." *medRxiv*, 2024, www.medrxiv.org/content/10.1101/2024.09.01.24312894.full.pdf. Accessed 2 July 2026.

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>

---

## License

MIT — see [LICENSE](./LICENSE).

<div align="right"><kbd><a href="#table-of-contents">↑ back to top ↑</a></kbd></div>
