# Scale Rules

Formal rules for which schemata fire at which document scale, in what order, and with what constraints. These are structural rules, not preferences.

## Activation Table

| Schema | Sentence | Paragraph | Section | Document |
|--------|----------|-----------|---------|----------|
| Barrier Bridge | ACTIVE | ACTIVE | ACTIVE | defer → Ripple Read |
| Chain Repair | SKIP (unless multi-claim sentence) | PRIMARY | ACTIVE | ACTIVE (logical derivation only) |
| Compression Pass | PRIMARY | ACTIVE | ACTIVE | defer → Ripple Read |
| Flow Weld | PRIMARY | ACTIVE | ACTIVE | defer → Ripple Read |
| Ripple Read | NOT ACTIVE | MONITOR (no edit) | MONITOR (no edit) | PRIMARY + EXCLUSIVE |

## Ordering Constraints

Schemata execute in phases. Within a phase, order is flexible.

**Phase 1 — Content schemata (what the text says):**
- Barrier Bridge
- Chain Repair

These run first because they add, restructure, or rewrite content. Running Compression Pass or Flow Weld before content is stable wastes work.

When both fire on the same passage: Barrier Bridge handles knowledge grounding. Chain Repair handles logical derivation. If a missing intermediate step is both a knowledge gap AND a reasoning chain break, Barrier Bridge bridges the knowledge first, then Chain Repair verifies the logical chain holds with the bridge in place.

**Phase 2 — Form schema (how the text reads):**
- Compression Pass

Runs after Phase 1 because it evaluates text that has already been bridged and repaired.

**Constraint:** Bridges inserted by Barrier Bridge are weight-bearing content. Compression Pass must not strip them unless a shorter bridge carries the same grounding. When in doubt, keep the bridge.

**Phase 3 — Integration schema (does the edit hold):**
- Flow Weld

Runs after Phases 1 and 2 because it checks whether ALL preceding edits created seams. Running it mid-edit would produce false positives.

**Phase 4 — Document schema (does the whole hold):**
- Ripple Read

Runs after localized editing is complete. Triggers on accumulated edit count (>3 without a full read), structural changes, or termination check. Dispatches gaps back to Phase 1-3 schemata with specific, located feedback.

## Re-Entry Rules

- If Ripple Read dispatches a gap to a Phase 1-3 schema, that schema runs, then Flow Weld runs on the repair, then Ripple Read runs again.
- If any schema's repair triggers another schema's recognition trigger (e.g., Chain Repair inserts a step that contains a barrier), the triggered schema runs before Flow Weld.
- The cycle continues until Ripple Read produces a clean pass (zero ledger entries, termination condition met).

## Authority Boundaries

| Concern | Authority |
|---------|-----------|
| Knowledge gaps (terms, concepts, prerequisites) | Barrier Bridge |
| Logical derivation (reasoning chains, thesis arc) | Chain Repair |
| Word-level efficiency and rhythm | Compression Pass |
| Edit integration (seam detection) | Flow Weld |
| Document-level coherence, rhythm, audience consistency | Ripple Read |
| Termination decision | Ripple Read (exclusive) |

No schema may override another schema's authority. When a concern crosses boundaries (e.g., a bridge that is also a reasoning step), both schemas evaluate within their own authority. Conflicts are resolved by the authority boundary: Barrier Bridge decides if a bridge is grounded. Chain Repair decides if a logical step is derived. Compression Pass decides if a word earns its place. Flow Weld decides if a seam exists. Ripple Read decides if editing is done.

## Audience Calibration

All schemata that reference a "target reader" inherit the audience definition from the editorial context provided at dispatch time. The dispatch layer (SKILL.md) is responsible for establishing who the target reader is before any schema runs. No schema defines its own audience.
