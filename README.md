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

# 7. The Six Schemata Files — The Execution Layer

editor.md says the editor must read all six files in ${CLAUDE_PLUGIN_ROOT}/schemata/ before its first pass. Those files are the actual procedural instructions. Each has the same structure: Recognition Trigger, Execution Sequence, Scale, Completion Test, Measurable Features.
barrier-bridge.md

Trigger: Text contains concepts, terms, or logical steps that assume knowledge the target reader does not have.

Execution:

    SCAN — sentence by sentence. Does this sentence contain a term the reader would need to look up? A logical connection that skips an intermediate step? A concept that only makes sense if you know where the argument ends up? Expertise presented as self-evident ("obviously," "simply," "just")?

    BRIDGE — for each barrier: name it explicitly ("The reader does not know X"), identify what the reader does know at this point, write the bridge, then ground it with one of: how the jump felt on first encounter, what was confusing and why, or what missing context made it click.

    VERIFY — re-read the bridged sentence in context (paragraph before, bridged paragraph, paragraph after). Did the bridge flow in? Did it flow out? Did it introduce a new barrier? If yes, re-enter SCAN on the bridge text.

Completion test: Every barrier bridged. No bridge introduced new barriers. Each bridge grounded in experience, confusion, or failure.

Measurable features: 0 ungrounded domain terms, 0 logical leaps, 0 self-evidence markers, each bridge contains at least one grounding element.
chain-repair.md

Trigger: A reasoning chain where the reader is told a conclusion rather than walked through the steps that produce it.

Execution:

    MAP THE CHAIN — number each claim, concept, or logical step in order. This is the chain as presented.

    WALK THE CHAIN AS A READER — at each transition, ask: does step N give me everything I need for step N+1? Or does N+1 require knowledge not yet stated? Mark every transition where the answer is "I need something that hasn't been stated." These are chain breaks.

    Exception: Experiential grounding inserted by Barrier Bridge is not subject to derivation requirements. Bridges ground through experience; chains derive through logic. Do not flag bridges as asserted claims.

    REPAIR EACH BREAK — identify the missing intermediate step(s), write them into the chain where they belong, then check upward and downward. If inserting broke either direction, rewrite the surrounding steps, not just the insertion.

    RE-MAP — number the repaired chain, walk it again. New breaks? Repair again. No breaks? Move to VERIFY.

    VERIFY — read the repaired passage as continuous prose. Does it read as natural progression, or as a list of steps? If steps, rewrite for prose flow while preserving logical progression.

Completion test: A reader can follow the reasoning from beginning to end, in order, deriving each step from the previous one. No step requires prior knowledge not established earlier.

Measurable features: 0 asserted connections, 0 transitions requiring unstated knowledge, chain walkable without external knowledge, reads as prose not enumeration (no 3+ consecutive sentences with identical syntactic structure).
compression-pass.md

Trigger: Text has been written or rewritten. Words exist on the page. Now evaluate whether each one earns its place.

Execution:

    WEIGHT TEST — sentence by sentence. Read aloud. Remove one word at a time. Does meaning change? If no, the word is dead weight. Does meaning hold but feeling change? If the sentence becomes technically correct but emotionally flat, the word is weight-bearing. Keep it. After removals, read the compressed sentence in context.

    FLUFF SCAN — qualifiers that soften but don't clarify ("somewhat," "rather," "quite," "a bit"). Hedges that protect the author ("arguably," "it could be said that"). Redundant pairs ("each and every," "first and foremost"). Throat-clearing openings ("It is important to note that").

    EXPANSION CHECK — after compression, re-read. Did I compress something that needed to breathe? A moment of recognition, a beat where the reader sits with an idea, an emotional turn? Did I strip a sentence that was doing two jobs — carrying meaning and feeling — and leave only the meaning? If yes, expand that specific point with weight-bearing words, not filler.

    RHYTHM CHECK — read the compressed passage for rhythm. Compression that destroys rhythm has gone too far. Short sentences need long ones around them. If the passage reads like a telegram, expand strategically for rhythm, not for content.

Completion test: Every word is doing work — carrying meaning, feeling, or rhythm. No word exists because it was there before. No gap exists where a weight-bearing word was stripped.

Measurable features: 0 dead-weight words, 100% sentences where removal of any word changes meaning or feeling, rhythm variation present, no net loss of emotional weight markers (moments of recognition, reflective beats, emotional turns).
flow-weld.md

Trigger: Any edit has been made. The edit exists. Now check whether it broke anything.

Execution:

    READ UP — read the full paragraph or section before the edit. What tone is established? What concept is the reader holding? What direction is the text moving? What does the reader expect next?

    READ THE EDIT — does it arrive where the preceding context was pointing? Or does the reader experience a seam — a shift in tone, concept, direction, or register?

    READ DOWN — read the full paragraph or section after the edit. Does the edited text set up what follows? Or is there a second seam?

    REPAIR — identify which side of the weld is broken (before→edit, edit→after, or both), identify what broke (tone, concept, register, direction), then rewrite whichever of the three segments — before, edit, or after — needs to change to restore continuous flow.

Completion test: A reader encountering the text for the first time would not detect where the edit was made. The before→edit→after sequence reads as if written in one sitting.

Measurable features: Tone consistency, concept continuity, register stability, direction coherence, seam detectability. Note the split: the editor flags suspected seams, the evaluator confirms or rejects on a clean read.
ripple-read.md

Trigger: More than 3 localized edits without a full document re-read. A structural change (sections moved, reordered, merged, split). An edit changed the emotional register of a passage. Another schema deferred a document-level concern here. The editor believes editing may be complete (termination check).

Execution:

    CLEAR STATE — stop editing. Do not hold assumptions about what the document says. Discover it by reading.

    FULL READ — start to finish, tracking five ledgers:

        Flow ledger: at each paragraph boundary, smooth or jarring? If seam, what broke?

        Coherence ledger: at each section boundary, did the section that just ended set up what follows?

        Rhythm ledger: stretches of uniform sentence length or paragraph density?

        Reasoning arc: does the document-level argument build from beginning to end? Can you state the thesis and trace how each section contributed in order?

        Audience drift: is the assumed reader the same person at the end as at the beginning?

    GAP IDENTIFICATION — compile ledger entries. For each gap: classify (flow, coherence, rhythm, arc, audience), locate (paragraph/section numbers), diagnose (name the mechanism, not "it feels off").

    DISPATCH — route each gap to the responsible schema. Flow gaps → Flow Weld. Coherence gaps → Chain Repair or Barrier Bridge. Rhythm gaps → Compression Pass. Arc gaps → Chain Repair at document scale. Audience gaps → Barrier Bridge.

    TERMINATION CHECK — only when a clean full read produces zero ledger entries: is the termination condition met?

Completion test: Zero entries in all five ledgers. Termination condition met. No dispatch required.

Measurable features: 0 flow seams, 0 coherence gaps, 0 rhythm flatlines, reasoning arc traceable, audience consistency, termination condition met, dispatch count reported (0 = clean pass).
scale-rules.md — the coordination layer

This file is the formal rule set for which schemata fire at which scale, in what order, and with what constraints.

Activation table: Each schema has a scale profile. Barrier Bridge is active at sentence, paragraph, and section, defers to Ripple Read at document. Chain Repair skips sentence (unless multi-claim), primary at paragraph, active at section, active at document for logical derivation only. Compression Pass primary at sentence, active at paragraph and section, defers at document. Flow Weld primary at sentence, active at paragraph and section, defers at document. Ripple Read is not active at sentence, monitors paragraph and section without editing, and is primary and exclusive at document.

Ordering constraints: Phase 1 (Barrier Bridge + Chain Repair) runs first because these add, restructure, or rewrite content. Running Compression Pass or Flow Weld before content is stable wastes work. When both fire on the same passage: Barrier Bridge handles grounding, Chain Repair handles derivation. If a missing intermediate step is both a knowledge gap and a reasoning chain break, Barrier Bridge bridges the knowledge first, then Chain Repair verifies the logical chain holds with the bridge in place. Phase 2 (Compression Pass) runs after Phase 1 because it evaluates already-bridged text. Bridges inserted by Barrier Bridge are weight-bearing content; Compression Pass must not strip them unless a shorter bridge carries the same grounding. Phase 3 (Flow Weld) runs after Phases 1 and 2 because it checks whether all preceding edits created seams. Phase 4 (Ripple Read) runs after localized editing is complete.

Re-entry rules: If Ripple Read dispatches a gap to a Phase 1–3 schema, that schema runs, then Flow Weld runs on the repair, then Ripple Read runs again. If any schema's repair triggers another schema's recognition trigger, the triggered schema runs before Flow Weld. The cycle continues until Ripple Read produces a clean pass.

Authority boundaries: No schema may override another's authority. Barrier Bridge decides if a bridge is grounded. Chain Repair decides if a logical step is derived. Compression Pass decides if a word earns its place. Flow Weld decides if a seam exists. Ripple Read decides if editing is done. When a concern crosses boundaries, both schemas evaluate within their own authority.

Audience calibration: All schemata inherit the audience definition from the editorial context provided at dispatch time. The dispatch layer (SKILL.md) establishes who the target reader is before any schema runs. No schema defines its own audience.

# 8. The Test Evidence — Does the Loop Actually Work?

The plugin includes a worked test case that demonstrates the full system on a paragraph with three planted defects.
The input: test-paragraph.md

The test paragraph is about "vibe coding" — using AI to generate code by describing what you want in plain English. The target reader is a non-technical professional who has never written code but is curious about AI tools. The preceding context ends with: "That first moment of seeing your words turn into something functional is genuinely thrilling. But the thrill masks something important."

The paragraph itself contains three planted defects:

    Tacit-knowledge wall. The opening sentence uses "impedance mismatch," "mental model," and "execution model." The next sentence adds "LLM," "tokenizes your intent," "latent representation," "training distribution," "statistical inference," and "executable logic." A non-technical reader cannot cross this.

    Skipped reasoning step. The paragraph jumps from "the LLM processes your words through statistical inference" to "Obviously, this is why prompt engineering matters so much." The intermediate step — explaining why statistical inference over ambiguous natural language makes precise prompting critical — is missing. The reader is told a conclusion without being shown the reasoning.

    Register break with preceding context. The preceding context is warm, experiential, second-person: "seeing your words turn into something functional is genuinely thrilling." The test paragraph opens cold and technical: "The real issue with vibe coding isn't the code quality — it's the impedance mismatch between your mental model and the system's execution model." The reader was in a reflective emotional state; the text yanks them into jargon without transition.

The baseline: baseline-test-output.md

This is what ordinary single-pass editing produces. The prose is cleaner. Sentences are shorter. But the three defects remain:

    The tacit-knowledge wall is still there — "the gap between what you think you asked for and what the AI actually built" is clearer, but the mechanism is still unexplained.

    The reasoning step is still skipped — "This is why how you phrase your request matters so much" is asserted without the intermediate explanation.

    The register break is still there — the opening is still abstract and analytical, not warm and experiential.

This is the comparison point. Single-pass editing optimizes for reads well to someone who already understands it — the opposite of the job when explaining across a knowledge gap.
The editor's run: editor-test-output.md

This is the full editor run on the test paragraph. It shows each phase in detail.

Phase 1 — Barrier Bridge + Chain Repair:

The Barrier Bridge SCAN identifies 14 barriers across sentences 1–5 and 7, plus one self-evidence marker ("Obviously"). The BRIDGE work replaces the entire technical apparatus with a "two pictures" metaphor: you have an image in your head of what you want; the AI has its own version assembled from millions of code examples it studied. Those two pictures are never exactly the same. The AI's output looks right, so you assume they matched. They almost certainly did not.

The grounding element is experiential: "And here is what makes that confusing at first: the AI's output looks right. It runs. It seems to do what you asked. So you assume the pictures matched." This is the "what was confusing and why" grounding the schema requires.

The Chain Repair MAP numbers 11 steps in the original. The WALK identifies two breaks: transition 3→4 (from "your request gets filtered" to "this is why prompt engineering matters") and transition 4→5 (from "prompt engineering matters" to "the gap is where bugs live"). The repair inserts intermediate steps: because the AI is matching against your words, the specific words you choose steer the result; even with careful phrasing, a gap almost always remains; that gap is where problems hide.

The VERIFY step re-reads the repaired chain as prose. No 3+ consecutive sentences with identical syntactic structure. Verified clean.

Phase 2 — Compression Pass:

The Weight Test removes "during training" (dead weight — "studied" does the same work), "so much" (replaced with "exact"), "though" (softener), "actually" (dead weight), "essentially" (hedge), and "waiting to cause trouble" (expansion that doesn't add meaning).

The Fluff Scan finds nothing new.

The Expansion Check confirms that reflective beats are preserved: "They almost certainly did not. Not completely." and "That gap is where problems hide." and the closing "It will run. Every time. Exactly as written, not as intended."

The Rhythm Check confirms variation: short punches alternate with longer explanatory sentences. Short paragraphs break up longer ones.

Phase 3 — Flow Weld:

Seven edit points are checked. Six are clean. One — Edit Point 4 — has a potential seam: the transition from the emotional beat ("They almost certainly did not. Not completely.") to the mechanical explanation ("Because the AI matches against your exact words...") feels slightly abrupt. The repair inserts a transitional breath: "But here is where it gets practical." This gives the reader a beat — acknowledgment that we are shifting from "what is happening" to "what you can do about it."

Phase 4 — Ripple Read:

The CLEAR STATE step wipes assumptions. The FULL READ tracks all five ledgers. Flow ledger: 0 seams. Coherence ledger: 0 gaps. Rhythm ledger: 0 flatlines. Reasoning arc: traceable from first paragraph to thesis. Audience drift: none.

The TERMINATION CHECK finds the condition met on a clean pass. The editor proposes termination.
The evaluator's independent scoring: evaluator-test-output.md

This evaluates the unedited test paragraph. The evaluator scores it against the feature set before reading any trace. It catches all three planted defects independently:

    Tacit knowledge barrier — DETECTED. Barrier Bridge features catch 6 ungrounded domain terms: impedance mismatch, mental model/execution model, tokenizes, latent representation, training distribution, semantic distance.

    Reasoning chain that skips a step — DETECTED. Chain Repair features catch the leap from tokenization mechanics to "prompt engineering matters" without the intermediate explanation.

    Flow break with preceding context — DETECTED. Flow Weld features catch the hard seam at entry: warm experiential register to cold technical register with no transition.

The verdict is FAIL with specific, located feedback for each schema.
What the test evidence demonstrates

The test suite shows four things:

    The original paragraph has three planted defects.

    A baseline single-pass edit does not fix them.

    The evaluator can detect all three independently, before seeing any editor trace.

    The editor can repair all three through the schemata.

The confirmation step — evaluator scoring the editor's output — is not shown in the provided files, but the editor's output ends with "PROPOSING TERMINATION" and "This assessment is submitted to the evaluator agent for confirmation or rejection." That is the next step in the loop.
# 9. The Questionnaire — The Intake Layer

skill-questionnaire-tests.md is the most extensive test file. It specifies the skill's interactive intake with manually verifiable acceptance criteria. The reason for this thoroughness is stated directly in SKILL.md: "a vague target reader silently corrupts every downstream schema, so the validation below is load-bearing, not ceremony."

The file covers:

    Section 1–2: Target Reader (Q1). Valid inputs must contain all three components: role/domain, at least one explicit "knows" item, at least one explicit "doesn't know" item. "Non-technical," "general audience," and "someone curious about AI" always fail. Test 2.7 shows a partial pass: role and knows are present and specific, but "doesn't know" is too broad ("the technical side"). The skill asks one targeted clarification identifying exactly what is missing.

    Section 3–4: Purpose (Q2). Valid inputs must contain a specific action verb attached to a specific object. "Make it clear," "improve the flow," "help them understand" always fail. Test 4.5 shows the escalation pattern: first vague answer gets a targeted clarification; second consecutive vague answer gets a direct checklist with examples.

    Section 5–6: Source Text (Q3). Valid inputs must be retrievable in the current session: pasted directly, file path, message reference, or public URL. Temporal references ("yesterday," "last week") always fail. Test 5.6 and 5.7 show the supplemental question handling: if the submission is valid but preceding context status is unknown, the skill accepts the submission, asks one supplemental question, and resolves the field before dispatch. The Unknown state must never survive into the Step 4 confirmation block.

    Section 7: Full end-to-end scenario. A complete valid flow from invocation through Q1, Q2, Q3, pre-dispatch confirmation, and dispatch. Plus a pre-dispatch adjustment scenario where the user corrects the purpose field before confirming.

    Section 8: Acceptance criteria. Twenty-one rules covering validation, clarification patterns, template formatting, and dispatch requirements.

The questionnaire is the gate. If the target reader is vague, Barrier Bridge cannot know which terms need grounding. If the purpose is vague, Chain Repair cannot know what reasoning chain the reader needs to follow. If the source text is not retrievable, nothing runs.

# 10. How it all fits together

The architecture has four layers:

Layer 1 — Intake (SKILL.md). The skill fires, asks three questions, validates each answer against strict criteria, reformats into templates, presents the full context for confirmation, and only dispatches after explicit user approval. This layer prevents bad context from entering the system.

Layer 2 — Execution (editor.md + schemata/). The editor agent reads all six schemata files, then runs Phase 1 (Barrier Bridge + Chain Repair), Phase 2 (Compression Pass), Phase 3 (Flow Weld), and Phase 4 (Ripple Read). If Ripple Read finds gaps, it dispatches back to the responsible schema. The editor produces edited text, a schema trace, and a termination assessment.

Layer 3 — Verification (evaluator.md). The evaluator agent receives the original text, the editor's output, the editor's schema trace, and the editorial context. It scores the output against every measurable feature before reading the trace. It then verifies the trace against its independent scoring. It returns feature scores, trace discrepancies, and a verdict (PASS or FAIL). If FAIL, it sends specific, located feedback back to the editor.

Layer 4 — Loop (SKILL.md). The skill orchestrates the cycle: dispatch to editor, dispatch to evaluator, if FAIL pass feedback back to editor and repeat, if PASS confirm termination. Maximum 5 cycles. If failures are not decreasing after cycle 3, surface to the user.

The trustworthiness comes from the separation between Layers 2 and 3. The editor cannot declare completion — only Ripple Read can propose termination, and only the evaluator can confirm it. The evaluator cannot see the editor's explanation until after it has scored the text on its own. This prevents the single-agent failure mode where an agent grades its own work and drifts toward passing itself.
