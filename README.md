# Editorial Recension _(Editorial-Recension)_

[![Standard Readme](https://img.shields.io/badge/standard--readme-fde047.svg)](https://github.com/RichardLitt/standard-readme)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![Agent Plugins 1.0.0](https://img.shields.io/badge/Agent%20Plugins-1.0.0-059669.svg)](https://github.com/agentplugins/agent-plugins-spec)

A two-agent editing loop that rewrites prose until readers without the author's expertise can follow it.

Editorial Recension is a two-agent editorial system packaged as a portable [Agent Plugins 1.0.0](https://github.com/agentplugins/agent-plugins-spec) package that runs on any conformant agent client. Its purpose is narrow and specific: edit prose until a reader who lacks the author's domain expertise can follow the reasoning chain — not until it "reads well" to someone who already understands it.

The system has two agents and a controller. The **editor** agent holds five named schemata — Barrier Bridge, Chain Repair, Compression Pass, Flow Weld, and Ripple Read — as its perceptual apparatus, and runs them in phases over your text. The **evaluator** agent scores the result against a measurable feature set *before* it reads the editor's explanation of what it did, then confirms or rejects the editor's claim that the text is done. They loop until the evaluator confirms the termination condition, or until five cycles are spent.

This repository is the plugin source. The product name is "Editorial Recension"; the repository folder and plugin package are `Editorial-Recension` / `editorial-recension` — the difference is casing only, not a rename.

## Table of Contents

- [Background](#background)
- [Install](#install)
  - [Dependencies](#dependencies)
  - [What ships in the package](#what-ships-in-the-package)
  - [Supported clients](#supported-clients)
  - [Any other agent](#any-other-agent)
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

Editorial Recension is built to close that gap. Its architecture comes from four older disciplines, translated into editorial moves:

- **Forensic linguistics** — the evaluator is modeled on a questioned-document examiner: it checks a produced document against known standards instead of asking whether it feels good.
- **Species counterpoint** — the five schemata are meant to be perceived simultaneously, the way a musician hears pitch, rhythm, and harmony at once, then executed in phases so the moves don't interfere with each other.
- **Oral-formulaic composition** — the grounding requirement: every bridge is anchored in how a jump felt, what was confusing, or what made it click, not in a definition.
- **Collaborative voice reconstruction** — the split of authority between the agent that rebuilds the text and the agent that verifies it.

## Install

Editorial Recension is a portable [Agent Plugins 1.0.0](https://github.com/agentplugins/agent-plugins-spec) package: a root `plugin.json` manifest (schema `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json`) plus a top-level `skills/` directory. The repository is markdown and JSON — no build step — and installs on any conformant client.

One rule applies everywhere: **install the whole directory, not just the skill folder.** The skill references `agents/` and `schemata/` through relative paths (`../../` from the skill file), so those paths resolve only when the bundle is installed intact. Clients without a native subagent mechanism can run the contents of `agents/editor.md` and `agents/evaluator.md` inline as prompts.

### Dependencies

- Any [Agent Plugins 1.0.0](https://github.com/agentplugins/agent-plugins-spec)-compatible agent client — see [Supported clients](#supported-clients).
- No runtime, package manager, or build tooling. The package is markdown and JSON.

### What ships in the package

| Component | Path | Role |
|---|---|---|
| Manifest | `plugin.json` | Agent Plugins 1.0.0 manifest — what clients validate and load |
| Skill | `skills/editorial-recension/SKILL.md` | The controller: intake questionnaire, dispatch, loop |
| Agents | `agents/editor.md`, `agents/evaluator.md` | The editor and evaluator the skill dispatches |
| Schemata | `schemata/*.md` (6 files) | The five editing schemata plus `scale-rules.md`, the coordination layer |
| Tests | `tests/` | Worked-case regression evidence; not loaded at runtime |
| Marketplaces | `.agents/plugins/marketplace.json`, `.github/plugin/marketplace.json` | Codex and Copilot CLI marketplace catalogs pointing at the package root |

### Supported clients

Install paths marked **verified** were exercised or checked against the client's current documentation (2026-09-12). Cursor is documented as loading Agent Plugins standard packages natively; Cline is not listed because its plugin system takes TypeScript SDK modules, not Agent Plugins packages — copy the repository into a `SKILL.md`-capable location instead (see [Any other agent](#any-other-agent)).

**Hermes Agent** — *verified end to end (install → validate → remove):*

```sh
hermes plugins install AlastairZeved/Editorial-Recension --no-enable
hermes plugins enable editorial-recension
hermes gateway restart
```

Hermes scans community plugin sources at install time and may ask you to confirm before proceeding. `hermes plugins validate editorial-recension` checks the manifest without enabling anything.

**Claude Code** — *verified against current vendor docs:*

```sh
git clone https://github.com/AlastairZeved/Editorial-Recension.git
claude --plugin-dir ./Editorial-Recension
```

`--plugin-dir` loads the plugin for that session; see the [Claude Code plugins docs](https://code.claude.com/docs/en/plugins) for making it permanent. The repository ships no Claude Code marketplace file, so the `/plugin marketplace add` route does not apply.

**OpenClaw** — *verified against current CLI docs:*

```sh
openclaw plugins install git:github.com/AlastairZeved/Editorial-Recension
```

**Codex** — *verified against current Codex docs; the repository ships a repo marketplace (`.agents/plugins/marketplace.json`):*

```sh
codex plugin marketplace add AlastairZeved/Editorial-Recension
```

Then pick Editorial Recension from that marketplace in the Plugins Directory and install it. Codex loads this repository through its root Agent Plugins manifest — the documented package format. The optional `.codex-plugin/plugin.json` overlay is deliberately not shipped: its OpenAI-specific settings are superseded by the root manifest's `extensions["com.openai"]` object, and a present root object replaces the overlay entirely. Marketplace sources can be pinned (`codex plugin marketplace add AlastairZeved/Editorial-Recension --ref main`) or added from a local checkout (`codex plugin marketplace add ./Editorial-Recension`).

**Cursor** — *verified against current vendor docs; no adapter file needed:*

```sh
git clone https://github.com/AlastairZeved/Editorial-Recension.git
```

Open **Customize** in the Cursor sidebar, find Editorial Recension, and select **Install** (choose project or user scope). Cursor loads Agent Plugins standard packages — a root `plugin.json` plus `skills/` — without changes, so no Cursor-specific manifest ships here.

**GitHub Copilot** — *verified against current vendor docs; the repository ships a Copilot marketplace (`.github/plugin/marketplace.json`):*

```sh
copilot plugin marketplace add AlastairZeved/Editorial-Recension
copilot plugin install editorial-recension@editorial-recension
```

`copilot plugin install AlastairZeved/Editorial-Recension` also works: the install command accepts a GitHub repository root directly, without a marketplace.

### Any other agent

Two universal routes:

- **SKILL.md-capable agents** — copy the whole repository (or clone it) into the agent's skill/plugin discovery directory. Any agent that reads `SKILL.md` files picks up `skills/editorial-recension/SKILL.md`; the `../../` references resolve because the bundle is intact.
- **Validation without installing** — `npx plugins.sh validate https://github.com/AlastairZeved/Editorial-Recension` checks the package against the Agent Plugins schema from any machine. The root manifest reports conformant; the `.claude-plugin/` client adapter reports schema warnings that are cosmetic (it follows Claude Code's manifest format, not the portable one). This repository is not yet listed in the [plugins.sh directory](https://plugins.sh) — listing is a manual form submission reviewed by the registry's maintainer — so `npx plugins.sh install AlastairZeved/Editorial-Recension` will not resolve until that submission lands. Use the client routes above, or `hermes plugins install` / `openclaw plugins install`, which do not depend on the registry.

## Usage

There is no CLI binary to call. You use it inside a session with your agent, by asking for the thing the skill is built for:

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

**Layer 1 — Intake** (`skills/editorial-recension/SKILL.md`). The skill validates the three questions, formats them into templates, and dispatches only after you confirm the context block. It escalates on vague answers: one targeted clarification, then a direct checklist — it does not accept vagueness out of politeness.

**Layer 2 — Execution** (`agents/editor.md` + `schemata/`). The editor reads all six schemata files, then runs four phases: Barrier Bridge and Chain Repair together, then Compression Pass, then Flow Weld, then Ripple Read. If Ripple Read finds gaps, it dispatches them back to the responsible schema with specific, located feedback. The editor returns edited text, a schema trace of what fired where, and a termination assessment.

**Layer 3 — Verification** (`agents/evaluator.md`). The evaluator scores the output against every measurable feature *before* reading the editor's trace, then compares its independent scoring to the trace. Discrepancies are the most important findings: either the editor ran a schema without the work showing (execution failure), or skipped a schema and claimed it ran (compliance failure). It returns feature scores, discrepancies, and a PASS/FAIL verdict.

**Layer 4 — Loop** (`skills/editorial-recension/SKILL.md`). On FAIL, the evaluator's located feedback goes back to the editor, which re-enters at the phase where the failures were found. On PASS, the termination condition is confirmed and the final output is presented. Limits:

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

There is no code API. The package's surface is three components, loaded through the root `plugin.json` manifest:

- **Skill — `skills/editorial-recension/SKILL.md`.** Auto-fires on prose-editing requests. Input: the user's request and text. Output: the intake questionnaire, then the final edited text with trace and verdict.
- **Agent — `agents/editor.md`.** Input: the formatted editorial context (target reader, purpose, source text, preceding context) plus the schemata library. Output: edited text, schema trace, termination assessment.
- **Agent — `agents/evaluator.md`.** Input: the same editorial context, the original text, the editor's output, and the editor's trace (read only *after* independent scoring). Output: feature scores, trace discrepancies, PASS/FAIL verdict.

## Maintainers

[@AlastairZeved](https://github.com/AlastairZeved)

## Thanks

To [Richard Litt](https://github.com/RichardLitt) and the [Standard Readme](https://github.com/RichardLitt/standard-readme) spec, which this README follows.

## Contributing

Questions and bug reports go to [GitHub issues](https://github.com/AlastairZeved/Editorial-Recension/issues). PRs are accepted — there is no CONTRIBUTING file or Code of Conduct in the repository yet, so please:

- Open an issue describing the problem before a large change.
- Keep terminology consistent with the schemata (`schemata/`) and the agents (`agents/`) — Barrier Bridge, Chain Repair, Compression Pass, Flow Weld, Ripple Read, and the feature set are the project's vocabulary.
- If you change behavior, add or update the corresponding evidence in `tests/` so the worked case keeps demonstrating the full loop.
- Keep the packaging layers in sync: the portable layer (root `plugin.json` + `skills/`) is the standard-conformant package and the source of truth; client adapters (`.claude-plugin/`, `agents/` for Claude Code, `.agents/plugins/marketplace.json` for Codex, `.github/plugin/marketplace.json` for Copilot CLI) describe the same system and must not drift from it. If a component moves, update the manifests, the marketplace entries, and any path references on both sides.

## License

MIT © Alastair Zeved. See [LICENSE](./LICENSE) for the full text.
