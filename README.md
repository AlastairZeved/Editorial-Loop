# Editorial-Loop
Editorial Loop is a two-agent editorial system packaged as a Claude Code plugin. Its purpose is narrow and specific: edit prose until a reader who lacks the author’s domain expertise can follow the reasoning chain; not until it “reads well” to someone who already understands it.

# 1. Plugin loading and discovery

Claude Code reads .claude-plugin/plugin.json and discovers:

    skills/editorial-loop/SKILL.md — the auto-firing skill and dispatch layer

    agents/editor.md — the editor subagent

    agents/evaluator.md — the evaluator subagent

    schemata/ — six reference files the editor agent reads before editing

    tests/ — regression evidence, not loaded at runtime

The skill’s description field is the trigger. It fires when you ask to edit prose, essays, guides, documentation, cross-domain writing, or anything meant to carry a reader through a reasoning chain. It does not fire for code comments, commit messages, quick responses, first drafts, or writing where the audience already shares the author’s expertise.

# 2. The skill is the front door and the loop controller

SKILL.md is not the editor. It is the dispatch layer and input validator. When it fires, it does not start editing immediately. It runs a strict intake sequence.
Step 1 — Ask Q1: Target Reader

It asks for three things:

    role or domain context

    what the reader already knows

    what the reader does not know

This is load-bearing. A vague target reader silently corrupts every downstream schema, so the skill validates hard.

A valid Q1 answer must contain all three. “Non-technical,” “general audience,” and “someone curious about AI” always fail. They have no role, no specific known vocabulary, and no specific gaps.

If the first answer is vague, the skill asks one targeted clarification question. If the second answer is still vague, it stops asking prose questions and restates the requirement as a direct checklist with examples. It does not accept vagueness out of politeness.

Once valid, it reformats the answer into:
text

TARGET READER: [role/domain]
KNOWS: [specific vocabulary, concepts, frameworks]
DOESN'T KNOW: [specific gaps]

Step 2 — Ask Q2: Purpose

It asks what the reader should be able to do, decide, or understand after reading.

A valid Q2 answer needs a specific action verb attached to a specific object: “explain the mechanism to their team,” “decide whether to sign,” “ask informed questions about FHIR compliance,” “evaluate whether Y.”

It fails if it contains only writing-quality goals like “make it clear,” “improve the flow,” or tautologies like “help them understand this.”

Same validation pattern: one targeted clarification, then a direct checklist if the second answer is vague.

Once valid, it reformats into:
text

PURPOSE: [concrete outcome + measurable indicator]
READER SHOULD: [specific action verb + object]

Step 3 — Ask Q3: Source Text

It asks for the text to edit. It accepts:

    direct paste

    file path

    message reference in the current conversation

    public URL

A valid Q3 answer must be retrievable in the current session. “What I wrote yesterday” fails because it has no path or paste. Vague references like “earlier” or “the document” fail unless they resolve unambiguously.

If the submission is valid but preceding context is unknown, the skill asks one supplemental question: is there a preceding paragraph or sentence the agents should factor in for flow? If the user says no, it marks:
text

PRECEDING CONTEXT: None — Flow Weld operates at document boundaries only

If the user provides it, it appends it. The Unknown state must never survive into dispatch.

Once valid, it confirms:
text

SOURCE TEXT: [description]
SOURCE TYPE: [Direct paste | File path | Message reference | URL]
PRECEDING CONTEXT: [quoted, described, or “None — ..."]

Step 4 — Pre-Dispatch Confirmation

It assembles the full context block:
text

EDITORIAL CONTEXT — PLEASE CONFIRM

TARGET READER: [...]
KNOWS: [...]
DOESN'T KNOW: [...]

PURPOSE: [...]
READER SHOULD: [...]

SOURCE TEXT: [description]
SOURCE TYPE: [type]
PRECEDING CONTEXT: [status]

It asks: “Does this look right? Say yes or tell me what to adjust and I’ll dispatch the agents.”

It does not dispatch until the user explicitly confirms or says to proceed. If the user requests an adjustment, it updates only the affected field, re-presents the full block, and asks again.

# 3. Dispatch to the editor agent

Once confirmed, the skill sends the editor agent:

    the formatted target reader template

    the formatted purpose template

    the source text

    the preceding context field exactly as formatted

    the schemata library from ${CLAUDE_PLUGIN_ROOT}/schemata/ and scale-rules.md

The editor agent’s identity is the editing loop itself. It does not decide whether to edit. It is already editing. It holds five named schemata as perceptual apparatus, not a checklist.
The five schemata
Phase	Schema	What it does
1	Barrier Bridge	Finds where the reader’s knowledge ends and the author’s assumptions begin. Grounds terms and leaps.
1	Chain Repair	Finds where reasoning is asserted rather than derived. Rebuilds the chain.
2	Compression Pass	Removes words that don’t earn their place — without stripping weight-bearing bridges.
3	Flow Weld	Checks bidirectional flow at every edit point. Removes seams edits left behind.
4	Ripple Read	Reads the whole document. Tracks flow, coherence, rhythm, reasoning arc, audience consistency. Owns the termination decision.
How the editor runs them

Phase 1: Runs Barrier Bridge and Chain Repair simultaneously. Barrier Bridge handles grounding; Chain Repair handles derivation. Where they overlap, Barrier Bridge goes first to ground the knowledge, then Chain Repair verifies the logic holds.

Phase 2: Runs Compression Pass on the repaired text. Every word must earn its place. Bridges inserted in Phase 1 are weight-bearing and are not stripped unless a shorter bridge carries the same grounding.

Phase 3: Runs Flow Weld on every edit made in Phases 1 and 2. Checks bidirectional flow at every edit point. If a seam exists, it rewrites whichever segment — before, edit, or after — needs to change. Not just the edit.

Phase 4: Runs Ripple Read. Full document, start to finish. Tracks five ledgers: flow, coherence, rhythm, reasoning arc, audience consistency. If any gap exists, it dispatches that gap back to the responsible schema — Phase 1, 2, or 3 — with specific, located feedback. Then re-runs from that phase forward.

It repeats until Ripple Read produces a clean pass: zero ledger entries, termination condition met.
What the editor produces

For each pass:

    The edited text

    A schema trace: which schemata fired, where, what they found, what was rewritten

    Its assessment of termination: is the condition met, or do gaps remain?

The schema trace is not optional. The evaluator uses it to verify the editor’s work.
What the editor does not do

    It does not evaluate its own output for quality. The evaluator does that.

    It does not decide the audience. The dispatch layer provides it.

    It does not skip schemata because the text “seems fine.” It runs all phases every pass. If a phase finds nothing, its trace says “no issues found at this scale.” That is different from not running it.

    It does not patch. Every repair is a rewrite. A patch preserves broken structure. A rewrite finds the right structure.

Termination condition

The editor cannot declare editing complete. Only Ripple Read can propose termination, and only when:

    A reader without the author’s domain expertise can follow the reasoning chain, in the order it was presented, using language and structure they already have, without silently disengaging.

The evaluator then confirms or rejects.

# 4. Dispatch to the evaluator agent

The skill sends the evaluator:

    the same three-part editorial context

    the original source text

    the editor’s output text

    the editor’s schema trace

The evaluator is an adversarial verification agent. It does not edit. It does not suggest rewrites. It does not improve the text. It examines whether the editor’s output matches the methodology’s measurable feature profile.
The independence requirement

The evaluator scores the output against every feature before reading the editor’s schema trace. This is the core of the plugin’s trustworthiness. Reading the trace first would be checking homework against the answer key — confirmation bias, not verification.

Score first, compare second. Always.
The feature set

The evaluator holds measurable features extracted from each schema.

Barrier Bridge features:

    domain-specific terms without grounding: target 0

    logical leaps without intermediate steps: target 0

    self-evidence markers (“obviously,” “simply,” “just”): target 0

    each bridge contains at least one grounding element: how the jump felt on first encounter, what was confusing and why, or what missing context made it click

Chain Repair features:

    asserted connections (told, not shown): target 0

    transitions requiring unstated knowledge: target 0

    chain walkable from step 1 to conclusion without external knowledge: yes/no

    repaired chain reads as prose, not enumeration: indicator is no 3+ consecutive sentences with identical syntactic structure

    experiential grounding from Barrier Bridge is exempt from derivation requirements

Compression Pass features:

    dead-weight words remaining (qualifiers, hedges, redundancies, throat-clearing): target 0

    sentences where removal of any word changes meaning or feeling: target 100%

    rhythm variation present: yes/no

    emotional weight markers preserved: moments of recognition, reflective beats, emotional turns — no net loss pre- to post-compression

Flow Weld features:

    tone consistency across before/edit/after: consistent/inconsistent

    concept continuity: yes/no

    register stability: yes/no

    direction coherence: yes/no

    seam detectability: evaluator performs a clean read of each edit region without referencing the editor’s trace, notes seams, then compares to the trace. Discrepancies are findings.

Ripple Read features:

    flow ledger seam count: target 0

    coherence ledger gap count: target 0

    rhythm flatline count (3+ consecutive similar-density paragraphs): target 0

    reasoning arc traceable from first section to thesis: yes/no

    audience consistency: yes/no

    termination condition met on clean read: yes/no

How the evaluator works

Step 1: Independent feature scoring. Score the editor’s output against every feature before reading the trace.

Step 2: Trace verification. Read the editor’s schema trace. Compare:

    Did the editor claim a schema ran? Verify the output shows evidence of that schema’s work.

    Did the editor claim “no issues found” for a phase? Check whether you found issues in that phase during Step 1.

    Did the editor claim termination? Check whether your independent scoring agrees.

Discrepancies are the most important findings. They indicate either:

    execution failure — the editor ran the schema but the output doesn’t reflect it

    compliance failure — the editor skipped the schema and falsely reported running it

Step 3: Produce evaluation report. Format:
text

## Feature Scores
[Each feature, its target, its actual score]

## Trace Discrepancies
[Any discrepancy between editor's trace and independent scoring]

## Failures
[Each failure: schema name, feature name, location in text, expected vs. found]

## Verdict
PASS — all features meet targets, no trace discrepancies
FAIL — [specific failures with schema-level feedback for editor]

Step 4: If FAIL, return to editor. The evaluator sends specific, located feedback. Not “try again,” but “Barrier Bridge feature failed on paragraph 3: domain-specific term ‘X’ used without grounding.” The editor re-enters its loop at the appropriate phase.

Step 5: If PASS, confirm termination. If all features pass and the termination condition is met on the evaluator’s independent clean read, editing is complete.
What the evaluator does not do

    It does not edit text. Ever.

    It does not suggest improvements. It identifies failures against the feature set.

    It does not soften failures. A failure is a failure. Name it, locate it, specify it.

    It does not pass text that has feature failures because it “reads well overall.” The features are the standard.

    It does not evaluate style, voice, or aesthetic quality. It evaluates against the measurable feature set. Nothing else.

# 5. The loop

The skill orchestrates the cycle:

    Dispatch to editor.

    Editor returns edited text + schema trace + termination assessment.

    Dispatch to evaluator.

    Evaluator returns feature scores + trace discrepancies + verdict.

    If FAIL: pass the evaluator’s failure report back to the editor. The editor re-enters at the phase where failures were identified. Return to step 3.

    If PASS: confirm termination condition. Present final output.

Loop limits

    Maximum 5 editor↔evaluator cycles per editing session.

    If not converged after 5 cycles, surface remaining failures to the user with the best output so far and let them decide how to proceed.

    Each cycle should show convergence — fewer failures than the previous cycle. If failures are not decreasing after cycle 3, surface this to the user.

# 6. Final output to the user

The skill presents:

Edited Text — the final version.

What Changed — which schemata fired, major rewrites, barriers bridged, chains repaired.

Editorial Trace — condensed: schema, location, finding, action taken.
Evaluator Verdict — PASS with feature summary, or best-effort with remaining issues.

# 7. What is not included in what you pasted

The actual schemata/ files are not in the provided content. editor.md says the editor must read all six files — the five schemata plus scale-rules.md — before its first pass. Those files contain the full execution sequence, scale rules, and completion test for each schema. The summaries in editor.md and SKILL.md are the perceptual apparatus; the files are how each move is actually run.

The tests/ directory is also not included, but the README says it holds a worked case: a paragraph with three planted defects — a tacit-knowledge wall, a skipped reasoning step, and a register break — which the evaluator caught independently. baseline-test-output.md shows the same paragraph after ordinary single-pass editing for comparison.

# 8. How a session actually runs

    You ask Claude Code to edit an essay, guide, or cross-domain explanation.

    The editorial-loop skill fires based on its description.

    It asks you three questions: target reader, purpose, source text.

    It validates each answer. If vague, it pushes back with targeted clarification, then a checklist.

    It asks a supplemental question about preceding context if needed.

    It presents the full editorial context for confirmation.

    You confirm.

    It dispatches the editor agent with the context, source text, preceding context, and schemata library.

    The editor runs Phase 1 → Phase 2 → Phase 3 → Phase 4. If Ripple Read finds gaps, it loops internally back to the responsible phase.

    The editor returns edited text + schema trace + termination assessment.

    The skill dispatches the evaluator agent with the original text, edited text, trace, and editorial context.

    The evaluator scores independently before reading the trace. It then verifies the trace.

    If FAIL, the evaluator sends located feedback back to the editor, and the loop repeats.

    If PASS, the evaluator confirms termination.

    The skill presents the final edited text, what changed, the editorial trace, and the evaluator verdict.

The system’s trustworthiness comes from the separation: the editor cannot declare completion, and the evaluator cannot see the editor’s explanation until after it has scored the text on its own. That prevents the single-agent failure mode where an agent grades its own work and drifts toward passing itself.
