# Editorial Loop _(Editorial-Loop)_

[![Standard Readme](https://img.shields.io/badge/standard--readme-fde047.svg)](https://github.com/RichardLitt/standard-readme)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-7c3aed.svg)](https://code.claude.com/docs/en/plugins)

Two-agent editing loop for Claude Code that edits prose until readers without the author's expertise can follow it.

Editorial Loop is a two-agent editorial system packaged as a Claude Code plugin. Its purpose is narrow and specific: edit prose until a reader who lacks the author's domain expertise can follow the reasoning chain; not until it "reads well" to someone who already understands it.

The system has two agents and a controller. The **editor** agent holds five named schemata — Barrier Bridge, Chain Repair, Compression Pass, Flow Weld, and Ripple Read — as its perceptual apparatus, and runs them in phases over your text. The **evaluator** agent scores the result against a measurable feature set *before* it reads the editor's explanation of what it did, then confirms or rejects the editor's claim that the text is done. They loop until the evaluator confirms the termination condition, or until five cycles are spent.

This repository is the plugin source. The product name is "Editorial Loop"; the repository folder and plugin package are `Editorial-Loop` / `editorial-loop` — the difference is casing only, not a rename.

## Table of Contents

- [Background](#background)
- [Install](#install)
  - [Dependencies](#dependencies)
- [Usage](#usage)
  - [The intake questionnaire](#the-intake-questionnaire)
  - [What you get back](#what-you-get-back)
  - [When not to use it](#when-not-to-use-it)
- [How It Works](#how-it-works)
- [The Five Schemata](#the-five-schemata)
- [Design Principles](#design-principles)
- [Test Evidence](#test-evidence)
- [API](#api)
- [Maintainers](#maintainers)
- [Thanks](#thanks)
- [Contributing](#contributing)
- [License](#license)

## Background

The starting observation is in the repo's own test evidence: ordinary single-pass editing optimizes for "reads well to someone who already understands it." A cleaner, tighter edit can leave every barrier intact — jargon ungrounded, reasoning asserted rather than derived, the register yanked mid-text. That is the opposite of the job when the writing has to carry a reader across a knowledge gap.

Editorial Loop is built to close that gap. Its architecture comes from four older disciplines, translated into editorial moves:

- **Forensic linguistics** — the evaluator is modeled on a questioned-document examiner: it checks a produced document against known standards instead of asking whether it feels good.
- **Species counterpoint** — the five schemata are meant to be perceived simultaneously, the way a musician hears pitch, rhythm, and harmony at once, then executed in phases so the moves don't interfere with each other.
- **Oral-formulaic composition** — the grounding requirement: every bridge is anchored in how a jump felt, what was confusing, or what made it click, not in a definition.
- **Collaborative voice reconstruction** — the split of authority between the agent that rebuilds the text and the agent that verifies it.

## Install

Editorial Loop is a Claude Code plugin. This repository ships no marketplace file, so the `/plugin marketplace add` route does not apply. Load it directly with `--plugin-dir`:

```sh
git clone https://github.com/AlastairZeved/Editorial-Loop.git
cd Editorial-Loop
claude --plugin-dir .
```

`--plugin-dir` loads the plugin for that session. To have it available on every launch, copy the directory into your Claude Code plugins/skills directory (see the [Claude Code plugins docs](https://code.claude.com/docs/en/plugins) for the current mechanics). When loaded as a plugin, Claude Code resolves `${CLAUDE_PLUGIN_ROOT}` to the plugin's own directory, which is how the skill locates `agents/` and `schemata/` at runtime.

### Dependencies

- [Claude Code](https://code.claude.com/docs/en/plugins) — the CLI runtime. This is the only dependency; the plugin is markdown and JSON, with no build step.

## Usage

There is no CLI binary to call. You use it inside a Claude Code session, by asking for the thing the skill is built for:

```text
> Edit this paragraph so someone without my background can follow the reasoning.
```

The skill fires on requests to edit prose, essays, guides, documentation, or any writing meant to carry a reader through a reasoning chain — especially cross-domain writing, where the author has expertise the reader does not.

### The intake questionnaire

The skill does not start editing immediately. It asks three questions and validates the answers, because a vague target reader silently corrupts every downstream schema:

1. **Target reader** — role or domain, what they already know, and what they don't. "Non-technical" and "general audience" always fail; the skill needs to know sentence by sentence which terms need a bridge.
2. **Purpose** — a specific action the reader should be able to take after reading ("decide whether to sign," "ask informed questions about X"). "Make it clear" fails; it is a property of the writing, not a reader outcome.
3. **Source text** — a paste, a file path, a message reference in the conversation, or a public URL. "What I wrote yesterday" fails; it is not retrievable in this session.

Valid answers are reformatted into a context block, which you confirm before any agent is dispatched:

```text
EDITORIAL CONTEXT — PLEASE CONFIRM

TARGET READER: Marketing manager at a fintech startup
KNOWS: Project management concepts, reading data dashboards
DOESN'T KNOW: ML background, how language models work internally

PURPOSE: Understand why AI-generated code is risky to trust
READER SHOULD: Explain the mechanism to their team

SOURCE TEXT: pasted paragraph, ~4 sentences
SOURCE TYPE: Direct paste
PRECEDING CONTEXT: None — Flow Weld operates at document boundaries only
```

If the text is part of a larger document, include the preceding paragraph when asked — the agents use it to check flow at the entry seam.

### What you get back

```text
Edited Text        — the final version
What Changed       — which schemata fired, major rewrites, bridges added
Editorial Trace    — condensed: schema, location, finding, action taken
Evaluator Verdict  — PASS with a feature summary, or best-effort with remaining issues
```

### When not to use it

- Code comments, commit messages, quick responses.
- Writing where the audience already shares the author's domain expertise and jargon is appropriate.
- First drafts that haven't been written yet — this is an editing loop, not a generation tool.

## How It Works

The architecture has four layers.

**Layer 1 — Intake** (`skills/editorial-loop/SKILL.md`). The skill validates the three questions, formats them into templates, and dispatches only after you confirm the context block. It escalates on vague answers: one targeted clarification, then a direct checklist — it does not accept vagueness out of politeness.

**Layer 2 — Execution** (`agents/editor.md` + `schemata/`). The editor reads all six schemata files, then runs four phases: Barrier Bridge and Chain Repair together, then Compression Pass, then Flow Weld, then Ripple Read. If Ripple Read finds gaps, it dispatches them back to the responsible schema with specific, located feedback. The editor returns edited text, a schema trace of what fired where, and a termination assessment.

**Layer 3 — Verification** (`agents/evaluator.md`). The evaluator scores the output against every measurable feature *before* reading the editor's trace, then compares its independent scoring to the trace. Discrepancies are the most important findings: either the editor ran a schema without the work showing (execution failure), or skipped a schema and claimed it ran (compliance failure). It returns feature scores, discrepancies, and a PASS/FAIL verdict.

**Layer 4 — Loop** (`skills/editorial-loop/SKILL.md`). On FAIL, the evaluator's located feedback goes back to the editor, which re-enters at the phase where the failures were found. On PASS, the termination condition is confirmed and the final output is presented. Limits:

- Maximum **5 editor↔evaluator cycles** per session.
- Each cycle should converge — fewer failures than the last. If failures are not decreasing after cycle 3, the skill surfaces this and lets you decide.
- If not converged after 5 cycles, you get the best output so far with the remaining failures named.

## The Five Schemata

| Phase | Schema | What it does |
|-------|--------|--------------|
| 1 | **Barrier Bridge** | Finds where the reader's knowledge ends and the author's assumptions begin; grounds terms and leaps in experience, not definitions. |
| 1 | **Chain Repair** | Finds where reasoning is asserted rather than derived, and rebuilds the chain so each step is derivable from the last. |
| 2 | **Compression Pass** | Removes words that don't earn their place — without stripping weight-bearing bridges. |
| 3 | **Flow Weld** | Checks bidirectional flow at every edit point and removes the seams edits leave behind. |
| 4 | **Ripple Read** | Reads the whole document across five ledgers (flow, coherence, rhythm, reasoning arc, audience) and owns the termination decision. |

Each schema file in `schemata/` has the same structure: Recognition Trigger, Execution Sequence, Scale, Completion Test, Measurable Features. `schemata/scale-rules.md` is the coordination layer:

- **Activation by scale.** Barrier Bridge, Compression Pass, and Flow Weld are primary at sentence/paragraph scale; Chain Repair is primary at paragraph scale; Ripple Read is not active below document scale and is primary and exclusive at document scale.
- **Ordering.** Phase 1 runs first because it adds and restructures content; compressing or welding before content is stable wastes work. Flow Weld runs after, because it checks whether all preceding edits created seams. Ripple Read runs last and dispatches gaps back.
- **Authority boundaries.** No schema may override another's authority: Barrier Bridge decides if a bridge is grounded, Chain Repair if a step is derived, Compression Pass if a word earns its place, Flow Weld if a seam exists, Ripple Read if editing is done.
- **Audience calibration.** Every schema inherits the audience definition from the dispatch layer. No schema defines its own reader.

## Design Principles

These are the commitments the system is built around. They explain behavior that might otherwise look stubborn.

1. **The standard is the reader, not the prose.** Editing stops when a reader without the author's domain expertise can follow the reasoning chain, in the order presented, using language and structure they already have, without silently disengaging — not when the text "reads well" to an expert.
2. **Score first, compare second.** The evaluator scores the text against every feature before reading the editor's trace. Reading the trace first is checking homework against the answer key — confirmation bias, not verification.
3. **Every repair is a rewrite.** A patch preserves broken structure; a rewrite finds the right structure. The editor does not patch.
4. **Bridges are weight-bearing.** Text inserted to ground a concept must survive compression: Compression Pass may not strip a bridge unless a shorter bridge carries the same grounding.
5. **The schemata are perceptual apparatus, not a checklist.** All phases run on every pass. A phase that finds nothing reports "no issues found at this scale" — which is different from not running it.
6. **Authority is partitioned, and termination is exclusive.** The editor cannot declare editing complete; only Ripple Read can propose termination, and only the evaluator can confirm it. Neither agent grades its own work.
7. **The intake is load-bearing, not ceremony.** A vague target reader silently corrupts every downstream schema, so the questionnaire validates hard before anything runs.
8. **Failures are named, located, and not softened.** The evaluator does not pass text with feature failures because it "reads well overall." The features are the standard.

## Test Evidence

The `tests/` directory contains a worked case: one paragraph with three planted defects, run through the system. The files are regression evidence, not loaded at runtime.

- [`tests/test-paragraph.md`](./tests/test-paragraph.md) — a paragraph about "vibe coding," written for a non-technical reader, with three planted defects: a tacit-knowledge wall of ungrounded ML jargon, a skipped reasoning step ("Obviously, this is why prompt engineering matters"), and a register break from the warm preceding context into cold technical exposition.
- [`tests/baseline-test-output.md`](./tests/baseline-test-output.md) — what ordinary single-pass editing produces. Cleaner prose, shorter sentences — and all three defects still present.
- [`tests/evaluator-test-output.md`](./tests/evaluator-test-output.md) — the evaluator scoring the *unedited* paragraph, blind to any trace. It independently catches all three defects: 6 ungrounded domain terms, the asserted leap to "prompt engineering matters," and the hard register seam at entry. Verdict: FAIL, with located feedback per schema.
- [`tests/editor-test-output.md`](./tests/editor-test-output.md) — the full editor run. Barrier Bridge finds 14 barriers and replaces the technical apparatus with a grounded "two pictures" metaphor; Chain Repair maps an 11-step chain, finds 2 breaks, and repairs them with the missing intermediate steps; Compression Pass strips dead weight while keeping the grounding; Flow Weld checks 7 edit points and welds the 1 seam it finds; Ripple Read returns zero ledger entries across all five dimensions and proposes termination.

Together the files demonstrate four things: the paragraph has three defects, a baseline single-pass edit does not fix them, the evaluator detects all three independently before seeing any trace, and the editor repairs all three through the schemata. The evaluator's confirmation of the editor's finished output is the loop's next step beyond these files; the editor's run ends by submitting its termination proposal for exactly that check.

[`tests/skill-questionnaire-tests.md`](./tests/skill-questionnaire-tests.md) separately specifies the intake: 21 manually verifiable acceptance criteria covering validation rules, clarification patterns, template formatting, and dispatch requirements, with worked scenarios for each.

## API

There is no code API. The plugin's surface is three components Claude Code loads from the manifest (`.claude-plugin/plugin.json`):

- **Skill — `skills/editorial-loop/SKILL.md`.** Auto-fires on prose-editing requests. Input: the user's request and text. Output: the intake questionnaire, then the final edited text with trace and verdict.
- **Agent — `agents/editor.md`.** Input: the formatted editorial context (target reader, purpose, source text, preceding context) plus the schemata library. Output: edited text, schema trace, termination assessment.
- **Agent — `agents/evaluator.md`.** Input: the same editorial context, the original text, the editor's output, and the editor's trace (read only *after* independent scoring). Output: feature scores, trace discrepancies, PASS/FAIL verdict.

## Maintainers

[@AlastairZeved](https://github.com/AlastairZeved)

## Thanks

To [Richard Litt](https://github.com/RichardLitt) and the [Standard Readme](https://github.com/RichardLitt/standard-readme) spec, which this README follows.

## Contributing

Questions and bug reports go to [GitHub issues](https://github.com/AlastairZeved/Editorial-Loop/issues). PRs are accepted — there is no CONTRIBUTING file or Code of Conduct in the repository yet, so please:

- Open an issue describing the problem before a large change.
- Keep terminology consistent with the schemata (`schemata/`) and the agents (`agents/`) — Barrier Bridge, Chain Repair, Compression Pass, Flow Weld, Ripple Read, and the feature set are the project's vocabulary.
- If you change behavior, add or update the corresponding evidence in `tests/` so the worked case keeps demonstrating the full loop.

## License

MIT © Alastair Zeved. See [LICENSE](./LICENSE) for the full text.
