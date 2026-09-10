# Schema: Ripple Read

## Recognition Trigger
Any of:
- More than 3 localized edits have been made without a full document re-read
- A structural change occurred (sections moved, reordered, merged, or split)
- An edit changed the emotional register of a passage
- Another schema's Scale section deferred a document-level concern here
- The editor agent believes editing may be complete (termination check)

## Execution Sequence

**CLEAR STATE:** Stop editing. Do not hold any assumptions about what the document currently says. You are about to discover what it says by reading it.

**FULL READ — start to finish:**

Read the entire document from the first word to the last. As you read, track:

1. **Flow ledger:** At each paragraph boundary, note whether the transition is smooth or jarring. Record: paragraph number, transition quality (smooth/seam), and if seam — what broke (tone, concept, direction, register, pace).

2. **Coherence ledger:** At each section boundary, note whether the section that just ended set up the section that follows. Record: section transition, coherence (yes/no), and if no — what's missing or mismatched.

3. **Rhythm ledger:** Note the rhythm pattern across the document. Are there stretches of uniform sentence length? Stretches where every paragraph is the same density? Record: location of any rhythm flatlines (3+ consecutive paragraphs with similar length/density).

4. **Reasoning arc:** Does the document-level argument build from beginning to end? At the end of the read, can you state the thesis and trace how each section contributed to it in order? If you can't → the arc is broken. Record: where the arc breaks.

5. **Audience drift:** Is the assumed reader the same person at the end of the document as at the beginning? Did the text start addressing a novice and end addressing an expert (or vice versa)? Record: location of any audience shifts.

**GAP IDENTIFICATION:**

After the full read, compile all ledger entries that recorded problems. These are the gaps.

For each gap:
1. Classify: flow gap, coherence gap, rhythm gap, arc gap, or audience gap
2. Locate: paragraph/section numbers from the ledger
3. Diagnose: what specifically broke and why (this is not "it feels off" — name the mechanism)

**DISPATCH:**

For each identified gap, determine which schema handles the repair:
- Flow gaps at paragraph boundaries → Flow Weld
- Coherence gaps at section boundaries → Chain Repair (logical) or Barrier Bridge (knowledge)
- Rhythm gaps → Compression Pass
- Arc gaps → Chain Repair at document scale
- Audience gaps → Barrier Bridge (recalibrate for consistent target reader)

Send specific, located feedback to the appropriate schema. Not "check section 3" but "section 3 assumes the reader understood the concept introduced in section 1, paragraph 4, but the bridge there was stripped during compression — Barrier Bridge needs to re-bridge, Compression Pass should not strip it again."

**TERMINATION CHECK:**

After all gaps have been repaired and a clean full read produces no new ledger entries:

The loop ends when — and only when — this is true:

**A reader without the author's domain expertise can follow the reasoning chain, in the order it was presented, using language and structure they already have, without silently disengaging.**

If this condition is met on a clean full read with zero gap entries → editing is complete.
If any gap entry exists → editing continues. Return to the schema that owns the gap.

## Scale
- **Sentence level:** Not active. Ripple Read does not edit individual sentences.
- **Paragraph level:** Monitors paragraph-boundary transitions (flow ledger) but does not edit. Dispatches to Flow Weld.
- **Section level:** Monitors section-boundary coherence (coherence ledger) but does not edit. Dispatches to Chain Repair or Barrier Bridge.
- **Document level:** PRIMARY AND EXCLUSIVE. This is the only schema that reads and evaluates at document scale. Owns: rhythm across sections, reasoning arc, audience consistency, and termination condition.

## Completion Test
A full read from start to finish produces zero entries in all five ledgers (flow, coherence, rhythm, reasoning arc, audience drift). The termination condition is met. No gap requires dispatch to another schema.

## Measurable Features
- Flow ledger seam count: target 0
- Coherence ledger gap count: target 0
- Rhythm flatline count (3+ consecutive similar-density paragraphs): target 0
- Reasoning arc traceable from first section to thesis: yes/no
- Audience consistency (same assumed reader throughout): yes/no
- Termination condition met on clean read: yes/no
- Number of dispatches to other schemata on this pass: reported (0 = clean pass)
